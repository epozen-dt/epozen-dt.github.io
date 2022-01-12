---
layout: post
title: "Apache Pulsar - 환경 구축 및 테스트"
date: 2022-01-12
author: 심건우
---

이번 포스팅에서는 Apahce 사의 Pulsar로 환경 구축 및 테스트를 진행하겠습니다.

## Apache Pulsar란?
 Apache Pulsar란, 서버 간 메시징을 위한 멀티-테넌트 고성능 솔루션이라 할 수 있습니다.
 
 처음 Yahoo에서 개발했으며, 현재는 Apache Software Foundation이 관리하고 있습니다.
 
 주요기능은 다음과 같습니다.
 
  - 멀티 클러스터 지원
  - 클러스터 전체에서 메시지를 완벽하게 지리적으로 복제(geo-replication)
  - 매우 낮은 publish 및 end-to-end latency
  - 백만 개가 넘는 topic으로 확장 가능
  - 다양한 client API(Java, Go, Python, C++ 등) 지원
  - Apache BookKeeper에서 제공하는 영구 메시지 저장소를 통한 메시지 전달 보장
  - Server-less 경량 컴퓨팅 프레임 워크인 Pulsar Functions를 통해 stream-natvie 데이터 처리 기능 제공

## 구조
 구조는 아래 이미지와 같습니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/149073723-4e345c73-ac48-4fa0-b769-0e614db64bf9.png)

 이미지 출처 : https://pulsar.apache.org/docs/ko/concepts-architecture-overview/
 
 브로커는 Producer로부터 들어오는 메시지들을 처리하고 Load-Balancing하여, Consumer에게 전송합니다.
 
 다양한 조정 작업들을 처리하기 위해 구성된 저장소와 상호 작용하고, BookKeeper 인스턴스(Bookies)에 메시지를 저장합니다.
 
 
## 설치 및 실행
 이제 Pulsar 설치 및 실행을 진행하겠습니다.
 
 본 포스팅에선, docker-compose를 활용했습니다.
 
 또, 별도의 클러스터를 구성하지 않고 Pulsar에서 제공하는 Standalone 모드를 적용했습니다.
 
 1. 도커 이미지 pull
  ```
  docker pull apachepulsar/pulsar:latest
  ```
 2. docker-compose.yaml 작성
  ```yaml
  version: "3"
   services:
     pulsar:
       image: apachepulsar/pulsar:latest
       command: bin/pulsar standalone
       environment:
         PULSAR_MEM: " -Xms512m -Xmx512m -XX:MaxDirectMemorySize=1g"
       ports:
         - 6650:6650
         - 8080:8080
       restart: unless-stopped
  ```
 3. 실행
  ```
  docker-compose up -d
  ```
## Producer & Consumer (Spring Boot)
 
 펄사 어플 수정 (JSON스키마 적용, 7번오라클 접속X)

