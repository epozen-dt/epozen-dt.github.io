---
title: "Solace pub sub 구축 및 웹 테스트"
last_modified_at: 2024-01-17
author: 조평연
---

본 포스팅은 Solace PubSub+ 구축 및 웹 테스트를 알아보는 내용입니다.

# 1. Solace 란?
- 종합적인 이벤트 스트리밍 및 관리 플랫폼을 통해 이벤트 중심 아키텍처를 적용, 관리 및 활용할 수 있도록 도와주는 플랫폼을 제공하는 회사입니다.

# 2. Solace PubSub+ 플랫폼 이란
- 이벤트 기반 아키텍처 (EDA)를 하이브리드 클라우드, 멀티 클라우드 및 IoT 환경에서 설계, 배포 및 관리하는 플랫폼 입니다.

# 3. 구축방법
## 3-1) 포트 내용
포트 8080 : 
- Solace PubSub+ Broker Manager로 메시지 브로커 컨테이너를 구성할 때 이 포트를 사용합니다.

포트 55555(Windows | Linux) / 포트 55554(Mac OS) : 
- 귀하의 애플리케이션은 Solace API를 사용하여 이 포트의 메시지 브로커에 연결할 수 있습니다.
- Mac 사용자의 경우 Mac OS가 이제 포트 55555를 예약하므로 포트 55554를 기본 Solace SMF 포트 55555에 매핑합니다.

포트 8008 : 
- JavaScript 샘플 애플리케이션은 이 포트를 사용하여 메시지 브로커를 통해 웹 메시징 트래픽을 전달합니다.

포트 1883 및 8000 : 
- 각각 TCP 및 WebSocket을 통한 MQTT 연결용 포트입니다.

포트 5672 : 
- Apache QPID API를 사용하는 AMQP 1.0 애플리케이션 연결합니다.

포트 9000 : 
- REST를 사용하여 Solace의 RESTful API 포트로 메시징 및 이벤트 데이터를 보냅니다.

포트 2222 : 
- 고급 구성을 위해 SSH를 사용하여 Solace 명령줄 인터페이스(CLI)에 연결합니다.

<br>

## 3-2) docker run 활용
- 공식 문서에서 제공하는 docker run 명령어를 이용하여 리눅스 환경에서 이미지를 내려 받고 컨테이너를 올립니다.

- 아래와 같이 작성합니다.

docker run -d -p 8080:8080 -p 55555:55555 -p 8008:8008 -p 1883:1883 -p 8000:8000 -p 5672:5672 -p 9000:9000 -p 2222:2222 <br>
--shm-size=2g --env username_admin_globalaccesslevel=admin --env username_admin_password=admin --name=solace solace/solace-pubsub-standard

<br>

## 3-3) docker-compose 활용
- 공식 문서에서 제공되지는 않지만 아래와 같이 구축 가능합니다.

- 아래와 같이 작성합니다.

version: '2' <br>
services: <br>
&nbsp; solace: <br>
&nbsp;&nbsp; image: solace/solace-pubsub-standard <br>
&nbsp;&nbsp; ports: <br>
&nbsp;&nbsp;&nbsp; - "8080:8080" <br>
&nbsp;&nbsp;&nbsp; - "55555:55555" <br>
&nbsp;&nbsp;&nbsp; - "8008:8008" <br>
&nbsp;&nbsp;&nbsp; - "1883:1883" <br>
&nbsp;&nbsp;&nbsp; - "8000:8000" <br>
&nbsp;&nbsp;&nbsp; - "5672:5672" <br>
&nbsp;&nbsp;&nbsp; - "9000:9000" <br>
&nbsp;&nbsp;&nbsp; - "2222:2222" <br>
&nbsp;&nbsp; shm_size: "2g" <br>
&nbsp;&nbsp; environment: <br>
&nbsp;&nbsp;&nbsp; - username_admin_globalaccesslevel=admin <br>
&nbsp;&nbsp;&nbsp; - username_admin_password=admin <br>
&nbsp;&nbsp;&nbsp; - SOLACE_JAIL_LOCAL_SERVICE=false <br>
&nbsp;&nbsp; container_name: solace

<br>

# 4. 웹 테스트
## 4-1) url 에 접속하여 미리 설정한 id, pw 를 이용해 로그인
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fctf94N%2FbtsDs2sFQEN%2FlfUe9RxQXNwYp0nUFzJ3y1%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 4-2) Message VPNs 접속
- 기본으로 생성된 Message VPNs에 접속합니다.

<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdAuutI%2FbtsEqYBNSVu%2F50qAShyunIm1mkdtGKNzR1%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 4-3) 샘플테스트

- 테스트 순서는 아래와 같습니다.

1. 연결을 클릭하여 게시자 애플리케이션을 메시지 브로커에 연결합니다.

2. 연결을 클릭하여 구독자 애플리케이션을 메시지 브로커에 연결합니다.

3. 구독자 애플리케이션에서 구독을 클릭하여 주제를 구독합니다.

   ex) solace/try/this/topic

4. 게시자 애플리케이션에서 게시를 클릭하여 solace/try/this/topic 주제에

   "Hello World" 메시지를 게시합니다.

   게시한 메시지는 구독자 애플리케이션 의 메시지 아래에 표시됩니다.
<br>

- 결과는 아래와 같습니다

<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FYrqas%2FbtsEmC1lDts%2FnKeBKk60KtjV6jrk5JdZj1%2Fimg.png" width="70%" height="70%" />
</p>

<br>

# 5. 참고문헌
- https://docs.solace.com/Get-Started/Solace-PubSub-Platform.htm (solace PubSub+ 공식문서)
- https://solace.com/products/event-broker/software/getting-started/ (웹 테스트 참고 공식문서)