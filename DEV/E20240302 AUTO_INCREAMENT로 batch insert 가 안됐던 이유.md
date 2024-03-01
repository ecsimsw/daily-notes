## E20240302 batch insert 가 안됐던 이유.md

1. 내가 id 생성을 AUTO_INCREAMENT로 한다.
2. AUTO_INCREMENT는 DBMS에 insert 쿼리가 날라가야 id 값을 얻을 수 있다.
3. 하이버네이트는 'transactional write-behind'으로 id 를 채택하고 있는데 insert 하고 id 가 만들어져야 연관 관계가 있는 엔티티의 FK로 사용될 수 있음을 말하는거 같다.

> JDBC 를 써야함
> saveAll 은 list를 순회하면서 저장하는 정도. 대신 직접 반복문을 돌면 트랜잭션을 찾아 들어가는 과정이 필요한데 이 과정이 없어져서 성능이 개선됨.
> 이거 트랜잭션 로그로 확인하면 보임, save 반복하면 기존 트랜잭션 찾고 조인하는거 본 기억이 있는데 일단 지금은 패스. 

```
logging.level.ROOT=INFO
logging.level.org.springframework.orm.jpa=DEBUG
logging.level.org.springframework.transaction=DEBUG
```
