# Boolean

부울 대수 연산을 위해 코틀린은 아래와 같은 연산자를 제공함:

* `!` (not)
* `&&` (and)
* `||` (or)

```kotlin
fun main() {
  val opens = 9
  val closes = 20
  println("Operating hours: $opens - $closes")
  val hour = 6
  println("Current time: " + hour)

  val isOpen = hour >= opens && hour < closes
  println("Open: " + isOpen)
  println("Not open: " + !isOpen)

  val isClosed = hour < opens || hour >= closes
  println("Closed: " + isClosed)
}
```
