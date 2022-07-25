---
title: "SpringBoot - PostgreSQL CDC"
last_modified_at: 2022-3-31
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **SpringBoot - PostgreSQL CDC 구현** 에 대한 내용을 공유해보려고 합니다.


## **CDC(Change Data Capture)**


CDC란, 데이터가 변화되었을 때 이를 인지하고 이루어지는 액션을 의미합니다.
즉, 사용자가 만든 테이블의 변경 사항을 추적(데이터 변화 캡쳐)하는 것을 말합니다.  

아래는 SpringBoot에 Postgres CDC를 연결하는 2가지 방법에 대해서 다룹니다.
* 1. PostgreSQL Notify&Listen (트리거 기반)
* 2. PostgreSQL Logical Replication Streaming (WAL로그 기반)


### **1. PostgreSQL Notify & Listen (트리거 기반)**

PostgreSQL Notify&Listen 이벤트를 통한 방식입니다.  
Notify 이벤트가 클라이언트에 의해 트리거되면 알림을 전송하도록 하며, 이 이벤트에는 Payload(데이터)가 포함되어 있어 트리거에서 지정한 CRUD이벤트에 대해서 해당 내용을 담아서 수신합니다. 트리거가 적용된 함수를 폴링하는 별도의 데이터 파이프라인으로 Notify&Listen 기능이 수행한다고 보면 됩니다.  


>**구현**  

**1. 트리거 프로시져 생성 :: Notify를 사용하여 이벤트 보내기** 

- 행에 변화가 생길 때마다 실행되도록 설정합니다.
- pg_notify를 통해 alert이라는 채널에 대해서만 트리거 발생하도록 설정합니다.
- TG_OP 으로, Insert/delete/update 설정합니다.
- 데이터 형식은 Json 문자열 형태로 전달되도록 설정합니다.

```
CREATE OR REPLACE FUNCTION notify_event() RETURNS TRIGGER AS $$    DECLARE 
        data json;
        notification json;
    BEGIN
        -- Convert the old or new row to JSON, based on the kind of action.
        -- Action = DELETE?             -> OLD row
        -- Action = INSERT or UPDATE?   -> NEW row
        IF (TG_OP = 'DELETE') THEN
            data = row_to_json(OLD);
        ELSE
            data = row_to_json(NEW);
        END IF;

        -- Contruct the notification as a JSON string.
        notification = json_build_object(
                          'table',TG_TABLE_NAME,
                          'action', TG_OP,
                          'data', data);    
                         
        -- Execute pg_notify(channel, notification)
        PERFORM pg_notify('alert',notification::text);
         
        -- Result is ignored since this is an AFTER trigger
        RETURN NULL; 
    END;
$$ LANGUAGE plpgsql;  
```

**2. 트리거 등록**

- 변경사항이 발생할 때마다 위의 프로시져가 호출됩니다.

```
# 테이블에 등록
CREATE TRIGGER alert_notify_event
AFTER INSERT OR UPDATE OR DELETE ON <<테이블>>
    FOR EACH ROW EXECUTE PROCEDURE notify_event();
 
```

**3. Listen :: 채널 감지**  

- 다음과 같이, 채널에 대한 Listen 작업과 데이터를 받을 수 있도록 설정합니다.
- ex. 스케쥴러 혹은 Runner 등을 통해서도 가능합니다.

```
# 채널 Listen
stmt.execute("LISTEN alert");

# 채널로 오는 이벤트 데이터 수신 예시(*일부)
# payload 형식 : [BEGIN]  [table / action / data]  [COMMIT (날짜_시간)]
org.postgresql.PGNotification notifications[] = pgconn.getNotifications();
```

>**장단점**  

쉽게 구현이 가능하나, 
원래 명령문의 실행 시간을 증가시켜 성능 저하가 우려되며 DB 버전 변경 및 테이블에 이상이 있을 경우, 트리거 관리가 어려워질 수 있습니다.

  
### **2. PostgreSQL Logical Replication Streaming (WAL로그 기반)**

Postgresql에서 제공하는 논리적 복제를 통해 데이터의 변경 사항을 실시간으로 외부 시스템으로 스트리밍하는 방식입니다.  
데이터베이스의 미리 쓰기 로그(WAL)를 가져와 행 수준 변경 이벤트에 대한 엑세스를 제공합니다. 즉, 실시간 변경 사항에 대한 엑세스를 위해 PostgreSQL은 스트리밍 복제 프로토콜을 제공하며, 그 중 논리적 복제를 통해 외부로 스트리밍합니다.

>**구현**  

**1. 수동 설정 변경 및 확인** 
- 사전에 해당 버전이 논리 복제를 지원하는 지 확인해야 됩니다.
- 논리 복제가 가능하도록 Config를 변경해야 합니다.  

```
## postgresql.conf

# 동시 연결 최대 수 (복제 대상, 대기 서버) / 0 이면 복제를 비활성화
max_wal_senders = 10

# pg_xlog에 저장되는 과거 로그 파일 세그먼트의 최소 수
wal_keep_segments = 10

# 논리 복제 가능하도록 레벨 변경 (중요)
wal_level = logical

# 복제 슬롯 최대 수 (해당 슬롯을 통해 이벤트 캐치 로그가 왔다갔다 함 / 슬롯 분리 = 용도에 따라 이벤트 분리)
max_replication_slots = 10
```

```
## pg_hba.conf

# 복제 스트림에 복제 권한이 있는 사용자 연결 활성화
host    all             all             [networkIP ?.?.?].0/24    trust

host    replication     replication [networkIP ?.?.?.?]/32    md5
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```

**2. 복제 연결 생성**

- 복제 API 사용합니다. (pgjdbc API를 통해 복제 슬롯 생성)  
- 복제 슬롯을 사용하여 서버에 WAL 로그를 예약하고 WAL 로그를 필요한 형식으로 디코딩합니다.

```
## replicationSlot 생성
replConnection.getReplicationAPI()
        .createReplicationSlot()
        .logical()
        .withSlotName(pg.getSlotName())
        .withOutputPlugin(pg.getOutputPlugin())
        .make();

con.setAutoCommit(pg.getAutoCommit());
```

**3. 복제 슬롯 생성**

- 복제 슬롯을 통해 해당 스트림으로 모든 변경 사항을 전송합니다.

```
## 해당 replicationSlot을 통해 스트리밍으로 DB 변화 캐치
## WAL EVENT
PGReplicationStream stream =
        replConnection.getReplicationAPI()
                .replicationStream()
                .logical()
                .withSlotName(pg.getSlotName())
                .withSlotOption("include-xids", pg.getIncludeXids())
                .withSlotOption("skip-empty-xacts", pg.getSkipEmptyXacts())
                .withSlotOption("include-rewrites", pg.getIncludeRewrites())
                .withSlotOption("include-timestamp", pg.getIncludeTimestamp())
                .withStatusInterval(pg.getStatusInterval(), TimeUnit.SECONDS)
                .start();
```


>**장단점**  

논리 복제 로그라 DB에 직접 쿼리하지 않기 때문에 DB부하가 적은편입니다.
또한, TimeStamp 및 트리거 생성 등이 불필요하다는 장점이 있습니다.  
다만, 사용시 slot 설정에 대한 관리가 필요하며, 오래된 버전에서는 지원하지 않습니다.
높은 트랜잭션 볼륨을 가져야하기도 하며, 일부 테이블만 지정해서 가져오지 못하고 모든 테이블에 대해서 변경로그를 가져오기 때문에 필요한 변경로그만 잘 캐치해야 됩니다.


## **정리**

이번 포스팅에서는 PostgreSQL CDC에 대해서 알아보았으며, SpringBoot와 연결 구현 방법에 대해서도 알아보았습니다.
또한, 설명한 방법 외에도 다른 CDC 방법이 있을 수 있다는 점 참고하시길 바랍니다.


> **참고 자료**  

* [PostgreSQL Replication](https://www.postgresql.org/docs/10/logical-replication.html)  
* [PostgreSQL Extensions to the JDBC API](https://jdbc.postgresql.org/documentation/head/replication.html)  
* [PostgreSQL Notify & Listen](https://postgresql.kr/blog/pg_listen_notify.html)  

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
