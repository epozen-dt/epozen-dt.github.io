---
title: "Solace REST Example"
last_modified_at: 2023-12-22
author: Do-soo, KIM
---


> 이번 포스팅에서는 Solace에서 지원하는 Open Protocol 중, REST에 대해서 정리해 보려고 합니다.

### 개요
---

Solace는 JMS 및 OpenMAMA와 같은 개방형 API와 AMQP, MQTT, REST와 같은 개방형 유선 프로토콜을 지원하여 애플리케이션, 기타 미들웨어 및 데이터 이동 기술과의 통합을 용이하게 합니다. 이를 통해 다양한 애플리케이션, 빅 데이터 시스템, 클라우드 서비스 및 IoT 장치 간에 실시간 데이터 흐름을 얻을 수 있습니다.

이 중에서도 REST를 이용해서 Solace PubSub+ 이벤트 브로커와 메시지를 주고 받는 방법을 알아보겠습니다.

Solace REST 메시징 API를 사용하면 HTTP 클라이언트가 HTTP POST 요청을 사용하여 이벤트 브로커와 메시지를 보내고 받을 수 있습니다. 이를 통해 REST 클라이언트는 Solace에서 제공하는 API를 사용할 필요 없이 Solace PubSub+ 이벤트 브로커 클라이언트와 메시지를 보내고 받을 수 있습니다.

아래 그림은 애플리케이션이 RESTful API를 사용하여 HTTP를 통해 메시지를 보내고 받는 방법을 보여줍니다.


<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/8337ae15-b12f-4f83-9981-98ea6ce0e07c" width="90%" height="90%" />
</p>

REST Producer는 HTTP POST 요청 body에 메시지 내용을 보내고, 응답 내용은 HTTP 200 OK 응답의 body에 전달됩니다. 

#### REST 수행 단계
---

아래 그림은 이벤트 브로커에 대한 연결 설정부터 데이터 흐름 시작까지의 단계 수행 과정입니다.<br>

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/009395d1-8edf-41db-ac62-725cd436570a" width="90%" height="90%" />
</p>


1. REST 메시지 생성

2. REST 메시지 송신

3. 브로커 인증

4. 브로커 승인

5. 브로커 경로

6. URI 기반 브로커 지속

7. Subscribe 설정

8. 웹후크를 통해 애플리케이션에 대한 연결 open

9. 메시지 삭제

### REST 테스트
---

자, 그럼 이제 REST를 통해 메시지를 보내볼까요?

필자는 curl을 사용해 테스트 해 보았습니다.<br>
여러분들은 편하게 Postman 같은 무료 프로그램을 사용하셔도 됩니다.

Q/test라는 Queue로 "Hello World Queue"라는 메시지를 보내보겠습니다.<br>
사용방법은 다음과 같습니다.

```
curl -X POST -d {message} http://{ip:port}/QUEUE/{queue name} --header "Content-Type: text/plain"
```

그럼 메시지를 한번 보내볼까요?


<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/4c410e06-c99b-4a35-a437-c60185eb0275" width="100%" height="100%" />
</p>

별 다른 응답이 없어서 제대로 보내졌는지 모르겠네요.<br>
Queue로 보낸 메시지를 직접 읽어서 확인해봐야 합니다.

공식 문서 상에서는 REST Consumer가 있는 것처럼 설명이 되어 있지만 어딜 찾아봐도 consumer 예제는 없었습니다.<br>
그래서 귀찮기는 하지만, Solace에서 제공하는 메시지 송수신 테스트 유틸인 SDKPerf를 사용해서 메시지를 읽어보도록 하겠습니다.

사용방법은 다음과 같습니다.

```
sdkperf_java.bat -cip={ip:port} -cu={id@vpn} -cp={pw} -sql={destination} -md -q
```

그럼 아까 메시지가 제대로 송신되었는지 한번 읽어볼까요?

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/b9575928-91a1-46ec-a040-13a2132e6a38" width="100%" height="100%" />
</p>

아까 보냈던 메시지를 잘 읽어왔네요.

여기까지 REST를 이용해 Solace 브로커에 메시지를 보내는 방법을 살펴보았습니다.

> Reference

- https://docs.solace.com (Solace 공식 문서)




