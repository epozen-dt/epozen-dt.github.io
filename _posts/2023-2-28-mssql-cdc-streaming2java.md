---
title: "MSSQL log based CDC - streaming to java"
last_modified_at: 2023-02-28
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **MSSQL log based CDC 적용 방법**에 대해 공유해보려고 합니다.


## **MSSQL log based CDC**

데이터베이스의 트랜잭션 로그를 기반으로 변경된 데이터를 실시간 캡쳐하고, 이를 외부 데이터 소스나 시스템에서 이용할 수 있는 방법입니다. 

> **캡쳐 인스턴스와 CDC 테이블(변경 테이블)**

로그 기반 CDC에서는 캡쳐 인스턴스와 CDC 테이블에 대해서 먼저 알아야합니다.

**캡쳐 인스턴스**는 로그 기반 CDC에서 가장 먼저 설정해야하는 객체로, 캡쳐를 할 대상 테이블이나 컬럼을 설정가능하며, 로그 덤프 파일에 대한 위치 정보와 레코드 읽기 등을 정의한 객체입니다.

**변경 테이블**은 캡쳐한 로그 레코드를 저장하기 위한 테이블로, CT(Change Tracking) 테이블에 삽입됩니다. 변경 테이블에 삽입된 로그 레코드는 캡처한 순서대로 정렬되며, 이를 이용해 변경 내용을 추적합니다.

정리하면, 로그 기반 CDC에서는 캡쳐 인스턴스와 변경 테이블을 사용하여 변경 내역을 추적하게 됩니다.

> **상세 설명** 

* 데이터 추적 방법  
    + 트랜잭션 로그를 사용해 변경 내용 추적합니다. 

* 변경 내용 저장 방식  
    + 추적된 변경 내용을 CDC 테이블(schema_table_CT)에 저장합니다.

* 데이터 무결성 보장  
    + 트랜잭션 로그에서 추적된 변경 내용은 해당 트랜잭션 내에서 발생한 변경 내용을 보장하므로, 무결성 을 보장합니다. 
    + 예를 들어, 한 트랜잭션에서 A테이블, B테이블의 레코드를 동시에 업데이트할 때, 해당 트랜잭션에서 발생한 변경 내용은 로그에서 모두 추적됩니다.

* 데이터베이스 성능  
    + 트랜잭션 로그를 읽기 때문에 성능에 어느정도 영향이 있습니다.


> **테스트 환경**
  
* MSSQL 2019 Developer Edition (64-bit) on Linux (Ubuntu 20.04.5 LTS)
* JAVA 8
* IDE: IntelliJ


### **1. MSSQL 컨테이너 생성** 

```
version: '3.7'
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2019-latest
    user: root
    container_name: sql1
    hostname: sql1
    ports:
      - 1433:1433
    volumes:
      - ./volume/sql1data:/var/opt/mssql
    environment:
      ACCEPT_EULA: Y
      SA_PASSWORD: password1!
```

* SQL Server Agent 를 사용하도록 설정 (기본적으로 사용하지 않도록 설정되어있음)

```
-- (컨테이너 생성 이후, 내부에서 실행)
$ su -
$ /opt/mssql/bin/mssql-conf set sqlagent.enabled true

-- (restart 도커 컨테이너)
```


### **2. CDC 활성화를 위한 쿼리문 작성** 

크게 4단계의 과정을 거쳐 CDC를 수행할 수 있게 됩니다.

1. CDC 활성화 (sp_cdc_enable_db)  

```
USE master;
-- CDC를 위한 데이터베이스 생성
CREATE DATABASE [데이터베이스명];

-- 데이터베이스에 대한 사용자 권한 부여
USE [데이터베이스명];


-- CDC 활성화 (Database 범위에서의 활성화)
EXECUTE sys.sp_cdc_enable_db;
select name as DBName,is_cdc_enabled, * from sys.databases
```

2. CDC 대상 테이블 설정 (sp_cdc_enable_table 저장 프로시저 실행하여 CDC를 적용할 테이블에 대한 메타 데이터를 생성)

```
-- CDC 대상 테이블 생성
CREATE TABLE cdc_table
(
   id INT PRIMARY KEY,
   name NVARCHAR(50)
);


-- CDC 활성화 (Table 단위에서의 활성화)
-- CDC 테이블에 적용
EXEC sys.sp_cdc_enable_table  
@source_schema = N'dbo',  
@source_name   = N'cdc_table',  
@role_name     = NULL,  
@supports_net_changes = 1;
```

3. 변경 내용 추적 (CDC 대상테이블에 대한 변경 내용을 추적, 트랜잭션 로그에서 변경 내용을 읽고 해당 변경 내용을 CDC 테이블에 저장)

```
-- 해당 과정이 없어도 무방
DECLARE @begin_lsn BINARY(10), @end_lsn BINARY(10)
SET @begin_lsn = sys.fn_cdc_get_min_lsn('dbo_cdc_table')
SET @end_lsn = sys.fn_cdc_get_max_lsn()
SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_cdc_table(@begin_lsn, @end_lsn, N'all');
```

4. 변경 내용 확인

```
INSERT INTO dbo.cdc_table
(id, name)
VALUES(1, 'test-1');

-- 실제 테이블
SELECT * FROM dbo.cdc_table;
-- CDC 테이블
SELECT * FROM cdc.dbo_cdc_table_CT;
```

  
### **3. Java 코드 작성** 

*일부 코드만을 담았습니다.*

**3.1 Maven Dependency 추가**  

```
<dependency>
	<groupId>com.microsoft.sqlserver</groupId>
	<artifactId>mssql-jdbc</artifactId>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

**3.2 변경 Events 캐치 (cdc 스키마의 변경데이터 확인)**

* CDC 기능에서 제공하는 내장 함수인 "cdc.fn_cdc_get_all_changes_" 를 통해 CDC 테이블의 변경 내용 캐치
* LSN (로그 시퀀스 넘버) 업데이트를 통해 새로 들어오는 변경 데이터만 수신

```
/* CDC 스키마의 해당 테이블에 접근해 변경 데이터를 지속적으로 가져오도록 하는 쿼리문 */
private static final String CDC_QUERY = "SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_cdc_table (?, ?, 'all')";
```

```
// 최초 실행시 CDC 테이블의 시작 LSN 값을 가져옴
Long startLsn = jdbcTemplate.queryForObject("SELECT CONVERT(BIGINT, sys.fn_cdc_get_min_lsn('dbo_cdc_table'))", Long.class);
// 컬럼명 조회
List<String> columnNames = jdbcTemplate.queryForList(COLUMN_QUERY, String.class);

while (true) {
    // 최신 LSN 값을 가져와서 쿼리 파라미터로 설정
    Long endLsn = jdbcTemplate.queryForObject("SELECT CONVERT(BIGINT, sys.fn_cdc_get_max_lsn())", Long.class);
    if (endLsn > startLsn) { // 최신 데이터만 가져오기

        // CDC 변경 데이터 가져오기
        List<Map<String, Object>> results = jdbcTemplate.queryForList(CDC_QUERY, startLsn, endLsn);
        for (Map<String, Object> row : results) {
            System.out.println(row);
        }
        // 변경 데이터 처리 후, 최신 LSN 값으로 startLsn 갱신
        startLsn = endLsn;
    }
}
```

### **4. 테스트** 

**4.1 변경 데이터 추출 확인**  

```
INSERT INTO dbo.cdc_table
(id, name)
VALUES(101, '101=>INSERT'), (102, '102=>INSERT');
```

* **테스트 결과, 변경 이벤트(INSERT 등) 추출을 확인 가능**
  
![Screenshot_1](https://user-images.githubusercontent.com/92897860/221783888-adb87d98-ffc9-427b-bfe9-af10f9f60b73.png)


## **정리**

MSSQL 로그 기반 CDC를 통해 Java 에서의 행 수준의 변경 데이터에 대한 추출 가능을 확인했습니다.     
테스트 수준의 해당 코드는 https://github.com/chong1ha/mssql-cdc 에서 확인 가능합니다.

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
