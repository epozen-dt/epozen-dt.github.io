---
title: "online-learning-evaluation"
last_modified_at: 2023-12-28
author: 김혜원
---

본 포스팅은 온라인 학습 시 모델 평가 방법에 대해 공유하기 위해 작성한 글입니다.

---

### 온라인 학습(online-learning)?

시계열 온라인 학습은 신규 데이터가 입력될 때마다 모델을 지속적으로 업데이트 하는 것을 말합니다. 기존에는 신규 데이터가 들어오면 이전 데이터와 병합하여 모델을 학습시켰습니다. 그러나 온라인 학습은 병합 과정 없이 신규 데이터에 대해서만 모델을 업데이트 합니다. 그래서 스트리밍 데이터의 빠른 속도와 큰 규모의 데이터를 처리하는데 용이합니다. 금융, 전자상거래, 헬스케어 등 다양한 분야에서 사용됩니다.


### 온라인 학습 시 모델 평가 방법

온라인 학습에서는 Precision at top-K , Normalized Discounted Cumulative Gain (NDCG) 등을 사용하여 모델의 성능을 평가합니다. 이 방법들은 추천 시스템에서 사용되는 방법으로, Precision(정밀도)를 이용합니다. Precision at top-K는 상위 K개의 랭킹에 대해 정밀도를 구하는 방법입니다. 추천된 결과 중 관련이 있는 것은 O, 없는 것은 X로 표기하여 관련도를 구하여 1에 가까울수록 좋습니다.

NDCG는 관련도의 합(Cumulative Gain, CG)에 추천 순서에 대한 가중치를 적용한 후 정규화한 것으로, 1에 가까울수록 좋습니다. 아래 식을 이용하여 구할 수 있습니다.

* NDCG 식

![NDCG](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F6ywVh%2FbtqO0T5DClO%2Fm1JXFa1KyTUVVkAj4kFIDK%2Fimg.jpg)

![DCG](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fy0nSc%2FbtqO12ONqiJ%2F4Eh5SiCp8kS3B0ikWQ72H0%2Fimg.jpg)

![IDCG](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F54cW5%2FbtrdDKx2P9V%2FzyKHWYB1soiVuYubXd1kIk%2Fimg.png)

* IDCG(Ideal DCG) : 최선의 추천을 했을 경우 받는 DCG값

------
> 참고

[1] [온라인 학습](https://arxiv.org/pdf/1802.02871.pdf)
[2] [NDCG](https://walwalgabu.tistory.com/entry/4-NDCG-Normalized-Discounted-Cumulative-Gain%ED%8F%89%EA%B0%80%EC%A7%80%ED%91%9C)
[3] [IDCG](https://sungkee-book.tistory.com/11)