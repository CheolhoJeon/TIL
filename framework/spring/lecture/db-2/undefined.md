# 스프링 트랜잭션 이해

## 스프링 트랜잭션 전파1 - 커밋, 롤백

트랜잭션이 둘 이상 있을 때 어떻게 동작하는지 자세히 알아보고, 스프링이 제공하는 <mark style="color:blue;">**트랜잭션 전파(propagation)**</mark> 개념도 함께 알아본다.

트랜잭션 전파를 이해하는 과정을 통해서 <mark style="color:blue;">**스프링 트랜잭션의 동작 원리**</mark>도 더 깊이있게 이해해보자.

먼저 간단한 스프링 트랜잭션 코드를 통해 기본 원리를 학습하고, 이후에 실제 예를 통해 어떻게 활용하는지 알아보겠다.

간단한 예제 코드로 스프링 트랜잭션을 실행해보자.

{% embed url="https://gist.github.com/d450dde01855d2f1bbcd31a1e0f7f757" %}
BasicTxTest.java
{% endembed %}



