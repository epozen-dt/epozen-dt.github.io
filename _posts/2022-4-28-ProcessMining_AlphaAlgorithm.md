---
layout: post
title: "프로세스 마이닝 - 알파 알고리즘 적용"
date: 2022-4-28
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **프로세스 마이닝, 이벤트 로그로부터 알파 알고리즘을 통해 프로세스 모델을 도출하는 과정** 에 대한 내용을 공유해보려고 합니다.

※ 프로세스 마이닝에 대해 기본적인 내용을 알아야 가능합니다.

# **프로세스 마이닝**

> **프로세스 마이닝 이전의 업무**  

업무 수행과 관련된 각종 정보가 이벤트 로그 형태로 정보 시스템에 기록되었지만, 
다음과 같은 방식은 비지니스 프로세스 개선에 많은 시간과 비용이 소모되었습니다.
- 가시화된 프로세스가 없음
- 업무 담당자 별 인터뷰 및 회의를 통해 AS-IS 프로세스 모델을 만듬
- 주관적이며, 왜곡 발생 가능성 존재

> **프로세스 마이닝 도입**  

위와 같이, AS-IS 프로세스 구조는 전체 그림을 파악하기 어려우며, 객관성이 떨어집니다. 특히 유지보수 시에 발생하는 잦은 프로세스의 변경에서는 실제 프로세스와 정의된 이상 프로세스 간의 격차가 더 커질 수 있습니다.  
 
따라서, 프로세스 마이닝 도입을 통해 다양한 업무 영역과 수준에 대해 실제 업무 프로세스를 가시화하고 이해할 수 있게 됩니다.

> **프로세스 마이닝**  

프로세스 과정 중 발생한 데이터를 기반으로 한 데이터 분석 기법입니다.   
업무에 사용되는 다양한 시스템(LMS, CRM, SCM 등)에 기록된 데이터인 이벤트 로그 분석을 통해서 프로세스를 파악할 수 있습니다.

![9](https://user-images.githubusercontent.com/92897860/165440676-c88889cf-81dc-401d-ad50-560ec523a9e4.png)

프로세스 마이닝으로 해결할 수 있는 문제는 다음과 같습니다.
- 현재 프로세스에 대한 분석 (실제 프로세스 파악 가능)
- 문제점 도출 및 개선 (병목, 낭비 구간, 숨겨진 프로세스, 재작업 등 문제점 도출 및 개선)
- 예측 (미래 예측 및 사전 대응 가능)


> **프로세스 데이터 (이벤트 로그)**  

프로세스 마이닝 기법을 적용하기 위해서는 이벤트 로그 데이터를 추출해야 합니다.
각 시스템에서 수집한 데이터는 다음과 같은 형태로 이루어져야 합니다. 

![8](https://user-images.githubusercontent.com/92897860/165440658-51fd0865-4fb0-4d25-bdc8-9094b44ee8e5.png)


## **알파 알고리즘**

이벤트 로그로부터 프로세스 모델을 도출하는 프로세스 발견 알고리즘 중 기본이 되는 알고리즘입니다.    
아래와 같은 이벤트 로그가 있다고 가정하에 알파 알고리즘을 적용하여 프로세스 모델을 추출하겠습니다.

> **1. 이벤트 로그 추출**  

알파 알고리즘에서는 이벤트 로그의 요소 중 액티비티의 순서에만 집중합니다.  
즉, 액티비티가 어떤 순서로 일어났는지에 대한 정보만 필요로 합니다.  
즉, 하나의 케이스 안에서 액티비티가 어떤 순서로 일어났는지에 대한 정보만 필요하며 L1 은 바로 그 형태입니다.

![a1](https://user-images.githubusercontent.com/92897860/165695904-cb337edd-95b1-420e-89cf-5363efae8b7e.png)


> **2. Log-based ordering relations 변환**  

먼저 , 액티비티 간의 relation 4가지 종류에 대해 알아야합니다.  
- **(>) Direct Succession :** 이벤트 로그 내의 하나의 trace 내에서 바로 뒤에 따라오는 관계의 액티비티  
- **(->) Casuality :** a가 b에 대해서 > 인 경우  
- **(||) Paraller :** >의 역이 성립하는 경우  
- **(#) Choice :** 아무 관계가 없는 경우  

다음과 같이, 이벤트 로그(L1)의 액티비티 간의 log-based ordering relations 를 파악해야 합니다.

![a5](https://user-images.githubusercontent.com/92897860/165697120-46b0c13d-7428-48c2-8ea2-453aaf509e2d.png)


> **3. Footprint Matrix 변환**  

Footprint Matrix란, 액티비티 간의 log-based ordering relations 을 Matrix 형태로 표현한 것을 의미합니다.  

![a3](https://user-images.githubusercontent.com/92897860/165696017-013d595d-fedf-4e70-9975-46d351432602.png)


> **4. 알파 알고리즘**  

먼저 알파 알고리즘의 기본 아이디어는 Footprint Matrix를 가지고 place의 5가지 기본 패턴을 적용해 place를 도출(프로세스 모델)합니다.

5가지 place의 기본 패턴은 아래와 같습니다.   
- **sequnce pattern**, 조건 a->b 인 경우
- **XOR-split pattern**, 조건 a->b, a->c, b#c 인 경우
- **AND-split pattern**, 조건 a->b, a->c, b||c 인 경우
- **XOR-join pattern**, 조건 a->c, b->c, a#b 인 경우
- **AND-join pattern**, 조건 a->c, b->c, a||b 인 경우

![a6](https://user-images.githubusercontent.com/92897860/165700540-dc2995bc-be03-4f50-8720-4e450e5a893f.png)


알파 알고리즘은 아래와 같습니다.

![a4](https://user-images.githubusercontent.com/92897860/165695971-590e948b-d390-43a2-88f3-1ae7d7f49eba.png)


> **5. 알파 알고리즘 적용**  

※ 모든 계산은 Footprint Matrix가 있다는 가정하에 진행됩니다.

**5.1 T_L, T_I, T_O 계산**   
아래와 같이 각 조건에 맞는 집합을 계산합니다.
  - T_L : 모든 액티비티의 조합
  - T_I : Trace 로 시작하는 집합
  - T_O : Trace 로 끝나는 집합

  ![b1](https://user-images.githubusercontent.com/92897860/165702164-b7f10382-90f0-4eea-aefe-bd7209361a87.png)

**5.2 X_L 계산** 

조건 Casuality(->)이면서 Choice(#)이 아닌 집합을 계산합니다.  
이때, X_L은 행과 열 따로 계산합니다.

![b4](https://user-images.githubusercontent.com/92897860/165702158-9c6ad08c-30ae-4ffc-be08-0f3f5f628f9d.png)

계산 결과는 다음과 같습니다.

![b2](https://user-images.githubusercontent.com/92897860/165702131-8c7db3ef-ba36-4ef4-bd91-8455d0f4221b.png)

**5.3 Y_L 계산** 

위의 X_L의 합집합에서 Maximal 이 아닌 값(집합)을 제외합니다.  
X_L 합집합에서 (c, g) (d, g) (e, g) (f, g) 의 4개의 항목이 제외되었습니다.

![b3](https://user-images.githubusercontent.com/92897860/165702152-8b6687fe-6cc6-49e0-a991-18b155d0152d.png)


**5.4 프로세스 모델(페트리넷) 도출** 

페트리넷의 구성요소들을 그립니다.  
- 각 액티비티 T_L 을 표시(사각형)
- 시작과 종료인 T_I, T_O를 표시
- Y_L 인 Transition 을 표시(원)
- 각 place 와 Transition 을 연결(화살표)

다음과 같이, 이벤트 로그로부터 알파 알고리즘을 통해 프로세스 모델을 도출할 수 있습니다.

![a2](https://user-images.githubusercontent.com/92897860/165695929-faea6657-034b-4fc5-8fd1-ff224726f07b.png)


## **참고 자료**  

* [프로세스 마이닝](https://process-mining.tistory.com/120)  
* [Data Science In Action](https://right1203.tistory.com/9?category=344380)  
* [알파 알고리즘 적용](https://process-mining.tistory.com/12)  

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
