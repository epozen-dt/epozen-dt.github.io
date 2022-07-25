---
title: "jQuery에 대해서 알아보기"
last_modified_at: 2022-03-25
author: Do-soo, KIM

---

이번 포스팅은 프론트엔드 개발자라면 접하게 되는 jQuery에 대해 소개하고자 합니다.

jQuery를 소개하기에 앞서 자바스크립트에 대해서 잠깐 언급하고 가겠습니다. 

만약 자바스크립트가 없다면 모든 웹 페이지는 정적일 수 밖에 없습니다. 미리 어떻게 표시될지 정해진 텍스트를 그대로 표시만 하기 때문이죠.

웹 브라우저가 표시 중인 내용을 변경하는 유일한 방법은 새로운 URL을 웹 서버에 요청해서 새 문서를 받아오는 방법이었는데 이 과정은 아주 느릴 뿐만 아니라, 자원도 낭비합니다.

로그인 계정과 비밀번호를 form에 입력하면 입력한 데이터를 서버에 보내서 승인된 사용자인지 확인한 다음, 그렇지 않은 경우라면 서버가 오류 메시지가 들어 있는 페이지를 리턴하는 과정이 되는 것입니다. 
 

1993년 모자익 웹브라우저를 만들었던 마크 인드리슨은 넷스케이프라는 회사를 창립하고, 1994년 넷스케이프 내비게이터 브라우저를 출시하였는데 보다 대화형에 가까운 웹 페이지가 필요하다는 사실을 인식하고, 1995년 자바스크립트라는 프로그래밍 언어를 소개하게 하였습니다. 

자바스크립트는 그 후 에크마 인터내셔널이라는 표준 위원회에 의해 ECMA-262로 표준화 되었습니다. 

그래서 자바스크립트가 에크마스크립트 라고도 알려져 있고, 줄여서 EC라고도 부르게 됩니다.

자바스크립트는 Core 문법, Core 라이브러리, BOM(Browser Object Model), DOM(Document Object Model)으로 구성되어 있습니다.

이 중, DOM이 jQuery와 관련이 있다고 할 수 있는데 바로 jQuery는 자바스크립트 DOM을 보다 쉽게 사용하게 해 주는 라이브러리라고 보면 됩니다.

jQuery 코드 내부는 자바스크립트 코어 문법과 자바스크립트 DOM으로 구성되어 있습니다.

변수 선언이나 반복문은 자바스크립트 기본 문법이고, getelementByID나 .style 등은 DOM 기능과 관련된 것들입니다.

이처럼 jQuery는 특정 기능을 하는 처리 코드를 포장만 하고 있을 뿐, 노드를 찾는다거나 스타일을 변경하는 중요한 작업은 모두 jQuery 내부의 자바스크립트 DOM이 처리합니다.

그렇기 때문에, 먼저 DOM에 대해서 간단히 소개 하겠습니다.

### 1. DOM 이란?

DOM은 W3C에서 개발하고 있는 프로그래밍 규격으로 Document의 내용, 구조, 스타일 등을 접근하여 동적으로 수정, 추가, 삭제 등을 할 수 있도록 프로그램으로 제공하는 언어체계입니다.

*W3C(Wolrd Wide Web Consortium): 월드 와이드 웹을 위한 표준을 개발하고 장려하는 조직

우리가 잘 아는 HTML 문서는 바로 DOM이 계층 구조로 구성된 것입니다. 

웹브라우저는 웹페이지라고 하는 HTML문서를 로딩하여 DOM 객체를 생성하게 됩니다. 

개발자는 이렇게 만들어진 DOM 객체에 접근하여 동적인 효과를 구현하는 작업을 하는 것입니다. 

결국 DOM은 'Document Object Model'이라는 풀네임 그대로 해석해 보자면 'HTML 문서(Document) 정보를 자바스크립트 객체(Object)의 형태로 만든 데이터(Model) 구조' 정도로 정의해 볼 수 있습니다.

DOM의 핵심 객체 구조는 아래 그림과 같이 tree 형태로 구성됩니다.

![image](https://user-images.githubusercontent.com/92505439/160043347-f1968de0-fe32-47eb-bf97-46a8ea72f0ef.png)

[DOM 객체 구조, 출처: https://ny-kim.tistory.com/m/7]

### 2. 자바스크립트와 jQuery

그럼, DOM을 다루는 데에 있어서 자바스크립트와 jQuery가 어떻게 다른지 살펴보도록 하죠! 

HTML 문서를 하나 만들어볼까요?

```html
<html>
    <head>
    </head>
    <body>
        <div id="content">
        </div>
    </body>
</html>
```

많이 익숙한 코드이니, 따로 설명은 필요 없겠죠?

이 HTML 문서를 웹브라우저에서 보면 아무 것도 표시되지 않는 흰 화면만 보일 것입니다.

이 문서에 "My name"이라는 텍스트를 표시하고 싶다면 어떻게 하면 될까요?

```html
<html>
    <head>
    </head>
    <body>
        <div id="content">
        My name
        </div>
    </body>
</html>
```
물론, 위 코드처럼 HTML 문서를 수정하면 됩니다. 

하지만, 퍼블리싱 개발자와 협업 시에 HTML 파일을 수정하게 되면 소스 파일 관리에 어려움이 있지 않을까요? 

그래서 자바스크립트 라는 것이 필요하게 된거겠죠? HTML 파일은 건드리지 않고 독립적으로 관리하면 편하니까요.^^

자 그럼 이걸 자바스크립트로 어떻게 하면 될까요?

```
<script>
    var content = document.querySelector('#content');
    var newText = document.createTextNode('My name'); 
    content.appendChild(newText);
</script>
```
네.. 익숙한 코드라 별로 어렵지 않죠?

그럼 jQuery로 한번 바꿔 볼까요?

```
<script>
    $('#content').text('My name');
</script>
```
자! 어떤가요? 

코드가 매우 간결하죠? 이런 장점 때문에 jQuery를 많이 사용한다고 볼 수 있습니다.

그런데, 사실 꼭 jQuery를 사용하는 것이 좋다라고 단정지을 수는 없습니다. 

장점도 있지만 단점도 있기 때문이죠..(속도가 더 느리다던가...)

개인적인 생각으로는 서비스가 제공해야 하는 특성에 따라 자바스크립트나 jQuery를 적절하게 활용하는 것이 좋은 것 같습니다. 아마도 그것이 프론트엔드 개발자의 Skill과 연관되지 않을까요?

이상으로 간단하게 나마 한편의 역사처럼 jQuery를 간단하게 소개해 보았습니다. 

그럼 이번 포스팅은 여기서 마치겠습니다.

### References
https://yubylab.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-DOM%EA%B3%BC-jQuery

https://ny-kim.tistory.com/m/7