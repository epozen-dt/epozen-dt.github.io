---
title: "Solace pub sub 구축 및 웹 테스트"
last_modified_at: 2024-01-17
author: 조평연
---

본 포스팅은 Solace pub sub 구축 및 웹 테스트를 알아보는 내용입니다.

# 1. Solace 란?
- 종합적인 이벤트 스트리밍 및 관리 플랫폼을 통해 이벤트 중심 아키텍처를 적용, 관리 및 활용할 수 있도록 도와주는 플랫폼을 제공하는 회사입니다.

# 2. Solace PubSub+ 플랫폼 이란
- 이벤트 기반 아키텍처 (EDA)를 하이브리드 클라우드, 멀티 클라우드 및 IoT 환경에서 설계, 배포 및 관리하는 플랫폼 입니다.

# 3. 구축방법
## 3-1) 포트 내용
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqvKTF%2FbtsDrgkG1bz%2FuTGES1iVNKGJlX97Z4lW9k%2Fimg.png)
<br>

## 3-2) docker run 활용
- 공식 문서에서 제공하는 docker run 명령어를 이용하여 리눅스 환경에서 이미지를 내려 받고 컨테이너를 올립니다.

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbGgNBE%2FbtsDxgDWGpM%2FRJkVJSaAkrB9Bjfku45bB0%2Fimg.png)
<br>

## 3-3) docker-compose 활용
- 공식 문서에서 제공되지는 않지만 아래와 같이 구축 가능합니다.

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fxc9xH%2FbtsDuRR0Yew%2FMhlGqigu4LPO5KO8O4Nqlk%2Fimg.png)
<br>

# 4. 웹 테스트
## 4-1) url 에 접속하여 미리 설정한 id, pw 를 이용해 로그인

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fctf94N%2FbtsDs2sFQEN%2FlfUe9RxQXNwYp0nUFzJ3y1%2Fimg.png)
<br>

## 4-2) Message VPNs 접속

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpBjju%2FbtsDrju0yJD%2FsQDwjSc5J02lZS34qKn6IK%2Fimg.png)
<br>

## 4-3) 샘플테스트

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBDb2x%2FbtsDyQR3Acd%2FR0GAeXT0KzXiGEk76eLS8K%2Fimg.png)
<br>

### 결과는 아래와 같습니다
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWIIOf%2FbtsDxTVMPan%2Fz69OJRahc4TkRBPhn8SnE1%2Fimg.png)
<br>