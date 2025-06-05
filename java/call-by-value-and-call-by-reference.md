---
icon: paste
---

# Call By Value & Call By Reference

## Q. Call By Value와 Call By Reference의 차이는 무엇인가요?

**A.**

* **Call By Value는** 함수에 인자를 전달할 때, 그 값을 복사하여 전달하는 방식입니다.\
  함수 내부에서 인자를 수정해도 원본 값은 변하지 않습니다. \
  이 방식의 장점은 원본 데이터가 안전하게 보존된다는 점이며, \
  단점은 값을 복사하기 때문에 메모리 사용량이 증가할 수 있다는 점입니다.
* **예**: `int a = 10`을 함수 `void changeValue(int x)`에 전달하면, \
  함수는 `a`의 복사본 `10`을 `x`로 받습니다. \
  함수에서 `x = 20`으로 변경해도 원본 a는 여전히 10입니다.
* **Call By Reference는** 인자의 메모리 주소를 직접 참조하여 전달하는 방식입니다. \
  함수 내부에서 인자를 수정하면 원본 데이터도 함께 변경됩니다. \
  복사를 하지 않으므로 속도가 빠르고 메모리를 절약할 수 있지만, \
  원본 데이터가 의도치 않게 바뀔 위험이 있습니다.
* **예**: 배열 `arr = [1, 2, 3]`의 주소를 함수에 전달하면, \
  함수에서 `arr[0] = 10`으로 수정하면 원본 배열도 `[10, 2, 3]`으로 바뀝니다.

## Q. Java에서는 Call By Value와 Call By Reference가 어떻게 적용되나요?

**A.**

Java는 **모든 인자 전달이 Call By Value** 방식으로 동작합니다. \
기본형이든 참조형이든, 값을 복사하여 전달합니다. \
특히 참조형의 경우, 객체의 "주소값" 때문에 Call By Reference로 오해하기 쉽지만, \
이는 엄밀히 Call By Value입니다. 그 이유를 다음과 같이 정리했습니다.

* **기본형 (Primitive Type)**: int, double 같은 기본형은 값 자체를 복사하여 전달합니다. \
  예를 들어, int a = 10을 함수에 전달하면 함수 내부에서 a를 변경해도 원본 값은 그대로 유지됩니다.
*   **참조형 (Reference Type)**: \
    객체(Object)나 배열은 객체의 메모리 주소를 가리키는 **참조값**을 복사하여 전달합니다. \
    따라서 함수 내부에서 객체의 필드를 수정하면 원본 객체도 바뀝니다. \
    하지만 이는 Call By Reference가 아닙니다. 이유는 두 가지입니다:

    1. Java가 전달하는 것은 실제 메모리 주소가 아니라, 주소를 가리키는 **참조값**의 복사본입니다.
    2. 함수 내부에서 전달받은 참조값을 다른 객체로 변경하면(예: `obj = new Object()`), \
       원본 참조에는 영향을 주지 않습니다. 즉, 참조값 자체를 복사한 것이므로 Call By Value로 동작합니다.

    예 : `Person p = new Person("Alice")`를 함수 `void changePerson(Person q)`에 전달하면, \
    함수에서 `q.setName("Bob")`을 호출하면 원본 `p`의 이름도 "Bob"으로 바뀝니다. \
    하지만 `q = new Person("Charlie")`를 하면 원본 `p`는 여전히 "Bob"을 가리킵니다. \
    이는 `q`가 `p`의 참조값 복사본을 받았기 때문입니다.

결론적으로, Java에서는 기본형은 값 자체를, 참조형은 참조값을 복사하여 전달합니다.&#x20;

## Q. 왜 리스트를 함수 인자로 받고 수정하면 원본 리스트가 바뀌나요?

Java의 참조형이 **참조값을 복사**하여 전달하기 때문입니다. \
리스트(ArrayList, LinkedList 등)는 참조형 객체로, \
함수에 전달될 때 리스트의 메모리 주소를 가리키는 참조값이 복사됩니다. \
함수 내부에서 이 참조값을 통해 리스트의 요소를 수정하면, \
원본 리스트가 가리키는 메모리 공간이 변경되므로 원본도 함께 바뀝니다. \
하지만 리스트 자체를 다른 리스트로 교체하면 원본은 영향을 받지 않습니다.

*   **예**:

    ```java
    import java.util.ArrayList;
    import java.util.List;

    List<String> list = new ArrayList<>();
    list.add("Apple");
    void modifyList(List<String> param) {
        param.add("Banana"); // 원본 리스트에 "Banana" 추가
        param = new ArrayList<>(); // param이 새로운 리스트를 가리킴
        param.add("Orange"); // 새로운 리스트에 "Orange" 추가
    }
    ```

    * **결과**: 함수 호출 후, list는 `["Apple", "Banana"]`입니다.
      * `param.add("Banana")`는 원본 리스트의 메모리 공간을 수정하여 원본 list에 영향을 줍니다.
      * 하지만 `param = new ArrayList<>()`는 param의 참조값만 새로운 리스트로 변경하므로 \
        원본 list에는 영향을 주지 않습니다. `param.add("Orange")`는 새로운 리스트에만 적용됩니다.

이는 Java가 실제 메모리 주소가 아닌 **참조값의 복사본**을 전달하기 때문입니다. \
즉, 기본형은 값 자체를, 참조형은 참조값을 복사하여 전달한다고 할 수 있습니다.&#x20;
