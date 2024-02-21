## E20240220 table based lock 을 row lock 으로.md

Mysql 은 row 나 레코드 자체가 아니라 인덱스에 락을 건다. 그래서 검색 조건에 인덱스가 아닌 컬럼이 포함되면 table lock 으로 lock이 처리된다.


```
SELECT * FROM performance_schema.data_locks;
SELECT * FROM performance_schema.events_transactions_current;
```

