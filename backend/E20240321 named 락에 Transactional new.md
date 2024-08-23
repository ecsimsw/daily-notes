### E20240321 named 락에 Transactional new.md
네임드 락 키워드를 많이 봤다.     
분산환경에서 락이 필요한데 DB 를 공유하고 있는 경우 사용할 수 있겠다.
     
비관적 락이나 낙관적 락처럼 DB table/row 에 락을 거는게 아니다.    
그저 락을 위한 DB 메타데이터를 사용한다고 생각한다.     
그래서 락이 필요한 조회에 보다는 스케줄러 단일 실행 보장이나 공유 자원에 접근 제어 등에 쓰면 좋겠다.    

Named lock은 주로 Transaction 을 서비스와 따로 얻어 사용한다.     
Named lock 은 트랜잭션과 상관없이 직접 release 를 해줘야하기에 어떤 작업을 마치고 release 가 필요한데, 기존에 존재하는 트랜잭션에서 release 후 기존 트랜잭션을 커밋/롤백하기엔 위험해서다.     
무슨 말이냐면 아래처럼 하면 commit 과 release lock 사이에 간극이 발생한다는 것이다.     
```
A transaction begin
try lock
// handle shared resource
release lock
A transaction commit
```
     
그래서 트랜잭션을 분리한다.
```
L transaction begin
try lock

A transaction begin
// handle shared resource
A transaction commit

release lock
L transaction commit
```
     
그렇기에 한 동작 안에서 여러 트랜잭션이 처리되어야 하고, 최소 두개의 커넥션이 필요하니 Connection pool 에 대한 고민이 필요하곘다.

### 근데 이러면 비관적 락이 더 유리한거 아니야?

공유 자원의 트랜잭션과 락의 주기를 같이 하는 비관적 락이 더 유리해보인다.     
코드나 설정도 깔끔하고 커넥션도 덜 먹는다.      

락이 필요한 공통 자원이 DB 의 row 라면 나는 그냥 비관적 락을 쓰겠다.     
그헣지 않다면 named lock 도 의미가 있을 것이다. 예를 들어 hi와 bye 를 격리해서 출력하는 상황이라고 해보자. 비관적 락을 어떻게 사용하면 좋을까.     
이럴 떈 편하게 네임드락을 쓰면 어떨까 싶다.     

```
L transaction begin
get lock
say hi
say bye
release lock
L transaction commit
```

그렇지 않고서 유명한 예제인 재고나 내 예제인 사용량 업데이트처럼 딱 한 로우에 락이 필요한 경우에서 굳이 네임드락을 써야하는 이유가 있을까?     
비관적 락과 달리 타임 아웃이 가능하다지만 반대로 타임 아웃 없이 트랜잭션과 주기를 같이하는게 제일 성능에 좋을 것 같고,      
정 타임 아웃이 필요하면 비관적 락도 hint 를 준다면??      

DB 트랜잭션이 아니라 다른 작업 (이벤트 처리, 스케줄링)처럼 비관적 락을 쓰기 애매한 곳에서 사용하는 이유는 명확히 알겠는데     
흠 아직 DB 격리를 위해서 비관적 락보다 네임드 락을 사용하는 이유는 잘 모르겠다.     

