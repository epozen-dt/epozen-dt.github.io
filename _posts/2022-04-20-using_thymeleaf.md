---
title: "Thymeleaf(타임리프) 기본 문법 사용하기"
last_modified_at: 2022-04-20
author: Do-soo, KIM

---

이번 포스팅은 스프링부트 기반의 웹 서비스 개발에 사용하는 thymeleaf에 대해 소개하고, 기본 문법을 사용해 보고자 합니다.

thymeleaf는 템플릿 엔진의 한 종류인데, 그렇다면 템플릿 엔진에 대해 먼저 살펴봐야 겠죠?


템플릿 엔진이란 템플릿 프로세서(Templete Processor)를 사용하여 웹 템플릿(Web Templete)를 결합하여 완성된 웹페이지를 만들어 내는 시스템입니다. 

템플릿 양식과 특정 데이터 모델에 따른 입력 자료를 결합하여 원하는 결과 문서를 출력하는 소프트웨어(또는 컴포넌트)를 말합니다. 그 중 웹 템플릿 엔진(Web Templete Engine)이란 웹 문서가 출력되는 엔진을 말합니다. 즉, 웹 템플릿 엔진은 웹 템플릿들과 웹 컨텐츠 정보를 처리하기 위해 설계된 소프트웨어입니다.  웹 템플릿 엔진은 view(html) 와 Data Logic(DB Connection)을 분리해 주는 기능을 합니다.




보통 자바에서 웹 개발시 JSP(Java Server Page)를 사용하는데 JSP를 사용하면 <% %>형태의 스크립트릿을 사용하여 개발하게 됩니다. 그러나 이 방식은 스크립트릿과 Html이 혼재된 상태가 되고 html태그의 반복적인 사용으로 인해 수정하기 어려운 상황이 됩니다. 이러한 문제 때문인지 스프링 부트 공식 문서에서도 JSP 사용을 권장하지 않고 있습니다. 

아래 공식 문서 내용을 참고하세요. 

![springboot](https://user-images.githubusercontent.com/92565548/164140841-3310aa49-e18e-46eb-8d2b-53f6f0301384.png)

부족한 제 영어실력으로 대충 해석해 보자면, "알려진 제한 사항들이 있으니 가능한 한 JSP를 사용하지 마라." 이런 말입니다. 


그래서 우리는 JSP를 저 멀리하고 스프링 부트에서 권장하는 템플릿 엔진 중에서 Thymeleaf에 대해서 다뤄보려고 합니다.



### 1. 소개



웹 서비스 설계 시, 기본적으로 MVC 패턴을 많이 사용합니다. 

MVC 패턴은 서비스의 역할을 Model, View, Controller로 나누어 이들 사이의 상호 작용을 통제하는 디자인 패턴입니다.(MVC 패턴에 대해서는 시간이 되면 포스팅하도록 노력해 보겠습니다.ㅠㅠ) 

이 중 View는 사용자가 보는 화면, 즉 Contoller가 사용자에게 보내주는 것에 해당합니다. 위에서 언급한 템플릿 엔진이 바로 이러한 View를 만들어 줍니다. 

Thymeleaf로 View를 만들기 위한 기본 문법을 이 포스팅을 통해 사용해 보려고 하는 것입니다.

Maven 기준으로 스프링 부트에서 thymeleaf를 사용하려면, pom.xml에 의존성 추가만 해 주면 됩니다.


```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId> 
</dependency> 
```

그리고 thymeleaf 문법을 사용할 html 파일에 다음과 같이 태그를 추가해 주면 됩니다.


```html
<html xmlns:th="http://www.thymeleaf.org">
```


### 2. 기본 문법 사용하기


사용할 준비를 마쳤으니, 이제 한번 사용해 볼까요?


#### 2.1 반복문

코딩을 하다보면 for문 같은 반복문을 많이 사용하게 되는데요. thymeleaf도 반복문을 제공합니다. 

thymeleaf에서의 반복문 키워드는 다음과 같습니다.

```html
th:each="item : ${items}"
```
Controller로부터 전달된 items라는 List 데이터를 반복하면서 뭔가를 하겠다는 코드 입니다.

아래 예제 코드를 보면서 더 자세히 살펴볼까요?

우선 설명을 위해 아래와 같이 controller를 하나 만들어 두겠습니다.


```java
@GetMapping("...")
public String ...(Model model) {
    addUser(model);
    return "...";
}

private void addUser(Model model) {
    List<User> list = new ArrayList<>();
    list.add(new User("userA", 10));
    list.add(new User("userB", 20));
    list.add(new User("userC", 30));

    model.addAttribute("users", list);
}

@Data
static class User {
    private String username;
    private int age;

    public User(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

controller에서 users라는 List 데이터를 model을 통해 view로 전달합니다.

View에서는 controller로부터 전달받은 list 데이터를 반복하면서 테이블을 생성하고 데이터를 넣어주는 기능을 만들어 보겠습니다.

코드는 아래와 같습니다.

```html
<table border="1">
    <tr>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user : ${users}">
        <td th:text="${user.username}">username</td>
        <td th:text="${user.age}">0</td>
    </tr>
</table>
```

별로 어렵지 않죠?


#### 2.2 조건문

반복문 못지 않게 많이 사용하는 문법이 if/else 등의 조건문이죠? 조건문은 어떻게 쓰는지 살펴 보죠.

조건문 키워드는 th:if, th:unless, th:switch가 있습니다.

아래 예제 코드는 th:if와 th:else를 사용한 코드입니다.



```html
<table border="1">
    <tr>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user : ${users}">
        <td th:text="${user.username}">username</td>
        <td>
            <span th:text="${user.age}">0</span>
            <span th:text="'미성년자'" th:if="${user.age lt 20}"></span> 
            <span th:text="'미성년자'" th:unless="${user.age ge 20}"></span> 
        </td>
    </tr>
</table>
```

나이 < 20 이면(th:if="${user.age lt 20}) '미성년자', 나이  >= 20 이지 않으면(th:unless="${user.age ge 20}) '미성년자'라고 표시해 주는 겁니다. 

th:if는 우리가 통상 알고 있는 if와 동일하고 th:unless는 if not이라고 보시면 됩니다.

다음은 th:switch 사용 예제입니다.

```html
<table border="1">
    <tr>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user : ${users}">
        <td th:text="${user.username}">username</td>
        <td th:switch="${user.age}"> 
            <span th:case="10">10살</span> 
            <span th:case="20">20살</span>
            <span th:case="*">기타</span> 
        </td>
    </tr>
</table>
```

일반적인 switch~case문과 동일합니다. 

'th:case="*"' 문은 case: default와 동일한 구문입니다.


#### 2.3 자바스크립트 Inline(인라인)

사실 이 포스팅을 작성하는 진짜 목적은 바로 이 부분입니다.

위에서 설명했던 문법들은 모두 HTML 문서 내에 삽입되어 작성하게 되는데, 사실 지난 번 jQuery 사용 관련 포스팅에서 잠깐 언급했지만,
퍼블리싱 담당자가 따로 있다면 HTML 파일 자체는 그대로 두고, 웹 동적 구현은 모두 자바스크립트로 따로 만드는 것이 소스 관리 측면에서는 효율적입니다.

다행히도 Thymeleaf 에서도 위와 같은 문법을 자바스크립트에서 사용할 수 있도록 제공해주고 있는데 이것이 바로 Inline 기능입니다. 

Inline 기능을 사용하려면 다음과 같이 선언해 주면 됩니다.

```html
<script th:inline="javascript">
    
</script>
```

그럼 Inline 자바스크립트를 사용하여 th:each 반복문 구현 방법을 살펴보겠습니다.

```html
<script th:inline="javascript">
    [# th:each="user : ${users}"] 
        var tr = document.createElement("tr");
        table.appendChild(tr);
        td.innerHTML = /*[[${user.username}]]*/;
        tr.append(td);
        td = document.createElement("td");
        td.innerHTML = /*[[${user.age}]]*/;
        tr.append(td);
    [/]
</script>
```

위 코드는 위에서 살펴보았던 th:each 예제 코드와 동일한 결과입니다.

자바스크립트 Inline을 사용하면 코드가 다소 길어지긴 하는데, 코드 관리 측면에서는 장점이 있으니 프로젝트 진행 상황에 따라 적절한 방법을 선택하여 사용하시면 될 것 같습니다.

여기까지해서, Thymeleaf에 대한 기본 문법을 살펴보았는데요. 제가 프로젝트 진행 중에 자주 쓰게 되는 문법 위주로만 소개하였습니다. 
좀 더 많은 문법을 공부하실 분들은 구글링 해보면 많은 자료가 있으니 한번 참고해 보세요.

그럼 이번 포스팅은 여기서 마치겠습니다.

### References

```
https://blog.naver.com/PostView.naver?blogId=donny1848&logNo=222328779811&parentCategoryNo=&categoryNo=41&viewDate=&isShowPopularPosts=true&from=search
https://shallow-learning.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-7-Spring-MVC-View-%EB%A7%8C%EB%93%A4%EA%B8%B0-Thymeleaf
https://velog.io/@won-developer/Thymeleaf-%EA%B8%B0%EB%B3%B8%EA%B8%B0%EB%8A%A5-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B02%ED%8E%B8#-%EB%8B%A4%EB%A3%A8%EB%8A%94-%EA%B8%B0%EB%8A%A5%EB%93%A4
```