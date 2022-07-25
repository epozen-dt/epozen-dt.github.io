---
title: "프로세스 마이닝-휴리스틱 마이닝(Heuristic Mining)에 대해서"
last_modified_at: 2022-07-18
author: Do-soo, KIM
---

휴리스틱 마이닝은 프로세스 마이닝의 가장 기본이 되는 알고리즘인 알파 알고리즘을 발전시킨 형태로, 기본적으로 frequency와 이를 바탕으로 한 dependency를 고려하여 맵을 생성하는 알고리즘 입니다.

알파 알고리즘과 달리 빈도 수를 고려할 수 있고 single activity, 즉 프로세스가 두 단계 이상 존재하지 않고 하나의 작업으로만 구성된 케이스를 생략할 수 있다는 장점을 가지고 있습니다.

알파 알고리즘에 대한 내용은 이전 포스팅 내용을 참고하세요.

[[프로세스 마이닝 - 알파 알고리즘 적용]](https://epozen-dt.github.io/2022/04/28/ProcessMining_AlphaAlgorithm.html) (클릭하시면 해당 포스팅으로 이동합니다.)



### 1. 프로세스맵 도출 과정

자, 그럼 휴리스틱 마이닝 방법으로 프로세스 맵을 도출하는 과정을 살펴보겠습니다.

아래의 이벤트 로그 예시를 기준으로 설명하겠습니다.

![image](https://user-images.githubusercontent.com/92565548/179429679-172b4392-af39-442b-91f8-b7a36c051da0.png)

먼저, Directly follows relation matrix를 계산해야 합니다.

Directly follows relation은 하나의 작업이 다른 작업에 연결되는 빈도수를 표현한 것으로, 흔히 frequency라고도 표현합니다.

Directly follows relation matrix 산출결과는 다음과 같습니다.

![image](https://user-images.githubusercontent.com/92565548/179429717-0f6a757d-30d3-4bc2-9848-78c2f9bbdf20.png)

다음으로 dependency matrix를 생성합니다.

Dependency는 휴리스틱 마이닝 맵을 구성하는 가장 기본이 되는 값으로 다음과 같은 식에 의해 산출할 수 있습니다.

![image](https://user-images.githubusercontent.com/92565548/179429770-0588e2a7-f02f-4f06-b2ac-d325ea173c03.png)

위 산출식에 표기된 a><sub>L</sub>b가 바로 directly follows relation 입니다.

네! 눈치채셨죠? 위 산출식을 대입하기 위해 앞에서 먼저 directly follows relation matrix를 계산했던 것입니다.

dependency matrix결과는 다음과 같습니다.

![image](https://user-images.githubusercontent.com/92565548/179429839-fc527125-8f75-4328-ba42-ce2696b2048a.png)

이제 산출된 Directly follows relation 와 dependency를 threshold로 하여 적절하게 값을 적용하여 맵을 작성하면 됩니다.

이 포스팅에서는, Directly follows relation = 2, dependency = 0.7의 threshold를 사용하여 map을 작성해 보겠습니다.

위의 Directly follows relation matrix와 dependency matrix 상에서 2개의 threshold를 모두 만족하지 못하는 항목을 모두 생략하여 도출된 프로세스 맵은 다음과 같습니다.

![image](https://user-images.githubusercontent.com/92565548/179430469-6f547bdd-1f1a-458d-a27b-6b399561ef93.png)


이처럼, 휴리스틱 마이닝 기법으로 프로세스 맵을 도출하는 경우, threshold에 의해 너무 빈도가 낮거나 필요없는 trace 나 관계들을 생략할 수 있어 프로세스 맵을 좀 더 간단하게 표현할 수 있는 장점이 있습니다.


### 2. threshold 종류

위에서 적용했던 dependency threshold 이외에도 다양한 threshold를 적용하여 좀 더 보기 좋은 맵을 생성할 수 있습니다.

threshold 종류는 다음과 같습니다.

- Relative to Best Threshold

두 이벤트 사이의 directly follows 수가 설정한 threshold보다 많은 것만을 표시합니다. 즉, dependency threshold는 dependency를 필터링했다면, 이는 frequency를 필터링합니다.

- Length-one Loop Threshold

이 threshold는 length가 1인 loop(자신으로 돌아오는 loop)에 대한 필터링을 위한 threshold 입니다. 설정한 threshold보다 높은 위 값을 가지는 sequence만 표시합니다.

- Length-two Loop Threshold

이 threshold는 length가 2인 loop ( a > b > a 형태의 loop)에 대한 필터링을 위한 threshold 입니다. length-one loop threshold와 마찬가지로, 설정한 threshold보다 높은 위 값을 가지는 sequence만 표시합니다.

- Long Distance Threshold

이 threshold는 길이에 상관 없이 한 액티비티 뒤에 다른 액티비티가 일어난 경우에 대한 필터링을 위한 threshold 입니다. 설정한 threshold보다 높은 위 값을 가지는 sequence만 표현합니다.

- AND Threshold

이 threshold는 a 다음에 오는 b와 c 액티비티가 서로 XOR 관계 (a 이후에 b와 c 중 하나를 선택해서 일어남)인지 AND 관계인지 (a 이후에 b와 c가 모두 일어남)인지를 판단하기 위한 threshold 입니다. 설정한 threshold보다 값이 높으면 AND, 낮으면 XOR로 취급합니다.


### 3. 알파 알고리즘과 비교

그럼, 지금까지 살펴본 휴리스틱 마이너 기법이 알파 알고리즘과 어떤 차이가 있는지 살펴 볼까요?

알파 알고리즘에서는 빈도(frequency)를 전혀 고려하지 않습니다. 

a>b가 100번 일어났든지 1번 일어났든지 신경 쓰지 않고 오로지 activity 간의 relation만을 고려합니다. 

하지만 이렇게 빈도를 고려하지 않으면 1000번 일어난 관계와 1번 일어난 관계가 모두 똑같이 취급을 받게 됩니다. 

그래서 휴리스틱 마이너에서는 dependency graph에서는 directly follows frequency를 고려하므로 알파 알고리즘보다 더 좋은 프로세스 맵 도출 기법이라고 이야기하는 것입니다. 

그렇다면 빈도 이외 dependency까지 고려하는 이유는 무엇일까요? 

이는 concurrency를 고려하기 위해서입니다. 

예를 들어, <a,b,c>와 <a,c,b> 이벤트 로그가 있다고 했을 때 a 다음에 b, c가 병렬적으로 발생한다는 것을 알 수 있습니다. 

하지만 directly follows frequency만 고려하게 된다면 b>c, c>b가 일어났기 때문에 b와 c가 병렬적으로 일어났음에도 불구하고 b>c와 c>b를 화살표로 연결하게 될 것입니다. 

하지만 이 경우에 dependency threshold를 고려하게 된다면, 각각의 dependency가 0이기 때문에 이들은 표시되지 않게 됩니다. 

바로 이러한 경우에서 dependency를 고려해야 한다는 것을 알 수 있습니다.

지금까지 휴리스틱 마이닝에 대해서 살펴 보았습니다. 많은 도움이 되셨길 바라며, 여기서 마치겠습니다.


### [References]

https://process-mining.tistory.com/46<br>
(포스팅에 사용된 그림의 출처를 포함합니다.)

