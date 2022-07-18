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
    _id: "config-rs",
    configsvr: true,
    members: [
      { _id : 0, host : "[config-server-1 ip]:[config-server-1 port]" },
      { _id : 1, host : "[config-server-2 ip]:[config-server-2 port]" },
      { _id : 2, host : "[config-server-3 ip]:[config-server-3 port]" }
    ]
  }
)
```

![image](https://user-images.githubusercontent.com/87160438/179454088-97fc62ed-deb4-44fa-b101-c2dcfbcc0bba.png)


#### 5. 확인

```
rs.status()
```

![image](https://user-images.githubusercontent.com/87160438/179454195-e7ea89b9-84c8-4e79-914e-938b9582cfd0.png)


![image](https://user-images.githubusercontent.com/87160438/179454292-f96e482d-3624-446f-835e-8d38ffa26360.png)


![image](https://user-images.githubusercontent.com/87160438/179454376-bef45bbf-aaaa-4883-9220-911c29fce2c9.png)


![image](https://user-images.githubusercontent.com/87160438/179454477-54b23de5-c1cb-45db-93ba-19afea3beaba.png)


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

![image](https://user-images.githubusercontent.com/87160438/179455097-72dd76b5-4660-4132-ad3d-aee97e706b27.png)


```
docker ps
```

![image](https://user-images.githubusercontent.com/87160438/179455122-f7f96032-3b72-47ba-9155-2cd785ce87ea.png)


#### 3. 컨테이너 접속

```
docker exec -it [컨테이너 아이디] /bin/bash
```

![image](https://user-images.githubusercontent.com/87160438/179455197-6eabe847-3723-4396-b83c-5749096f92f0.png)


#### 4. replicat set 구성

```
mongo mongodb://[shard-1-sever-1 ip]:[shard-1-server-1 port]
```

![image](https://user-images.githubusercontent.com/87160438/179455670-0944bc9f-238d-4c3f-9c14-abdba4109482.png)


```
rs.initiate(
  {
    _id: "shard-1-rs",
    members: [
      { _id : 0, host : "[shard-1-server-1 ip]:[shard-1-server-1 port]" },
      { _id : 1, host : "[shard-1-server-2 ip]:[shard-1-server-2 port]" },
      { _id : 2, host : "[shard-1-server-3 ip]:[shard-1-server-3 port]" }
    ]
  }
)
```

![image](https://user-images.githubusercontent.com/87160438/179456330-2c3d8f49-ee3c-4761-be8e-9fc4e6eaf138.png)


#### 5. 확인

```
rs.status()
```

![image](https://user-images.githubusercontent.com/87160438/179456410-2a5a749c-826d-41e1-a983-850fc0eab9c9.png)


![image](https://user-images.githubusercontent.com/87160438/179456514-4f6c4702-6073-43a2-81fe-e638171b7792.png)


![image](https://user-images.githubusercontent.com/87160438/179456625-605dd2fa-ba2a-4b52-9c7b-cffaa6e32f24.png)


![image](https://user-images.githubusercontent.com/87160438/179456731-72eb7246-2394-446a-8dd6-6c4f83e5671f.png)


## mongos router
 config-server의 설정을 참조하여, 데이터를 각 샤드에 분산 및 결과 반환
 
#### 1.docker-compose.yml 작성

```yml
version: '3'

services:

  mongos:
    container_name: mongos
    image: mongo
    command: mongos --configdb cfgrs/[config-server-1 ip]:[config-server-1 port],[config-server-2 ip]:[config-server-2 port],[config-server-3 ip]:[config-server-3 port] --bind_ip 0.0.0.0 --port 27017
    ports:
      - 60000:27017
```

#### 2.shard 클러스터 등록
#### 3.확인

## 결과
#### 1. 데이터 삽입 확인
#### 2. 보안 관련 추가 필요(TLS)

 
