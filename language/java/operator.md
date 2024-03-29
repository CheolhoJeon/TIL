# Operator

### 1. 연산자와 연산식

* **`연산(operations)`:** 데이터를 처리하여 결과를 산출하는 것
* **`연산자(operator)`:** 연산에 사용되는 표시나 기호
* **`피연산자(operand)`:** 연산되는 데이터
* **`연산식(expressions)`:** 연산자와 피연산자를 이용하여 연산의 과정을 기술한 것

### 2. 단항 연산자

* **`단항 연산자`:** 피연산자가 단 하나뿐인 연산자
* **단항 연산자의 종류**
  * 부호 연산자( **`+`, `-`** )
    * int 이하 정수 타입에 대한 산출 타입: int
  * 증감 연산자( **`++`, `–`** )
  * 논리 부정 연산자( **`!`** )
    * boolean 타입에만 사용 가능
  * 비트 반전 연산자( **`~`** )
    * 부호가 반대인 새로운 값이 산출
    * int 이하 정수 타입에 대한 산출 타입: int

### 3. 이항 연산자

* **`이항 연산자`:** 피연산자가 두 개인 연산자
* **이항 연산자의 종류**
  * 산술 연산자( **`+`, `-`, `*`, `/`, `%`** )
    * 피연산자들의 타입이 동일하지 않을 경우, 피연산자들의 타입을 일치시킨 후 연산을 수행
      * byte + byte => int + int = int
      * int + long => long + long = long
      * int + double => double + double
    * **오버플로우 탐지**
      * 연산 후의 산출값이 산출 타입으로 충분히 표현 가능한지 살펴봐야 함
      * 바로 산술 연산자를 사용하지 말고 메소드를 이용하는 것이 좋음
    * **정확한 계산은 정수 사용**
      * 정확하게 계산해야 할 때는 실수 타입을 사용하지 않는 것이 좋음
    * **NaN과 Infinity 연산**
      * `/`와 `%`의 결과가 `Infinity` 또는 `NaN`이 나오면 다음 연산을 수행하면 안됨
      * **`Double.isInfinite()`**, **`Double.IsNaN()`**
    * **입력값의 NaN 검사**
      * 입력값으로써 문자열을 입력 받을 때, “NaN” 인지 확인해야함
      * **`Double.isNaN()`**
  * 문자열 연결 연산자( **+** )
  * 비교 연산자( **`<`, `<=`, `>`, `>=`, `==`, `!=`** )
    * String 객체의 문자열만 비교하고 싶다면 == 연산자 대신에 **`equals()`** 사용
  * 논리 연산자( **`&&`, `||`, `&`, `|`, `^`, `!`** )
    * **`&&`**`,`` `**`||`**이 **`&, |`** 보다 효율적
  * 비트 연산자( **`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`** )
    * 비트 논리 연산자( **`&`, `|`, `^`** )
    * 비트 이동 연산자( **`<<`, `>>`, `>>>`** )
  * 대입 연산자( **`=`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `^=`, `|=`, `<<=`, `>>=`, `>>>=`** )

### 4. 삼항 연산자

* **`삼항 연산자`:** 세 개의 피연산자를 필요로 하는 연산자
* **`[조건식] ? [값 또는 연산식] : [값 또는 연산식]`**
