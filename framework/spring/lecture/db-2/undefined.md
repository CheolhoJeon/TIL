# 스프링 트랜잭션 이해

## 스프링 트랜잭션 소개

해당 절에서는 스프링 트랜잭션을 더 깊이있게 학습하고, 또 스프링 트랜잭션이 제공하는 다양한 기능들을 자세히 알아본다

먼저 본격적인 기능 설명에 앞서 지금까지 학습한 스프링 트랜잭션을 간략히 복습하면서 정리해보자.

### 스프링 트랜잭션 추상화

각각의 데이터 접근 기술들은 트랜잭션을 처리하는 방식에 차이가 있다. 예를 들어 JDBC 기술과 JPA 기술은 트랜잭션을 사용하는 코드 자체가 다르다

#### JDBC 트랜잭션 코드 예시

```java
public void accountTransfer(String fromId, String toId, int money) throws SQLException {
  Connection con = dataSource.getConnection();
  try {
    con.setAutoCommit(false); //트랜잭션 시작 //비즈니스 로직
    bizLogic(con, fromId, toId, money);
    con.commit(); //성공시 커밋 
  } catch (Exception e) {
    con.rollback(); //실패시 롤백
    throw new IllegalStateException(e);
  } finally {
    release(con);
  }
}
```

#### JPA 트랜잭션 코드 예시

```java
public static void main(String[] args) {
  //엔티티 매니저 팩토리 생성
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
  EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성 
  EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득
  
  try {
    tx.begin();  //트랜잭션 시작 
    logic(em);   //비즈니스 로직 
    tx.commit(); //트랜잭션 커밋
  } catch (Exception e) { 
    tx.rollback(); //트랜잭션 롤백
  } finally {
    em.close(); //엔티티 매니저 종료
  }
  emf.close(); //엔티티 매니저 팩토리 종료
}
```

따라서 JDBC 기술을 사용하다가 JPA 기술로 변경하게 되면 트랜잭션을 사용하는 코드도 모두 함께 변경해야 한다.

스프링은 이런 문제를 해결하기 위해 트랜잭션 추상화를 제공한다. <mark style="color:blue;">**트랜잭션을 사용하는 입장에서는 스프링 트랜잭션 추상화를 통해 둘을 동일한 방식으로 사용할 수 있게 되는 것이다.**</mark>

스프링은 <mark style="color:blue;">`PlatformTransactionManager`</mark>라는 인터페이스를 통해 트랜잭션을 추상화한다.

#### PlatformTransactionManager 인터페이스

```java
package org.springframework.transaction;

public interface PlatformTransactionManager extends TransactionManager {
  TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
  
  void commit(TransactionStatus status) throws TransactionException;
  void rollback(TransactionStatus status) throws TransactionException;
}
```

* 트랜잭션은 트랜잭션 시작(획득), 커밋, 롤백으로 단순하게 추상화할 수 있다.

![](../../../../.gitbook/assets/image.png)

* <mark style="color:blue;">**스링은 트랜잭션을 추상화해서 제공할 뿐만 아니라**</mark>, 실무에서 주로 사용하는 데이터 접근 기술에 대한 <mark style="color:blue;">**트랜잭션 매니저의 구현체도 제공**</mark>한다.&#x20;
* 여기에 더해서 스프링 부트는 어떤 데이터 접근 기술을 사용하는지 자동으로 인식해서 적절한 트랜잭션 매니저를 선택해서 스프링 빈으로 등록해주기 때문에 트랜잭션 매니저를 선택하고 등록하는 과정도 생략할 수 있다.\
  예를 들어, <mark style="color:blue;">`JdbcTemplate`</mark>, <mark style="color:blue;">`MyBatis`</mark>를 사용하면 <mark style="color:blue;">`DataSourceTransactionManager`</mark>를 스프린으로 등록하고, <mark style="color:blue;">`JPA`</mark>를 사용하면 <mark style="color:blue;">`JpaTransactionManager`</mark>를 스프링 빈으로 등록해준다.

{% hint style="info" %}
스프링 5.3 부터는 JDBC 트랜잭션을 관리할 때 <mark style="color:blue;">`DataSourceTransactionManager`</mark>를 상속받아서 약간의 기능을 확장한 <mark style="color:blue;">`JdbcTransactionManager`</mark>를 제공한다. 둘의 기능 차이는 크지 않으므로 같은 것으로 이해하면 된다.
{% endhint %}

### 스프링 트랜잭션 사용 방식

<mark style="color:blue;">`PlatformTransactionManager`</mark>를 사용하는 방법은 크게 2가지가 있다.

#### 선언적 트랜잭션 vs 프로그래밍 방식 트랜잭션 관리

* <mark style="color:blue;">**선언적 트랜잭션 관리(Declarative Transaction Management)**</mark>
  * <mark style="color:blue;">`@Transactional`</mark> 애노테이션 하나만 선언해서 매우 편리하게 트랜잭션을 적용하는 것은 선언적 트랜잭션 관리라 한다.
  * 선언적 트랜잭션 관리는 과거 XML에 설정하기도 했다.
  * 이름 그대로 해당 로직에 트랜잭션을 적용하겠다 라고 어딘가에 선언하기만 하면 트랜잭션이 적용되는 방식이다.
* <mark style="color:blue;">**프로그래밍 방식의 트랜잭션 관리(Programmatic Transaction Management)**</mark>
  * 트잭션 매니저 또는 트랜잭션 템플릿 등을 사용해서 트랜잭션 관련 코드를 직접 작성하는 것을 프로그래밍 방식의 트랜잭션 관리라 한다.



* **프로그래밍 방식의 트랜잭션 관리를 사용하게 되면, 애플리케이션 코드가 트랜잭션이라는 기술 코드와 강하게 결합된다.**
* 선언적 트랜잭션 관리가 프로그래밍 방식에 비해서 훨씬 간결하고 실용적이기 때문에 <mark style="color:blue;">**실무에서는 대부분 선언적 트랜잭션 관리를 사용한다**</mark>

### 선언적 트랜잭션과 AOP

<mark style="color:blue;">`@Transactional`</mark>을 통한 선언적 트랜잭션 관리 방식을 사용하게 되면 기적으로 프록시 방식의 AOP가 적용된다.&#x20;

#### 프록시 도입 전



