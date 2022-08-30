---
title: "Spring Boot: custom 라이브러리 추가하여 maven 빌드하기"
last_modified_at: 2022-08-30
author: Do-soo, KIM
---

### 1. 개요


우리가 Spring Boot 프로젝트를 개발할 때 다른 라이브러리로부터 클래스를 사용하게 된다면 Maven 저장소에 등록된 라이브러리(.jar 파일)을 사용하는 경우가 있을테고 
또 어떤 경우는 자신이 직접 만든 라이브러리나 또는 오픈 소스로 제공되는 라이브러리(이하 포스팅에서는 custom 라이브러리라고 하겠습니다.)를 사용하여 개발하는 경우도 있을 겁니다.

사실 Maven 저장소에 등록된 라이브러리를 사용하는 경우라면 간단히 pom.xml 파일에 의존성만 추가해 주면 간단하겠지만, 
custom 라이브러리를 사용하게 된다면 보통 아래 그림과 같이 프로젝트 속성에서 해당 라이브러리를 추가하는 방법을 사용합니다.

![image](https://user-images.githubusercontent.com/92565548/187381816-fad53951-4901-4bd9-9b50-bee1dae465c0.png)

이 그림은 이클립스에서 설정하는 화면입니다.

물론, 이렇게 하여 개발하고 테스트하면 아무 문제가 없지만 문제는 바로 이 프로젝트 빌드 후에 배포했을 경우 입니다. 

해당 프로젝트를 Executable jar로 build하고 이 jar 파일을 실행해 보았는데 runtime 중에 custom 라이브러리로부터 사용된 클래스를 찾지 못하는 오류가 발생하였습니다.

결국엔 이 문제를 해결하기 위한 방법에 대해서 이번 포스팅에서 소개하고자 합니다.

---

### 2. 방법

Spring Boot 기반에서 Maven build tool을 사용한다면, spring-boot-maven-plugin을 반드시 사용하게 되어 있습니다.

이 plugin을 사용할 때는 프로젝트에 custom 라이브러리를 추가하려면 includeSystemScope 옵션을 반드시 활성화해 주어야 합니다.

pom.xml 파일에서 다음과 같이 추가해 주면 됩니다.

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <configuration>
    <includeSystemScope>true</includeSystemScope>
  </configuration>
</plugin>
```

includeSystemScope 옵션을 true로 설정하여 활성해 주는 것입니다.

이제 다음 단계로, 추가할 라이브러리를 직접 프로젝트 폴더에 복사해 주는 겁니다. 

maven 저장소 역할을 하는 다른 저장소를 하나 만들어 주는 것이라고 생각하시면 쉽게 이해가 되실 겁니다.

라이브러리를 따로 저장할 폴더를 생성하여 그 폴더로 추가할 라이브러리 파일을 복사하면 됩니다. 

보통은 jar 파일을 많이 사용하시겠죠?

저는 프로젝트 폴더 내 src/main/resource 아래애 libs라는 폴더를 만들고 사용할 jar 파일을 저장해 두었습니다.

jar 파일이 저장된 이 절대 경로는 다음 단계에서 사용됩니다.

이제, 마지막으로 다음과 같이 pom.xml 파일에 추가한 라이브러리에 대한 의존성을 추가해 줍니다. 

여기서 중요한 것은 바로 scope 옵션은 반드시 system이라고 설정해 주어야 합니다.

```xml
<dependency>
    <groupId>epz.custom</groupId>
    <artifactId>myLibrary</artifactId>
    <version>4.0.1</version>
    <scope>system</scope>
    <systemPath>${pom.basedir}/src/main/resources/libs/myLibrary-4.0.1.jar</systemPath>
</dependency>
```

systemPath 옵션에 jar 파일을 저장한 절대경로를 적어주는 것입니다.

제가 앞에서 말씀드렸죠? 절대경로가 사용될 것이라고...^^

이제 모든 준비를 마쳤네요.

이제, build한 excutable jar 파일을 실행하여도 runtime 오류 없이 잘 돌아가게 됩니다.^^

이번 포스팅은 여기서 마치겠습니다.


### References

https://sup2is.tistory.com/53
