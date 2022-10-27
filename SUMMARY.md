# Table of contents

* [About Me](README.md)

## OS

* [Concepts](os/concepts/README.md)
  * [Introduction](os/concepts/introduction.md)
  * [Process](os/concepts/process.md)
  * [Threads & Concurrency](os/concepts/threads-and-concurrency.md)

## Network

* [Protocol](network/protocol/README.md)
  * [WebSocket](network/protocol/websocket.md)
  * [RSocket](network/protocol/rsocket/README.md)
    * [Motivations](network/protocol/rsocket/motivations.md)

## DATABASE

* [Concepts](database/concepts/README.md)
  * [회복과 병행 제어](database/concepts/undefined/README.md)
    * [트랜잭션](database/concepts/undefined/undefined.md)
    * [장애와 회복](database/concepts/undefined/undefined-1.md)
    * [병행 제어](database/concepts/undefined/undefined-2.md)
* [SQL](database/sql/README.md)
  * [Basics](database/sql/basics/README.md)
    * [날짜와 타임스탬프 다루기](database/sql/basics/undefined.md)
    * [NULL 값을 디폴트 값으로 대치하기](database/sql/basics/null.md)
    * [문자열 다루기](database/sql/basics/undefined-1.md)
    * [2개의 값 비율 계산하기](database/sql/basics/2.md)
    * [그룹핑](database/sql/basics/undefined-2.md)
    * [가로 <-> 세로](database/sql/basics/less-than-greater-than.md)
    * [테이블 결합하기](database/sql/basics/undefined-3.md)
    * [계산된 테이블에 이름 붙여 사용하기](database/sql/basics/undefined-4.md)
  * [Recipes](database/sql/recipes/README.md)
    * [시계열 기반으로 데이터 집계하기](database/sql/recipes/undefined.md)
    * [다면척인 축을 사용해 데이터 집약하기](database/sql/recipes/undefined-1.md)
    * [사용자 전체의 특징과 경향 찾기](database/sql/recipes/undefined-2.md)

## Language

* [Java](language/java/README.md)
  * [Introduce](language/java/introduce.md)
  * [Variable And Type](language/java/variable-and-type.md)
  * [Operator](language/java/operator.md)
  * [If Statement & Looping](language/java/if-statement-and-looping.md)
* [Kotlin](language/kotlin/README.md)
  * [Basics](language/kotlin/basics/README.md)
    * [Hello, World](language/kotlin/basics/hello-world.md)
    * [var & val, Data Types](language/kotlin/basics/var-and-val-data-types.md)
    * [Functions](language/kotlin/basics/functions.md)
    * [Boolean](language/kotlin/basics/boolean.md)
  * [Effective Kotlin](language/kotlin/effective-kotlin/README.md)
    * [가독성](language/kotlin/effective-kotlin/undefined/README.md)
      * [Item 16: 프로퍼티는 동작이 아니라 상태를 나타내야한다](language/kotlin/effective-kotlin/undefined/item-16.md)

## Design

* [OOP](design/oop/README.md)
  * [설계 품질과 트레이드 오프](design/oop/undefined.md)
  * [서브클래싱과 서브타이핑](design/oop/undefined-1.md)
* [MSA](design/msa.md)
  * [Microservice Patterns](design/msa/microservice-patterns/README.md)
    * [모놀리식과 마이크로서비스](design/msa/microservice-patterns/undefined.md)
    * [마이크로서비스 쿼리 구현](design/msa/microservice-patterns/undefined-1.md)
* [CQRS](design/cqrs/README.md)
  * [CQRS Documents by Greg Young](design/cqrs/cqrs-documents-by-greg-young.md)
* [DDD](design/ddd/README.md)
  * [Implementing DDD](design/ddd/implementing-ddd/README.md)
    * [애그리거트](design/ddd/implementing-ddd/undefined.md)
    * [바운디드 컨텍스트의 통합](design/ddd/implementing-ddd/undefined-1.md)
  * [DDD - Eric Evans](design/ddd/ddd-eric-evans/README.md)
    * [도메인의 격리](design/ddd/undefined.md)
    * [소프트웨어에서 표현되는 모델](design/ddd/undefined-1.md)
    * [도메인 객체의 생명 주기](design/ddd/undefined-2.md)

## Test

* [TDD](test/tdd/README.md)
  * [TDD START](test/tdd/tdd-start/README.md)
    * [TDD 시작](test/tdd/tdd-start/tdd.md)
    * [JUnit5 기초](test/tdd/tdd-start/junit5.md)
  * [TDD REAL](test/tdd/tdd-real/README.md)
    * [좋은 코드](test/tdd/tdd-real/undefined.md)
    * [테스트 주도 개발 기초](test/tdd/tdd-real/undefined-1/README.md)
      * [코드 기능 명세](test/tdd/tdd-real/undefined-1/undefined.md)
      * [테스트 기법](test/tdd/tdd-real/undefined-1/undefined-1.md)
      * [코드 분해](test/tdd/tdd-real/undefined-1/undefined-2.md)
      * [단위 테스트](test/tdd/tdd-real/undefined-1/undefined-3.md)
      * [테스트 우선 개발](test/tdd/tdd-real/undefined-1/undefined-4.md)
      * [정리된 코드](test/tdd/tdd-real/undefined-1/undefined-5.md)

## Etc

* [CPS](etc/cps.md)

## Framework

* [Spring](framework/spring/README.md)
  * [Lecture](framework/spring/lecture/README.md)
    * [스프링 DB 2편 - 데이터 접근 활용 기술](framework/spring/lecture/db-2/README.md)
      * [스프링 트랜잭션 이해](framework/spring/lecture/db-2/undefined.md)
  * [Annotations](framework/spring/annotations/README.md)
    * [@Transactional](framework/spring/annotations/transactional.md)
  * [Document](framework/spring/document/README.md)
    * [Spring Framework](framework/spring/document/spring-framework/README.md)
      * [Data Access](framework/spring/document/spring-framework/data-access.md)

## Optimizing

* [@MockBean 성능 이슈](optimizing/mockbean.md)
