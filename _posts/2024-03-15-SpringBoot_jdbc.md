---
title: "SpringBoot jdbc 연동: DB 접속 비밀번호 암호화 방법"
last_modified_at: 2023-03-15
author: Do-soo, KIM
---


> 이번 포스팅에서는 SpringBoot 프로젝트에서 jdbc 연동 시, DB 접속 비밀번호 암호화를 적용하는 방법에 대해서 정리해 보려고 합니다.

### 개요
---

SpringBoot에서 jdbc를 연동하는 방법을 다양하겠지만 필자의 경우는 이런 식으로 하였습니다.

1. pom.xml에 dbms에 대한 dependency 추가

2. application.properties DB 커넥션 정보 추가

3. DataSourceConfig 클래스를 이용한 Datasource 설정

이런 방식으로 jdbc 연동 시, db 접속 정보 중 비밀번호는 application.properties 파일의 spring.datasource.password 키 값에 설정을 하게 되죠.

```
spring.datasource.password = 1234
```

보통 위와 같이 저렇게 대 놓고 비밀번호를 노출해서 사용하시죠?

누가 봐도 참 편치 않은 상황입니다.

저 비밀번호를 암호화해서 사용하는 방법이 있으니 걱정 안 하셔도 됩니다.

### 적용 방법
---

그럼 어떻게 하면 되는지 설명하겠습니다.

먼저 application.properties 파일에서 spring.datasource.password 항목은 삭제합니다.

이 값은 클래스에서 사용할 것이니까요.

바로 DataSourceConfig에서 직접 설정하도록 수정할 겁니다.

기존에는 아마 다음과 같이 코드가 되어 있을 겁니다.

```java
public DataSource dataSource() {
    return DataSourceBuilder.create().build();
}
```
여기에 암호화된 비밀번호를 복호화해서 값을 가져오도록 하는 코드를 추가해 주면 됩니다.

아래 수정된 코드를 보시면 됩니다.

```java
public DataSource dataSource() throws Exception {
    String decryptedPassword = Decryptor.decrypt("암호화된 비밀번호");
    return DataSourceBuilder.create().password(decryptedPassword).build();
}
```
위 코드에서 Decryptor라는 복호화 클래스는 필자가 사용하는 것이므로, 각자 사용하시는 걸로 수정하시면 됩니다.

이렇게 하면 비밀번호를 대 놓고 노출하지 않고 사용할 수 있게 됩니다.

그럼 여기서 마칠게요.

> Reference

- https://doozi0316.tistory.com/entry/Spring-Boot-MyBatis-MySQL-%EC%97%B0%EB%8F%99-%EB%B0%A9%EB%B2%95




