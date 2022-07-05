# @Transactional



트랜잭션이란?



스프링에서는 두 가지 형태의 트랜잭션 관리 기법을 제공한다

* [Declarative Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative)(이하, 선언적 트랜잭션 관리)
* [Programmatic Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic)

스프링 공식문서에 따르면 대부분의 개발자가 선언적 트랜잭션 관리방식을 사용하고 있으며, 대부분의 케이스에서 권고된다고 안내하고 있다. 이 문서의 주제인 `@Transactional`은 <mark style="color:blue;">**애노테이션 기반의 선언적 트랜잭션 관리방식**</mark>에서 사용되는 핵심 애노테이션이다(XML 기반의 선언적 트랜잭션 관리 방식도 존재한다).

차근차근히 살펴보도록 하자.

## 선언 위치

[참고](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-annotations)

클래스, 메서

나중에 다시 기술

## 역할

## 트랜잭션 매니저



* AOP와 트랜잭션 메타데이터의 조합은 적절한 TransactionManager 구현과 함께 TransactionInterceptor를 사용하여 메서드 호출을 중심으로 트랜잭션을 구동하는 AOP 프록시를 생성합니다.



```java
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

* [TransactionManager](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionManager.html)
  * 트랜잭션 매니저임을 의미하는 마커(Marker) 인터페이스
* [PlatformTransactionManager](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-strategies)
  * 트랜잭션 관리를 위한 오퍼레이션의 집합을 제공하는 인터페이스
* [TransactionStatus](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionStatus.html)
  * 트랜잭션의 상태를 나타냄
* TransactionException
  * PlatformTransactionManager의 모든 오퍼레이션을 통해 던져질 수 있는 익셉션의 상위 타
* [TransactionDefinition](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html)
  * 스프링의 트랜잭션 관련 프로퍼티를 정의하는 인터페이
  * Propagation:&#x20;
  * isolation:&#x20;
  * Timeout:&#x20;
  * Read-only status:&#x20;



