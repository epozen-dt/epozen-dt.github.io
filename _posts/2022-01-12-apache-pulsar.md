---
title: "Apache Pulsar - 환경 구축 및 테스트 (Spring Boot 활용)"
last_modified_at: 2022-01-26
author: 심건우
---

이번 포스팅에서는 Apahce 사의 Pulsar로 환경 구축 및 테스트를 진행하겠습니다.

## Apache Pulsar란?
 Apache Pulsar란, 서버 간 메시징을 위한 멀티-테넌트 고성능 솔루션이라 할 수 있습니다.
 
 처음 Yahoo에서 개발했으며, 현재는 Apache Software Foundation이 관리하고 있습니다.
 
 주요기능은 다음과 같습니다.
 
  - 멀티 클러스터 지원
  - 클러스터 전체에서 메시지를 완벽하게 지리적으로 복제(geo-replication)
  - 매우 낮은 publish 및 end-to-end latency
  - 백만 개가 넘는 topic으로 확장 가능
  - 다양한 client API(Java, Go, Python, C++ 등) 지원
  - Apache BookKeeper에서 제공하는 영구 메시지 저장소를 통한 메시지 전달 보장
  - Server-less 경량 컴퓨팅 프레임 워크인 Pulsar Functions를 통해 stream-natvie 데이터 처리 기능 제공

## 구조
 구조는 아래 이미지와 같습니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/149073723-4e345c73-ac48-4fa0-b769-0e614db64bf9.png)

 이미지 출처 : https://pulsar.apache.org/docs/ko/concepts-architecture-overview/
 
 브로커는 Producer로부터 들어오는 메시지들을 처리하고 Load-Balancing하여, Consumer에게 전송합니다.
 
 다양한 조정 작업들을 처리하기 위해 구성된 저장소와 상호 작용하고, BookKeeper 인스턴스(Bookies)에 메시지를 저장합니다.
 
 
## 설치 및 실행
 이제 Pulsar 설치 및 실행을 진행하겠습니다.
 
 본 포스팅에선, docker-compose를 활용했습니다.
 
 또, 별도의 클러스터를 구성하지 않고 Pulsar에서 제공하는 Standalone 모드를 적용했습니다.
 
  1. 도커 이미지 pull
  
  Apache Pulsar 공식 이미지 pull
  
  ```
  docker pull apachepulsar/pulsar:latest
  ```
  
  2. docker-compose.yaml 작성
  
  ```yaml
  version: "3"
  services:
    pulsar:
      image: apachepulsar/pulsar:latest
      command: bin/pulsar standalone
      environment:
        PULSAR_MEM: " -Xms512m -Xmx512m -XX:MaxDirectMemorySize=1g"
      ports:
        - 6650:6650
        - 8080:8080
      restart: unless-stopped
  ```
  
  3. 실행
  
  ```
  docker-compose up -d
  ```
  
## Producer & Consumer (Spring Boot)
 구축한 Pulsar 환경을 통해 데이터를 주고 받아 보겠습니다.
 
 Producer와 Consumer 모두 Spring Boot 기반의 어플리케이션을 만들었습니다.
 
 Producer에 POST 요청을 보내면 Consumer에서 읽어들이는 구조이고, 대략적인 구성도는 아래와 같습니다.
 
 ![image](https://user-images.githubusercontent.com/87160438/149269549-033c8d5c-68c9-4041-91e9-a38d5fb03f2f.png)

 
 아래는 Producer와 Consumer 어플리케이션 구현 과정 및 테스트 결과입니다.
 
 구현에 앞서, Pulsar에서 제공하는 Java Client를 사용하려면 pom.xml에 아래와 같은 Maven dependency를 추가해야 합니다.
 
 ```xml
  <dependency>
    <groupId>org.apache.pulsar</groupId>
    <artifactId>pulsar-client</artifactId>
    <version>2.8.1</version>
  </dependency>
 ```
 
 ### Producer 구현
 - 구조
 
 ```
 - src
  - main
    - java
      - config
        - PulsarProducerConfig.java
      - controller
        - PulsarProducerController.java
      - dto
        - TestDto.java
      - service
        - PulsarProducer.java
      - PulsarProducerApplication.java
    - resources
      - application.yaml
 ```
 - PulsarProducerConfig.java

```java
@Component
@Configuration
public class PulsarProducerConfig {
    @Value("${topic-name}")
    private String topicName; // topic name

    @Value("${pulsar.service-url}")
    private String serviceURL; // pulsar service url

    public String getTopicName() {
        return topicName;
    }

    public void setTopicName(String topicName) {
        this.topicName = topicName;
    }

    public String getServiceURL() {
        return serviceURL;
    }

    public void setServiceURL(String serviceURL) {
        this.serviceURL = serviceURL;
    }
}
```

 - PulsarProducerController.java

```java
@RestController
@RequestMapping(value = "/user")
public class PulsarProducerController {
    @Autowired
    PulsarProducer pulsarProducer;

    @PostMapping(value = "/publish")
    public void sendMessage(@RequestParam("name") String name, @RequestParam("age") Integer age) throws PulsarClientException {
        // name과 age POST 요청
        pulsarProducer.sendMessage(name, age);
    }
}
```

 - TestDto.java

```java
public class TestDto {
    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String toString() {
        return "name : " + getName() + ", " + "age : " + getAge().toString();
    }
}
```

 - PulsarProducer.java

```java
@Service
public class PulsarProducer implements ApplicationRunner {
    @Autowired
    PulsarProducerConfig pulsarProducerConfig;

    private static PulsarClient pulsarClient;
    private static Producer<TestDto> producer;

    public void run(ApplicationArguments args) throws Exception {
        // pulsar 클라이언트 생성
        pulsarClient = PulsarClient.builder().serviceUrl(pulsarProducerConfig.getServiceURL()).build();
        // producer 생성
        producer = pulsarClient.newProducer(Schema.JSON(TestDto.class)).topic(pulsarProducerConfig.getTopicName()).create();
    }
    
    public void sendMessage(String name, Integer age) throws PulsarClientException {
        TestDto testDto = new TestDto();
        testDto.setName(name);
        testDto.setAge(age);
        System.out.println(testDto.toString());
        // TestDto 타입의 객체
        producer.send(testDto);
    }
}
```

 - PulsarProducerApplication.java

```java
@SpringBootApplication
public class PulsarProducerApplication {
    public static void main(String[] args) {
        SpringApplication.run(PulsarProducerApplication.class, args);
    }
}
```

 - application.yaml

```yaml
server:
  port: 8001

topic-name: "test-topic"

pulsar:
  service-url: "pulsar://{pulsar service url}:6650"
```

 ### Consumer 구현
 - 구조
 
 ```
 - src
  - main
    - java
      - config
       - PulsarConsumerConfig.java
      - dto
       - TestDto.java
      - service
       - PulsarConsumer.java
      - PulsarConsumerApplication.java
    - resources
      - application.yaml
 ```
 
 - PulsarConsumerConfig.java

```java
@Component
@Configuration
public class PulsarConsumerConfig {
    @Value("${topic-name}")
    private String topicName;

    @Value("${pulsar.service-url}")
    private String serviceURL;

    public String getTopicName() {
        return topicName;
    }

    public void setTopicName(String topicName) {
        this.topicName = topicName;
    }

    public String getServiceURL() {
        return serviceURL;
    }

    public void setServiceURL(String serviceURL) {
        this.serviceURL = serviceURL;
    }
}
```

 - TestDto.java

```java
public class TestDto {
    private String name; 
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String toString() {
        return "name : " + getName() + ", " + "age : " + getAge().toString();
    }
}
```

 - PulsarConsumer.java

```java
@Service
public class PulsarConsumer implements ApplicationRunner {
    @Autowired
    PulsarConsumerConfig pulsarConsumerConfig;

    private static PulsarClient pulsarClient;
    private static Consumer<TestDto> consumer;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        consumerPulsar();
    }

    public void consumerPulsar() throws Exception {
        // pulsar 클라이언트 생성
        pulsarClient = PulsarClient.builder().serviceUrl(pulsarConsumerConfig.getServiceURL()).build();
        // consumer 생성 (JSON 스키마 사용)
        consumer = pulsarClient.newConsumer(Schema.JSON(TestDto.class)).topic(pulsarConsumerConfig.getTopicName()).subscriptionName("epz-test").subscribe();
        while (true) {
            // 메시지 수신 (TestDto 타입)
            Message<TestDto> message = consumer.receive(); 
            try {
                // 수신한 메시지 출력
                System.out.println(message.getValue().toString());
                // 메시지 인식 여부 
                consumer.acknowledge(message);
            } catch (Exception e) {
                consumer.negativeAcknowledge(message);
            }
        }
    }
}
```

 - PulsarConsumerApplication.java

```java
@SpringBootApplication
public class PulsarConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(PulsarConsumerApplication.class, args);
    }
}
```

 - application.yaml

```yaml
server:
  port: 8002

topic-name: "test-topic"

pulsar:
  service-url: "pulsar://{pulsar service url}:6650"

```

 
 ### 테스트 결과
 
 아래 결과는 Producer와 Consumer 모두 실행 후, Post 요청을 한 결과입니다.
 - POST 요청 (Postman 활용)
![image](https://user-images.githubusercontent.com/87160438/149278266-b24bfb49-1df8-4c50-900d-6899ec1e9d9d.png)


 - Producer 로그
![image](https://user-images.githubusercontent.com/87160438/149278119-adc16bc4-28b0-4612-9091-aa25953c2abe.png)


 - Consumer 로그
![image](https://user-images.githubusercontent.com/87160438/149278170-f318506d-454d-4015-9c5e-7c045363488a.png)


 
 여기까지 Apache Pulsar 환경 구축, Spring Boot 기반의 간단한 테스트를 해봤습니다.
 
 추후에 클러스터 구성 및 성능 비교 테스트도 진행할 예정입니다.
  
 감사합니다.
