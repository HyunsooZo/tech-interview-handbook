---
icon: spinner-scale
---

# Thread

## Q1. 자바에서 스레드(Thread)에 대해 설명해주세요.

**A.**\
스레드는 프로세스 내에서 실행되는 **최소 실행 단위**로, 여러 작업을 동시에 처리할 수 있게 합니다. \
예: 파일 다운로드와 UI 업데이트 동시 수행.

* **동시성(Concurrency)**: \
  스레드는 동시에 실행되는 것처럼 보이지만, CPU가 시분할(Time Slicing)로 스레드를 빠르게 전환하여 처리합니다.
* **스레드 생성 방법은**
  1. Thread 클래스를 상속받아 run() 메서드 오버라이드 하는 **Thread 클래스 상속** 방법과
  2. Runnable 인터페이스를 구현해 run() 메서드 정의하는 **Runnable 인터페이스 구현** 방법이 있습니다.
  3. 자바에서는 단일 상속 제한의 이유로 Runnable 인터페이스를 구현하는 것이 권장됩니다.

***

## Q2. 스레드로 여러 작업을 동시에 수행할 때 발생할 수 있는 문제점에 대해 설명해주세요.

**A.**

1. **Race Condition (경쟁 조건)**
   * 여러 스레드가 **공유 자원**에 동시에 접근해 **경쟁 조건이** 발생할 수 있습니다.
   * 예를들어 두 스레드가 동시에 변수 값을 증가시킬 때 잘못된 결과가 발생할 수 있습니다.
   * synchronized 키워드 또는 ReentrantLock으로 임계 영역 보호하여 해결 가능합니다.
2. **DeadLock (교착 상태)**
   * 두 스레드가 서로의 자원을 점유한 상태로 무한 대기하는 교착 상태가 발생할 수 있습니다.
   * 예를들어 스레드 A가 자원 1을 점유하고 자원 2를 기다리며, \
     스레드 B가 자원 2를 점유하고 자원 1을 기다리는 상태를 말합니다.
   * 자원 획득 순서 일정하게 유지, 타임아웃 설정, synchronized 또는 ReentrantLock 사용하여 \
     해결 할 수 있습니다.&#x20;
3. **Starvation (기아 상태)**
   * 특정 스레드가 자원을 할당받지 못해 실행되지 못하는 기아상태가 발생할 수 있습니다.
   * 스레드 우선순위 조정, 공정한 Lock 사용으로 해결 가능합니다.

***

## Q3. synchronized 키워드를 사용하는 것과 ReentrantLock을 사용하는 것은 어떠한 차이점이 있나요?

**A.**

<table data-header-hidden><thead><tr><th width="55.40234375"></th><th></th><th></th></tr></thead><tbody><tr><td><strong>항목</strong></td><td><strong>synchronized</strong></td><td><strong>ReentrantLock</strong></td></tr><tr><td><strong>정의</strong></td><td>자바의 기본 Lock 메커니즘. <br>다른 스레드가 접근하지 못하도록 임계 영역 설정.</td><td>java.util.concurrent.locks 패키지의 <br>Lock 구현체로, 유연한 동기화 제어 제공.</td></tr><tr><td><strong>사용 방식</strong></td><td>메서드나 블록에 키워드 사용. <br>Lock 획득/해제 자동 처리.</td><td>lock()과 unlock() 메서드 명시적 호출. <br>finally 블록에서 unlock() 필수.</td></tr><tr><td><strong>유연성</strong></td><td>단순, 타임아웃/조건 대기 불가.</td><td>타임아웃(tryLock()), <br>조건 대기(Condition), <br>공정한 Lock 지원.</td></tr><tr><td><strong>권장 사용</strong></td><td>간단한 동기화 상황.</td><td>세밀한 제어(타임아웃, 조건 대기 등) 필요한 상황.</td></tr></tbody></table>
