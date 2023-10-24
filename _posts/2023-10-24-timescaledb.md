---
title: "TimescaleDB 특징, 설치 방법 및 테스트"
last_modified_at: 2023-10-25
author: 조평연
---

본 포스팅은 Timescaledb의 특징을 알아보고 설치 방법 및 테스트를 진행한 내용입니다.

# 용어정리
### 1) 시계열 데이터
- 일정 시간동안 수집되어 시간 축에 의해 정렬될 수 있는 데이터이다.

- 시계열 데이터의 특징으로는 시간에 관해 순서가 매겨져 있다는 점과, 연속한 관측치는 서로 상관관계를 갖고 있다.

- 예시로는 현재 CPU 온도, 연도별 실업률, 주식 가격 등이 있다.

### 2) 시계열 데이터베이스
- 시계열 데이터를 처리하기 위해 최적화된 데이터베이스로, 실시간으로 쌓이는 대규모 데이터들을 처리하기 위해 고안되었다.

- InfluxDB, Prometheus, TimescaleDB는 등의 시계열 데이터베이스를 사용한다.

- 데이터 중복도가 낮은 경우 timescaledb가 좋고 중복도가 높은 경우 influxdb가 좋다.

### 3) TimescaleDB
- PostgreSQL을 기반으로 시계열 데이터 지원을 강화한 오픈 소스 시계열 RDBMS이다.

- 대부분의 시계열 데이터는 NoSQL 형태이지만 TimescaleDB는 RDBMS의 한 종류인 PostgreSQL 확장으로 만든 RDBMS 시계열 데이터베이스이다.

- 프로젝트에서 이미 RDMS를 사용하고 있다면 기존의 SQL 코드를 재활용 할 수 있어 부담 없이 채택할 수 있다.

# TimescaleDB 특징
- 특정 테이블이 TIMESTAMP 또는 DATE 타입의 필드를 포함할 경우 시계열 테이블로 취급된다.

 - 시계열 데이터를 저장하는 테이블을 Hypertable 이라고 부르며 TimescaleDB의 핵심 개념이라고 할 수 있다.

- 시계열 테이블에는 데이터 변경점이 발생할 때마다 시계열 필드를 기준으로 테이블을 여러 덩어리 (chunked)로 쪼개는 최적화 작업을 백그라운드에서 자동으로 수행한다.

- 백그라운드에서 내부 알고리즘의 판단에 의해 자동으로 특정 시일이 지난 덩어리를 압축한다.

# 설치방법
로컬이든 원격이든 그 환경에 직접 다운받을 수 도 있지만 여기서는 도커를 이용해서 다운받도록 하겠다. <br>

### 1) docker-compose.yml 파일 생성 및 실행

```Bash
version: '3'

services:
  timescaledb:
    image: timescale/timescaledb:latest-pg14
    ports:
      - 25432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - /project/timescaledb/data:/var/lib/postgresql/data
```
<br>

### 2) 설치 확인 (Dbeaver)

Host : 본인 local 또는 원격 ip<br>
Port : 25432 (본인 원하는 포트 사용 가능)<br>
Database : 본인 지정<br>
Username : 본인 지정<br>
Password : 본인 지정<br>

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbiZTrZ%2Fbtsp2apwmaq%2FMsbpkk4meZIOcXH96OgStk%2Fimg.png)

<br>

# 테스트
Dbeaver 에서 쿼리를 통한 테스트와 자바 클라이언트를 통해 아래와 같은 테스트를 진행해 볼 것이다.
- TIMESTAMP 포함된 테이블 생성
- 생성된 테이블을 Hypertable 로 변환 실행
- 데이터 삽입 / 조회

## Dbeaver 에서 쿼리를 통한 테스트

### 1) Dbeaver SQL 편집기를 이용

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbKHBim%2FbtspYeMXFzs%2Fc96SqHWfmzRcckxaNkuMk1%2Fimg.png)

<br>

### 2) 테이블 생성

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbtlE4v%2Fbtsp3nCaZ7V%2FuOGt0Qs2ZESI7TIptHxPkK%2Fimg.png)

<br>

### 3) Hypertable 변환

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FmwaRj%2FbtspTIVTxRX%2FCmskdNwX67eRrUc9pKX2J1%2Fimg.png)

<br>

### 4) 다중 컬럼 인덱스 생성

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FHxe7k%2FbtspTJHghVf%2FH3PzEPIch8lN299oc9Uiy1%2Fimg.png)

<br>

## Java Client 를 통한 테스트

### 1) Dependency 추가

![그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcLIxRl%2Fbtsp2aiMnPS%2FPiL2WorZ4lB9BrsmRcQR60%2Fimg.png)

### 2) 코드

```java
package com.example.timescaledbPra;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Timestamp;
import java.time.Instant;

import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TimescaledbPraApplication {

	public static void main(String[] args) {

		String url = "jdbc:postgresql://192.168.10.123456789:25432/본인 지정";
		String user = "본인 지정";
		String password = "본인 지정";

		try (Connection conn = DriverManager.getConnection(url, user, password)) {

			System.out.println("DB 연결 성공");

			// conditions 테이블 값 삽입
			String insertValueQuery = "INSERT INTO conditions (time, location, device, temperature, humidity) "
				+ "VALUES (?, ?, ?, ?, ?)";

			try (PreparedStatement pstmt = conn.prepareStatement(insertValueQuery)) {
				pstmt.setObject(1, Timestamp.from(Instant.now()));
				pstmt.setString(2, "Seoul2");
				pstmt.setString(3, "Device012");
				pstmt.setDouble(4, 26.5);
				pstmt.setDouble(5, 48.3);
				pstmt.executeUpdate();
			}

			System.out.println("데이터가 추가 되었습니다");

			// conditions 테이블 조회
			String selectValuesQuery = "SELECT * FROM conditions";

			try (PreparedStatement pstmtSelect = conn.prepareStatement(selectValuesQuery);
				 ResultSet rs = pstmtSelect.executeQuery()) {
				while (rs.next()) {
					Timestamp timestamp = rs.getTimestamp("time");
					Instant time = timestamp.toInstant();
					String location = rs.getString("location");
					String device = rs.getString("device");
					double temperature = rs.getDouble("temperature");
					double humidity = rs.getDouble("humidity");
					System.out.printf("Row (time: %s, location: %s, device: %s, temperature: %.1f, humidity: %.1f)%n",
						time, location, device, temperature, humidity);
				}
			}

		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}
```

<br>

# 참고 문헌

## TimescaleDB 공식 문서

[공식문서 1](https://docs.timescale.com/)

## TimescaleDB Java Client 공식 문서

[공식문서 2](https://docs.timescale.com/quick-start/latest/java/)