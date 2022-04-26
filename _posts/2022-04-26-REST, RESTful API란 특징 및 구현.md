---
layout: post
title: "REST API"
date: 2022-4-26
author: zeuskwon
---



# REST, RESTful API란 특징 및 구현

이번 포스팅은 HTTP 프로토콜을 사용할 때 필요한 REST API에 대해서 포스팅 하겠습니다.



# REST란?

### REST 기본 개념

- 자원을 이름으로 구분하여 해당 자원의 상태(정보)를 주고 받는 모든 것을 의미합니다
- WWW(World Wide Web)과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 개발 아키텍처의 한 형식입니다
- 네트워크 상에서 Client와 Server 사이의 통신 방식 중 하나이다

### REST 구체적인 설명

HTTP URI(Uniform Resource Identifier)를 통해 자원(Resource)를 명시하고, HTTP Method(POST, GET, PUT, DELETE)를 통해 Resource를 처리하도록 설계된 아키테처를 의미합니다. 

- CRUD Operation

CRUD는 대부분의 컴퓨터 소프트웨어가 가지는 기본적인 데이터 Create(생성), Read(읽기), Update(갱신), Delete(삭제)를 묶어서 일컫는 말입니다.

​	- Create : 생성(POST)

​    \- Read : 조회(GET)

​    \- Update : 수정(PUT)

​    \- Delete : 삭제(DELETE)

​    \- HEAD : header정보 조회(HEAD)

### REST 구성 요소

​	1. **자원(Resource) : URI**

-    Client는 URI를 이용해서 자원을 지정하고 해당 자원의 상태(정보)에 대한 조작을 Server에 요청합니다

​	2. **행위(Verb): HTTP Method**

-    HTTP 프로토콜은 GET, POST, PUT, DELETE롸 같은 메서드를 제공합니다

​	3. **표현(Representation of Resource)**

-    \REST에서 하나의 자원은 JSON, XML, TEXT, RSS등 여러 형태의 Representation으로 나타내어 질 수 있습니다

### REST의 장단점

#### 장점

- 서버와 클라이언트의 역할을 명확하게 분리합니다
- HTTP 프로토콜의 인프라를 그대로 사용하므로 REST API 사용을 위한 별도의 인프라를 구축할 필요가 없습니다
- HTTP 프로토콜의 표준을 최대한 활용하여 여러 추가적인 장점을 함께 가져갈 수 있게 해줍니다
- HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능합니다
- Hypermedia API의 기본을 충실히 지키면서 범용성을 보장합니다
- 여러 가지 서비스 디자인에서 생길 수 있는 문제를 최소화합니다

#### 단점

- 표준이 존재하지 않는다.
- 사용할 수 있는 메소드(HTTP Method 형태)가 제한적입니다
- 구형 브라우저가 아직 제대로 지원해주지 못하는 부분이 존재합니다(PUT, DELETE)

### REST의 특징

1. Server-Client(서버-클라이언트 구조)
2. Stateless(무상태)
3. Cacheable(캐시 처리 가능)
4. Layered System(계층화)
5. Uniform Interface(인터페이스 일관성)

### REST를 사용하는 이유

- 애플리케이션 분리 및 통합
- 다양한 클라이언트의 등장
- 다양한 브라우저와 안드로이드, 아이폰과 같은 여러 종류의 디바이스 통신 필요성

REST가 나온 이유는 확장성과, 분산시스템, 데이터 소모량 감소입니다. 초기에는 브라우저만 있었으니 크게 상관없었지만 다양한 애플리케이션 클라이언트이 등장하면서 REST에 관심을 가지게 되었습니다.



# REST API란?

### REST API의 개념

#### API(Application Programming Interface)의 정의

- 응용 프로그램에서 사용할 수 있도록, 운영체제나 프로그래밍 언어가 제공하는 기능을 제어할 수 있게 만든 인터페이스를 뜻한다. – 위키백과
- 쉽게 말해서 컴퓨터 프로그램간 상호작용을 촉진하여, 서로 정보를 교환 가능하도록 하는 것입니다

#### REST API의 정의

- REST기반으로 서비스 API를 구현한 것입니다
- 최근 OpenAPI(누구나 사용할 수 있도록 공개된 API: 구글맵, 공공데이터 등), 마이크로 서비스(하나의 큰 애플리케이션을 여러 개의 작은 애플리케이션으로 쪼개어 변경과 조합이 가능하도록 만든 아키텍처) 등을 제공하는 업체 대부분은 REST API를 제공합니다

#### REST API특징

사내 시스템들도 REST 기반으로 시스템을 분산해 확장성과 재사용성을 높여 유지보수 및  운용을 편리하게 할 수 있게 합니다. 

REST는 HTTP표준을 기반으로 구현하므로, HTTP를 지원하는 프로그램 언어로 클라이언트, 서버로 구현할 수 있습니다.

### REST API설계

#### REST API 설계 규칙

기억해야할 원칙은 두 가지라고 합니다.

> 첫 번째 : URI는 정보의 자원을 표현
>
> 두 번째 : 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE)로 표현

​    

##### **<규칙>**

- URI는 명사를 사용
- 슬래시(/)로 계층 관계를 표현
- URI의 마지막에는 슬래시(/)를 붙이지 않음
- URI는 소문자로만 구성
- 가독성이 떨어지는 경우 하이픈(-)을 사용
- 밑줄(_)은 URI에 사용하지 않음
- 행위는 URL에 작성하지 않음
- 경로 부분 중 변하는 부분은 유일한 값으로 대체 (즉, :id는 하나의 특정 resource를 나타내는 고유값이다)
  - ​    Ex) student를 생성하는 route: POST/students
  - ​    Ex) id=12인 student를 삭제하는 route: DELETE/students/12

- resource의 도큐먼트 이름으로는 단수 명사를 사용
- resource의 컬렉션 이름으로는 복수 명사를 사용
- resource의 스토어 이름으로는 복수 명사를 사용

**예시)**

- #### 마지막에 "/" 포함하지 않는다.

  Bad

  ```http
  http://api.test.com/users/
  ```

  Good

  ```http
  http://api.test.com/users
  ```



- #### _(underbar) 대신 -(dash)를 사용한다.

  (dash)의 사용도 최소한으로 설계하지만 정확한 의미나 표현을 위해 단어의 결합이 불가피한 경우  -(dash)를 사용한다

  Bad

  ```http
  http://api.test.com/users/post_commnets
  ```

  Good

  ```http
  http://api.test.com/users/post-commnets
  ```

#### 

- #### 소문자를 사용한다.

  Bad

  ```http
  http://api.test.com/users/postCommnets
  ```

  Good

  ```http
  http://api.test.com/users/post-commnets
  ```



- #### 행위(method)는 URL에 포함하지 않는다.

  Bad

  ```http
  POST http://api.test.com/users/1/delete-post/1
  ```

  Good

  ```http
  POST http://api.test.com/users/1/posts/1
  ```



- #### 컨트롤 자원을 의미하는 URL 예외적으로 동사를 허용한다.

  함수처럼, 컨트롤 리소스를 나타내는 URL은 동작을 포함하는 이름을 짓는다.

  Bad

  ```http
  http://api.test.com/posts/duplicating
  ```

  Good

  ```http
  http://api.test.com/posts/duplicate
  ```



##### REST API설계

![캡처](https://user-images.githubusercontent.com/102636902/165222669-5858a0b6-fef9-4ffb-b6a6-1d7856e8f56a.PNG)


##### 응답 상태 코드 

- 1XX : 전송 프로토콜 수준의 정보 교환
- 2XX : 클라이언트 요청이 성공적으로 수행됨
- 3XX : 클라이언트는 요청을 완료하기 위해 추가적인 행동을 취해야 함
- 4XX : 클라이언트의 잘못된 요청
- 5XX : 서버쪽 오류로 인한 상태코드



##### REST API개발 예제

아래 코드는 URI에서 {id}에 해당하는 부분을 매개변수 값으로 보겠다는 것입니다

  ![그림입니다. 원본 그림의 이름: CLP000083b8040e.bmp 원본 그림의 크기: 가로 717pixel, 세로 120pixel](file:///C:\Users\epozen\AppData\Local\Temp\tmpDEC4.jpg)  

## RESTful API

##### RESTful이란

- REST API를 제공하는 웹서비스를 나타내기 위한 용어입니다
- REST API를 제공하는 웹서비스를 “RESTful”하다고 할 수 있습니다
- RESTful은 REST를 REST답게 쓰기 위한 방법으로, 누군가가 공식적으로 발표한 것은 아닙니다

##### RESTful API 목적

- 이해하기 쉽고 사용하기 쉬운 REST API를 만드는 것입니다
- RESTful API를 구현하는 근본적인 목적이 성능 향상에 있는 것이 아니라 일관적인 컨벤션을 통한 API의 이해도 및 호환성을 높이는 것이 주 동기이니, 성능이 중요한 상황에서는 굳이 RESTful한 API를 구현할 필요는 없습니다

