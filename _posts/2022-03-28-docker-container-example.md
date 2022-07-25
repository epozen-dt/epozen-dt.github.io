---
title: "도커 컨테이너로 서비스 실행하기"
last_modified_at: 2022-03-28
author: 심건우
---

이번 포스팅에선, 도커 관련 정보를 간략히 설명하고, 도커 컨테이너로 서비스를 실행하는 방법을 설명하겠습니다.

관련 정보는 도커 공식 문서를 참고했음을 밝힙니다.

[도커 공식 문서](https://docs.docker.com/get-started/overview/)

## 도커 (Docker)  
 - 도커는 응용 프로그램 개발, 전송 및 실행을 위한 개방형 플랫폼입니다.
 - 도커를 사용하면, 응용 프로그램과 환경을 분리하여 신속하고 일관성 있게 서비스를 제공할 수 있습니다.

## 도커 구조 (Docker architecture)
 - 도커는 클라이언트-서버 구조를 사용함니다.
 - 도커 클라이언트는 도커 데몬과 통신합니다.
 - 도커 데몬은 도커 컨테이너의 빌드, 실행 및 배포 작업을 수행합니다.
 - 도커 클라이언트와 데몬은 동일한 시스템에서 실행하거나, 도커 클라이언트를 원격 도커 데몬에 연결할 수 있습니다.
 - 도커 클라이언트와 데몬은 REST API, UNIX 소켓, 네트워크 인터페이스로 통신합니다.

![image](https://user-images.githubusercontent.com/87160438/160334143-0fcc4f26-7a25-429a-848b-0446c9ffb28a.png)

[이미지 출처](https://docs.docker.com/get-started/overview/)
 
## 도커 데몬 (Docker daemon)
 - 도커 데몬은 Docker API 요청을 수신하고 도커 이미지, 도커 컨테이너, 네트워크 및 볼륨과 같은 Docker 개체를 관리합니다.
 - 데몬은 다른 데몬과 통신하여 도커 서비스를 관리할 수도 있습니다.

## 도커 클라이언트 (Docker client)
 - 도커 클라이언트는 도커 사용자가 도커와 상호작용하는 주된 도구입니다.
 - 'docker run' 같은 명령어를 사용하면 클라이언트는 명령어를 도커로 전송하고, 도커는 명령어를 실행합니다. (Docker API 사용)
 - 도커 클라이언트는 다수의 데몬과 통신할 수 있습니다.

## 도커 레지스트리 (Docker registries)
 - 도커 레지스트리에는 도커 이미지들이 저장됩니다.
 - 도커 허브(Docker hub)는 누구나 사용 가능한 공용 레지스트리이며, 도커는 기본적으로 도커 허브에서 이미지를 검색합니다.
 - 필요에 따라 독자적인 레지스트리를 생성할 수 있습니다.
 - 'docker pull' 명령 또는 'docker run' 명령을 사용하면, 설정된 도커 레지스트리에서 관련된 도커 이미지를 가져옵니다(pull).
 - 'docker push' 명령을 사용하면, 설정된 도커 레지스트리에 관련 도커 이미지를 전송합니다(push).

[도커 허브 주소](https://hub.docker.com/)
 
## 도커 이미지 (Docker image)
 - 도커 이미지는 도커 컨테이너를 작성하기 위한 지침이 포함된 읽기 전용 템플릿입니다.
 - 대부분의 경우, 이미지는 다른 이미지를 기반으로 하며 일부 추가 수정이 가능합니다.
 - 독자적인 이미지를 작성할 수도 있고, 다른 사람들이 만들어 도커 레지스트리에 게시한 이미지만 사용할 수도 있습니다.
 - 자신의 이미지를 빌드하려면, 이미지를 만들고 실행하는데 필요한 단계를 간단한 구문을 사용하여 정의한 도커 파일(Dockerfile)을 만듭니다.
 - 도커 파일의 각 명령은 이미지에 레이어를 생성합니다.
 - 도커 파일을 변경하고 이미지를 재구축하면 변경된 레이어만 재구축됩니다.
 
## 도커 컨테이너 (Docker Container)
 - 도커는 컨테이너라고 하는 격리된 환경에서 응용 프로그램을 패키징하고 실행할 수 있는 기능을 제공합니다.
 - 분리 및 보안을 통해 특정 호스트에서 여러 컨테이너를 동시에 실행할 수 있습니다.
 - 컨테이너는 응용 프로그램 실행에 필요한 것을 포함하므로, 호스트의 환경에 의존적이지 않습니다.
 - 도커 컨테이너는 도커 이미지의 실행 가능한 인스턴스입니다.
 - Docker API 또는 CLI를 사용하여 컨테이너 생성, 시작 중지, 이동 또는 삭제가 가능합니다.
 - 도커 컨테이너는 하나 이상의 네트워크에 연결 가능합니다.
 - 도커 컨테이너는 스토리지에 연결 가능합니다.
 - 도커 컨테이너는 현재 상태를 기반으로 새 이미지를 생성할 수 있습니다.
 - 도커 컨테이너를 제거하면 영구 스토리지에 저장되지 않은 상태의 변경 사항은 사라집니다.

## 도커 컴포즈 (Docker-compose)
 - 도커 컴포즈는 하나 이상의 도커 컨테이너를 포함하는 도커 응용 프로그램을 정의하고 실행하기 위한 도구입니다.
 - 도커 컴포즈에서는 YAML 파일을 사용하여 응용 프로그램의 서비스를 구성합니다.
 - YAML 파일 작성 후, 하나의 명령어로 YAML 파일에 정의한 서비스를 구성하고 시작할 수 있습니다.
 
## 예제 1. (도커 허브 활용, 도커 이미지 커스터마이징)
 도커 허브에서 도커 이미지를 가져오고, 일부 수정하여 컨테이너로 서비스해보겠습니다.
 
#### 1. 도커 허브에서 이미지 찾기 (도커 이미지 : PostgreSQL)
 - 도커 허브 이미지 검색
 
    ![image](https://user-images.githubusercontent.com/87160438/160339945-0853e3a3-3936-4ccb-99a3-7a1cca82bb04.png)

 - 관련 정보 확인 및 커맨드 복사
 
    ![image](https://user-images.githubusercontent.com/87160438/160340181-6f07aac6-c79a-4a1e-8299-6f97b95e7f7b.png)

#### 2. 도커 이미지 pull
 - 복사한 커맨드 입력 (필요에 따라 태그 추가)

```
 docker pull postgres:bullseye
```

 - 확인

```
docker images postgres:bullseye
```

![image](https://user-images.githubusercontent.com/87160438/160341546-c81ec4ce-9488-4d5f-8af0-a1a909843d8b.png)
 
 
#### 3. 도커 이미지 일부 수정
 - Dockerfile 작성 (필요에 따라 기능 수정, 관련 문법은 공식 문서 참조)

```
FROM postgres:bullseye
RUN apt-get update && apt-get install -y --no-install-recommends postgresql-14-wal2json
```

 - 수정 이미지 빌드

```
docker build . -t pg-test:1.0
```

![image](https://user-images.githubusercontent.com/87160438/160343303-04a2c17c-dbb9-4f48-a919-f210ecb23b62.png)


#### 4. 도커 컨테이너 생성
 - docker-compose.yaml 작성 (관련 문법은 공식 문서 참조)

```yaml
version: "3"

services:
  postgres:
    image: pg-test:1.0
    container_name: pg-test
    restart: always
    ports:
      - 5433:5432
    environment:
      - POSTGRES_USER=test
      - POSTGRES_PASSWORD=test
      - POSTGRES_DB=test
    volumes:
      - ./postgres_data/:/var/lib/postgresql/data
    command:
      - "postgres"
      - "-c"
      - "wal_level=logical"
      - "-c"
      - "shared_preload_libraries=wal2json"
```

 - 컨테이너 생성 커맨드 입력 (백그라운드 실행)

```
docker-compose up -d
```

![image](https://user-images.githubusercontent.com/87160438/160344009-e1920ca7-1c1b-4ce3-9606-601ebe09676b.png)


 - 컨테이너 상태 확인

```
docker ps -l
```

![image](https://user-images.githubusercontent.com/87160438/160344280-d39c465e-c99a-467c-b30a-aee8bbb370cb.png)


 - 컨테이너 로그 확인

```
docker logs -f (컨테이너 ID)
```

![image](https://user-images.githubusercontent.com/87160438/160344516-88d83efa-a252-46fc-afe9-6ef0c19a19cb.png)

    
#### 5. 서비스 접속 (PostgreSQL)
 - docker-compose.yaml 에 작성한 설정값으로 PostgreSQL 접속 (DBeaver Test connection)

![image](https://user-images.githubusercontent.com/87160438/160345020-18ae9517-d716-4c10-a760-64c68fd88c99.png)


![image](https://user-images.githubusercontent.com/87160438/160345118-fb18e2d8-02aa-4c6f-9b8b-643d55f26403.png)


#### 6. 컨테이너 삭제

```
docker-compose down
```

![image](https://user-images.githubusercontent.com/87160438/160345975-10423058-9b21-481f-bb6e-87c24fad61cf.png)


## 예제 2. (도커 이미지 직접 생성)
 응용 프로그램을 서비스할 수 있는 이미지를 직접 생성하고, 빌드하여 컨테이너로 서비스해보겠습니다.
 
 모든 과정은 예제 1과 동일하며, 아래 두 파일의 내용과 일부 커맨드 내 ID와 태그만 중복이 일어나지 않게 구분하면 됩니다.
 
 응용 프로그램은 Spring Boot 작업물을 Maven을 통해 jar 파일로 패키징한 것을 사용했습니다.
 
 - Dockerfile

```
FROM openjdk:8

RUN apt-get update
RUN mkdir /alert-proto
ADD ./alert-0.0.1-SNAPSHOT.jar /alert-proto

ENTRYPOINT ["java", "-jar", "/alert-proto/alert-0.0.1-SNAPSHOT.jar"]
```

 - docker-compose.yaml

```yaml
version: "3"

services:
  alert-proto:
    image: alert-proto:1.0
    container_name: alert-proto
    restart: always
    ports:
      - 8082:8080
```

## 정리
 간단하게 도커 컨테이너로 서비스를 실행하는 법을 정리해봤습니다.
 
 호스트 환경에 의존적이지 않고, 일관성 있게 서비스를 제공할 수 있다는게 가장 두드러진 장점인 것 같습니다.
 
 이상입니다.