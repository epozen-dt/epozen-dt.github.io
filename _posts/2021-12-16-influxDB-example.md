---
title: "InfluxDB 예제 - Java Client를 활용한 간단한 Write & Read"
last_modified_at: 2021-12-16
author: 심건우

---

이번 포스팅에서는 InfluxDB에서 지원하는 Java Client를 활용하여 데이터 Write / Read 하는 방법을 간단한 예제를 통해 알아보겠습니다.

## InfluxDB

 InfluxDB는 InfluxData에서 개발한 오픈 소스 시계열 데이터 베이스입니다.
 
 주요 특징은 아래와 같습니다.
   - NoSQL Database
   - Go 언어로 작성
   - 빠른 읽기 및 쓰기 가능
   - 다양한 Client 지원 (ex. Go, Java, Python, Node.js 등)
   - Schemaless 디자인
   - 보존 정책에 따라 유효하지 않은 데이터는 자동 만료됨
   - nanosecond 단위의 unix time을 사용함 (unix time : <1970년 1월 1일 00:00:00> 부터의 경과 시간을 초로 환산하여 정수로 나타낸 것)

 다음은 InfluxDB에서 주로 사용되는 용어들을 RDB(Rule-Based Database)의 용어와 대응시켜 설명한 것입니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146504462-00c31860-a497-42fe-8b28-2ba637c43c3c.png)

 
## 설치 및 실행

 InfluxDB를 설치하는 방법은 Installer, Docker image 등이 있습니다.
 
 본 포스팅에선 Docker image를 사용하는 방법으로 진행하겠습니다.
 
 먼저 InfluxDB의 Docker image를 Pull 합니다.
 
 ```
 docker pull influxdb:latest
 ```
  
 다음으로 Docker Container를 생성하기 위한 docker-compose.yaml 파일을 작성합니다.
 
 ```yml
 version: "3"
 services:
   influxdb:
     image: influxdb:latest
     ports:
       - '8086:8086'
     environment:
       - INFLUXDB_DB=test
       - INFLUXDB_ADMIN_USER='아이디'
       - INFLUXDB_ADMIN_PASSWORD='비밀번호'
 ```
 
 InfluxDB 컨테이너를 생성합니다.
 
 ```
 docker-compose up -d
 ```
  
## 초기 설정

 처음 실행 시 필요한 설정을 위해 "http://(컨테이너 ip):8086"에 접속하여 "Get Started"를 클릭합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146516378-8cc96b54-ea3a-4d10-a94e-71f58d5c2032.png)

 
 Initial User 관련 설정 입력 후 "Continue"를 클릭합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146513779-a5eff725-c592-44e3-9e08-39d424655538.png)


 "Quick Start"를 클릭합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146513841-b9da1bc6-9cf4-4a16-887e-f02236b35ae1.png)

## Bucket 생성

 Bucket이란, InfluxDB에서 데이터가 저장되는 공간이라고 할 수 있겠습니다.
 
 이제 Bucket을 생성해보겠습니다.
 
 아래와 같이 Load Data의 Buckets 메뉴에서 "Create Bucket"을 클릭합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146514467-3637a5d3-10e7-4807-8143-26e2de7b2053.png)


 Bucket 이름 지정 후, "Create"를 클릭합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146514543-92caf86b-69da-4893-a286-c5c1db8db069.png)


## Token 생성

 Client에서 InfluxDB에 접근할 때 보안을 위해 필요한 Token을 생성해보겠습니다.
 
 Load Data의 API Tokens 메뉴에서 "Generate API Token"을 클릭합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146514877-934c61c5-c32c-4f30-b18f-489fd81d152a.png)
 
 
 필요에 따라 Access Type을 설정합니다. 본 포스팅에선 Read/Write Type으로 설정했습니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146514993-8d40e1ad-7f23-45da-b083-7dc5e5a8283c.png)
 
 
 API Token이 적용될 Bucket 지정 후, "Save"를 클릭합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146515071-cd973cd8-6a57-412c-ba13-81b91512b1e1.png)


 아래와 같이 생성된 Token을 확인합니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146515175-30895941-e7f5-46f9-84b0-114943548ab4.png)
 
 
 ![image](https://user-images.githubusercontent.com/87160438/146515206-fc0217a5-4524-42b3-9534-aa57a6c1ce13.png)


## Java Client 생성

 아래 그림은 InfluxDB에서 제공하는 Client Libraries입니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146515760-b683f9eb-a063-462d-b7ce-b3700443fdd0.png)

 
 본 포스팅에선 Java Client를 사용하여 예제를 진행했습니다.
 
 먼저, Maven Dependency를 추가합니다.
 
 ```xml
  <!-- https://mvnrepository.com/artifact/com.influxdb/influxdb-client-java -->
  <dependency>
   <groupId>com.influxdb</groupId>
   <artifactId>influxdb-client-java</artifactId>
   <version>3.4.0</version>
  </dependency>
 ```
 
 다음으로 아래와 같이 InfluxDB 연결 정보를 설정하고 Client를 생성합니다.
 
 ```java
  // influxDB url
  private static String url = "http://(InfluxDB 주소):8086";
  // org Name (초기 설정에서 지정한 Organization Name)
  private static String org = "test";
  // bucket name
  private static String bucket = "write/read";
  // bucket Token
  private static char[] token = "(토큰)".toCharArray();

  // Client 생성
  InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
 ```
 
## Write 테스트

 생성한 Client로 데이터를 Write 해보겠습니다.
 
 테스트 코드는 아래와 같고, Point 클래스를 활용했습니다.
 
 ```java
public void writeInfluxDBWithPoint() {
      // 클라이언트 생성
      InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
      // write api 호출
      WriteApiBlocking writeApiBlocking = influxDBClient.getWriteApiBlocking();
      Random random = new Random();
      // random 값
      double i = random.nextDouble();
      // bucket에 삽입될 point
      Point point = Point.measurement("test-measurement")
              .time(Instant.now().toEpochMilli(), WritePrecision.MS)
              .addTag("test-tag", "tag-1")
              .addField("test-field", i);
      // write
      writeApiBlocking.writePoint(point);
      // 클라이언트 연결 종료
      influxDBClient.close();
}
 ```
 
실행 결과는 아래와 같이 Data Explorer에서 확인할 수 있습니다. (Write 시 삽입한 정보 지정 후, "Submit" 클릭)

![image](https://user-images.githubusercontent.com/87160438/146727476-7b175cbd-45ec-4c15-a79c-1269d5a75d2f.png)


## Read 테스트

 생성한 Client로 위에서 Write한 데이터를 Read 해보겠습니다.
 
 테스트 코드는 아래와 같고, Flux 쿼리를 활용했습니다. (쿼리 내용은 bucket에 있는 데이터를 처음부터 끝까지 가져온다는 것과 같습니다.)
 
 ```java
 public void readInfluxDB() {
     InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
     // flux 쿼리
    List<FluxTable> tables = influxDBClient.getQueryApi()
            .query("from(bucket: \"" + bucket + "\") |> range(start:0)");
    for (FluxTable fluxTable : tables) {
        List<FluxRecord> records = fluxTable.getRecords();
        for (FluxRecord fluxRecord : records) {
            System.out.println(fluxRecord.getTime() + ", " +
                    fluxRecord.getValueByKey("_field") + ", " +
                    fluxRecord.getValueByKey("_value"));
        }
    }
    influxDBClient.close();
}
 ```
 
 아래 그림은 차례로 Data Explorer와 위 코드를 실행한 결과를 비교한 화면입니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/146731754-3a78c96a-6101-4115-a9c4-46777365eaf1.png)


 ![image](https://user-images.githubusercontent.com/87160438/146731782-8295119d-94ea-4afe-879a-259ea6c99de2.png)


여기까지 InfluxDB관련 설정, Java Client를 활용한 Write/Read 예제를 진행했습니다.

감사합니다.
