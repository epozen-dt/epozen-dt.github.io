---
title: "Springboot Application에서 Kafka Schema-Registry 사용하기"
last_modified_at: 2021-10-20
author: Do-soo, KIM

---

이번 포스팅에서는 Springboot Applcation에서 Kafka Schema-Registry를 사용하기 위해서 설정하는 방법을 알아보고자 합니다.

여기서는 maven을 기준으로 설명할 것이며, 혹시 gradle을 이용하시려면 다른 자료를 참조하시면 됩니다.

프로젝트 생성 후, pom.xml 파일에 몇 가지만 추가하면 적용할 수 있습니다.

다음은 pom.xml에 관련 설정을 추가한 예시입니다.

주석으로 넘버링한 부분이 관련하여 추가한 내용입니다.(1번부터 5번까지 넘버링 되어 있습니다.)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/> 
    </parent>
    <groupId>io.confluent.developer</groupId>
    <artifactId>kafka-avro</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>kafka-avro</name>
    <description>Demo project for Spring Boot</description>
<properties>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.confluent</groupId>
        <!-- <1> -->
        <artifactId>kafka-schema-registry-client</artifactId>                                   
        <version>5.3.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.avro</groupId>
        <!-- <2> -->
        <artifactId>avro</artifactId>                                                           
        <version>1.8.2</version>
    </dependency>
    <dependency>
        <groupId>io.confluent</groupId>
        <!-- <3> -->
        <artifactId>kafka-avro-serializer</artifactId>                                          
        <version>5.3.0</version>
    </dependency>
    <dependency>
        <groupId>io.confluent</groupId>
        <artifactId>monitoring-interceptors</artifactId>
        <version>5.3.0</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

        <plugin>
            <groupId>org.apache.avro</groupId>
            <artifactId>avro-maven-plugin</artifactId>
            <version>1.8.2</version>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>schema</goal>
                    </goals>
                    <configuration>
                        <!-- <4> -->
                        <sourceDirectory>${project.basedir}/src/main/avro</sourceDirectory>     
                        <outputDirectory>${project.build.directory}/generated-sources</outputDirectory>
                        <stringType>String</stringType>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>

<repositories>
    <repository>
        <!-- <5> -->
        <id>confluent</id>                                                                      
        <url>https://packages.confluent.io/maven/</url>
    </repository>
</repositories>
</project>
```

