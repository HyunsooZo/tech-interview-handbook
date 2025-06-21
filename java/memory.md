---
icon: brain-circuit
---

# Memory

## Q. 자바의 메모리 구조에 대해 설명해주세요.

**A.**\
자바의 메모리 구조는 JVM(Java Virtual Machine)에서 관리되며, \
크게 **스택(Stack)**, **힙(Heap)**, **메서드 영역(Method Area)**&#xC73C;로 나뉩니다.

* **스택**\
  각 스레드마다 독립적으로 존재하며, 메서드 호출 시 생성되는 프레임이 저장됩니다. \
  프레임에는 지역 변수, 매개변수, 리턴 값이 포함되며, LIFO(후입선출) 구조로 동작합니다. \
  메서드 종료 시 프레임은 제거됩니다.
* **힙**\
  모든 스레드가 공유하는 영역으로, 객체와 배열이 동적으로 할당됩니다. \
  힙은 **Young Generation**(Eden, Survivor)과 **Old Generation**으로 나뉘며, \
  가비지 컬렉터(GC)가 사용되지 않는 객체를 정리합니다.
* **메서드 영역**\
  클래스 구조(메서드 코드, 클래스 변수(static)), 런타임 상수 풀(상수 값)이 저장되며, 모든 스레드가 공유합니다. \
  JVM 시작 시 생성됩니다.

이 구조는 자바 프로그램의 효율적인 메모리 관리와 스레드 안전성을 보장합니다.

## Q. 자바에서 메모리 누수(Memory Leak)가 발생하는 원인과 이를 방지하는 방법은 무엇인가요?

**A.**\
자바에서 메모리 누수는 더 이상 필요 없는 객체가 메모리에 남아 해제되지 않는 현상입니다. \
주요 원인과 방지 방법은 다음과 같습니다:

* **정적 필드 참조**: 정적 필드가 객체를 참조하면 프로그램 종료까지 메모리 해제 안 됨.
  * **방지**: 불필요한 참조를 null로 설정하거나 정적 필드 사용 최소화.
* **리소스 미해제**: 파일, 네트워크 연결 등 닫지 않으면 누수 발생.
  *   **방지**: try-with-resources로 자동 해제.

      ```java
      try (FileInputStream fis = new FileInputStream("file.txt")) {
          // 작업
      } // 자동 해제
      ```
* **잘못된 equals()/hashCode()**: HashMap/Set에서 중복 객체 추가로 메모리 낭비.
  * **방지**: equals()와 hashCode()를 올바르게 구현.
* **내부/익명 클래스**: 외부 클래스 인스턴스를 암묵적으로 참조해 누수.
  * **방지**: 정적 내부 클래스 또는 람다 표현식 사용.
* **캐시 부적절 사용**: 캐시에서 객체 제거 안 하면 누수.
  *   **방지**: WeakHashMap 또는 주기적 캐시 정리.

      ```java
      Map<String, Object> cache = new WeakHashMap<>();
      ```
* **리스너/콜백 미제거**: 불필요한 리스너가 메모리 점유.
  * **방지**: 리스너 해제(예: removeListener() 호출).

이러한 방법으로 메모리 누수를 방지해 안정적인 애플리케이션을 유지할 수 있습니다.

## Q. 자바에서 메모리 최적화를 위해 객체 풀링(Object Pooling)은 어떻게 활용되나요?

**A.**\
객체 풀링은 자주 사용되는 객체를 생성/삭제 대신 재사용해 메모리 할당과 해제 비용을 줄이는 기법입니다. \
자바에서 메모리 최적화에 유용합니다.

* **활용 방식**:
  * 객체 풀(예: ObjectPool)을 구현해 미리 생성된 객체를 저장하고, 필요 시 제공/반환.
  *   예: 데이터베이스 연결(Connection Pool), 스레드 풀(ThreadPoolExecutor).

      ```java
      public class ObjectPool<T> {
          private List<T> pool = new ArrayList<>();
          public T getObject() {
              return pool.isEmpty() ? createObject() : pool.remove(0);
          }
          public void returnObject(T obj) {
              pool.add(obj);
          }
      }
      ```
* **장점**:
  * 객체 생성/해제 오버헤드 감소.
  * 메모리 사용량 최적화, 특히 고비용 객체(예: DB 연결)에 효과적.
* **단점**:
  * 풀 관리 복잡성 증가.
  * 잘못된 구현 시 메모리 누수 발생(예: 반환 안 된 객체).
* **사용 사례**: Apache Commons Pool, HikariCP(데이터베이스 연결 풀).
  *   예: HikariCP로 DB 연결 재사용.

      ```java
      HikinariDataSource ds = new HikariDataSource(config);
      try (Connection conn = ds.getConnection()) {
          // DB 작업
      } // 연결 반환
      ```

객체 풀링은 고성능 애플리케이션에서 메모리 효율성을 높이지만, 풀 크기와 반환 로직을 신중히 관리해야 합니다.
