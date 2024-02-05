---
title: "Elastic Alert 조사 및 테스트"
last_modified_at: 2024-02-05
author: 조평연
---

본 포스팅은 Elastic Alert을 알아보는 내용입니다.

# 1. 규칙
- 알림을 받기 위해 여러가지 규칙중 하나를 선택해서 데이터를 분석하고 그에 따른 결과를 받을 수 있습니다.

- 그 중 대중적으로 다 제공하는 Metric threshold을 이용해서 임계점 검사를 해보았습니다.

- Metric threshold 란 측정항목 집계가 임계값을 초과하면 경고합니다.

- 임계값을 10으로 뒀을때 10을 초과하는 데이터가 생기면 그에 따른 경고 알림을 보내는 것입니다.

## 1-1) Rules(1)
- 규칙을 생성합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbEFGSw%2FbtsEuOFKu4V%2FCMViTYpzhzAHtBCLtm5AH0%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 1-2) Rules(2)
- 규칙을 이름을 작성합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FC7tRD%2FbtsEnSW6Ufn%2FhISgkmf3MhTPirBpCRClj0%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 1-3) Rules(3)
- 규칙을 타입을 작성합니다.
- 필터로 Metric을 선택한 뒤 metric threshold 를 선택합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbWVEdg%2FbtsEtjznCYo%2FDfWJmo4hUG01K36kmbnKGk%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 1-4) Rules(4)
- 임계점 검사할 대상과 검사 범위 값을 설정합니다.
- 원하는 임계점 값을 선택하고 측정 기간을 선택합니다.
- 테스트 용도로 알림을 빠르게 오게 하기 위해 13으로 했습니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb55dvY%2FbtsEu4aCrHY%2F2B1WOUxJu8Rk5kMJwUzj00%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 1-5) Rules(5)
- 데이터 검사 주기 설정헙니다.
- 1분마다 임계점보다 낮은지 확인합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FRmPgi%2FbtsEmrTb60e%2Fkxf9lBwa5ixrwcgmCSHR0k%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 1-6) Rules(6)
- 알림을 보낼 매체를 지정합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpXlTY%2FbtsEoq66NsY%2Fres9eEAKcNRBKDQxlJhgZ1%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 1-7) Rules(7)
- 이메일 알림에 대한 상세 설정을 합니다.
- 테스트에서는 elastic에서 지정한 default값 사용합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbGWvWb%2FbtsEnQ550sP%2FKO6bD9aL3AUV2KXErHIW80%2Fimg.png" width="70%" height="70%" />
</p>
<br>

## 1-8) Rules(8)
- 이메일 주소와 제목 및 내용을 작성합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoRoq5%2FbtsEm9ENX7D%2FaitZtUPEVnB5dVJ6cVzAcK%2Fimg.png" width="70%" height="70%" />
</p>
<br>

# 2. 이메일 확인
- 설정한 이메일 제목과 설정 값에 대해 정상적으로 메일 온것을 확인합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbVX7XP%2FbtsEonCAcRX%2FnMNOk0KW6MAIdZqgYWwFBk%2Fimg.png" width="70%" height="70%" />
</p>
<br>

- 작성한 메세지와 동일함을 확인합니다.
<p align="center">
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrX4tQ%2FbtsEuA8HQjI%2FmArcjpT6Fy2rME7p5JPd30%2Fimg.png" width="70%" height="70%" />
</p>
<br>

# 3. 참고문헌
- https://www.elastic.co/guide/en/observability/8.11/metrics-threshold-alert.html (Elastic 공식문서)