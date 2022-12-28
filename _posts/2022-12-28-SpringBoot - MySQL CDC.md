---
title: "SpringBoot - MySQL CDC Binlog"
last_modified_at: 2022-12-28
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **SpringBoot - MySQL CDC 구현**에 대해 공유해보려고 합니다.


## **MySQL Binlog CDC**

Binlog 혹은 Binary log 는 MySQL 의 내용이나 구조를 변경하는 모든 이벤트(INSERT, DELETE, UPDATE 등)에 대한 정보를 포함하는 로그 파일입니다. 기본적으로 Transaction commit 시 기록되어지며, 바이너리 형태로 저장됩니다. 주로 Replication 과 Recovery 등 데이터 동기화를 목적으로 사용되어집니다.

**이번 포스팅에서는 Debezium 에서 사용하는 MySQL Binlog Connector를 활용하여 MySQL 내 변경 데이터를 실시간으로 추출할 수 있는지에 대해서 테스트를 진행합니다.**

> **MySQL Binlog Connector 기능** 

자세한 내용은 https://github.com/shyiko/mysql-binlog-connector-java 에서 확인 가능합니다.   

* Auto Binlog 파일 이름/위치 추적 가능  
* TLS 보안 통신 가능  
* JMX 친화적  
* 실시간 통계 지원  
* 타사 종속성 없음  
* Java/Python/Go 지원  
* Maven Central 가용성  

> **테스트 환경**
  
* Java JDK 8
* Maven
* MySQL Community Server 8
* IDE: IntelliJ


### **1. MySQL 기본 설정** 

* 외부 접속 및 CDC 권한 부여

```
-- 권한: Replication Client, Slave, Super 
mysql> select host, user from mysql.user;
mysql> grant all privileges on *.* to 'mysql'@'192.168.10.%';
mysql> flush privileges;

-- (모든 IP 허용)
'mysql'@'%'
-- (특정 IP 대역대 허용)
'mysql'@'111.11.11.%'
-- (특정 IP만 허용)
'mysql'@'111.11.11.1'
```
  
### **2. Java 코드 작성** 


**2.1 Maven Dependency 추가**  

```
<dependency>
		<groupId>com.zendesk</groupId>
		<artifactId>mysql-binlog-connector-java</artifactId>
		<version>0.27.5</version>
</dependency>
```

**2.2 BinaryLogClient 연결**

* CDC 클라이언트 생성

```
-- Binlog conn
BinaryLogClient binaryLogClient = new BinaryLogClient(host, port, user, password);
```

**2.3 변경 Events 캐치**

* 들어온 변경 이벤트(INSERT/DELETE/UPDATE) 실시간 캐치
* DDL과 같은 Query 이벤트 실시간 캐치

```
@Override
public void onEvent(Event event) {

    /* Event Process 확인 */
    if (log.isInfoEnabled()) {
            log.info("## Received " + event);
    }

    /* QUERY Event 확인 */
        if (event.getHeader().getEventType() == EventType.QUERY) {
            QueryEventData data = event.getData();
            log.info("{0} " + data.toString());
    }

    /* 들어온 INSET SQL 확인 */
    if (event.getHeader().getEventType() == EventType.EXT_WRITE_ROWS ) {

        WriteRowsEventData data = event.getData();
        Serializable[] s_data = data.getRows().get(0);
        log.info("{2} " + Arrays.toString(Arrays.stream(s_data).toArray()));
    } else if (event.getHeader().getEventType() == EventType.EXT_UPDATE_ROWS) {

        UpdateRowsEventData data = event.getData();

    } else if (event.getHeader().getEventType() == EventType.EXT_DELETE_ROWS) {
        DeleteRowsEventData data = event.getData();
        Serializable[] s_data = data.getRows().get(0);
        log.info("{4} " + Arrays.toString(Arrays.stream(s_data).toArray()));
    }

}
```

```
-- 연결
binaryLogClient.registerEventListener(this);
binaryLogClient.connect();

-- 연결 해제
binaryLogClient.unregisterEventListener(this);
binaryLogClient.disconnect();
```

### **3. 테스트** 

**3.1 변경 데이터 추출 확인**  

* 프로시저를 통해 데이터 INSERT 를 여러번 수행

```
-- 프로시저 작성
sql> 
CREATE PROCEDURE `INSERT_DATA` ()
BEGIN
	DECLARE i INT DEFAULT 1;
    WHILE i <= 10 DO
        INSERT INTO test.post (id, name) VALUES (i, 'TEST');
        SET i = i + 1;
        COMMIT;
    END WHILE;
END

-- 실행
sql> CALL INSERT_DATA();
```

* **테스트 결과, MySQL binlog 파일을 읽고 Query 이벤트(프로시저 생성 등) 및 변경 이벤트(INSERT 등) 추출을 확인 가능**
  

![1](https://user-images.githubusercontent.com/92897860/209756615-d12eb2ca-383e-40a9-98a6-5391481ce2a1.png)


**3.2 이벤트 흐름 파악**  

```
sql> INSERT INTO test.post (id, name) VALUES (2122, 'TEST');
sql> commit;
```

* **변경 이벤트에 대해서 하나의 트랜잭션 단위로 가져옴을 확인 가능**
* [INSERT] GTID => QUERY => TABLE_MAP => EXT_WRITE_ROWS => XID
  
![2](https://user-images.githubusercontent.com/92897860/209757494-5c60fcc1-c1b2-4f41-91aa-bf74d9b08c43.png)


## **정리**

Java 에서의 행 수준의 변경 데이터에 대한 추출 가능을 확인했습니다.   
필요한 데이터를 추출할 수 있는가에 대해서 초점을 두고 진행했기 때문에 그 점 염두에 두시면 좋겠습니다.  

해당 코드는 https://github.com/chong1ha/mysql-binlog-cdc 에서 확인 가능합니다.


> **참고 자료**  

* [mysql-binlog-connector-java](https://github.com/osheroff/mysql-binlog-connector-java)  
* [mysql-binlog-connector-java-code](https://www.javatips.net/api/mysql-binlog-connector-java-master/src/test/java/com/github/shyiko/mysql/binlog/BinaryLogClientTest.java)  

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
