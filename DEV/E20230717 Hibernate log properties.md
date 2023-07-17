## Hibernate log properties

ref, `https://kwonnam.pe.kr/wiki/java/hibernate/log`
```
org.hibernate : 모든 메시지
org.hibernate.SQL = DEBUG : SQL 구문
org.hibernate.type.descriptor.sql = TRACE : 바인딩된 파라미터 값(Hibernate 4,5)
org.hibernate.type = TRACE 은 모든 JDBC 파라미터를 찍으며, 이 경우 로그가 과도하게 찍히기 때문에
org.hibernate.type.BasicTypeRegistry = WARN 으로 로그 출력을 줄여줄수도 있다.
org.hibernate.orm.jdbc.bind = TRACE : 바인딩된 파라미터 값(Hibernate 6)
org.hibernate.SQL_SLOW = INFO : Hibernate 5.4.5 이상 버전에서 Slow Query
hibernate.session.events.log.LOG_QUERIES_SLOWER_THAN_MS=밀리초 property 를 지정해주면 이 값을 따름.
org.hibernate.pretty : flush 시점의 세션이 있는 Entity 들의 상태(최대 20 개만)
org.hibernate.cache : 2차 캐시 상태
org.hibernate.stat = DEBUG : 모든 쿼리의 분석 통계
org.hibernate.tool.hbm2ddl = DEBUG : DDL 로그
org.hibernate.transaction : 트랜잭션 정보
org.hibernate.jdbc : JDBC 리소스 처리 상태 로깅
org.hibernate.hql.ast.AST : Log HQL and SQL ASTs during query parsing
org.hibernate.secure : Log all JAAS authorization requests
```
