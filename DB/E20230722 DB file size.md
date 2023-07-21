## E20230722 DB file size

### Select db file size
```
SELECT table_schema AS "Database", SUM(data_length + index_length) / 1024 / 1024 AS "Size (MB)" FROM information_schema.TABLES GROUP BY table_schema
```

### 대충 

테이블이 1개, 컬럼이 4개, 인덱스 id만, data 100개인 db가 8mb가 나온다
