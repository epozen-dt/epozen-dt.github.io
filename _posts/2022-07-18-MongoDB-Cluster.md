---
layout: post
title: "MongoDB 클러스터 구성 예제(docker-compose)"
date: 2022-07-18
author: 심건우
---

지난 포스팅에서, docker-compose를 활용한 MongoDB 환경 설정 및 Python Client 예제를 진행했다.

[MongoDB 설치(docker-compose) 및 Python Client 예제](https://epozen-dt.github.io/2022/06/27/MongoDB.html)

이번 포스팅에선, docker-compose를 활용해서 MongoDB 클러스터 구축 예제를 진행해보겠다.

* 참고 링크

[MongoDB 구조 개념](https://velog.io/@minchoi/MongoDB-%EA%B5%AC%EC%A1%B0-%EA%B0%9C%EB%85%90-%EC%83%A4%EB%94%A9-%EC%83%A4%EB%93%9C-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0-%EB%A0%88%ED%94%8C%EB%A6%AC%EC%B9%B4%EC%85%8B)

[docker-compose 이용해서 MongoDB Cluster 구조 구성하기](https://velog.io/@minchoi/docker-compose-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-MongoDB-Cluster-%EA%B5%AC%EC%A1%B0-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0)

[docker container로 mongo cluster 구성하기](https://boying-blog.tistory.com/35)

[Set up Sharding in MongoDB using Docker containers](https://www.youtube.com/watch?v=7Lp6R4CmuKE&list=LL&index=5&t=1150s&ab_channel=JustmeandOpensource)

## 클러스터 구조

![image](https://user-images.githubusercontent.com/87160438/179451795-23de0e2d-ad80-4cfc-8615-37ff7e13eedd.png)


## config-server
 샤드 분산 정보 관리 
 
#### 1. docker-compose.yml 작성

```yml
version: '3'

services:

  config-server-1:
    container_name: config-server-1
    image: mongo
    command: mongod --configsvr --replSet config-rs --port 27017 --dbpath /data/db
    ports:
      - 40001:27017
    volumes:
      - ./config-server-1:/data/db

  config-server-2:
    container_name: config-server-2
    image: mongo
    command: mongod --configsvr --replSet config-rs --port 27017 --dbpath /data/db
    ports:
      - 40002:27017
    volumes:
      - ./config-server-2:/data/db

  config-server-3:
    container_name: config-server-3
    image: mongo
    command: mongod --configsvr --replSet config-rs --port 27017 --dbpath /data/db
    ports:
      - 40003:27017
    volumes:
      - ./config-server-3:/data/db

volumes:
  config-server-1: {}
  config-server-2: {}
  config-server-3: {}
```

#### 2. 컨테이너 생성

```
docker-compose up -d
```

![image](https://user-images.githubusercontent.com/87160438/179452817-40cd800d-6667-448c-8a87-8e4f10ebdf02.png)


```
docker ps
```

![image](https://user-images.githubusercontent.com/87160438/179452862-57f4e44e-28eb-492c-826a-9b4d49fded60.png)


#### 3. 컨테이너 접속

```
docker exec -it [컨테이너 아이디] /bin/bash
```

![image](https://user-images.githubusercontent.com/87160438/179452924-3f9b30df-733c-40f9-bf9b-466d271ab24b.png)


#### 4. replicat set 구성

```
mongo mongodb://[config-server-1 ip]:[config-server-1 port]
```

![image](https://user-images.githubusercontent.com/87160438/179453160-8cc1ae14-2aad-4e18-ad8f-f0f9bd50afcf.png)


```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "[config-server-1 ip]:[config-server-1 port]" },
      { _id : 1, host : "[config-server-2 ip]:[config-server-2 port]" },
      { _id : 2, host : "[config-server-3 ip]:[config-server-3 port]" }
    ]
  }
)
```



#### 5. 확인

## shard-server
 데이터의 실제 저장소
 
#### 1. docker-compose.yml 작성

```yml
version: '3'

services:

  shard-1-server-1:
    container_name: shard-1-server-1
    image: mongo
    command: mongod --shardsvr --replSet shard-1-rs --port 27017 --dbpath /data/db
    ports:
      - 50001:27017
    volumes:
      - ./shard-1-server-1:/data/db

  shard-1-server-2:
    container_name: shard-1-server-2
    image: mongo
    command: mongod --shardsvr --replSet shard-1-rs --port 27017 --dbpath /data/db
    ports:
      - 50002:27017
    volumes:
      - ./shard-1-server-2:/data/db

  shard-1-server-3:
    container_name: shard-1-server-3
    image: mongo
    command: mongod --shardsvr --replSet shard-1-rs --port 27017 --dbpath /data/db
    ports:
      - 50003:27017
    volumes:
      - ./shard-1-server-3:/data/db

volumes:
  shard-1-server-1: {}
  shard-1-server-2: {}
  shard-1-server-3: {}
```

#### 2. 컨테이너 생성

```
docker-compose up -d
```

#### 3. 컨테이너 접속

```
docker exec -it [컨테이너 아이디] /bin/bash
```

#### 4. replicat set 구성
#### 5. 확인

## mongos router
 config-server의 설정을 참조하여, 데이터를 각 샤드에 분산 및 결과 반환
 
#### 1.docker-compose.yml 작성
#### 2.shard 클러스터 등록
#### 3.확인

## 결과
#### 1. 데이터 삽입 확인
#### 2. 보안 관련 추가 필요(TLS)

 
