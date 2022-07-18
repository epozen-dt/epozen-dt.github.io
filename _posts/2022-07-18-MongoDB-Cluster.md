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


파워포인트 클러스터 구조 이미지 첨부

## config-server
 샤드 분산 정보 관리
 
### 1.docker-compose.yml 작성
- docker-compose.yml 작성
```yml
version: '3'

services:

  config-server-1:
    container_name: config-server-1
    image: mongo
    command: mongod --configsvr --replSet cfgrs --port 27017 --dbpath /data/db
    ports:
      - 40001:27017
    volumes:
      - config-server-1:/data/db

  config-server-2:
    container_name: config-server-2
    image: mongo
    command: mongod --configsvr --replSet cfgrs --port 27017 --dbpath /data/db
    ports:
      - 40002:27017
    volumes:
      - config-server-2:/data/db

  config-server-3:
    container_name: config-server-3
    image: mongo
    command: mongod --configsvr --replSet cfgrs --port 27017 --dbpath /data/db
    ports:
      - 40003:27017
    volumes:
      - config-server-3:/data/db

volumes:
  config-server-1: {}
  config-server-2: {}
  config-server-3: {}
```


### 2.레플리카셋 구성
### 3.확인
## shard-server
 데이터의 실제 저장소
### 1.docker-compose.yml 작성
### 2.클러스터 구성
### 3.확인
## mongos router
 config-server의 설정을 참조하여, 데이터를 각 샤드에 분산 및 결과 반환
### 1.docker-compose.yml 작성
### 2.shard 클러스터 등록
### 3.확인
## 데이터 분산 확인

 