---
title: "PgBouncer 설치 및 사용 방법"
last_modified_at: 2024-12-18
author: 이현지
---

<br>


# PgBouncer
PostgreSQL 데이터베이스의 연결을 효율적으로 관리하는 연결 풀러로서 클라이언트와 데이터베이스 사이에 위치하는 미들웨어입니다.

 

## 주요 기능
 

### 연결풀링

데이터베이스와의 연결을 관리하여 클라이언트의 요청에 대해 미리 생성된 연결을 재사용합니다.<br>
이를 통해 데이터베이스에 대한 연결 수를 줄이고, 연결 생성 및 종료에 드는 오버헤드를 최소화합니다. 


### 다양한 풀링 모드

#### - Session pooling :<br>
클라이언트 세션이 pgBouncer에 연결될 때마다 새로운 데이터베이스 연결을 생성하며 세션이 종료될 때까지 연결이 유지됩니다. 
#### - Transaction pooling :<br>
각 트랜잭션이 시작될 때 데이터베이스 연결을 할당하고, 트랜잭션이 완료되면 연결을 반환합니다.<br>가장 효율적인 모드이며, 많은 클라이언트가 동시에 연결할 수 있도록 합니다.
#### - Statement pooling :<br>
SQL문 단위로 연결을 생성하고 종료합니다. 특정 SQL문이 세션 상태에 의존하지 않을 때만 사용할 수 있습니다.


### 연결 관리
연결의 수를 제한하고, 비정상적으로 오랜 시간 동안 사용되지 않는 연결을 종료하는 등의 기능을 제공합니다.

### 로드 밸런싱
여러 데이터베이스 인스턴스에 대한 로드 밸런싱을 지원합니다. <br>
이를 통해 클라이언트 요청을 여러 데이터베이스 서버에 분산시켜 성능을 향상시킬 수 있습니다.

### 세션 및 트랜잭션 통계
연결, 세션, 트랜잭션에 대한 통계를 수집하고, 이를 통해 데이터베이스의 성능을 모니터링할 수 있습니다.<br>
이러한 통계는 관리자가 시스템의 상태를 이해하고, 필요에 따라 조치를 취하는 데 유용합니다.

### 간단한 설정 및 관리
설정 파일을 통해 쉽게 구성할 수 있으며, 명령줄 인터페이스를 통해 관리할 수 있습니다.<br>
또한, 다양한 인증 방법을 지원하여 보안성을 높일 수 있습니다.

### Failover 및 장애 조치
데이터베이스 서버의 장애 발생 시 자동으로 다른 서버로 연결을 전환할 수 있는 기능을 제공합니다.<br>
이를 통해 시스템의 가용성을 높일 수 있습니다.

### SSL 지원
SSL을 통한 안전한 연결을 지원하여, 데이터 전송 중 보안을 강화할 수 있습니다.


## 설치 방법
### docker-compose.yaml
<br>

```yaml
version: '3.8'
 services:
   pgbouncer:
    image: bitnami/pgbouncer:latest
    container_name: pgbouncer
    environment:
      - PGBOUNCER_DATABASE=DB명
      - POSTGRESQL_HOST=DB컨테이너명
      - POSTGRESQL_USERNAME=사용자명
      - POSTGRESQL_PASSWORD=사용자비밀번호
      - PGBOUNCER_POOL_MODE=session
      - PGBOUNCER_DEFAULT_POOL_SIZE=100
      - TZ=Asia/Seoul
    ports:
      - "6432:6432"
    restart: always
    networks:
      - 네트워크명
 networks:
  네트워크명:
    external: true
```

PgBouncer는 기본적으로 6432 포트를 사용합니다.<br>
<b>네트워크명</b> : postgresql DB가 사용중인 네트워크 입력<br>
<b>PGBOUNCER_POOL_MODE</b> : 풀링모드 설정 - session (default), transaction, statement<br>
<b>PGBOUNCER_DEFAULT_POOL_SIZE</b> : 최대 풀 개수 설정<br><br>

(이외 다른 설정 환경 변수는 아래 URL 참고) <br>
https://hub.docker.com/r/bitnami/pgbouncer <br>

![환경변수예시](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbn5CfU%2FbtsKTYxhswy%2F0aG45KWiIpbkIcqKJbJxck%2Fimg.png)


### application.properties

스프링 애플리케이션과 PgBouncer 연결 시 application.properties 파일에 아래 설정값들을 추가해줍니다.<br>
PostgreSQL DB와 연결 시 작성하는 DB 정보에서 url의 포트 부분만 PgBoucer의 포트(일반적으로 6432)로 바꿔주면 됩니다.<br>

```
spring.datasource.url = ${DB_URL:jdbc:postgresql://호스트:PgBouncer포트/DB명}
spring.datasource.driverClassName = org.postgresql.Driver
spring.datasource.username = 사용자명
```

### 연결 확인
다음 명령어를 통해 PgBouncer에 접속할 수 있습니다.<br>
```
psql -h 호스트 -p 포트 -U DB사용자명 pgbouncer
```
ex) psql -h localhost -p 6432 -U user pgbouncer<br>
로컬호스트에 떠 있는 PgBouncer에 user라는 사용자로 pgbouncer라는 DB에 접속<br>

### DB 정보 확인
PgBouncer에 접속한 후 다음 명령어로 PgBouncer에 연결된 DB 정보를 확인할 수 있습니다.<br>

```
SHOW DATABASES;
```
<br>

이상으로 PgBouncer 설치 방법에 대해 알아보았습니다. 

<br>
