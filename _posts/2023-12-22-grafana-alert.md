---
title: "Grafana Alert 조사 및 테스트"
last_modified_at: 2023-12-22
author: 조평연
---

본 포스팅은 Grafana Alert을 알아보는 내용입니다.

# 1. Grafana 이란?
- Grafana 오픈 소스는 오픈 소스 시각화 및 분석 소프트웨어이다

- 이를 통해 저장된 위치에 관계없이 측정항목, 로그 및 추적을 쿼리, 시각화, 경고 및 탐색할 수 있다 

- 시계열 데이터베이스(TSDB) 데이터를 통찰력 있는 그래프와 시각화로 전환하는 도구를 제공한다

# 2. Grafana Alerting 이란?
- 들어오는 측정항목 데이터 또는 로그 항목을 모니터링하고 특정 이벤트나 상황을 관찰한 다음 해당 항목이 발견되면 알림을 보내도록 알림 시스템이다

- 시스템 문제가 발생한 직후 이에 대해 알아볼 수 있다

# 3. Grafana Alerting 설정 방법
## 3-1) 구글 보안 설정
- Grafana로 부터 이메일을 받으려면 보안 수준이 낮은 앱 엑세스를 허용해 줘야한다

- 하지만 2022.05.30 부로 구글에서 더 이상 사용자 이름과 비밀번호만 사용하여 로그인하도록 요청하는 서드 파티 앱 또는 기기의 사용을 지원하지 않아 보안 수준이 낮은 앱 엑세스를 허용해주지 못한다

- 이를 해결하기 위해 2단계 인증 후 앱 비밀번호를 생성해주어야 한다

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FXE9Z0%2FbtsBrMro8zk%2Fg4yyIdmjFT1lkuYmxI6hx1%2Fimg.png)
<br>

### 계정 -> 보안 -> 2단계인증

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbLtTx8%2FbtsBq1I44yu%2FCHRbwQEAEE8eVJ3owQkw8k%2Fimg.png)
<br>

### 계정 -> 보안 -> 앱 비밀번호

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fv8M6X%2FbtsBrfN1Xdl%2FisJKIv3iaNy5I2jpZW4nGk%2Fimg.png)
<br>

## 3-2) Grafana.ini 설정
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FceoxYh%2FbtsBG5YoWBM%2FRjsiLYvZbiyd6B2nrVrKsk%2Fimg.png)
<br>

### 수정 전
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcYHspc%2FbtsBog74nPw%2FitJRZRPyYRqhRdbvFfBPKK%2Fimg.png)
<br>

### 수정 후
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbmr2rR%2FbtsBtJOKjyP%2FkDMKs46W04uIjO7QNeDmuk%2Fimg.png)
<br>

## 3-3) DB 연결
- 사용하고자 하는 DB를 선택 후 url, user, password 입력 후 생성

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBFL2H%2FbtsBrMrpQ33%2FS632lEnsmBVnj3YrUQD4x0%2Fimg.png)
<br>

## 3-4) Alerting 세팅
- 아래와 같이 Alerting 메뉴를 볼 수 있다

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdTNJWX%2FbtsBgwXOS0L%2FTuV6K8ZK20vjckwiG9xFY1%2Fimg.png)
<br>

## 3-4-1) Alert rules
- 임계값이나 여러 표현식을 사용하여 알람을 설정하는 메뉴이다

## Alert rules 생성
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FYNJc1%2FbtsBqtF0pTN%2FrmZZiJpKX5DMU3k9aqnB10%2Fimg.png)
<br>

## Alert rules 이름 작성
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbc8zMo%2FbtsBqJPmQBK%2FHwdyLI1vL4hiEk9wk1FJbK%2Fimg.png)
<br>

## 확인할 테이블 쿼리 작성
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzDGnv%2FbtsBqC3TbXL%2FikHTQ0Ve0O0jXP3yiVvFCk%2Fimg.png)
<br>

## 임계값 설정
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdrGApz%2FbtsButEBG1C%2FuzA5Ia7NLRnJepvpBRBwz0%2Fimg.png)
<br>

## evaluation Interval 시간, pending period 시간 설정
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbzSFiE%2FbtsBqtTzn0s%2F0LunJqwpChOqj1nKuS8sFK%2Fimg.png)
<br>
<br>
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fllses%2FbtsBuxfRGih%2FDVou6oXGcc3R4ASc8J9QB0%2Fimg.png)
<br>

## 주석 설정
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbJTgjh%2FbtsBuowHTdR%2F51aqFTvBpsknscKzmdQA5K%2Fimg.png)
<br>

## 라벨 설정 (임계점 검사에서는 불필요)
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcHH7Uw%2FbtsButLnn2a%2FKhuP5uZHmOz1fyeOw3l09k%2Fimg.png)
<br>

## 3-4-2) Contact points
- Alert message의 템플릿을 만들거나 Alert를 어떠한 플랫폼(gmail, slack 등)으로 전송할지 정하는 메뉴이다

## 알림 받을 이메일 설정 (이외 20종류의 방법 존재 integration 에서 설정)
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkIsx7%2FbtsBuzLvNoD%2FI6iXmtO0GhfOPKUiScznjk%2Fimg.png)
<br>

## Test 후 지정한 이메일로 온 메일 확인
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbClCqK%2FbtsBueukggi%2Fg9xAIJDENIaW6NQp4jfFh1%2Fimg.png)
<br>

## 3-4-3) Notification policies
- Label을 체계적으로 정리하여 그룹화 시켜 확인할 수 있는 메뉴이다

## 알림정책에서 그룹화를 통해 알림 간격, 반복 간격 등 설정 가능
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtxBBq%2FbtsBr6wAdwB%2FDxfn4kFLL1sdkdebQ13lWK%2Fimg.png)
<br>
<br>
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F1mGDv%2FbtsBla8iU9z%2FmiKaxYUbyH0P3DCzUG4fA0%2Fimg.png)
<br>

## 3-4-4) Silences
- 특정 시간 또는 기간동안 Alert를 울리지 않게 하는 메뉴이다

## 특정 시간 및 기간 동안 알림을 받지않게 설정 가능
![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbubHLF%2FbtsBq1bkxI7%2FvbqgpkKkfk3OVvoCG77zak%2Fimg.png)
<br>

## 3-4-5) groups
- 그룹화된 Alert 내용을 한번에 볼 수 있는 메뉴이다

## 3-4-6) Admin
- Alert의 Json파일을 확인할 수 있고 외부의 Alertmanager랑 연동할 수 있다

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxRIxA%2FbtsBmSGoxBw%2F0eAXQfLCrdrQDKRvqrkGlk%2Fimg.png)
<br>

# 4. 테스트 결과
- val 값이 10 초과일 경우 알림이 오게 설정하고 db에서 val 값을 11로 주었을때 아래와 같이 이메일이 오는걸 확인했다

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F5g8V0%2FbtsBtGYRGsY%2F4EwhqHMcwDFTEajbKQRCO0%2Fimg.png)
<br>

5. 참고 문헌
- Grafana Labs 링크 : https://grafana.com/docs/grafana/latest/alerting/
- 사진 출처 출처 : https://vuddus526.tistory.com/494