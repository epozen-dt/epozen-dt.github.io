---
title: "Transformer - Positional encoding"
last_modified_at: 2022-9-30
author: 이한솔
---
Transformer는 2017년 NIPS를 통해 발표 되었으며, 번역 부분 SOTA를 차지한 우수한 모델입니다.

Transformer의 입력 방식은 Embedding vector에 Positional Encoding을 더해서 입력하게 되는데, 지난 시간에는 Embedding vector를 만드는 Embedding 방식에 대해 알아보았습니다. ([Transformer - Embedding](https://epozen-dt.github.io/Transformer_embedding/))

이 포스팅에서는 Positional Encoding 대해 알아보겠습니다.

---

# **목차**

1. Positional Encoding의 필요성
2. Positional Encoding 위치 표현
   - 수식 계산

<br>

---

# **Positional Encoding의 필요성**

RNN은 이전 분석 결과가 현재 분석에 영향을 주기 때문에 순차적(sequential)으로 연산하는데<br>이는 병렬 처리가 불가능 하다는 문제점이 있습니다.

<img src="https://user-images.githubusercontent.com/96156882/193163310-b9573f41-921f-4055-9628-6a40b94ce669.png" width="600">

Transformer 모델은 RNN을 사용하지 않는 Attention 기반의 모델로, 순차적으로 연산하지 않기 때문에 순서 위치 정보가 필요합니다.

> **Positional Encoding**은 Embedding vector에 위치 정보를 더해주는 것입니다.

<br>

---

# **Positional Encoding 위치 표현**

그렇다면 위치 정보를 어떻게 표현해주면 좋을까요?

첫번째는 **단순 정수**로 표현하는 방법을 생각해 볼 수 있습니다.<br>
하지만 단순 정수로 표현할 경우 숫자가 너무 빨리 커져서 weight가 갈수록 커지게 되어 학습이 불안정 하다는 문제점이 생깁니다.

<img src="https://user-images.githubusercontent.com/96156882/193163700-193c410e-c587-4dd5-b84c-cba63f350ddf.png" width="300">

<br>

두번째는 **Normalization**으로 표현하는 방법을 생각해 볼 수 있습니다.<br> 첫번째 토큰은 0, 마지막 토큰은 1로 두고 0~1로 Normalization 하는 방법인데, 이는 input 시퀀스 길이에 따라 값이 달라질 수 있다는 문제점이 있습니다. 

<img src="https://user-images.githubusercontent.com/96156882/193163998-6ebee3e0-5646-4778-81b3-ace92ef98660.png" width="300">

<br>

그래서 생각한 방법이 바로 **sin, cos 함수**를 이용하는 것입니다.<br>
-1~1 사이에서만 반복되는 주기 함수로 값이 너무 커질 것을 걱정하지 않아도 되고, input 길이가 달라도 항상 같은 값을 가질 수 있습니다.

<img src="https://user-images.githubusercontent.com/96156882/193164299-df704d08-30b6-48b5-bfc1-d97de4e31947.png" width="500">

<br>

> ### **수식 계산**

편의를 위해 단어 개수 4개, 차원이 4 일 때를 가정하여 수식 계산을 해보도록 하겠습니다.

<img src="https://user-images.githubusercontent.com/96156882/193164564-af7ffd37-8988-49e7-b422-93b1faf0f3c6.png" width="500">

계산 결과 i 값에 따라 다른 주기를 가지는 것을 확인할 수 있었습니다.


#### ***-1~1 사이를 반복하는 주기 함수이기 때문에 겹치는 값이 있지 않을까?*** 
의문이 들 수 있습니다. 하지만 positional encoding은 스칼라 값이 아닌 벡터 값이라는 것을 기억해야합니다. 

스칼라 값일 경우, 즉 차원이 하나일 경우에는 겹칠 수 있습니다.

<img src="https://user-images.githubusercontent.com/96156882/193165242-6bab6346-a927-423e-a433-e2acc18349bf.png" width="300">

하지만 벡터 값일 경우<br>
예시로 차원이 4개일 경우를 가정해보면 하나의 위치 벡터가 4개의 차원으로 표현된다면 각 요소는 서로 다른 4개의 주기를 갖게 되기 때문에 겹칠 걱정을 하지 않아도 됩니다.<br>
(물론 모든 주기의 공배수만큼 지난 위치는 겹칠 수 있겠지만, 그 정도면 이미 대부분의 단어 위치를 표현 할 수 있습니다.)

<img src="https://user-images.githubusercontent.com/96156882/193165512-4f1d6cc7-b436-4609-9ba1-6baca934615c.png" width="350">

<br>

---

저번 포스팅에 이해 이번 포스팅까지 Transformer의 입력 방식에 대해 알아보았습니다.<br>
다시 한번 정리하면, 기계가 이해할 수 있도록 숫자(벡터)로 바꾸는 Embedding vector에 위치 정보를 넣어주는 Positional Encoding을 더해서 입력하게 됩니다.

---

### 참고자료
- https://tigris-data-science.tistory.com/entry/%EC%B0%A8%EA%B7%BC%EC%B0%A8%EA%B7%BC-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-Transformer5-Positional-Encoding 
- https://www.blossominkyung.com/deeplearning/transfomer-positional-encoding

