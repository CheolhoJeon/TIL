# Functions

*   함수(f_unction_)는 서브 루틴(_subroutines_)라고 불림:

    ```kotlin
    fun functionName(arg1: Type1, arg2: Type2, ...): ReturnType {
      // Lines of code ...
      return result
    }
    ```
* `fun` 키워드 뒤에는 함수 이름과 괄호 안에 매개변수 목록이 옴
* 코틀린은 매개변수의 타입을 유추할 수 없으므로, <mark style="color:blue;">각 매개변수의 타입을 명시적으로 지정해야함</mark>
* 함수는 타입을 가지며, `:` 뒤에 명시됨\

* 함수 시그니처 뒤에는 중괄호 안에 포함된 함수 본문이 위치함
* `return` 문은 함수의 반환 값을 제공\

*   함수가 하나의 표현식만 포함하는 경우, <mark style="color:blue;">함수를 축약하여 표현할 수 있음</mark>:

    ```kotlin
    fun functionName(arg1: Type1, arg2: Type2, ...): ReturnType = result
    ```
* 이러한 형식을 _<mark style="color:red;">expression body</mark>_라고 함
* 중괄호 대신 등호 뒤에 표현식을 사용할 수 있음
*   이때 코틀린의 타입 추론을 통해 <mark style="color:blue;">반환 타입을 생략할 수 있음</mark>

    ```kotlin
    fun cube(x: Int): Int {
      return x * x * x
    }

    fun bang(s: String) = s + "!"

    fun main() {
      println(cube(3))
      println(bang("pop"))
    }
    ```
