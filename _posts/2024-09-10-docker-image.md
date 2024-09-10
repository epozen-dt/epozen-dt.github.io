---
title: "docker image 생성 및 배포"
last_modified_at: 2024-09-10
author: 조평연
---

본 포스팅은 docker image 생성 및 배포 방법 을 알아보는 내용입니다.

# 1. Dockerfile 이란?
- 도커파일은 도커 이미지를 만들기 위해 작성하는 파일을 뜻합니다.
- docker build 명령어를 통해 도커파일에서 작성한 내용을 기반으로 이미지 생성 가능합니다.

# 2. 문법
- FROM : 어떤 base 이미지를 사용하는지 쓰는곳
- COPY : 호스트에서 이미지에 파일 추가
- ADD : 호스트에서 이미지에 파일 추가 (url, 압축파일 까지 커버)
- ENTRYPOINT : 빌드한 이미지를 컨테이너로 생성할때 단 한번 실행
- CMD : 빌드한 이미지 생성 및 시작할때 실행 (Docker run, start)
- WORKDIR : run 명령어가 실행되는 위치를 지정, 컨테이너의 위치 지정

# 3. 준비 파일
- ext 폴더
- lib 폴더
- test-1.0.jar 파일
- Dockerfile 파일
- docker-compose.yml

# 4. Dockerfile 내용
```bash
FROM openjdk:17
ADD test-1.0.jar /apps/backend/test-1.0.jar
ADD lib /apps/ backend /lib
ADD ext /apps/ backend /ext
WORKDIR /apps/ backend
CMD ["java","-jar","test-1.0.jar"]
```

# 5. docker-compose.yml 내용
```bash
services:
    test:
        image: test:1.0
        container_name: test
        ports:
            - 8080:8080
        environment:
            APPKEY_FILE: ext/app.key
            DB_ENCRYPT_PASSWD: 1234
            TZ: Asia/Seoul
        restart: always
```

# 6. 이미지 생성
- docker build -t [이미지이름]:[태그] [Dockerfile 경로]
- ex) docker build -t test:1.0 .

# 7. 도커 run 으로 실행 방법
- docker run -p 8080:8080 -d test-1.0

# 8. 도커 컴포즈로 실행 방법
- docker-compose up -d

# 9. 참고문헌
- https://docs.docker.com/reference/cli/docker/image/