# Motivations

* 대규모 분산 시스템은 서로 다른 기술과 프로그래밍 언어를 사용하는 여러 독립적인 팀을 통해 모듈 방식으로 구현되는 경우가 많음
* 각 모듈은 안정적으로 서로 통신하며, 독립적인 상태에서 빠르게 진화해야함
* 효율적이고 확장 가능한 통신방식은 분산 시스템에서 중요한 관심사임
* 이는 사용자가 체감하는 지연시간과 시스템 구축 및 실행에 필요한 리소스의 크기에 상당한 영향을 미침
* [Reactive Manifesto](http://www.reactivemanifesto.org)에 기술되어 있으며, [Reactive Streams](http://www.reactive-streams.org)와 [Reactive Extensions](http://www.reactivex.io)에 구현되어 있는 아키텍처 패턴(Architectural pattern)은 비동시 메시징 방식을 선호하고 요청/응답 스타일의 통신 방식을 뛰어넘음(?)
* _RSocket_ 프로토콜은 reactive 원칙을 수용하는 형식적인(formal) 통신 프로토콜임
* 아래의 내용들은 새로운 프로토콜을 정의하게된 동기들을 나열함

## 1. Message Driven

* 네트워크 통신은 기본적으로 비동기임
* RSocket은 이를 수용하고 단일 네트워크 연결을 통해 다중화된(multiplexed)된 메시지 스트림으로 모든 통신을 모델링하며, 응답을 기다리는 동안 동기적으로 차단하지 않음
*   [리액티브 선언서(Reactive Manifesto)](http://www.reactivemanifesto.org)는 다음과 같이 기술하고 있음:

    > `Reactive System`는은 느슨한 결합(loose coupling), 격리(isolation), 위치 투명성(location transpareny)을 보장하고 구성요소 간의 바운더리를 설정하기 위해 비동기 메시지 전달에 의존합니다. 명시적인 메시지-전달(message-passing)을 활용하면 시스템의 메시지 큐를 구현 및 모니터링하고 필요할 때 back-pressure를 적용하여 부하 관리, 탄력성 및 흐름 제어가 가능합니다. Non-blocking 통신을 사용하면 수신자가 활성(active) 상태일 때만 리소스를 사용할 수 있으므로 시스템 오버헤드가 줄어듭니다.
*   또한, [HTTP/2 FAQ](https://http2.github.io/faq/#why-is-http2-multiplexed)는 지속적인 연결을 통한 다중화된(multiplexing) 메시지 지향 프로토콜을 채택하는 동기를 잘 설명하고 있음:

    > HTTP/1.x에는 "head-of-line blocking"이라는 문제가 있습니다. 이 문제에서는 한 번에 하나의 요청만 효과적으로 처리할 수 있습니다.

    > HTTP/1.1은 파이프라이닝으로 이 문제를 해결하려고 시도했지만 문제를 완전히 해결하지 못했습니다(크거나 느린 응답은 여전히 뒤에 있는 다른 사람을 차단할 수 있음). 또한 많은 중개자(intermediaries)와 서버가 파이프라이닝을 올바르게 처리하지 못하기 때문에 배포하기가 매우 어렵습니다.

    > 이렇게 하면 클라이언트가 여러 경험적 방법(종종 추측)을 사용하여 언제 어떤 요청을 오리진(origin)에 연결할지 결정해야 합니다. 페이지가 사용 가능한 연결 수의 10배(또는 그 이상)를 로드하는 것이 일반적이기 때문에 성능에 심각한 영향을 미칠 수 있으며 종종 차단된 요청의 "폭포(waterfall)"가 발생합니다.

    > 다중화는 여러 요청 및 응답 메시지가 동시에 전송되도록 허용하여 이러한 문제를 해결합니다. 한 메시지의 일부를 유선상의 다른 메시지와 섞는 것도 가능합니다.

    > 이것은 차례로 클라이언트가 페이지를 로드하기 위해 오리진당 하나의 연결만 사용할 수 있도록 합니다.
*   지속적 연결에 대해 논의하면서 FAQ가 이어짐:

    > HTTP/1을 사용하면 브라우저는 오리진 당 4\~8개의 연결을 엽니다. 많은 사이트가 여러 출처를 사용하기 때문에 단일 페이지 로드가 30개 이상의 연결을 열 수 있음을 의미할 수 있습니다.

    > 동시에 너무 많은 커넥션을 여는 하나의 응용 프로그램은 TCP가 사용되는 환경의 가정을 깨뜨립니다. 각 연결은 응답을 위해 데이터 플러드(flood)를 시작할 것이기 때문에, 중간 네트워크의 버퍼가 오버플로되어 정체 이벤트 및 재전송을 일으킬 위험이 있습니다.

    > 또한 너무 많은 연결을 사용하면 네트워크 리소스를 부당하게 독점하여 다른 응용 프로그램(예: VoIP)에서 해당 리소스를 "훔쳐"갑니다.

## 2. Interaction Models

* 부적절한 프로토콜은 시스템 개발 비용을 증가시킴
* 부적절한 프로토콜을 사용하는 것은 이 프토토콜이 허용하지 않는 틀에 시스템 설계를 끼워 맞추는 것일 수 있음
* 이로 인해 개발자는 오류를 처리하고 성능을 달성하기 위해 단점을 해결하는데 더 많은 시간을 할애해야함
* 여러 언어를 사용하는 환경에서 이러한 문제는 다른 언어가 이 문제를 해결하기 위해 다른 접근 방식을 사용하기 때문에 증폭되며, 이는 팀 간의 추가 조정을 필요로함
* 현재까지 사실상의 표준은 HTTP이고 모든 것이 요청/응답임
* 어떤 경우에는 지정된 기능에 대한 이상적인 통신 모델이 아닐 수 있음
* 그러한 예 중 하나는 푸쉬 알림임
* 요청/응답 모델은 클라이언트가 서버에서 데이터를 확인하기 위해 지속적으로 폴링하도록 애플리케이션을 강제함
* 폴링을 위해 초당 많은 양의 요청을 수행하는 애플리케이션의 예를 찾는 것은 어렵지 않음
* 이것은 클라이언트, 서버 및 네트워크 측면에서 낭비임
  * 인크라의 사이즈, 운용 복잡도 및 가용성 문제를 키움
  * 즉, 불필요한 비용이 발생함
* 또한 비용을 줄이기 위해 폴링이 더 긴 간격으로 축소되기 때문에 일반적으로 알림 수신 시 사용자 경험에 대한 레이턴시가 증가됨
* 이러한 이유로 RSocket은 하나의 상호 작용 모델에만 국한되지 않음
* 아래에 설명된 다양한 지원 상호 작용 모델은 시스템 설계를 위한 강력하고 새로운 가능성을 열어줌

### 2-1. Fire-and-Forget

* `Fire-and-forget` 모델은 응답이 필요하지 않은 상황에서 유용한 `요청/응답` 모델의 최적화된 형태임
* 응답을 스킵함으로써, 네트워크 사용량을 절약할 수 있을 뿐만 아니라 응답 또는 취소 요청을 기다리고 연결하는데 부기(bookkeeping)이 필요하지 않으므로 클라이언트 및 서버 처리 시간에서도 상당한 성능 최적화가 가능함
* 이 상호 작용 모델은 중요하지 않은 이벤트 로깅과 같이 메시지 손실을 허용하는 사용 사례에 유용함
*   사용법은 아래와 같음:

    ```java
    Future<Void> completionSignalOfSend = socketClient.fireAndForget(message);
    ```

### 2-2. Request/Response (single-response)

* 표준적인 요청/응답 체계는 여전히 지원되며, RSocket을 통한 대부분의 통신이 이러한 형태일 것으로 예상됨
* 이러한 요청/응답 상호 작용은 최적화된 “단 하나의 응답 스트림(streams of only 1 response)”으로 간주될 수 있으며, 단일 연결을 통해 다중화된 비동기 메시지임(?)
* 컨슈머는 응답 메시지를 "대기"하므로 일반적인 요청/응답처럼 보이지만 그 아래에서는 동기적으로 차단되지 않음
*   사용법은 아래와 같음:

    ```java
    Future<Payload> response = socketClient.requestResponse(requestPayload);
    ```

### 2-3. Request/Stream (multi-response, finite)

* request/stream 모델은 여러 메시지를 스트림으로 응답하며, request/response 모델의 확장된 형태임
* “collection” 혹은 “list”를 응답하는 것처럼 생각해보자
* 대신에 모든 데이터는 단일 메시지로 응답하는 것이 아닌, 각 요소가 순서대로 스트리밍됨
* 아래와 같은 유스케이스가 있을 수 있음:
  * 비디오 목록 가져오기
  * 카탈로그에서 제품 가져오기
  * 파일을 Line-by-Line 조회
*   사용법은 아래와 같음:

    ```java
    Publisher<Payload> response = socketClient.requestStream(requestPayload);
    ```

### 2-4. Channel

* 채널(channel)은 각 방향에 연결된 스트림을 활용한 양방향 통신 모델임
* 이 상호작용 모델(interaction model)의 이점을 활용하는 사용 사례의 예는 다음과 같음:
  * client requests a stream of data that initially bursts the current view of the world
  * deltas/diffs are emitted from server to client as changes occur
  * 클라이언트가 criteria, topics 등을 추가/제거하기 위해 시간이 지남에 따라 구독을 업데이트하는 상황
* 양방향 채널이 없으면 클라이언트는 효율적으로 구독을 업데이트만 하는 것이 아니라, 초기 요청을 취소하고 다시 요청하고 모든 데이터를 처음부터 새로 받아야함
*   사용법은 아래와 같음:

    ```java
    Publisher<Payload> output = socketClient.requestChannel(Publisher<Payload> input);
    ```

## 원문

* [RSocket - Motivations](https://rsocket.io/about/motivations)
