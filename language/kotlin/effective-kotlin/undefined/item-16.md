# Item 16: 프로퍼티는 동작이 아니라 상태를 나타내야한다

* 코틀린의 프로퍼티는 자바의 필드와 비슷해보이지만, 사실 서로 완전히 다른 개념임

```
// 코틀린의 프로퍼티
var name: String? = null

// 자바의 필드
String name = null;
```

* 둘 다 데이터를 저장한다는 점은 같음
* 하지만 프로퍼티에는 더 많은 기능이 있음
  * 게터, 터

```kotlin
var name: String = null
    get() = field?.toUpperCase()
    set(value) {
        if(!value.isNullOrBlank()) {
            field = value
        }
    }
```

* val를 사용하면 셋터가 생성되지 않
* var을 사용해서 만든 읽고 쓸 수 있는 프로퍼티는 게터와 세터를 정의할 수 있음
* 이러한 프로퍼티를 **파생 프로퍼(derived property)**라고 부름



