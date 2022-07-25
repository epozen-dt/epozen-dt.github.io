---
title: "순환 신경망-RNN, LSTM, GRU"
last_modified_at: 2022-01-26
author: 김수민
---

이번 포스팅에서는 순환 신경망에 대해 이야기 해보도록 하겠습니다.

---

## 목차

[1. RNN](#1-rnn)

[2. LSTM](#2-lstm)

[3. GRU](#3-gru)

[4. References](#4-references)

---

순환 신경망은 반복적이고 순차적인 데이터 학습에 특화된 가장 기본적인 신경망 입니다.

기본적으로 4가지 종류로 task를 구분할 수 있습니다.

![그림1](https://user-images.githubusercontent.com/87166420/151270200-a5159997-ce33-4906-b07c-d617b61c8dc8.png)



가장 기본적인 순환신경망에는 RNN, LSTM, GRU로 꼽을 수 있습니다. 지금부터 살펴보겠습니다.

---

## 1. RNN

가장 기본이 되는 신경망 구조 입니다.

시계열 데이터에서 이전 step의 결과와 현재의 입력값을 가지고 현재 step의 결과를 만들어내는 방식입니다.

![그림5](https://user-images.githubusercontent.com/87166420/151270313-875776ab-c8d8-4ca6-8ca8-b57bcd728916.png)



기본 연산과정을 살펴보겠습니다. 해당 과정을 좀 더 자세한 그림으로 그린다면 다음과 같습니다.

![그림2](https://user-images.githubusercontent.com/87166420/151270336-13a077b4-5f31-4c7f-b395-f916a39ad19a.png)


1. 처음 h_(t-1) 은 0 혹은 랜덤 값으로 초기화되어 시작합니다.

2. 초기화된 h_(t-1)를 W_0 을 곱한 결과와 x_t를 연산한 결과를 합쳐 activation function을 거쳐  h_t를 예측합니다.
3. 예측 결과(h_t)를 다음 과정에서 사용하면서 반복적으로 수행하며
4. loss 계산 후 해당 값을 미분해 모든 시퀀스에 전달하면서 학습이 이루어 집니다.



이런 RNN 모델은 시퀀스가 길어지게 되면 각 과정에서 사용되는 activation function에 의해 다음 값으로 전달되는 값들이 급격하게 줄거나 발산하는 vanishing gradient문제가 발생하게 됩니다.

이를 해결하기 위해 나온 모델이 바로 LSTM입니다.

---

## 2. LSTM

LSTM은 Long Short Term Memory 라는 뜻의 모델 입니다.

기존 RNN은 시퀀스가 길어짐에 따라 앞의 계산 결과가 제대로 전달되어지지 않는 모습을 보였습니다.

이를 해결하기 위해 LSTM에서는 결과로 출력하는 부분과 결과를 다음 결과로 전달하는 부분으로 구성했습니다.

이전 step에서의 중요 정보들을 다음 step으로 전달하는 구조를 cell state 라는 명칭으로 부릅니다.

그리고 기존 RNN에서와 같이 현재 step에서 내보낼 output을 전달하는 부분을 hidden state라는 명칭으로 부릅니다.

![그림3](https://user-images.githubusercontent.com/87166420/151270371-8e6e58e8-2bdf-4ebe-ae1f-dd63bcd259a0.png)



각 부분들에서 기억하고, 삭제하는 연산을 위해 gate를 사용하게 되는데, gate에는 Forget / Input / Output 3가지가 사용됩니다.

1. Forget Gate

	cell state에서 sigmoid 한 결과를 통해 어떤 정보를 버릴것인지를 결정하는 gate입니다.

![forget](https://user-images.githubusercontent.com/87166420/151270537-b82258d5-e273-4238-b545-42bcedb51606.png)


2. Input Gate

   입력된 정보들 중 어떤 것을 cell state에 저장할 것인지를 결정하는 gate 입니다.

![input](https://user-images.githubusercontent.com/87166420/151270555-5f443a54-e2ee-4147-ab76-0aa15892cb56.png)

3. Output Gate

   어떤 정보를 Output으로 출력할지를 결정하는 gate 입니다.

![output](https://user-images.githubusercontent.com/87166420/151270577-20d2830d-de1b-4866-b537-758bb09a270e.png)



이러한 gate들 중 Forget Gate와 Input Gate의 결과를 사용해 Cell state를 업데이트 하는 과정이 발생하며, 해당 과정에서의 결과를 다음 cell로 전달하게 됩니다.

이런 연산과정을 간략하게 말하자면 다음과 같다.

1. $C_{t-1}$을 가공해 $C_{t}$ 생성
   1. $C_{t-1}$에서 불필요한 정보 제거
   2. $C_{t-1}$에 새로운 정보를 추가 ($x_{t}$와 $h_{t-1}$ 연산)
2. $C_{t}$를 가공해 step에서의 $h_{t}$ 연산
3. $C_{t}$와 $h_{t}$ 전달


이러한 LSTM은 RNN의 단기기억만 할 수 있던 단점을 보완해 현재 제일 많이 사용되고 있는 순환 신경망 셀 입니다.

---

## 3. GRU

위의 LSTM연산에서의 복잡성을 줄이고자 개선한 모델이 바로 GRU입니다.

성능적으로는 LSTM과 크게 차이가 난다고 보기는 힘들지만, 경험적으로 데이터의 양이 적은 경우에 대해서는 GRU가 좀 더 나은 성능을 보인다고 합니다.

GRU는 Gated Recurrent Unit의 약자로, cell state와 hidden state가 합쳐져 있는 모양 갖고 있습니다.

![그림4](https://user-images.githubusercontent.com/87166420/151270429-3e97dc28-7a7c-49e9-b842-e1abc2241477.png)

해당 모델에서도 역시 연산을 Gate를 사용해 수행합니다.

GRU에서는 Reset과 Update Gate 두 개를 사용합니다.

1. Reset Gate

   과거의 정보를 적당히 리셋하도록 기능하는 Gate 입니다.

   직전 시점의 output과 현 시점의 정보에 가중치를 곱해 얻는 결과로 과거의 정보를 리셋하게 됩니다.

![reset](https://user-images.githubusercontent.com/87166420/151270634-ab3509fd-c0b9-4d84-bd1c-3d47a0aaa273.png)

2. Update Gate


   과거와 현재의 정보의 최신화 비율을 정하는 Gate 입니다.

   LSTM에서의 forget Gate와 Input Gate를 합쳐놓은 기능을 수행합니다.

![update](https://user-images.githubusercontent.com/87166420/151270649-72a259c1-28cd-4508-a542-e1814368fbc5.png)

연산과정을 간략하게 설명하면 다음과 같습니다.

1. Reset Gate를 계산해 임시  $h_{t}$를 생성합니다.
2. Update Gate를 통해 $h_{t-1}$ 과 $h_{t}$ 간의 비중을 결정합니다.
3.  $z_{t}$를 이용해 최종  $h_{t}$를 계산합니다.

---

이번 포스팅에서는 시계열 데이터를 다루는데 가장 많이 사용되는 순환신경망에 대해 알아보았습니다.

순환 신경망에는 크게 RNN, LSTM, GRU가 있으며 RNN은 장기 의존성 문제로 인해 많이 사용되지 않고, 이를 보완한 LSTM과 GRU를 주로 사용합니다.

LSTM과 GRU에서는 이전 데이터 값을 비중에 따라 다음으로 전달하며, 이 과정에서 activation function을 사용하지 않음으로써 vanishing gradient를 해결하고자 했습니다.



이상으로 포스팅을 마치도록 하겠습니다.

---

## 4. References

[https](https://github.com/heartcored98/Standalone-DeepLearning/blob/master/Lec7/Lec7-A.pdf)[://](https://github.com/heartcored98/Standalone-DeepLearning/blob/master/Lec7/Lec7-A.pdf)[github.com/heartcored98/Standalone-DeepLearning/blob/master/Lec7/Lec7-A.pdf](https://github.com/heartcored98/Standalone-DeepLearning/blob/master/Lec7/Lec7-A.pdf)

[https://](https://www.youtube.com/watch?v=bPRfnlG6dtU)[www.youtube.com/watch?v=bPRfnlG6dtU](https://www.youtube.com/watch?v=bPRfnlG6dtU)

[https](https://brunch.co.kr/@chris-song/9)[://brunch.co.kr/@](https://brunch.co.kr/@chris-song/9)[chris-song/9](https://brunch.co.kr/@chris-song/9)

[https://](https://www.youtube.com/watch?v=cs3tSnAsyRs)[www.youtube.com/watch?v=cs3tSnAsyRs](https://www.youtube.com/watch?v=cs3tSnAsyRs)

[https](https://blog.naver.com/winddori2002/221992543837)[://](https://blog.naver.com/winddori2002/221992543837)[blog.naver.com/winddori2002/221992543837](https://blog.naver.com/winddori2002/221992543837)

https://yjjo.tistory.com/18

