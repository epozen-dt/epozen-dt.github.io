---
title: "Spring-Rest-Docs"
last_modified_at: 2024-10-31
author: 김혜원
---

Spring Rest Docs에 대한 포스팅입니다.

---

&nbsp;

## Spring Rest Docs?

Spring Rest Docs는 자바에서 사용하는 Rest API 문서화 도구 중 하나로,
테스트 기반의 문서 생성 방식을 사용하여, API의 스펙이 실제 코드와 동기화되도록 합니다.
중요한 점은 문서에서 사용할 스니펫(snippets)을 얻기 위하여 테스트 코드가 반드시 있어야 한다는 것입니다. 

보통 Spring MockMvc 테스트와 JUnit을 활용하여 API 문서를 생성하며,
테스트 코드가 성공적으로 실행되면 스니펫(snippet) 형태의 문서 조각이 생성되어 최종 문서에 포함됩니다.
이는 API 변경 시 테스트와 문서가 함께 유지되도록 하여 문서의 신뢰성을 높여줍니다.

Spring Rest Docs를 통해 자동으로 문서화되는 기능을 통해 개발자는 API 문서를 수동으로 작성하는 번거로움을 덜고, 
실제 동작하는 코드 기반의 신뢰성 있는 문서를 생성할 수 있습니다.

&nbsp;

### 장점

1. 다양한 포맷 지원 (유지보수 용이)
    - AsciiDoctor 및 Markdown과 같은 포맷을 지원하여, 텍스트 기반으로 문서를 관리하고 커스터마이징할 수 있습니다. 

2. API 변경에 따른 자동 업데이트
    - 테스트 코드를 통해 API가 변경될 때마다 문서도 자동으로 업데이트되므로, 문서가 항상 최신 상태로 유지됩니다.

3. 높은 가독성
    - 문서 템플릿에 따라 필요한 정보를 구조화하여 높은 가독성을 제공하며, 읽는 사람이 쉽게 이해할 수 있는 문서를 제공합니다.


&nbsp;

### 예시 및 결과


    < 의존성 추가 >

            <!-- Spring REST Docs -->
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-mockmvc</artifactId>
            <scope>test</scope>
        </dependency>

        .
        .
        .
    
        <!-- Asciidoctor Maven Plugin for converting to HTML or PDF -->
        <plugin>
            <groupId>org.asciidoctor</groupId>
            <artifactId>asciidoctor-maven-plugin</artifactId>
            <version>2.0.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>process-asciidoc</goal>
                    </goals>
                    <phase>prepare-package</phase>
                    <configuration>
                        <sourceDirectory>${project.basedir}/src/docs/asciidoc</sourceDirectory>
                        <outputDirectory>${project.build.directory}/generated-docs</outputDirectory>
                        <resources>
                            <resource>
                                <directory>
                                    ${project.build.directory}/generated-docs
                                </directory>
                            </resource>
                        </resources>
                    </configuration>
                </execution>
            </executions>
        </plugin>

&nbsp;

예시는 @AutoConfigureMockMvc와 주어진 url 및 파라미터 값을 이용하여 객체를 생성하는 테스트 코드입니다.


![img3](https://github.com/user-attachments/assets/6681ae68-49c4-4ce4-8b34-779b1554fb4d)

&nbsp;

<실행 결과>

![img1](https://github.com/user-attachments/assets/fe90b124-ae14-406f-bc26-40e8a77019e4)
![img2](https://github.com/user-attachments/assets/8453e857-c384-48f8-96df-c9f8ee33f3f7)

&nbsp;

응답 결과를 문서에서 확인하면 아래와 같습니다.


![image](https://github.com/user-attachments/assets/fa2f4493-ffe2-4e8e-98a0-2d2c1193cdd4)

&nbsp;

< 사용된 버전 >

* java 17
* spring 6
* sprint rest docs 2.0.0

&nbsp;

------
> 참고

[1] [Spring Rest Docs 적용](https://techblog.woowahan.com/2597/)

[2] [내가 만든 API를 널리 알리기 - Spring REST Docs 가이드편](https://helloworld.kurly.com/blog/spring-rest-docs-guide/)
