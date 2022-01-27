# Motivations

* 대규모 분산 시스템은 서로 다른 기술과 프로그래밍 언어를 사용하는 여러 독립적인 팀을 통해 모듈 방식으로 구현되는 경우가 많음
* 각 모듈은 안정적으로 서로 통신하며, 독립적인 상태에서 빠르게 진화해야함
* 효율적이고 확장 가능한 통신방식은 분산 시스템에서 중요한 관심사임
* 이는 사용자가 체감하는 지연시간과 시스템 구축 및 실행에 필요한 리소스의 크기에 상당한 영향을 미침
* [Reactive Manifesto](http://www.reactivemanifesto.org)에 기술되어 있으며, [Reactive Streams](http://www.reactive-streams.org)와 [Reactive Extensions](http://www.reactivex.io)에 구현되어 있는 아키텍처 패턴(Architectural pattern)은 비동시 메시징 방식을 선호하고 요청/응답 스타일의 통신 방식을 뛰어넘음(?)
* _RSocket_ 프로토콜은 reactive 원칙을 수용하는 형식적인(formal) 통신 프로토콜임
* 아래의 내용들은 새로운 프로토콜을 정의하게된 동기들을 나열합니다.

## 1. Message Driven

* 네트워크 통신은 기본적으로 비동기임
* RSocket은 이를 수용하고 단일 네트워크 연결을 통해 다중화된(multiplexed)된 메시지 스트림으로 모든 통신을 모델링하며, 응답을 기다리는 동안 동기적으로 차단하지 않음

## 참고자료

* [RSocket - Motivations](https://rsocket.io/about/motivations)
