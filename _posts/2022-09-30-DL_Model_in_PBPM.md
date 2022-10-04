---
title: "DL_Model_in_PBPM"
last_modified_at: 2022-09-30
author: 김혜원
---

이번 포스팅은 Predictive Business Process Monitoring에 적용된 딥러닝 모델들에 대하여 알아보겠습니다.

---

>## Predictive Process Monitoring?

Predictive Process Monitoring이란 Process Mining의 하위분야로 진행중인 프로세스의 미래를 예측하는 것을 말합니다.
예측하고자 하는 목표에 따라 다음 활동 예측, 남은 시간 예측, 결과 지향 예측으로 나눌 수 있고, 필수로 들어가야하는 이벤트 속성은 Time stamp, Activity, CaseID로 숫자나 텍스트입니다.
같은 트레이스 안에 존재하는 이벤트들은 같은 케이스를 필수로 포함하고 있어야 하며, 시간은 증가 해야 합니다.



    <사용하는 용어>
    * event : 프로세스 내의 활동
    * trace : 유한하고 비어있지않은 시퀀스(이벤트 모음) 
    * event log : 트레이스 모음
    * prefix trace : 이벤트로그의 첫 이벤트를 가진 트레이스

프로세스 예측 연구 초기에는 적용 모델로 RNN을 사용했는데 RNN 특성상 시간적으로 멀어지면 학습 효과가 떨어지는 문제를 해결하기 위해 LSTM을 사용했습니다. 이후 적용된 모델들은 Transformer와 BERT가 있습니다. 프로세스의 입력은 Activity를 Embedding한 벡터이고, 출력은 예측값이 됩니다.


>## Model

 * Seq2seq(sequence-to-sequence)

아래 그림은 seq2seq 모델을 나타낸 것입니다. 이는 한 문장을 다른 문장으로 변환하는 모델로 Embedding된 시퀀스(벡터)를 입력으로 받고 RNN 모듈을 거쳐 하나로 합쳐집니다. 이때 합쳐진 하나의 벡터를 context vector라고 하는데 이는 디코더의 입력값이 됩니다.
각 RNN은 LSTM이나 GRU를 사용합니다.

![seq2seq](https://gaussian37.github.io/assets/img/dl/concept/attention/1.png)

이를 프로세스 예측에 적용하면 아래 그림과 같습니다.

![PBPM_LSTM](https://media.springernature.com/lw685/springer-static/image/chp%3A10.1007%2F978-3-319-59536-8_30/MediaObjects/450866_1_En_30_Fig2_HTML.gif?as=webp)

인코더 부분을 사용한 것과 같고, 임베딩된 벡터를 입력으로 받습니다. 따라서 전체 구조는 다음과 같이 정리할 수 있습니다.

    < 전체 구조 >
        1. Word Embedding
        2. LSTM 모듈 통과(인코더 - 디코더)
        3. 결과 출력



* Transformer

아래 그림은 트랜스포머 모델의 전체 구조를 나타낸 것입니다. 인코더 - 디코더 구조를 가지며 Multi-head Attention block이 핵심요소입니다.

![Transformer](https://gaussian37.github.io/assets/img/dl/concept/transformer/0.png)

이를 프로세스 예측에 적용하면 아래 그림과 같은 구조를 가집니다.

![ProcessTransformer](https://d3i71xaburhd42.cloudfront.net/06fd337667d8fd38f6685e146adddc03fc8ce6dc/7-Figure2-1.png)

ProcessTransformer에서는 기존 Transformer 구조의 인코더 부분만 사용합니다. 위치 정보를 가진 벡터를 입력으로 가지며, 예측 결과로 모델은 다음 활동이 일어날 확률과 발생할 시간, 진행중인 프로세스의 남은 시간을 출력합니다. 따라서, 전체 구조는 다음과 같습니다.

    < 전체 구조 >
        1. 단어 사전 생성
        2. Word Embedding + Positional encoding
        3. Multi-head Self-attention
        4. 결과 출력

예측 결과 평가지표로는 다음 활동 예측에 대해서는 accuracy를, 다음 활동이 발생할 시간 예측과 프로세스의 남은 시간 예측에 대해서는 MAE(Mean Absolute Error)를 사용합니다. 

지금까지 프로세스 예측에서 사용되는 시계열 모델들에 대해 알아보았습니다.

----

> 참고 문헌
* [PBPM](https://dl.acm.org/doi/10.1145/3301300)
* [Seq2seq](https://gaussian37.github.io/dl-concept-attention/)
* [PBPM_LSTM](https://link.springer.com/chapter/10.1007/978-3-319-59536-8_30)
* [Transformer](https://gaussian37.github.io/dl-concept-transformer/#transformer%EC%9D%98-%EC%9E%85%EB%A0%A5%EA%B3%BC-%EC%B6%9C%EB%A0%A5-1,"Transformer")
* [MAE](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=heygun&logNo=221516529668,'MAE')
* [ProcessTransformer](https://arxiv.org/abs/2104.00721)
