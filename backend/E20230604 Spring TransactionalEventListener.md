## TransactionalEventListener

Spring에서 Event를 처리할 때 Transaction에 묶인 Event를 아무 생각없이 EventListener으로 핸들링해선 안된다.

예를 들면 아래와 같이 한 트랜잭션으로 묶인 로직에서 Event를 발생시켜 `4. 이벤트 발생`의 핸들링을 분리했다고 해보자.
``` java
@Transactional
public OrderResponse order(OrderRequest orderRequest) {
    // 1. 재고 확인 로직 
    // 2. 재고 차감하는 로직
    // 3. 주문 정보를 저장하는 로직
    eventPublisher.publishEvent(OrderCreatedEvent.of(savedOrder)); // 4. 이벤트 발생
    // 5. 처리 결과들을 조합하는 로직
    return OrderResponse.of(result);
}
```

만약 5번에서 문제가 발생하여 트랜잭션을 rollback 해야하는 상황이라면 4번에 앞선 1,2,3의 처리는 롤백이 될 수 있으나 4번의 이벤트를 처리한 핸들러의 동작은 어찌할 수 없을 것이다.
``` java
@Async
@EventListener
public void sendMessageQueue(OrderCreatedEvent orderCreatedEvent) {
}
```

이때 TransactionalEventListener를 사용하면 트랜잭션에 묶인 로직에서 트랜잭션 처리 결과로 핸들링 여부를 결정할 수 있다.

``` java
@Async
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void sendMessageQueue(OrderCreatedEvent orderCreatedEvent) {
}
```

위처럼 phase를 TransactionPhase.AFTER_COMMIT으로 하는 경우, 앞선 order transaction의 처리가 정상적으로 commit 되는 경우에만 위 이벤트 핸들러가 실행된다. 

```
AFTER_COMMIT : Handle the event after the commit has completed successfully.
AFTER_COMPLETION : Handle the event after the transaction has completed.
AFTER_ROLLBACK : Handle the event if the transaction has rolled back.
BEFORE_COMMIT : Handle the event before transaction commit.
```
올바른 phase를 결정해서 트랜잭션 처리 결과에 따른 이벤트 처리를 의도하자. 
