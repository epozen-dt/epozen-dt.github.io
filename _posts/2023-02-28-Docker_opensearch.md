---
title: "Docker로 Opensearch Cluster 구축하기"
last_modified_at: 2023-02-28
author: 이한솔
---
<br>

이번 포스팅에서는 Docker로 Opensearch Cluster 구축하는 방법에 대해 공유하겠습니다.

> **Opensearch**는 Elastic Search 7.1에서 Fork 된 것으로 분산형 커뮤니티 기반 오픈 소스 검색 및 분석 제품입니다. 실시간 애플리케이션 모니터링, 로그 분석 및 웹 사이트 검색과 같이 다양한 사용 사례에 사용되고 있습니다.

<br>

# Opensearch Cluster 구성
Cluster 구성도는 다음과 같습니다. <br>
1번 서버에는 Cluster Manager Node와 Data Node1을, 2번 서버에는 Data Node2를 구성하였습니다. <br>
Opensearch Dashboard 또한 1번 서버에 설치하겠습니다.

<img src="https://user-images.githubusercontent.com/96156882/221719523-a5773918-f9a2-44c0-b4ef-2ba70363de47.png" width="700">

<br>

# 설정 파일
Docker를 이용해 설치하기 위해서 두가지의 설정 파일을 작성해야 합니다.<br>
설정 파일은 Cluster Manager 기준으로 작성하겠습니다. <br>

### **docker-compose.yaml** <br>
도커 설치를 위한 도커 컴포즈 파일입니다.

```yaml
version: '3'
services:
  opensearch-cluster_manager:
    image: opensearchproject/opensearch:latest
    container_name: opensearch-cluster_manager
    environment:
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms4g -Xmx4g"
      - TZ=Asia/Seoul
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536 
        hard: 65536
    volumes:
      - opensearch-cluster_manager:/usr/share/opensearch
      - ./manager-opensearch.yml:/usr/share/opensearch/config/opensearch.yml
      - ./tls/root-ca.pem:/usr/share/opensearch/config/root-ca.pem
      - ./tls/admin.pem:/usr/share/opensearch/config/admin.pem
      - ./tls/admin-key.pem:/usr/share/opensearch/config/admin-key.pem
      - ./tls/manager.pem:/usr/share/opensearch/config/manager.pem
      - ./tls/manager-key.pem:/usr/share/opensearch/config/manager-key.pem
    ports:
      - 9200:9200 # REST API
      - 9301:9301 # Node communication
      - 9600:9600 # Performance Analyzer
volumes:
  opensearch-cluster_manager:

```
1. Environment
- bootstrap.memory_lock=true : 디스크 스왑 방지
- "OPENSEARCH_JAVA_OPTS=-Xms4g -Xmx4g" : Java Heap 메모리 설정. 최소 및 최대 JVM 힙 크기를 시스템 RAM의 50% 이상으로 설정

2. ulimits : 프로세스들에 대한 시스템 자원사용을 제한
- soft : 새로운 프로그램을 생성하면 기본적으로 적용되는 한도 
- hard : 소프트한도에서 최대로 늘릴 수 있는 한도 <br>
  → -1로 memlock을 무제한으로 설정(soft 또는 hard 제한 없음)
- nofile : 해당 도메인 (사용자, 그룹)이 오픈할 수 있는 최대 파일 개수

3. volumes
- opensearch-cluster_manager:/usr/share/opensearch : opensearch-cluster_manager 볼륨을 생성하여 컨테이너에 마운트
- ./manager-opensearch.yml:/usr/share/opensearch/config/opensearch.yml : 현재 디렉토리에 있는 manager-opensearch.yml를 컨테이너 내부의 opensearch.yml에 매핑 <br>
  → _**클러스터 설정 및 암호화 설정 할 때 중요한 파일 입니다**_

<br>

### **config.yml** <br>
opensearch의 config 파일입니다.

```yml
cluster.name: opensearch-cluster
node.name: opensearch-cluster_manager
node.roles: [ cluster_manager ]

network.host: 0.0.0.0
network.publish_host: cluster_manager IP

http.port: 9200
transport.port: 9301

discovery.seed_hosts: [cluster_manager IP, data_node1 IP, data_node2 IP]
cluster.initial_cluster_manager_nodes: ["opensearch-cluster_manager"]

```

- cluster.name : 클러스터 이름. Cluster manager, Data Node 1,2에도 동일한 클러스터 이름을 기재
- node name : 노드 이름
- node.roles : 해당 노드의 역할. 데이터 노드일 경우 [ data ]

- network.host : Default로 Local에서만 접근이 가능하도록 설정이 되어 있으므로 외부에서 접속 가능하도록 만들기 위해서는 network.host 값을 0.0.0.0으로 바꿈
- network.publish_host : 현재 서버의 IP 주소. 노드가 클러스터의 다른 노드에 연결할 수 있도록 제공
- http.port : HTTP 요청에 바인딩할 포트
- transport.port : 노드 간 통신을 위해 바인딩할 포트
- discovery.seed_hosts : node가 시작했을 때, Cluster에 구성되기 위해 찾아가는 Host 주소의 List로 IP주소를 입력
- cluster.initial_cluster_manager_nodes : 클러스터에서 manager 노드의 초기 집합을 설정

<br>

# 결과
클러스터 구성 결과입니다.
![image](https://user-images.githubusercontent.com/96156882/221735354-9ab46c80-d70d-4680-bc6a-917225f4941e.png)


이번 포스팅에서는 docker를 통해 opensearch cluster 구축하는 방법에 대해 알아보았습니다. 대부분 opensearch 공식 문서를 참고하였습니다. <br>
[Opensearch 공식문서] (https://opensearch.org/docs/latest/)