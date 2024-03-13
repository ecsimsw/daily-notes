## MVCC

한 레코드를 여러 버전으로 관리하여 동시성을 제어하는 방법.

### Repeatable read

서로 다른 두 트랜잭션이 있을 때, A 트랜잭션에서 읽기 -> B 트랜잭션 쓰기 후 커밋 -> A 트랜잭션에서 다시 읽기가 일어난다면 앞선 읽기와 다음 읽기가 같은 값을 갖을 수 있을까? 
원래는 안될텐데 Repeatable read는 이게 가능하다. 그게 Repeatable read니까.    

처음 생각할 수 있는 방식은 lock 처리하는 것이다. 서로 다른 두 트랜잭션을 아예 격리 시켜버리면 된다. A읽기 -> A읽기 -> B쓰기로 처리해버리면 같은 읽기 값이 될테니. 문제는 이렇게하면 퍼포먼스가 안나올 것이다. 매번 대기가 필요할테니 말이다.    
    
Mysql 은 로그를 이용해서 MVCC를 처리한다. 레코드의 변경이 생기면 버퍼 풀에 변경 쿼리를 기록하고, 변경 전 컬럼 데이터(스냅샷)을 언두 로그에 저장한다.        
외부 트랜잭션은 격리 수준에 따라 Repeatable read 를 지원하면 언두 로그를 읽고, 지원하지 않으면 데이터 그 자체를 읽는다.     
이렇게 함으로써 매번 대기에서 벗어나면서도 여러번 조회에도 동일 레코드를 보장하는 것이다.        

그리고 이 언두 로그를 필요로 하는 트랜잭션이 모두 없어지면 그때 언두 로그를 제거한다.


### 실습
```
A> start transaction;
B> start transaction;
B> select * from member where id = 1;   // age = 12
A> update member set name=jinhwan, age=13 where id =1;
B> select * from member where id = 1;   // age = 12
A> commit
B> select * from member where id = 1;   // age = 12
B> commit

A> select * from member where id = 1;   // age = 13
```



