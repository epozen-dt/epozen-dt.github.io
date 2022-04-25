---
layout: post
title: "Zookeeper, Cluster 구성 및 Python Client 예제"
date: 2022-04-25
author: 심건우
---

이번 포스팅에선, Zookeeper 관련 정보를 간략히 설명하고, 간단한 Cluster 구성 및 Python Client 예제를 진행하겠습니다. 

관련 정보는 Zookeeper 공식 문서를 참고했음을 밝힙니다.

[Zookeeper 공식 문서](https://zookeeper.apache.org/doc/current/zookeeperOver.html)

## Zookeeper
 - 분산 어플리케이션을 위한 분산 오픈 소스 조정 서비스
 - 분산 어플리케이션 간 동기화, 설정 관리 등 기능 제공
 - 파일 시스템의 디렉터리 트리 구조와 유사한 데이터 모델 사용 

## 특징
 #### 1. 단순하다.
  - 표준 파일 시스템과 유사하게 구성된 공유 계층 네임스페이스를 통해 분산 프로세스 간 조정 가능
  - 네임스페이스는 Zookeeper 용어로 "znodes"라고 불리는 데이터 레지스터로 구성 (파일 및 디렉토리와 유사)
  - 파일 시스템과 다르게 Zookeeper 데이터는 메모리에 저장 -> 높은 처리량, 낮은 지연 시간
 #### 2. 복제 가능하다.
  - Zookeeper 자체는 앙상블이라 불리는 호스트 집합을 통해 복제됨
![image](https://user-images.githubusercontent.com/87160438/165010597-301cd827-6b62-4eb8-b4e3-d68c4899b2de.png)

[이미지 출처](https://zookeeper.apache.org/doc/current/zookeeperOver.html)

 #### 3. 순차적이다.
 #### 4. 속도가 빠르다.

## Data model 및 hierarchical namespace
![image](https://user-images.githubusercontent.com/87160438/165010787-48d1438d-574d-417a-8895-1f6946a2a5f7.png)

[이미지 출처](https://zookeeper.apache.org/doc/current/zookeeperOver.html)

## Nodes 와 Ephemeral nodes

## Conditional updates 및 watches

## Guarantees

## Simple API

## Cluster 구성

## Python Client 예제

