---
icon: block-question
---

# Generic

## Q1. 자바에서 제네릭이란 무엇인가요?

**A.**\
제네릭은 클래스나 메서드를 작성할 때 **구체적인 데이터 타입을 나중에 지정**할 수 있게 해주는 기능입니다. \
타입을 미리 정하지 않아 코드의 **재사용성**과 **타입 안전성**을 높여줍니다.

*   **예시**: RPG 게임에서 '아이템 상자'를 만든다고 할때, \
    이 상자는 포션, 무기, 방어구 등 어떤 아이템이든 담을 수 있어야 합니다.

    ```java
    public class ItemBox<T> {
        private T item;
        public void setItem(T item) { this.item = item; }
        public T getItem() { return item; }
    }
    ```

    사용 시:

    ```java
    ItemBox<Potion> potionBox = new ItemBox<>();
    potionBox.setItem(new Potion("체력 포션", 100));
    Potion potion = potionBox.getItem(); // 형변환 없이 사용
    ```
* **장점**:
  * **타입 안전성**: 컴파일러가 잘못된 타입 사용을 사전에 차단할 수 있습니다.
  * **형변환 제거**: 명시적 캐스팅 없이 코드가 간결해집니다.
  * **재사용성**: 다양한 타입에 동일한 로직 적용이 가능합니다.

## Q2. Object를 사용하면 모든 타입을 다룰 수 있는데, 왜 제네릭을 써야 하나요?

**A.**\
`Object` 타입은 모든 객체를 다룰 수 있지만, 제네릭을 사용하는 이유는 **안전성과 편의성** 때문입니다.

*   **Object의 문제점**:

    * 값을 꺼낼 때마다 **명시적 형변환** 필요.
    * **런타임 에러** 위험: 잘못된 타입이 들어가면 `ClassCastException` 발생.

    ```java
    List<Object> list = new ArrayList<>();
    list.add("Hello");
    list.add(123);
    Integer num = (Integer) list.get(0); // ClassCastException 발생
    ```
* **제네릭의 장점**:
  * **컴파일 시 타입 체크**: 잘못된 타입 삽입을 컴파일 단계에서 차단.
  * **형변환 불필요**: 코드 간결, 가독성 향상.
  *   **예시**:

      ```java
      List<String> list = new ArrayList<>();
      list.add("Hello");
      // list.add(123); // 컴파일 에러
      String str = list.get(0); // 형변환 없이 사용
      ```
* **결론**: 제네릭은 런타임 오류를 줄이고, 코드의 의도를 명확히 하며, 유지보수성을 높여줍니다.

## Q3. 제네릭 와일드카드란 무엇인가요?

**A.**\
와일드카드(`?`)는 제네릭에서 **특정 타입을 유연하게 처리**하기 위해 사용됩니다. \
메서드에서 다양한 타입을 다룰 때 유용합니다.

* **종류**:
  1. **비제한 와일드카드 (`?`)**
     * 모든 타입 허용, **읽기 전용**.
     *   **예시**: 어떤 리스트든 출력 가능.

         ```java
         public void printList(List<?> list) {
             for (Object o : list) {
                 System.out.println(o);
             }
             // list.add(123); // 컴파일 에러 (쓰기 제한)
         }
         ```
  2. **상한 제한 와일드카드 (`? extends Type`)**
     * `Type` 또는 그 하위 타입만 허용, **읽기 전용**.
     *   **예시**: `Number`의 하위 타입(`Integer`, `Double`) 처리.

         ```java
         public double sum(List<? extends Number> nums) {
             double total = 0;
             for (Number n : nums) {
                 total += n.doubleValue();
             }
             return total;
         }
         ```
  3. **하한 제한 와일드카드 (`? super Type`)**
     * `Type` 또는 그 상위 타입만 허용, **쓰기 가능**, 읽기는 `Object`로 제한.
     *   **예시**: `Integer`를 추가할 수 있는 리스트.

         ```java
         public void addNumbers(List<? super Integer> list) {
             list.add(1);
             list.add(2);
             // Integer n = list.get(0); // 컴파일 에러
         }
         ```
* **장점**: 메서드의 유연성 증가, 읽기/쓰기 역할 명확화.

### Q4. 자바 프로그램 실행 시 제네릭은 어떻게 동작하나요? 원시 타입을 제네릭으로 사용할 수 없는 이유는?

**A.**

* 제네릭은 **컴파일 시점**에 타입 안전성을 보장하고, \
  **실행 시점**에는 타입 소거로 타입 정보가 제거됩니다.
  * 컴파일 후 제네릭 타입은 Object로 대체.
  * 컴파일러가 자동으로 형변환 삽입.
  *   **예시**:

      ```java
      // 작성 코드
      List<String> list = new ArrayList<>();
      list.add("Hello");
      String str = list.get(0);
      // 컴파일 후
      List list = new ArrayList();
      list.add("Hello");
      String str = (String) list.get(0);
      ```
  * **목적**: Java 5에서 제네릭 도입 시 기존 코드와의 **하위 호환성** 유지.
* **원시 타입 사용 불가 이유**:
  * 제네릭은 Object 기반으로 동작하며, 원시 타입(int, double 등)은 형변환 불가.
  *   **대안**: 래퍼 클래스(Integer, Double) 사용.

      ```java
      List<Integer> list = new ArrayList<>(); // OK
      // List<int> list = new ArrayList<>(); // 컴파일 에러
      ```

## Q5. 제네릭 클래스와 제네릭 메서드의 차이점은 무엇인가요?

**A.**

* **제네릭 클래스**:
  * 클래스 전체가 특정 타입에 고정. 타입 파라미터는 클래스 선언에 정의합니다.
  *   **예시**:

      ```java
      public class ItemBox<T> {
          private T item;
          public void setItem(T item) { this.item = item; }
          public T getItem() { return item; }
      }
      ```

      ```java
      ItemBox<Potion> box = new ItemBox<>();
      box.setItem(new Potion("마나 포션", 50));
      ```
* **제네릭 메서드**:
  * 메서드 단위로 독립적인 타입 지정. 메서드 선언 시 `<T>` 를 사용합니다.
  *   **예시**:

      ```java
      public class Dungeon {
          public <T> void inspectItem(T item) {
              System.out.println("검사된 아이템: " + item);
          }
      }
      ```

      ```java
      Dungeon dungeon = new Dungeon();
      dungeon.inspectItem(new Potion("체력 포션", 100));
      dungeon.inspectItem(new Sword("롱소드"));
      ```
* **차이점**:
  * **적용 범위**: 제네릭 클래스는 클래스 전체, 제네릭 메서드는 메서드 단위.
  * **유연성**: 제네릭 메서드는 호출마다 다른 타입 처리 가능.
  * **선언 방식**: 메서드는 `<T>`를 메서드 앞에 명시.
  * **장점**: 제네릭 메서드는 특정 작업에서 타입 안전성과 간결함 제공, 컴파일러가 타입 체크로 안전성 보장.
