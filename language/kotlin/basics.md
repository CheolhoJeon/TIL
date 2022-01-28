# Basics

## 1. Why Kotlin

## 2. Basics

### Hello, World

* 코틀린은 `//`, `/**/` 주석 스타일을 지원함
*   코틀린 프로그램의 엔트리 포인트(entry point)는 `main()` 함수임

    ```kotlin
    fun main() {
    	println("Hello, world!")
    }
    /* Output:
    Hello, world!
    */
    ```
* 코틀린에서는 표현식(expression)이나 구문(statement)의 끝에 `;`을 사용할 필요가 없음
* 세미콜론은 오직 한 줄 내에서 여러 표현식 혹은 구문을 구분하기 위해서 사용됨

### `var` & `val`, Data Types

*   `val` 키워드를 사용하여 변경되지 않는 변수를 생성할 수 있음:

    ```kotlin
    val identifier: Type = Value
    val score: Int = 98
    ```
* `val`로 값을 할당한 경우, 값을 변경할 수 없음
*   코틀린의 _타입 추론(type inference)_은 초기값을 통해 자동으로 타입을 결정할 수 있음:

    ```kotlin
    val identifier = Value
    val score = 98
    ```
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
