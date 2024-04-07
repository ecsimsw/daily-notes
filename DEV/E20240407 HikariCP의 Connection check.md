## E20240407 Datasource 동적 라우팅.md

### HikariCP의 Connection check 
- 둘 중 하나가 꺼지면 에러를 내는 것이 아니다.
- 그 Datasource 에 요청이 갔을 때 그제서 사용 불가능함을 알린다.
```
2024-04-07T23:08:56,871 WARN  [http-nio-8084-exec-1] c.z.h.p.PoolBase: HikariPool-2 - Failed to validate connection com.mysql.cj.jdbc.ConnectionImpl@38fb5119
(No operations allowed after connection closed.). Possibly consider using a shorter maxLifetime value.
```
- 기존 DBCP는 SELECT 1 같은 커넥션 유효 검사를 주기적으로 했었는데 JDBC4 부터는 커넥션 확인을 쿼리로 하지 않는 것 같다.
- HikariCP도 testQuery 를 설정한게 아니라면 이를 따른다. (setConnectionTestQuery 로 test query를 수행할 수는 있으나 지양한다.)
```
// https://github.com/brettwooldridge/HikariCP/blob/dev/src/main/java/com/zaxxer/hikari/pool/PoolBase.java#L149
 boolean isConnectionDead(final Connection connection)
   {
      try {
         setNetworkTimeout(connection, validationTimeout);
         try {
            final var validationSeconds = (int) Math.max(1000L, validationTimeout) / 1000;

            if (isUseJdbc4Validation) {
               return !connection.isValid(validationSeconds);
            }

            try (var statement = connection.createStatement()) {
               if (isNetworkTimeoutSupported != TRUE) {
                  setQueryTimeout(statement, validationSeconds);
               }

               statement.execute(config.getConnectionTestQuery());
            }
         }
```

- 위 코드에서 알 수 있 듯, JDBC4Validation인지를 확인하면 대신 connection 의 isValid() 를 호출한다.
- (JDBC4Validation 이 아니라면 쿼리를 날리기 위해 createStatement 를 아래서 호출하는 것을 확인할 수 있다)


```
// https://github.com/mysql/mysql-connector-j/blob/805f872a57875f311cb82487efcfb070411a3fa0/src/main/user-impl/java/com/mysql/cj/jdbc/ConnectionImpl.java#L2488
public boolean isValid(int timeout) throws SQLException {
        synchronized (getConnectionMutex()) {
            if (isClosed()) {
                return false;
            }
            try {
                try {
                    pingInternal(false, timeout * 1000);
                } catch (Throwable t) {
                    try {
                        abortInternal();
                    } catch (Throwable ignoreThrown) {
                        // we're dead now anyway
                    }
                    return false;
                }

            } catch (Throwable t) {
                return false;
            }

            return true;
        }
    }
```

- Mysql-conntor-j 에선 이 isValid가 ping 으로 되어 있다.
- Connection 을 날리지 않고, ping 처럼 보다 빠르고 간단한 방법으로 처리하나 보다.

```
// https://github.com/mysql/mysql-connector-j/blob/805f872a57875f311cb82487efcfb070411a3fa0/src/main/user-impl/java/com/mysql/cj/jdbc/ConnectionImpl.java#L1467

@Override
public void pingInternal(boolean checkForClosedConnection, int timeoutMillis) throws SQLException {
    this.session.ping(checkForClosedConnection, timeoutMillis);
}
```

### 그래서 나는 어떻게 했냐

- 30초에 한번씩 쿼리를 직접 발생시켜주고, 그 헬스 상태를 Map 에 기록한다.
- 이때 Health 쿼리는 직접 짜는게 아니라 actuator 의 DataSourceHealthIndicator 를 이용한다.

``` java
@Scheduled(fixedDelay = 30000)
public void healthCheck() {
    DataSourceTargetContextHolder.setContext(DataSourceType.MASTER);
    STATUS_MAP.put(
        DataSourceType.MASTER, 
        indicatorMaster.getHealth(false).getStatus()
    );
    DataSourceTargetContextHolder.setContext(DataSourceType.SLAVE);
    STATUS_MAP.put(
        DataSourceType.MASTER,
        indicatorSlave.getHealth(false).getStatus()
    );
    DataSourceTargetContextHolder.clearContext();
}
```

- 그리고 그 헬스체크 상태에 따라 라우팅 방식을 바꿔주는 것이다.

``` java
public class DataSourceRoutingRule extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        if (DataSourceTargetContextHolder.hasDirectTargetDataSource()) {
            return DataSourceTargetContextHolder.getTargetContext();
        }
        var isReadOnlyQuery = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
        if (isReadOnlyQuery && !DataSourceHealth.isDown(DataSourceType.SLAVE)) {
            return DataSourceType.SLAVE;
        }
        if (!isReadOnlyQuery && DataSourceHealth.isDown(DataSourceType.MASTER)) {
            throw new DataSourceConnectionDownException("Server can read only now");
        }
        return DataSourceType.MASTER;
    }
}
```

- 정상인 상태에서는 Master, Slave 가 각각 부하를 나누고,
- Slave가 down 인 상태에서는 Master 만으로 처리한다.
- Master가 down 인 상태에서는 서비스는 읽기만 처리되고, 쓰기는 에러를 발생하도록 한다.

<img width="887" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/3139cc45-d574-4653-a5e7-e53af9e3e0ae">




