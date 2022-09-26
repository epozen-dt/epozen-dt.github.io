---
title: "Docker remote api - Python 클라이언트 라이브러리"
last_modified_at: 2022-09-26
author: 심건우
---

이번 포스팅에선 이전 포스팅에서 java로 접근했던 Docker remote api를 python 클라이언트로 접근해보았다.

## Docker remote API Python SDK
 Docker 공식 remote api 관련 sdk이다.
  - Apache License 2.0
  - Docker 측에서 직접 테스트 완료
  - Docker 측에서 버전 관리
  - pip를 통한 간단 설치(pip install docker)

[참고](https://docs.docker.com/engine/api/sdk/)

## 구현 방식 비교 (python vs java)
 아래 코드는 두 라이브러리 간 컨테이너 생성 및 환경 변수 설정 시, 차이를 비교한 것이다.
  - python은 dictionary에서 key를 통해 값을 가져오고, java는 builder 패턴을 사용

![image](https://user-images.githubusercontent.com/87160438/192229718-9e07b65d-7c0d-4b7e-b0b0-457176ced2e3.png)


## 코드 예시
 - 컨테이너 목록 조회

![image](https://user-images.githubusercontent.com/87160438/192230717-c47a16d4-1974-4db5-8cd5-ab1648204261.png)


 - 도커 이미지 pull

![image](https://user-images.githubusercontent.com/87160438/192230835-e6fd1b34-7b51-4af3-8c70-b4257f087a87.png)


 - 컨테이너 환경 변수 설정 및 실행

![image](https://user-images.githubusercontent.com/87160438/192231110-444726b2-0836-407a-a7b0-9767bde07270.png)


 - 컨테이너 중지 및 삭제

![image](https://user-images.githubusercontent.com/87160438/192231213-6307a2d1-d0b2-44c6-8c3d-dc82c464bd3c.png)
