---
description: 기능 명세를 만족하는 것 외에 지속적으로 코드를 정리하면 얻을 수 있는 이점에 대해 고민함
---

# 정리된 코드

### 작업 환경 정리

* **생산성**
  * 정리된 환경과 어지럽혀진 환경에서의 작업 생산성 차이
* **지속성**
  * 작업 환경의 생산성이 일정 수준 미만으로 떨어지면 더 이상 그 환경에서 작업 진행은 불가능
* <mark style="color:red;">**코드는 작업 환경이자 작업 결과물**</mark>

### 리팩터링

* 팩터라이제이션(factoriztion) 혹은 팩터링(factoring)은 수학에서 아래와 같은 의미를 지님
  * 15 ⇒ 3 \* 5
  * $x^2 - 4$ ⇒ (x-2)(x+2)
  * 즉, 특정 수학적 개체를 더 단순하거나 작은 개체들의 조합으로 표현하는 것
  * 여기서 주요 쟁점은 원래의 의미가 훼손되지 않는다는 것
* Re-factoring
  * 원래의 의미를 훼손하지 않으면서 코드의 구조를 바꾸는 작업임
  * 그렇다면 ‘그 코드가 가지는 원래 의미를 어떻게 유지할것인가’라는 문제가 발생함
  * **테스트를 통해 리팩터링 과정에서 의미가 훼손되었는지 검증할 수 있음**
  * 하지만 수동 테스트에만 의지하는 경우, 리팩터링 과정이 매우 괴로울 수 있음
  * <mark style="color:red;">**테스트 자동화**</mark>**를 통해 이러한 문제를 해결할 수 있음**

### 실습

```java
public class RefineText {

    public String refineText(String source, String ... bannedWords) {
        source = normalizeWhitesSpaces(source);
        source = compactWhiteSpaces(source);
        source = maskBannedWords(source, bannedWords);

        return source;
    }

    private String maskBannedWords(String source, final String[] bannedWords) {
        return Arrays.stream(bannedWords).reduce(source, this::maskBannedWord);
    }

    private String maskBannedWord(final String source, final String bannedWord) {
        return source.replace(bannedWord, "*".repeat(bannedWord.length()));
    }

    private String normalizeWhitesSpaces(final String source) {
        return source.replace("\\t", " ");
    }

    private String compactWhiteSpaces(final String source) {
        return source.contains("  ") ? compactWhiteSpaces(source.replace("  ", " ")) : source;
    }

}
```
