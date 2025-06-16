---
icon: tent-arrow-left-right
---

# 트랜잭셔널 아웃박스 패턴

## Q. 트랜잭셔널 아웃박스 패턴이란 무엇인가요?

**A.**\
트랜잭셔널 아웃박스 패턴은 분산 시스템에서 데이터베이스 쓰기와 메시지(이벤트) 발행을 원자적으로 처리하여 \
이중 쓰기 문제를 해결하는 설계 패턴입니다. \
예를 들어, 상품 데이터를 저장하고 관련 이벤트를 발행할 때, \
두 작업이 독립적으로 처리되면 데이터 정합성이 깨질 수 있습니다.

이 패턴은 데이터베이스에 **Outbox 테이블**을 생성해 이벤트 데이터를 동일한 트랜잭션 내에 저장합니다.

```java
@Transactional
public void createProduct() {
    Product product = new Product("신규 상품");
    productRepository.save(product);
    productOutboxRepository.save(new ProductEvent(product.getId()));
}
```

데이터베이스 트랜잭션의 원자성을 활용해 상품과 이벤트가 모두 저장되거나 모두 실패합니다. \
별도의 프로세스가 Outbox 테이블을 주기적으로 폴링하여 이벤트를 메시지 브로커에 발행하고, \
성공 시 레코드를 삭제하거나 상태를 업데이트합니다. 이를 통해 데이터 정합성과 신뢰성을 보장합니다.

## Q. 트랜잭셔널 아웃박스 패턴을 사용하지 않을 경우 어떤 문제가 발생하나요?

**A.**\
트랜잭셔널 아웃박스 패턴을 사용하지 않고 데이터베이스 쓰기와 이벤트 발행을 별도로 처리하면 \
**이중 쓰기 문제**가 발생할 수 있습니다.

```java
@Transactional
public void createProduct() {
    Product product = new Product("신규 상품");
    productRepository.save(product);
    eventPublisher.publish(new ProductEvent(product.getId()));
}
```

* **문제 시나리오**:
  * 데이터베이스 저장은 성공했지만, 네트워크 문제로 이벤트 발행이 실패하면 \
    이벤트가 누락되어 시스템 간 데이터 불일치 발생.
  * 반대로, 이벤트 발행은 성공했지만 트랜잭션 커밋이 실패해 롤백되면, 잘못된 이벤트가 전파됨.
* **결과**: 데이터 정합성 훼손, 서비스 장애, 또는 복잡한 복구 로직 필요.

트랜잭셔널 아웃박스 패턴은 이벤트 저장을 트랜잭션에 포함시켜 이러한 문제를 방지합니다.

## Q. 트랜잭셔널 아웃박스 패턴의 구현 시 고려해야 할 점은 무엇인가요?

**A.**\
트랜잭셔널 아웃박스 패턴을 구현할 때 다음 사항을 고려해야 합니다:

* **폴링 최적화**\
  Outbox 테이블 폴링은 데이터베이스 부하를 유발할 수 있으므로, 폴링 주기와 배치 크기를 적절히 설정해야 합니다.
* **중복 발행 방지**\
  이벤트의 고유 ID를 부여하고, 발행 성공 시 상태를 업데이트(예: `PROCESSED`)하거나 \
  레코드를 삭제하여 중복 발행을 방지합니다.
* **장애 처리**\
  이벤트 발행 실패 시 재시도 로직(예: 지수 백오프)을 구현하고, \
  실패 이벤트를 Dead Letter Queue(DLQ)에 저장해 추적 가능하도록 합니다.
* **스케일링**\
  대규모 트래픽에서는 테이블 크기 증가를 관리하기 위해 파티셔닝이나 CDC(Change Data Capture) 도구\
  (예: Debezium)를 고려할 수 있습니다.
*   **예시**:

    ```java
    @Scheduled(fixedDelay = 5000)
    public void publishOutboxEvents() {
        List<ProductEvent> events = productOutboxRepository.findUnprocessed();
        for (ProductEvent event : events) {
            try {
                eventPublisher.publish(event);
                event.markAsProcessed();
                productOutboxRepository.save(event);
            } catch (Exception e) {
                log.error("Event publishing failed: {}", event.getId(), e);
            }
        }
    }
    ```

이러한 점을 고려하면 안정적이고 확장 가능한 구현이 가능합니다.

### Q. 트랜잭셔널 아웃박스 패턴과 CDC(Change Data Capture)의 차이점은 무엇인가요?

**A.**\
트랜잭셔널 아웃박스 패턴과 CDC는 모두 이벤트 발행을 처리하지만, 접근 방식과 사용 사례가 다릅니다.

* **트랜잭셔널 아웃박스 패턴**:
  * 이벤트 데이터를 별도의 Outbox 테이블에 저장하고, 폴링 프로세스가 이를 주기적으로 읽어 발행.
  * 장점: 명시적으로 이벤트를 설계하고 관리, 기존 데이터베이스 트랜잭션 활용 가능.
  * 단점: 폴링에 의한 지연과 데이터베이스 부하 발생 가능.
  * 사용 사례: 이벤트 발행 로직을 애플리케이션에서 명확히 제어하고자 할 때.
* **CDC(Change Data Capture)**:
  * 데이터베이스 로그(예: MySQL binlog)를 분석해 변경 사항을 캡처하고 이벤트를 생성.
  * 장점: 데이터베이스 변경을 실시간으로 감지, 별도 테이블 관리 불필요.
  * 단점: 데이터베이스 로그 접근 권한과 추가 도구(예: Debezium, Kafka Connect) 필요.
  * 사용 사례: 데이터베이스 변경을 자동으로 이벤트 스트림으로 전환할 때.

트랜잭셔널 아웃박스는 애플리케이션 레벨에서 이벤트를 명시적으로 관리하며 간단한 구현에 적합하고, \
CDC는 대규모 데이터 변경을 실시간으로 처리하는 데 유리합니다.
