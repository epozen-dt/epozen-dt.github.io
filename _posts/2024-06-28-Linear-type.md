---
title: "Linear-type"
last_modified_at: 2024-06-28
author: 김혜원
---

본 포스팅은 Linear 모델의 종류에 대해 공유합니다.

---


&nbsp;


## 개요

다중 선형 회귀와 Dlinear 및 Nlinear에 대해 알아보았습니다. 다중 선형 회귀는 선형 회귀의 한 종류이고, Dlinear과 Nlinear은 'Are Transformers Effective for Time Series Forecasting?' 논문[[1]](https://arxiv.org/pdf/2205.13504)에서 소개된 개념으로 각 의미는 Decomposition layer, Normalization layer입니다. 각 개념은 다음과 같습니다.


&nbsp;

## Multiple Linear Regression(다중 선형 회귀)


다중 선형 회귀(Multiple Linear Regression)는 여러 독립 변수들이 하나의 종속 변수에 미치는 영향을 모델링하는 기법입니다. 이는 단순 선형 회귀(Simple Linear Regression)에서 하나의 독립 변수만 사용하는 것과 달리, 여러 독립 변수들을 동시에 고려하여 예측의 정확성을 높입니다. 시계열 데이터세트 중 많이 쓰이는 데이터세트로 ETT(Electrocity Transformer Temperator)가 있습니다.

&nbsp;

## Dlinear?

Decomposition(분해) linear는 Autofomer에서 사용된 방법으로, 이동 평균(Moving Average)trend(추세)와 seasonal(계절성)으로 분리하는 것을 말합니다. 

![image](https://github.com/khw927/epozen-dt.github.io/assets/107157737/d51b1d02-0af1-49fd-91d7-f2111408cc32)

분해 과정은 위 그림과 같습니다. 먼저 입력 데이터 $x$가 있을 때, n개의 블럭으로 나누어 줍니다. 이 블럭을 3개씩 묶어서 이동 평균 $x_t$를 구합니다. 이 때, 데이터의 처음과 끝은 1개씩만 존재하기 때문에 해당 블럭을 복사하여 각각 앞과 뒤에 연결을 해줍니다. 마지막으로, 원래 데이터 $x$에서 이동 평균 데이터(=Trend, 추세) $x_t$를 빼면  $x_s$(seasonal, 계절성)이 나옵니다. 

&nbsp;

## Nlinear?

Normalization(정규화) linear는 정규화된 데이터에 linear를 적용한 것입니다. 이 방법은 입력 시퀀스의 마지막 값을 빼고, 선형 레이어를 통과한 후, 다시 마지막 값을 더하여 최종 예측을 만듭니다. 

예를 들어, 입력 시퀀스 $x$가 [$x_1$, $x_2$, $x_3$, ··· $x_n$]일 때, 각 항에 마지막 값을 빼서 [$x_1$-$x_n$, $x_2$-$x_n$, $x_3$-$x_n$, ··· $x_n$-$x_n$]로 변환합니다. 그 다음 정규화된 시퀀스를 선형 모델에 입력으로 넣어 예측을 합니다. 마지막으로, 각 항에 $x_n$을 다시 더하여 원래 값으로 복원합니다.

&nbsp;


## 효과

단순한 선형 변환을 사용하기 때문에 계산적으로 간단하고, 시간이 오래 걸리지 않는다는 장점이 있습니다. 논문에서 제안한 두 모델은 시계열 데이터에 대해 시간 대비 효율적이고, Transfomer 기반 모델들보다 성능이 좋습니다. 

&nbsp;



&nbsp;


------
> 참고

[1] [논문](https://arxiv.org/pdf/2205.13504)

[2] [그림](https://data-newbie.tistory.com/944)



