---
title: "PyScript 소개"
last_modified_at: 2022-06-28
author: Do-soo, KIM
---

### 1. 개요


빅데이터와 인공지능의 열풍으로 최근 Python 언어의 인기가 대단하죠.

아래 그림에서 보듯, Python이 2022년 6월 현재 1위를 달리고 있습니다.

![image](https://user-images.githubusercontent.com/92565548/176110541-594077c7-bfb3-4dea-b07f-d09ac0ad4d80.png)

(출처: https://www.tiobe.com/tiobe-index/)

특히 데이터사이언스 분야에서 Python은 넘사벽 수준입니다. 

보통 웹 프로그램 개발할 때, 프론트엔드로서 자바스크립트를 많이 사용하는데요.

하지만 웹 개발 측면에서 Python의 아쉬웠던 점은 바로 프론트엔드 언어로서 사용할 수 없었다는 점이죠.

Flask, Django와 같은 프레임워크와 함께 백엔드에서만 제한적으로 사용될 수 밖에 없었죠.

Python을 자바스크립트처럼 프론트엔드에서 사용할 수 있게 되어, Python 만으로 웹 애플리케이션 개발이 가능하다면 얼마나 편리할까요? 

이런 마음을 대변이라도 하듯이, Anaconda가 Pycon 2022 컨퍼런스에서 이러한 고민 해결책을 내 놓았습니다.

Anaconda의 공식 사이트 내용에 따르면 HTML의 인터페이스, Pyodide, WASM, 최신 웹 기술을 사용하여 브라우저에서 Python 웹 애플리케이션을 만들 수 있게 해주는 "PyScript"라는 프레임워크를 발표하였습니다.

이제 Python만으로 웹 애플리케이션을 만들 수 있게 된 세상이 드디어 오게 된 것입니다.

(물론 아쉽게도 아직 상용제품 적용단계는 아닙니다. 뒷부분에서 이부분은 다시 언급하게 될 겁니다.)

---

### 2. 구성 기술

Anaconda 공식 사이트에서는 PyScript의 기술 스택을 다음과 같이 표현해 놓았네요.

![image](https://user-images.githubusercontent.com/92565548/176111020-7934f7f4-dffa-4e7f-832d-24a03fe8f380.png)

(출처: https://www.anaconda.com/blog/pyscript-python-in-the-browser)


아쉽게도, 스택에 대한 자세한 설명은 없어서 대충 이렇게나마 확인하는 것으로 만족해야 할 것 같군요.

그래도, PyScript를 이해하기 위해서 최소한 설명이 필요한 것은 있기에 그 부분만 간단히 설명하겠습니다.

WASM과 Pyodide 이 2가지 개념은 알고 가야 PyScript에 대해서 이해가 쉬울 겁니다.

(이미 알고 계신 분들이라면 이 내용은 그냥 skip 하셔도 됩니다.)

#### - WASM

WASM(WebAssembly)는 최신 웹브라우저에서 실행할 수 있는 새로운 유형의 코드라고 정의하고 있습니다. 

쉽게 말하자면 브라우저가 이해할 수 있는 언어인 것이죠.

우리가 언어로 프로그램을 개발할 때 소스 코드 컴파일 과정을 거쳐서 바이트 코드르 변환하여 OS에서 실행하는 과정을 거치는데, 이러한 바이트코드와 비슷한 개념이라고 보시면 됩니다.

웹 개발에서는 C, C++, RUST 등의 소스 코드를 LLVM compiler를 통해 바이트코드로 컴파일하여 WASM로 변환해서 브라우저에서 실행하는 방식으로 사용합니다.

#### - Pyodide

Pyodide는 WASM/Emscripten을 기반으로 하는 브라우저, Node JS용 Python 배포판이라고 정의되어 있습니다.

Python은 인터프리터 언어이므로, WASM으로 사용이 불가능합니다. 

따라서 WASM으로 변환이 되어야 브라우저에서 실행할 수 있습니다.

그래서 Pyodide는 Python의 모체라고 할 수 있는 CPython 인터프리터를 WASM 모듈로 컴파일하여 브라우저에서 실행 가능한 형태로 제공하는 것입니다.

PyScript는 이러한 WASM과 Pyodide를 이용하여 그 외 기타 기술들을 모아서 만들어 놓은 프레임워크인 셈이죠.

쉽게 설명한다고 했는데도 어렵네요.

---

### 3. 특징

그럼, PyScript 공식사이트에 소개된 특징을 살펴 보겠습니다.

- 주요 Python 데이터 사이언스 라이브러리(numpy, pandas, matplotlib, scikit-learn 등)를 지원

- Python과 Javascript 객체 및 네임스페이스 간의 양방향 통신 제공

- 실행할 페이지 코드에 포함할 패키지 및 파일 정의 허용

- 버튼, 컨테이너, 텍스트 상자 등과 같이 쉽게 사용할 수 있는 UI 구성 요소 제공

백문이 불여일견이라고 하였으니, 어려운 설명은 여기까지 하고 실제로 PyScript를 사용해 보면서 이해를 해 보도록 하겠습니다.

---

### 4. 사용해 보기

#### - PyScript 사용 정의하기

자바스크립트에서 사용했던 방식 그대로 CDN 정의만 해 주면 바로 사용할 수 있습니다.

```html
<html>
    <head>
        <link rel="stylesheet" href="https://pyscript.net/alpha/pyscript.css" />
        <script defer src="https://pyscript.net/alpha/pyscript.js"></script>    
    </head>
    <body>
    </body>
</html>
```

#### - 패키지 정의하기

` < `head` > ` 태그 안에 ` < `py-env` > ` 태그를 정의하여 사용합니다.

```html
<head>
    <py-env>
        - pandas
        - scikit-learn
        - matplotlib
        - statsmodels
    </py-env>    
</head>
```

#### - Python 코드 작성하기

` < `body` > ` 태그 안에 ` < `py-script` > ` 태그를 정의하여 사용합니다.

```html
<body>
    <b><p>Today is <br><u><label id='today'></label></u></p></b>
    <py-script>
      import datetime as dt

      def index():
        pyscript.write('today', dt.date.today().strftime('%A %B %d, %Y'))

      index()
    </py-script>    
</body>
```

#### - REPL(Read Eval Print Loop) 컴포넌트 생성하기

PyScript에서는 우리가 Python 개발 시 많이 사용하는 주피터 노트북 형태의 컴포넌트를 생성할 수 있습니다.

` < `body` > ` 태그 안에 ` < `py-repl` > ` 태그를 정의하여 사용합니다.

이렇게 하면, 주피터 노트북 화면처럼 직접 코드를 작성하여 실행할 수 있는 화면을 구성할 수 있습니다.

```html
<body>
    <py-repl>
import datetime as dt

def index():
pyscript.write('today', dt.date.today().strftime('%A %B %d, %Y'))

index()
    </py-repl>    
</body>
```

![image](https://user-images.githubusercontent.com/92565548/176118320-af16b39e-4b29-40f9-82af-f6d43c650e8f.png)


별 다른 서버 구성이나 프레임워크 없이, 위와 같이 html 파일만 작성해서 실행하면 끝입니다. 

바로 이게 PyScript의 최대 장점인 것이죠.

이 정도만 알면, 지금 바로 웹 애플리케이션을 만들어 볼 수 있습니다.

한번 만들어볼까요?

PyScript가 아직까지 지원하지 않는 패키지들이 있는 관계로, 지원하는 패키지로 한정하여 만들어 볼 수 있는 걸 생각해 보다가 SARIMA 예제가 딱이구나 라고 생각했습니다.

SARIMA에 대한 내용은 아래의 이전 포스팅을 참고하세요.

[[시계열 데이터 분석 - ARIMA & S-ARIMA 모형 적용 결과 비교 테스트 (Python vs Java)]](https://epozen-dt.github.io/2022/02/24/arima-java-example.html)

(클릭하시면 해당 포스팅으로 이동합니다.)

만들어 볼 예제는 시계열 데이터 예측 웹 애플리케이션으로, SARIMA 모형을 적용하여 비행기 탑승객 수 예측 결과를 웹으로 시각화 해 보겠습니다.

사용할 패키지는 Pandas, Scikit-learn, Matplotlib, Statsmodels 입니다.

아래 그림은 예측결과(노란색 표시)를 웹 화면으로 시각화 한 결과입니다.

실행 속도도 기록해 보았습니다. 약 7.75초 정도 걸렸습니다.

![image](https://user-images.githubusercontent.com/92565548/176118391-331f85b1-73db-4702-8069-0310db663d34.png){: width="50%" height="50%"}

비교를 위해 Python으로도 한번 만들어 보았습니다. 약 2.87초 정도가 걸렸네요.

![image](https://user-images.githubusercontent.com/92565548/176118455-cd23d0b0-b61e-4722-ac72-089016c830f6.png){: width="50%" height="50%"}

아직까진 PyScript가 성능에 문제가 있음을 확인할 수 있습니다.

### 5. 정리

아래 내용은 PyScript 공식 사이트에서 발췌한 내용입니다.

객관적으로 내용을 전달하기 위해 보시기엔 불편하시겠지만 영문 그대로 적어 보았습니다.

Please be advised that PyScript is very alpha and under heavy development. There are many known issues, from usability to loading times, and you should expect things to change often. We encourage people to play and explore with PyScript, but at this time we do not recommend using it for production.

대충 내용을 살펴보자면, PyScript가 현재까지는 알파 버전이고, 로딩 시간 등의 많은 알려진 이슈들이 있으니까, production으로 적용은 추천하지 않는다는 내용입니다.

그러니까 아직 테스트 버전 수준으로 계속적으로 개발이 진행될 것이라는 내용인 것이죠.

PyScript는 위 예제에서도 살펴보았듯이 화면에서 로딩 속도가 느린 문제가 있고, 

Tensorflow 등의 라이브러리도 아직은 지원되지 않고 있는데 이러한 것들도 반영이 되겠죠?

결국 상용화 버전에 다다르면 굉장한 위력을 가지는 또 하나의 프레임워크가 탄생되리라 예상해 봅니다.

그럼, PyScript가 상용화 단계까지 오는 순간을 기대해 보면서 포스팅은 여기서 마치겠습니다.

### References

```
https://www.anaconda.com/blog/pyscript-python-in-the-browser
https://pyscript.net/
```