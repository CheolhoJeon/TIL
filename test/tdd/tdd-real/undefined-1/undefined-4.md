# 테스트 우선 개발

* <mark style="color:red;">**테스트 코드는**</mark>
  * 가시적이고 구체적인 목표가 됨
  * 자가 검증을 할 수 있는 목표임
  * 반복 실행될 수 있음
  * 운영 코드의 클라이언트임



* <mark style="color:red;">**운영 코드보다 테스트 코드를 먼저 작성**</mark>
  * 명확하고 검증 가능한 목표를 설정한 후 목표를 달성
  * 프로세스가 코딩에 앞선 목표 설정을 강요
  * **프로그래머는 자신이 풀어야 할 문제를 구체적으로 이해해야 함**



### 1. 탭 **문자 처리하기**

```java
@ParameterizedTest
@ValueSource(strings = {
    "hello\t world",
    "hello \ttworld",
})
void testAboutTabCharacter(String source) {
    String actual = refineTextInstance.refineText(source);
    assertEquals("hello world", actual);
}
```

### 2. @MethodSource를 사용해 기댓 값 전달하기

```java
@ParameterizedTest
@MethodSource("provideStringsForBannedWord")
void testAboutBannedWords(String source, List<String> bannedWords, String expected) {
    String actual = refineTextInstance.refineText(source);
    assertEquals(expected, actual);
}

private static Stream<Arguments> provideStringsForBannedWord() {
    return Stream.of(
        Arguments.of("hello mockist", List.of("mockist", "purist"), "hello *******"),
        Arguments.of("hello purist", List.of("mockist", "purist"), "hello ******")
    );
}
```

{% embed url="https://gmlwjd9405.github.io/2019/11/27/junit5-guide-parameterized-test.html" %}

### 3. Faker를 사용해 입력 데이터 생성하기

```java
@Test
void testAboutGeneratedBannedWords() {
    String bannedWord = faker.lorem().word();
    String source = "hello " + bannedWord;

    String expected = "hello " + "*".repeat(bannedWord.length());
    String actual = refineTextInstance.refineText(source, bannedWord);

    assertEquals(expected, actual);
}
```

{% embed url="https://github.com/DiUS/java-faker" %}

```java
public class RefineText {

    public String refineText(String s, String ... bannedWords) {
        s = s.replace("    ", " ")
            .replace("\\t", " ")
            .replace("  ", " ")
            .replace("  ", " ")
            .replace("  ", " ")
            .replace("mockist", "*******")
            .replace("purist", "******");

        for (String bannedWord : bannedWords) {
            s = s.replace(bannedWord, "*".repeat(bannedWord.length()));
        }
        return s;
    }

}
```

