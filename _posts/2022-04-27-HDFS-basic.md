---
layout: post
title: "HDFS 블록 파일 시스템[1] - 기본 특징 및 구성"
date: 2022-04-27
author: 김수민
---

이번 포스팅에서는 HDFS에 대해 간단하게 설명드리겠습니다.

------

HDFS는 Hadoop Distributed File Syste으로, 대용량 데이터를 분산된 서버에 저장하고, 데이터를 빠르게 처리할 수 있게 하는 파일시스템 입니다.

![hadoop-hdfs](https://user-images.githubusercontent.com/87166420/165899323-b7ff35ba-08e9-4677-b087-9134b995191f.jpg)

​																																[[하둡 로고](https://mr-devlife.com/all-about-hdfs-concept/)]

HDFS는 구글 파일 시스템을 본떠 만든 오픈소스로, 다음과 같은 특징을 갖고 있습니다.

- 큰 용량의 데이터 저장 가능
  - 데이터 저장 시 **블록** 단위로 쪼개어 저장을 진행해 여러 장비에 걸쳐 데이터를 저장할 수 있습니다.
- 스트리밍 방식의 데이터 사용
  - HDFS로 저장된 데이터는 수정이 불가능 합니다.(단, 마지막 데이터에 추가로 데이터를 이어 붙이는 것은 가능함(append))
- 일반적 하드웨어 사용 가능
- 파일 블록 형태의 저장
  - 기본적인 블록의 크기는 128MB이며, 데이터 크기가 큰 경우 많은 블록을 띄우는 것을 방지하기 위해 크기를 키우기도 합니다. (최대 512YB까지 지원)
  - 데이터 크기가 블록 크기보다 작을 경우, 저장 파일 만큼만 디스크를 사용해 불필요한 공간 활용이 없습니다.
  - 보통은 3개의 복제본을 생성해 서로 다른 장비에 분산 저장하는 경우가 많습니다.

---

## 노드 종류

HDFS는  **Name Node(master)**와 **Data Node(worker)**로 구성되어 있습니다.

![hdfsarchitecture](https://user-images.githubusercontent.com/87166420/165899369-f98e219b-48fb-46c4-87e3-632b8547478c.png)

​																											[[HDFS Architecture](https://wikidocs.net/23582)]



- **Name Node** 기능

  - 메타 데이터 관리

    파일 시스템 트리와 그 트리에 포함된 모든 파일, 디렉터리에 대한 메타 데이터(namespace image, edit log) 유지

  - Data Node 모니터링

    Data Node가 3초마다 보내는 상태, 용량(heartbeat)을 체크해 장애 서버 여부 판단

  - 블록관리

    장애가 발생한 데이터노드의 블록을 새로운 Data Node에 복제

  - 클라이언트 요청 접수

    클라이언트 접근 시 기존 파일 저장 여부, 권한 환인의 절차를 거쳐 저장을 승인

- **Data Node** 기능

  - 클라이언트가 저장하는 파일을 로컬 디스크에 유지 (raw data, meta data 2가지 저장)
  - Name Node에 주기적으로 상태정보와 블록 위치정보를 전달(heartbeat)

  

이처럼 Name Node에는 Data Node에 대한 정보를 갖고 있으며, Data Node는 실질적인 데이터 저장을 합니다.

Data Node에 데이터가 저장되어 있으나, <u>Name Node가 손상된 경우, 가지고 있는 데이터를 읽어오거나 복구 할 수 없습니다.</u>

HDFS에서는 위의 이유 때문에 Name Node에 대해 HA(High Availability)를 지원합니다.

이런 전환 방식은 Controller 을 사용하여 관리하게 되는데, 일반적으로 Zookeeper 을 통해 구현됩니다.

---

## 노드 상태

우선 Name Node 상태에는 Active와 Standby 상태 2가지로 봅니다.

Active Name Node에 장애 발생 시, Standby Name Node 가 이어 Active 상태로 바뀌어 클러스터의 장애를예방하는 방식입니다.



Name Node에서 Data Node의 상태를 두 가지의 정보를 사용해 나타냅니다.

우선 활성 여부를 표현하기 위해 주기적인 heartbeat를 보내는지 여부에 따라 Live/Dead 상태로 표시됩니다.

노드에 문제가 발생해 Name Node에서 일정 시간 heartbeat를 받지 못하면 해당 Data Node의 상태는 Stale로, 그 후에도 반응이 없다면 Dead 로 변경합니다.



또 하나의 정보는 운영상태 정보 입니다.

운영상태는 Data Node에서 작업을 하기 위해 서비스를 잠시 중단하는 경우 블록을 보호하기 위해 설정하는 정보입니다.

상태에는 다음 5가지가 있습니다.

- NORMAL: 서비스 상태
- DECOMMISSIONED: 서비스 중단 상태
- DECOMMISSION_INPROGRESS: 서비스 중단 상태로 진행 중
- IN_MAINTENANCE: 정비 상태
- ENTERING_MAINTENANCE: 정비 상태로 진행 중

---

## 파일 접근

파일을 접근하는 과정을 간단하게 설명하겠습니다.

READ 과정을 간략하게 보자면 다음과 같습니다.

1. Client에서 Name Node에 block 위치를 요청
2. Name Node로부터 반환된 위치에 해당하는 Data Node에서 read
3. 만일 해당 Data Node의 상태가 정상이 아니면, Name Node에 알린 후 다른 블록에서 read

![file-read](https://wikidocs.net/images/page/23582/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-01-10_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_10.41.49.png)

​																												[[파일 읽기 프로세스](https://wikidocs.net/23582)]



WRITE 과정은 다음과 같습니다.

1. Client에서 Name Node로 create 시도
2. Data Node에서 사용할 Block Node 목록을 반환하면, 해당 Data Node에 파일 쓰기 요청
3. Data Node 간 복제 진행

![hdfs-write](https://wikidocs.net/images/page/23582/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-01-10_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_10.40.00.png)

​																												[[파일 쓰기 프로세스](https://wikidocs.net/23582)]



---

이번 포스팅에서는 Hadoop의 분산 파일 시스템인 HDFS에 대해 특징과 Node의 종류, 간단한 WRITE, READ 흐름에 대해 알아보았습니다.

다음 포스팅에서는 HDFS 를 docker로 구성하고, 간단한 파일 업로드 및 다운로드예제에 대해 알아보겠습니다.

---

### [참고 자료]

[1] https://wikidocs.net/37809

[2] https://yookeun.github.io/java/2015/05/24/hadoop-hdfs/

[3] https://mr-devlife.com/all-about-hdfs-concept/

[4] https://kadensungbincho.tistory.com/30
