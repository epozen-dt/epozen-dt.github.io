---
layout: post
title: "Zookeeper, 클러스터 구성 및 Python 클라이언트 예제"
date: 2022-04-25
author: 심건우
---

이번 포스팅에선, Zookeeper 관련 정보를 간략히 설명하고, 간단한 클러스터 구성 및 Python 클라이언트 예제를 진행하겠습니다. 

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
  - Zookeeper 서비스를 구성하는 서버는 모두 상호 간 인지 상태
  - 영구 저장소의 트랜잭션 로그 및 스냅샷과 함께 상태에 대한 in-memory 이미지 유지
  - 클라이언트는 단일 Zookeeper 서버에 연결
  - 클라이언트는 요청 송신 및 수신, 감시 이벤트, 생존 신호 송신 등 TCP 연결 유지
  - 서버에 대한 TCP 연결 소실 시, 클라이언트는 다른 서버에 연결

#### 3. 순차적이다.
  - Zookeeper는 모든 Zookeeper 트랜잭션의 순서를 의미하는 번호 순으로 각 업데이트 반영
  
#### 4. 속도가 빠르다.
  - 특히 "read-dominant" 작업에서 속도가 빠름
  - 쓰기 작업보다 읽기 작업이 일반적인 곳에서 약 10배 더 잘 작동

## Data model 및 hierarchical namespace
 - Zookeeper의 네임스페이스는 표준 파일 시스템의 네임스페이스와 유사
 - "/"로 노드 구분 가능
 - Zookeeper의 네임스페이스에 있는 모든 노드는 경로로 식별됨
  
![image](https://user-images.githubusercontent.com/87160438/165010787-48d1438d-574d-417a-8895-1f6946a2a5f7.png)

[이미지 출처](https://zookeeper.apache.org/doc/current/zookeeperOver.html)

## Nodes 와 Ephemeral nodes
 - 표준 파일 시스템과 달리 Zookeeper 네임스페이스의 각 노드는 하위 노드뿐만 아니라 데이터도 연결 가능
 - Zookeeper는 상태 정보, 구성 및 설정, 위치 정보 등 가벼운 데이터를 저장하도록 설계됨
 - 각 노드에 저장된 데이터는 일반적으로 바이트에서 킬로바이트 범위 안팎
 - Zookeeper에서 노드는 "znode"로 불림
 - znode는 데이터 변경, ACL 변경 및 타임스탬프에 대한 버전 번호를 포함하는 상태 구조를 유지하여 캐시 유효성 검사 및 일부 업데이트 허용
 - znode 데이터 변경 시, 버전 번호 증가
 - 네임스페이스의 각 znode에 저장된 데이터는 원자성을 띔 (단일 읽기/쓰기)
 - 읽기는 znode와 관련된 모든 데이터를 바이트 형태로 가져옴
 - 쓰기는 znode와 관련된 모든 데이터를 바이트 형태로 저장함
 - 각 노드에는 ACL(접근 제어 목록)이 존재
 - Ephemeral(임시) znode를 생성한 세션이 활성 상태일 때, 임시 znode 유지
 - Ephemeral(임시) znode를 생성한 세션이 종료 상태일 때, 임시 znode 삭제

## Conditional updates 및 watches
 - Zookeeper는 감시 기능을 지원
 - 클라이언트는 znode 감시 설정 가능
 - znode 변경 시, 감시 기능이 활성화 및 감시 기능 삭제
 - 감시 기능 활성화 시, 클라이언트는 znode가 변경되었다는 내용의 패킷 수신
 - 클라이언트와 Zookeeper 서버 하나 간의 연결이 끊기면, 클라이언트는 관련 알림을 수신

## Guarantees
 Zookeeper는 아래 항목들을 보증합니다.
 - 순차적 일관성 : 클라이언트의 업데이트가 전송된 순서대로 적용
 - 원자성 : 업데이트는 성공하거나 실패함 (부분적 결과 x)
 - 단일 시스템 이미지 : 연결된 서버에 관계없이 클라이언트는 동일한 서비스를 제공받음
 - 안정성 : 업데이트는 클라이언트가 업데이트를 덮어쓸 때까지 지속됨
 - 적시성 : 클라이언트에 제공되는 서비스는 일정 시간 내에 최신 상태로 유지됨

## Simple API
 - create : 트리 안에 노드를 생성 
 - delete : 노드를 삭제
 - exists : 해당 위치에 노드 존재 여부 확인
 - get data : 노드로부터 데이터를 읽어옴
 - set data : 노드에 데이터를 씀
 - get children : 노드가 가진 children 목록을 가져옴
 - sync : 데이터가 전파될 때가지 대기
 
## 클러스터 구성
 3개의 Zookeeper 서버로 클러스터를 구성했고, Docker 환경에서 진행했습니다.
 
 추가적으로 시각화 도구인 ZooNavigator를 사용하여 Zookeeper 내부 데이터를 확인했습니다.
 
 - docker-compose.yaml
 
```yaml
version: "3.0"
services:
  zk1:
    image: zookeeper:3.8
    restart: always
    hostname: zk1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181

    volumes:
      - "./zk-cluster/zk1/data:/data"
  zk2:
    image: zookeeper:3.8
    restart: always
    hostname: zk2
    ports:
      - "2182:2181"
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
    volumes:
      - "./zk-cluster/zk2/data:/data"

  zk3:
    image: zookeeper:3.8
    restart: always
    hostname: zk3
    ports:
      - "2183:2181"
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
    volumes:
      - "./zk-cluster/zk3/data:/data"

  zoo-navi:
    image: elkozmon/zoonavigator:1.1.2
    ports:
      - 9000:9000
    environment:
      HTTP_PORT: 9000
```

 - 실행

```
docker-compose up -d
```

## Python 클라이언트 예제
 Python 클라이언트를 통해 데이터를 쓰고, 읽어보았습니다.
 
 라이브러리는 kazoo를 사용했고, Jupyter Notebook을 통해 진행했습니다.
 
 [kazoo 관련 자료](https://kazoo.readthedocs.io/en/latest/)
 
 - notebook 작성
```python
# kazoo 라이브러리 import
from kazoo.client import KazooClient
```

```python
# 클라이언트 객체 생성
zk = KazooClient(hosts='호스트 ip:2181')
```

```python
# 클라이언트 실행
zk.start()
```

```python
# 노드 존재 여부 확인
zk.exists("/gunwu")
```

```python
# 노드 등록
zk.ensure_path("/gunwu")
```

    '/gunwu'

```python
# 테스트용 파일 read
f = open("./test.txt", "rb")
lines = f.read()
f.close()
print(lines)
```

    b'This is Zookeeper test'
    
```python
# 테스트 노드 생성 및 데이터 write
zk.create("/gunwu/test1", lines)
```

    '/gunwu/test1'

```python
# 테스트 노드에 등록된 데이터 read
data, stat = zk.get("/gunwu/test1")
print(data)
```

    b'This is Zookeeper test'
    
```python
# 테스트 노드에 등록된 데이터 변경
zk.set("/gunwu/test1", data + bytes("***", 'UTF-8'))
```

    ZnodeStat(czxid=4294971847, mzxid=4294971848, ctime=1650869606309, mtime=1650869613949, version=1, cversion=0, aversion=0, ephemeralOwner=0, dataLength=25, numChildren=0, pzxid=4294971847)

```python
# 테스트 노드에 등록된 데이터 read
data, stat = zk.get("/gunwu/test1")
print(data)
```

    b'This is Zookeeper test***'
    
```python
# 노드 삭제
# zk.delete("/gunwu", recursive=True)
```

```python
# 클라이언트 중지
zk.stop()
```

```python
# 클라이언트 연결 종료
zk.close()
```

 - ZooNavigator 확인
 
