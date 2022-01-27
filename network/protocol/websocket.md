# WebSocket

## 소개

* 웹 소켓(WebSocket)이 존재 않던 시절에는 클라이언트와 서버 간의 양방향 통신이 필요한 웹 애플리케이션을 \
  구현하기 위해 막대한 양의 HTTP 폴링이 필요했음
* 이로 인해 아래와 같은 문제가 발생함:
  * 전송/수신을 위해 각각 TCP 커넥션을 필요로하므로, 서버는 다수의 커넥션을 맺어야함
  * 각 메시지의 HTTP 헤더로 인한 오버헤드가 발생함
  * 클라이언트는 인커밍(incoming) 커넥션과 아웃고잉(outgoing) 커넥션 간의 관계(맵핑)를 유지해야함
    * 이를 유지하지 않으면, 엉뚱한 아웃고잉 커넥션으로 메시지를 전달할 여지가 생김



* 웹 소켓은 하나의 TCP 커넥션을 통해 양방향 통신을 지원함
* 즉, HTTP 폴링 방식의 대안이 될 수 있음
* 하지만 다른 설계 포인트와 마찬가지로 항상 HTTP 폴링 방식이 나쁜 것은 아님



* 웹 소켓 프로토콜은 HTTP를 사용하는 기존 인프라(프록시, 필터링, 인증)를 활용 할수 있도록 설계되었음
* 웹 소켓은 기존 HTTP 인프라 환경에서 기존 양방향 HTTP 기술의 목표를 해결하는 것이 목표임
* 이를 위해, HTTP 포트인 `80` 및 `443`에서 작동하고 HTTP 프록시 및 여러 중간자(intermediaries)를 \
  지원하도록 설계됨
* 그러나 이러한 설계 철학이 웹 소켓을 HTTP에 한정 짓는 것은 아님



## 참고자료

* [RFC- 6455 - The WebScoket Protocol](https://datatracker.ietf.org/doc/html/rfc6455#page-4)
* [Web on Servlet Stack](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket)
