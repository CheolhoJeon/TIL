# var & val, Data Types

*   `val` 키워드를 사용하여 변경되지 않는 변수를 생성할 수 있음:

    ```kotlin
    val identifier: Type = Value
    val score: Int = 98
    ```
* `val`로 값을 할당한 경우, 값을 변경할 수 없음
*   코틀린의 _<mark style="color:red;">타입 추론(type inference)</mark>_은 초기값을 통해 자동으로 타입을 결정할 수 있음:

    ```kotlin
    val identifier = Value
    val score = 98
    ```

``

*   `var`(variable) 선언은 `val`를 선언하는 방식과 동일함:

    ```kotlin
    var identifier1 = initialization
    var identifier2: Type = initialization
    ```
*   `val`와 다르게 `var`은 변수에 저장된 값을 변경할 수 있음:

    ```kotlin
    var hoursSpent = 20
    hoursSpent = 25
    ```
* 그러나 _자료형(type)_은 변경할 수 없음
