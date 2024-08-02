---
title: "Java로 Solace SEMP 요청"
last_modified_at: 2024-07-30
author: Sun-woong, Kim
---

Solace SEMP v1을 요청하여 XML 데이터를 가져오는 방법에 대해 공유합니다.

# 개요

Solace SEMP(Solace Element Management Protocol)는 Solace 정보를 가져오기 위해서 사용됩니다.

## 환경

Spring Boot 3.x

httpClient5 5.3.1

## 구현

RestTemplate 환경 설정 
```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        factory.setConnectTimeout(3000); // 연결시간초과, ms
        CloseableHttpClient httpClient = HttpClientBuilder.create()
            .setConnectionManager(PoolingHttpClientConnectionManagerBuilder.create()
                .setMaxConnPerRoute(10) // (IP, PORT) 1쌍에 대한 connection 수
                .setMaxConnTotal(100) // connection pool 적용
                .build())
            .build();

        factory.setHttpClient(httpClient); // HttpClient 세팅

        return new RestTemplate(factory);
    }
}
```

<br>

SEMP 요청 코드 구현

```java
@Service
@RequiredArgsConstructor
public class SEMPService() {
    private final RestTemplate restTemplate;
    
    public void call() {
        HttpHeaders headers = new HttpHeaders();
        headers.setBasicAuth("연결 ID", "연결 PW");
        headers.setContentType(MediaType.APPLICATION_XML);

        String body = "<rpc><show><queue><name>*</name><detail></detail></queue></show></rpc>";
        HttpEntity<String> httpEntity = new HttpEntity<>(body, headers);

        String url = "Solace 설치된 서버 URL";
        ResponseEntity<String> response = restTemplate.exchange(url, HttpMethod.POST, requestEntity, String.class);
    }
}

```

이상입니다.
