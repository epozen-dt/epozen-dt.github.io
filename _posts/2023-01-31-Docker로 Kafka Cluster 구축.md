---
title: "Docker Kafka Cluster 구축"
last_modified_at: 2023-01-31
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **Docker로 Kafka Cluster 구축하는 방법**에 대해 공유해보려고 합니다.


## **Docker로 Kafka Cluster 구축**

> **Getting started**

카프카에 대한 기본 내용은 이전 포스팅 [[Kafka 서버 Docker 설치 및 Java Producer/Consumer 구현]](https://epozen-dt.github.io/kafka-example/)을 참고하세요.  

> **Kafka Cluster 구성**   

아래 그림과 같이, 주키퍼와 브로커 모두 서버 3대의 앙상블(클러스터)로 구성하였습니다.

![Screenshot_2](https://user-images.githubusercontent.com/92897860/215413124-782ba003-5cf8-4da7-b68b-59ff7fd72642.png)


> **docker-compose.yml 설정 파일 작성**   

아래의 docker-compose.yml 파일은 xxx.xxx.xx.1 서버에 대한 설정 파일입니다. 
나머지 서버에서도 아래와 같이 작성 및 실행되어야 합니다.

* 작성 시에 고유 식별 ID(브로커, 주키퍼), hostname, container_name 등은 변경하셔야 합니다.
* extra_hosts 를 통해 컨테이너 내부의 /etc/hosts 파일에 등록 (datanode1, 2, 3)
* Kafka UI로는 Kafdrop을 사용 (다른 Kafka-ui, CMAK 등 사용 무방)

```
version: '3.8'

services:
  zookeeper1:
    image: confluentinc/cp-zookeeper:5.5.3
    restart: unless-stopped
    hostname: zookeeper1
    container_name: zookeeper1
    extra_hosts:
      - "datanode1:xxx.xxx.xx.1"
      - "datanode2:xxx.xxx.xx.2"
      - "datanode3:xxx.xxx.xx.3"
    environment:
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
      ZOOKEEPER_SERVERS: zookeeper1:2888:3888;datanode2:2888:3888;datanode3:2888:3888
      ZOOKEEPER_MAX_CLIENT_CNXNS: 60
      ZOOKEEPER_DATA_DIR: /var/lib/zookeeper 
      LOG_DIR: /var/log/zookeeper
    volumes:
      - /usr/share/zoneinfo/Asia/Seoul:/etc/localtime:ro
      - ./zookeeper-etc:/etc/kafka
      - ./zookeeper-data/data:/var/lib/zookeeper/data
      - ./zookeeper-data/log:/var/lib/zookeeper/log
      - ./zookeeper-varlog:/var/log/zookeeper
    #network_mode: host
    networks:
      - kafka-cluster-net
    ports:
      - "2181:2181"
      - "2888:2888"
      - "3888:3888"

  kafka1:
    image: confluentinc/cp-kafka:5.5.3
    restart: unless-stopped
    hostname: kafka1
    container_name: kafka1
    extra_hosts:
      - "datanode1:xxx.xxx.xx.1"
      - "datanode2:xxx.xxx.xx.2"
      - "datanode3:xxx.xxx.xx.3"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper1:2181,datanode2:2181,datanode3:2181"
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://xxx.xxx.xx.1:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_LOG4J_LOGGERS: "kafka.authorizer.logger=INFO"
      KAFKA_LOG4J_ROOT_LOGLEVEL: INFO
      KAFKA_DATA_DIR: /var/lib/kafka
      KAFKA_LOG_RETENTION_HOURS: 168
      LOG_DIR: /var/log/kafka
    volumes:
      - /usr/share/zoneinfo/Asia/Seoul:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - ./kafka-etc:/etc/kafka
      - ./kafka-data:/var/lib/kafka/data
      - ./kafka-varlog:/var/log/kafka
    depends_on:
      - zookeeper1
    #network_mode: host
    networks:
      - kafka-cluster-net
    ports:
      - "9092:9092"

  kafdrop:
    image: obsidiandynamics/kafdrop:latest
    hostname: kafdrop
    container_name: kafdrop
    extra_hosts:
      - "datanode1:xxx.xxx.xx.1"
      - "datanode2:xxx.xxx.xx.2"
      - "datanode3:xxx.xxx.xx.3"
    environment:
      - KAFKA_BROKERCONNECT=datanode1:9092,datanode2:9092,datanode3:9092
    depends_on: 
      - kafka1
    networks:
      - kafka-cluster-net
    ports:
      - "9000:9000"

networks:
  kafka-cluster-net:
    name: kafka-cluster-net
    driver: bridge
```  

**# 주키퍼 Environment**

* **ZOOKEEPER_SERVER_ID:** 주키퍼의 식별 ID
* **ZOOKEEPER_CLIENT_PORT:** 클라이언트가 주키퍼 접속을 위한 포트 (2888은 동기화를 위한 포트, 3888은 클러스터 구성 시 리더 선출을 위한 포트)
* **ZOOKEEPER_SERVERS:** 현재 내 서버는 내부IP 로 작성하고, 나머지 서버는 외부 IP로 작성
* **ZOOKEEPER_TICK_TIME:** 주키퍼는 클러스터를 구성할 때, 동기화를 위한 기본 틱 타임을 지정함(ms)
* **ZOOKEEPER_INIT_LIMIT:** 주키퍼 클러스터는 쿼럼이라는 과정을 통해 마스터를 선출하며, 이때 주키퍼들이 리더에게 커넥션을 맺을 때 지정할 초기 타임아웃 시간을 의미 (tick time * init_limit)
* **ZOOKEEPER_SYNC_LIMIT:** 주키퍼 리더와 나머지 서버들의 싱크 타임, 해당 시간 내에 싱크 응답이 오면 클러스터가 정상으로 구성되어 있음을 확인하는 시간
* **ZOOKEEPER_MAX_CLIENT_CNXNS:** 하나의 클라이언트에서 동시접속하는 갯수 제한 (0은 무제한)

**# 카프카 Environment**

* **KAFKA_BROKER_ID:** 클러스터 각 노드에 할당할 고유 브로커의 아이디 (중복 불가)
* **KAFKA_LISTENERS:** 리스너 리스트, 수신할 URI와 리스너 이름의 리스트
* **KAFKA_ZOOKEEPER_CONNECT:** 클러스터 구성에 사용할 주키퍼 호스트 정보(커넥션 대상)
* **KAFKA_ADVERTISED_LISTENERS:** 카프카 브로커를 가르키는 주소 목록 (외부에서 접속하기 위한 리스너 설정), 카프카 브로커는 초기 연결 시, 이를 클라이언트에게 보냄
* **KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR:** 토픽 파티션의 복제 갯수 (값이 1이면, 리더 브로커에만 파티션이 기록되며, 만약 3이라면 하나의 파티션이 총 3개로 분산 저장됨을 의미)
* **KAFKA_NUM_PARTITIONS:** 토픽의 파티션 갯수
* **KAFKA_LOG_RETENTION_HOURS:** 세그먼트 파일의 삭제 주기, 기본값 hours, 168시간(7일)
* **LOG_DIR:** 로그 데이터를 보관할 디렉토리


## **구축 확인**


> **토픽 생성**

주키퍼 컨테이너 내부에 접속 후 진행합니다.

* 토픽에 대해서 어느 서버든 Leader 가 되며, 나머지는 Follower가 됨
* 예시에는 하나의 토픽에 대하여 5개의 파티션으로 쪼개며, 복제본은 3개를 유지

```
-- 토픽 생성
$ kafka-topics --create --zookeeper datanode1:2181,datanode2:2181,datanode3:2181 --replication-factor 3 --partitions 5 --topic test-topic

-- 토픽 확인
& kafka-topics --describe --zookeeper datanode1:2181,datanode2:2181,datanode3:2181
```  

![Screenshot_3](https://user-images.githubusercontent.com/92897860/215686172-2812423c-4e00-4364-9f71-5385cdff50b7.png)

> **토픽에 데이터 넣기 (Producer)**

```
-- 메시지 게시  
$ kafka-console-producer --broker-list datanode1:9092 --topic test-topic
```  

> **토픽 데이터 확인 (Consumer)**

```  
$ kafka-console-consumer --bootstrap-server datanode1:9092 --topic test-topic --from-beginning
```  

![Screenshot_4](https://user-images.githubusercontent.com/92897860/215687960-904b7ccb-bd1c-4a62-aa71-f100043b8d66.png)


> **Kafdrop - 카프카 클러스터 모니터링 UI**

카프드랍이라는 카프카 클러스터 모니터링 UI 로 부터 test-topic을 확인한 결과입니다.

5개의 파티션 중 0~2번 파티션의 0번 offset부터 100 개의 메시지를 출력해봤을 때, test-topic의 메시지가 1번과 2번 파티션에 담긴 것을 확인할 수 있습니다.

![Screenshot_5](https://user-images.githubusercontent.com/92897860/215688669-02faf703-1232-4052-a927-7d5c46daaa64.png)


이번 포스트에서는 Docker로 Kafka Cluster 구축하는 방법에 대해 알아보았습니다.  
감사합니다.


> **참고 자료**  

* [Topic Replication 개념](https://www.popit.kr/kafka-%EC%9A%B4%EC%98%81%EC%9E%90%EA%B0%80-%EB%A7%90%ED%95%98%EB%8A%94-topic-replication/)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
