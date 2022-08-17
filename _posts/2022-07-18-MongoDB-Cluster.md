---
title: "MongoDB 클러스터 구성 예제(docker-compose)"
last_modified_at: 2022-07-18
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
 shard 분산 정보 관리 
 
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
 config-server의 설정을 참조하여, 데이터를 각 shard 에 분산 및 결과 반환
 
#### 1.docker-compose.yml 작성

```yml
version: '3'

services:

  mongos:
    container_name: mongos
    image: mongo
     command: mongos --configdb config-rs/[config-server-1 ip]:[config-server-1 port],[config-server-2 ip]:[config-server-2 port],[config-server-3 ip]:[config-server-3 port] --bind_ip 0.0.0.0 --port 27017
    ports:
      - 60000:27017

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8111:8081
    environment:
      - ME_CONFIG_MONGODB_SERVER=mongos
      - ME_CONFIG_MONGODB_PORT=27017
    depends_on:
      - mongos

```

#### 2. 컨테이너 생성

```
docker-compose up -d
```

![image](https://user-images.githubusercontent.com/87160438/179462392-5034a94f-2552-41ab-8735-884301671a08.png)


```
docker ps
```

![image](https://user-images.githubusercontent.com/87160438/179462416-2209c7fd-d16c-4d59-b186-3d184ddf5e6c.png)


#### 3. 컨테이너 접속

```
docker exec -it [mongos 컨테이너 아이디] /bin/bash
```

![image](https://user-images.githubusercontent.com/87160438/179462504-0231f889-059c-464d-921b-aa2681a7c07b.png)


#### 4.shard 클러스터 등록

```
mongo mongodb://[mongos server ip]:[mongos server port]
```

![image](https://user-images.githubusercontent.com/87160438/179462612-e803956f-9b98-40a1-a83b-a47018390ce7.png)


```
sh.addShard("shard-1-rs/[shard-1-server-1 ip]:[shard-1-server-1 port],[shard-1-server-2 ip]:[shard-1-server-2 port],[shard-1-server-3 ip]:[shard-1-server-3 port]")
```

![image](https://user-images.githubusercontent.com/87160438/179462728-7a2c6fb4-113d-4bb8-9a89-d2efac00421f.png)


#### 3.확인

```
sh.status()
```

![image](https://user-images.githubusercontent.com/87160438/185006266-f5a3e9d7-9da6-4132-b00c-277d077893fe.png)


## 결과
#### 1. 데이터 삽입 확인
- python 클라이언트 데이터 삽입 

```python
# 라이브러리 import
from pymongo import MongoClient
from pprint import pprint

# client 연결
client = MongoClient(host='host ip address', 
                     port=60000)
# Database
epozen = client.epozen
# Collection
team_dt = epozen.team_dt
# Document ('_id' 지정 안 하면 알아서 고유 id 부여)
team_dt.insert_one({
#     '_id': 1,
    'name': 'member_1', # str
    'age': 26, # int
    'hobby': ['math'], # list
    'pet': {'dog': 'happy'}, # dict
    'score': 60.0 # float
})
# Document 두 개 이상 삽입
team_dt.insert_many([
    {
        'name': 'member_3',
        'age': 28,
        'hobby': ['baseball', 'music'],
        'pet': {'cat': 'mike'},
        'score': 75.5
    },
    {
        'name': 'member_4',
        'age': 29,
        'hobby': [],
        'pet': {'dog': 'alice'},
        'score': 90.0
    },
    {
        'name': 'member_5',
        'age': 30,
        'hobby': ['music'],
        'pet': {},
        'score': 100.0
    },
])

```

- mongos 상태 변화 확인

```
rs.status()
```

```
sh.status()
```

![image](https://user-images.githubusercontent.com/87160438/179464610-82661c75-8f93-4584-996a-cc05e9d77f85.png)


- mongos 보안 관련 로그 (TCP fast Open)

![image](https://user-images.githubusercontent.com/87160438/185006441-867c8e6e-e595-4e2d-9b70-cac94f7a24f5.png)


- mongo-express 화면 

![image](https://user-images.githubusercontent.com/87160438/185006598-ee2ad9cd-e987-43ea-8196-a08102e2a77f.png)


- 생성된 collection

![image](https://user-images.githubusercontent.com/87160438/179466282-ffec9c3f-bc6d-4b67-857d-a52d8beae306.png)


 * 보안 관련 항목(TLS, id, pw 등) 미설정 -> replica set, balancer 미동작
 * shard와 데이터베이스는 정상 등록

## 정리
- docker-compose를 활용해 MongoDB 클러스터 구성 예제를 진행해보았다.
- 컨테이너 내부에 진입해야 하는 과정이 반복되어 약간 번거로웠다.
- 실제 환경에 적용 시, 보안 관련 사항 관리에 어려움이 있을 것으로 예상된다.

 
