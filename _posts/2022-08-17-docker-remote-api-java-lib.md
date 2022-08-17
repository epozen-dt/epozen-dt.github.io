---
title: "Docker remote api - Java 클라이언트 라이브러리"
last_modified_at: 2022-08-17
author: 심건우
---

이번 포스팅에선 docker remote api를 java 클라이언트로 접근해보았다.

## Docker remote API
 클라이언트에서 원격으로 도커 서비스를 제어할 수 있는 API
 
 ![image](https://user-images.githubusercontent.com/87160438/185047895-fc5ebed9-d137-4ca5-acf8-758af863df1f.png)
 

[이미지 출처](https://wiki.kicco.com/space/SYS/312967547/Docker+Remote+API+%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)


## Java 클라이언트 라이브러리 비교
 아래 표 내 3개의 라이브러리는 Docker 공식 문서의 Unofficial libraries 목록을 참고하여 선정
 
 각 라이브러리 관련 정보는 각 라이브러리의 github와 mvnrepository를 참고
 
 각 라이브러리의 취약점은 dependency들의 버전 업데이트로 회피 가능
 
 ![image](https://user-images.githubusercontent.com/87160438/185052474-d0854ae7-22cd-4595-9fa7-340b5e164e56.png)


[Unofficial libraries](https://docs.docker.com/engine/api/sdk/#unofficial-libraries)

[A github 링크](https://github.com/spotify/docker-client)

[A mvnrepository 링크](https://mvnrepository.com/artifact/com.spotify/docker-client)

[B github 링크](https://github.com/docker-java/docker-java)

[B mvnrepository 링크](https://mvnrepository.com/artifact/com.github.docker-java/docker-java)

[C github 링크](https://github.com/amihaiemil/docker-java-api)

[C mvnrepository 링크](https://mvnrepository.com/artifact/com.amihaiemil.web/docker-java-api)


## Docker remote 환경설정
- 도커 환경설정을 위해 root로 switch user

```
su root
```

- docker.service 편집

```
nano /usr/lib/systemd/system/docker.service
```

- docker.service 내 ExecStart 속성 아래와 같이 수정 (2375 port는 보안 미적용 port)


```
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
```

- 변경 사항 적용

```
systemctl daemon-reload
```

- docker 재시작

```
service docker restart
```

- 방화벽 외부 접근 가능 port 추가

```
firewall-cmd --permanent --zone=public --add-port=2375/tcp
```

- 변경 사항 적용

```
firewall-cmd-reload
```

- 외부 cmd에서 curl 요청 (컨테이너 목록 중 1개 조회)

```
curl http://[docker host server ip address]:2375/containers/json?all=1
```

- 설정 적용 전 요청 결과

![image](https://user-images.githubusercontent.com/87160438/185054918-313e8555-be8a-47f9-9c7b-d21894af1250.png)


- 설정 적용 후 요청 결과

![image](https://user-images.githubusercontent.com/87160438/185055236-1636a1aa-6d6d-41c3-bc64-d33dc85bddb7.png)


## 라이브러리 사용 예시 (spotify/docker-client)

[예시 출처](https://github.com/spotify/docker-client/blob/master/docs/user_manual.md)

- 컨테이너 목록 조회

![image](https://user-images.githubusercontent.com/87160438/185055356-2ac70e29-0f16-48be-a0c9-e456d5c45a8d.png)


- 도커 이미지 pull

![image](https://user-images.githubusercontent.com/87160438/185055379-e80a09fe-657b-4ed7-a15d-31474fb87e90.png)


- 컨테이너 환경 설정

![image](https://user-images.githubusercontent.com/87160438/185055441-cc05afe9-6013-4dd5-8de2-17c40a7679cb.png)


- 컨테이너 생성 및 실행

![image](https://user-images.githubusercontent.com/87160438/185055500-3d3b2273-e77e-455f-994b-df651d9b3623.png)


![image](https://user-images.githubusercontent.com/87160438/185055520-70c87954-2f23-450a-8eec-0ea3380446ec.png)


- 컨테이너 중지 및 삭제

![image](https://user-images.githubusercontent.com/87160438/185055547-8d137b27-fd9b-498f-9e4d-436f16bf5378.png)


![image](https://user-images.githubusercontent.com/87160438/185055564-f3064a28-29b8-4aec-9d23-eceb1e4b7ff9.png)


![image](https://user-images.githubusercontent.com/87160438/185055588-60e6ab91-d9c4-4495-9db6-d95fa773c4ed.png)
