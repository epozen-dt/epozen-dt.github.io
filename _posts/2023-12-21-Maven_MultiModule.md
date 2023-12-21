---
title: "스프링부트 메이븐 멀티모듈 프로젝트 관리 방법"
last_modified_at: 2023-12-21
author: Do-soo, KIM
---


> 이번 포스팅에서는 스프링부트 메이븐 멀티 모듈 프로젝트를 생성하고 사용하는 방법을 정리해 보려고 합니다.

### 개요
---

여러 개의 스프링부트 프로젝트를 개발하다보면 각 프로젝트마다 클래스를 공통적으로 사용하는 경우들이 있을 겁니다.<br>
이런 경우, 각 프로젝트마다 공통적으로 사용하는 클래스를 각각 만들어 사용하게 되면, 프로젝트 관리 상 굉장히 비효율적이겠죠?<br>
공통 클래스를 별도의 공통 모듈로 만들어서 각 프로젝트에서 사용할 수 있도록 해 주면 좋을 것 같은데 이럴 때 멀티 모듈 프로젝트로 관리해 주면 됩니다.


그럼 Eclipse에서 멀티 모듈 프로젝트를 생성하는 방법을 알아볼까요?

아래 그림과 같이, 이름이 Multi인 최상위 Maven 프로젝트 아래, 2개의 하위 프로젝트가 있는 구조를 만들어 볼 겁니다. 하위 프로젝트를 이제부터는 모듈이라고 부르겠습니다.<br>
각 모듈의 이름은 CommonModule, Module1 이라고 할텐데, CommonModule의 클래스를 Module1에서 공통으로 사용하는 구조로 만들려고 하는거라서 편의상 이름을 이렇게 붙였습니다.

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/a4b79388-c008-495a-9300-1a8f4a599e58" width="50%" height="50%" />
</p>



#### 상위 프로젝트 생성

이름이 Multi인 상위 프로젝트는 다음과 같이 생성하면 됩니다.

- File --> New --> Maven Project 실행


<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/d806d40b-4361-4546-915e-670e2fab657a" width="80%" height="80%" />
</p>

- Next 버튼 클릭 


<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/1a59a106-57f2-437e-9e3d-051fd8af2b93" width="80%" height="80%" />
</p>


- Finish 버튼 클릭<br>
&emsp;아래와 같이 프로젝트가 생성됩니다.

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/6bd58e76-7394-4009-90e4-e9fd260db823" width="50%" height="50%" />
</p>


- pom.xml 파일 수정<br>
&emsp;아래의 내용을 추가합니다.


```xml
<packaging>pom</packaging>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.8</version> <!-- 사용하는 버전에 따라 표기 -->
    <relativePath/>
</parent>
```

- Maven -> Update Project 실행

#### 하위 모듈 생성

그럼 이제 하위모듈을 생성해 보겠습니다.

- Project Explorer에서 Multi Project 마우스 오른쪽 버튼 클릭 --> New --> Other --> Maven --> Maven Module 클릭

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/94a15b93-6d63-4421-b8a3-2165eb4b10ad" width="80%" height="80%" />
</p>

- Next 버튼 클릭

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/50b406a2-fb07-4d87-a145-ed24ccb831ae" width="80%" height="80%" />
</p>

- Finish 버튼 클릭<br>
&emsp;또 다른 모듈인 Module1도 동일하게 생성해 줍니다.<br><br>
&emsp;완료되면 다음과 같이 정상적으로 2개의 모듈이 추가된 걸 볼 수 있습니다.

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/0d5134df-5e8c-4309-9b14-08d76f0b968f" width="50%" height="50%" />
</p>

- pom.xml 파일 수정<br>
&emsp;각 모듈의 pom.xml 파일에 아래의 내용을 추가합니다.

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.project-lombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
		</plugin>
    </plugins>
</build>
```
- Maven -> Update Project 실행

### 프로젝트 실행

---

이제 멀티 모듈 프로젝트 생성이 완료되었으니, 코드를 작성해 보도록 하겠습니다.

SpringBoot를 실행할 Main 클래스가 필요합니다.<br>
CommonModuleApplication이라는 이름의 클래스를 생성하고 코드를 작성합니다. 

```java
package com.epozen.commonmodule;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class CommonModuleApplication {

	public static void main(String[] args) throws Exception {
		SpringApplication.run(CommonModuleApplication.class, args);
	}
}
```

이제 다른 모듈에서 사용할 공통 클래스를 하나 만들어 보겠습니다.<br>
CommonContoller라는 Controller 클래스를 생성하여 코드를 작성합니다.

```java
package com.epozen.commonmodule;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CommonController {

	@GetMapping("/test")
	public String msg(@RequestParam String param) {
		return param;
	}
}

```

한번 실행해 볼까요?

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/e6a9e35c-c12d-4a17-b696-f1ac37853aa2" width="80%" height="80%" />
</p>

이제 이렇게 만들어진 클래스를 다른 하위 모듈에서 사용하도록 코드를 추가해 보겠습니다.<br>
module1 모듈에서 이 클래스를 참조하도록 코드를 작성합니다.

위에서 했던 것처럼 Main Class를 생성하여 작성한 다음, CommonModule의 CommonController의 메소드를 호출하여 실행하는 SubController 클래스를 생성하여 코드를 작성합니다.

```java
package com.epozen.module1;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.epozen.commonmodule.CommonController;

@RestController
public class SubController {

	@GetMapping("/module")
	public String msg(@RequestParam String param) {
		CommonController commonController = new CommonController();
		commonController.msg(param);
		return param;
	}
}
```
그런데 CommonModule의 CommonController 클래스를 찾을 수 없어 코드에 빨간 줄들이 보이게 될 겁니다.<br>
CommonController 클래스를 사용할 수 있도록 다음과 같이 진행합니다.

우선, CommonModule의 jar를 추출합니다. 

- CommonModule의 jar 추출을 위해 Project Explorer에서 CommonModule Project 마우스 오른쪽 버튼 클릭 --> Run As --> Maven install 클릭

- Module1 프로젝트 classpath에 추가

- Project Explorer에서 Module1 Project에 마우스 오른쪽 버튼 클릭 --> Build Path --> Configure Build Path 클릭 --> Add External JARS 클릭

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/583c6faa-a105-40a7-a917-99bc537d24a8" width="80%" height="80%" />
</p>


- CommonModule 의 dependency 추가<br>
&emsp;Module1의 pom.xml을 열어서 다음과 같이 추가합니다.

```xml
    <dependency>
        <groupId>com.epozen</groupId>
        <artifactId>commonmodule</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
```
이제, 코드의 빨간 줄들이 다 사라지고 실행할 준비가 되었습니다.
그럼 실행해 볼까요?


<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/b5ddb46e-149b-46e2-b832-6ba7671508ad" width="80%" height="80%" />
</p>

잘 실행되었네요.

Maven 멀티 모듈 프로젝트를 생성해 보고, 어떻게 사용하면 되는지 알아보았습니다.


### Reference
---
- https://codegear.tistory.com/23
- https://flowlog.tistory.com/73



