---
title: "Bias Correction"
last_modified_at: 2022-08-30
author: 김혜원
---

이번 포스팅은 Adam Optimizer에서 사용된 Bias Correction에 대해 알아보겠습니다.

---

>## Adam Optimizer


Adam은 AdaGrad 기법과 RMSProp 기법의 장점들을 합친 최적화함수로, moment를 계산하는 부분과 bias correction을 진행하는 부분으로 구성되어있습니다. 

앞선 기법들을 간단히 설명하자면 AdaGrad는 과거의 gradient 변화량을 참고하여 많이 변화한 변수들은 stepsize를 작게 주고, 변화가 거의 없는 변수들은 stepsize를 크게 주는 방법입니다. 이 방법은 학습이 반복될수록 stepsize가 무수히 작아지는것이 단점인데 이를 해결하기 위해 나온 방법이 RMSProp과 같은 지수이동평균을 적용한 것입니다.


두 방법이 합쳐진 Adam의 알고리즘은 지수이동평균(Exponential Moving Average, EMA)을 사용하여 가중치를 업데이트합니다.


![adam_algorithm](https://user-images.githubusercontent.com/37034031/57378858-98b32100-71e0-11e9-92c9-62c20e9a167e.png)

alpha는 학습률, θ는 가중치, beta_1과 beta_2는 모멘트 추정의 지수감쇠율, f(θ)는 가중치 θ가 주어졌을때 최적화 할 손실함수 t는 학습 횟수, g_t는 gradient를 말합니다.
지수이동평균을 이용하여 첫번째 모멘트(m_t) 추정값과 두번째 모멘트(v_t) 추정값을 구합니다. 

두 모멘트의 초기값이 0으로 설정되어 있기때문에 초기 추정값이 0으로 수렴하며 편향될 수 있습니다. 이 초기 편향은 gradient에 곱해진 (1-beta_1)와 (1-beta_2)로 나눠줌으로써 쉽게 상쇄되고, 이 과정을 Initialization Bias Correction이라고 부릅니다.

Bias Correction까지 진행한 후에 가중치를 업데이트하는데 이때 분모가 0이 되는 경우를 피하기 위해 ε을 더해줍니다. Bias Correction은 계산해보면 초기에만 영향을 미치며, 논문에는 beta_1=0.9, beta_2=0.999가 최적화값으로 나와있습니다.



------
참고문헌
* [Adam optimizer 논문](https://arxiv.org/abs/1412.6980,"Adam_optimizer")
* [Adam algorithm image](https://www.wenyanet.com/opensource/ko/6053c228a613963f8a03b297.html,"Adam_algorithm_image")
* [Adam optimizer 논문 리뷰](https://dalpo0814.tistory.com/29,"adam_paper_review")
* [Adam algorithm 설명](https://towardsdatascience.com/adabelief-optimizer-fast-as-adam-generalizes-as-good-as-sgd-71a919597af,"Adam_algorithm_explain")