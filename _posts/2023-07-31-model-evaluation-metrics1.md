---
title: "model-evaluation-metrics(1)"
last_modified_at: 2023-07-31
author: 김혜원
---

본 포스팅은 모델 성능 평가 지표에 대한 내용입니다.

---
&nbsp;

> ## 1. 모델 성능 평가 지표란?

&nbsp;

모델 성능 평가 지표는 모델을 검증할 때 쓰는 지표로, '학습이 얼마나 잘 됐는가?' 를 수치로 표현하는 것입니다. 본 포스팅에서 다룰 지표는 PSNR(Peak Signal-to-Noise Ratio, 최대 신호잡음비)과 SSIM(Structural Similarity Index) 입니다. 

손실함수로 MAE(Mean Absolute Error, 평균 절대 오차)와 MSE(Mean Square Error, 평균 제곱 오차)를 많이 사용합니다. 이 둘은 원본 이미지와 생성 이미지 사이의 오차를 구한 것으로, 각각 절댓값과 제곱을 취한 후 평균을 구한 것입니다. 

PSNR에서는 평균 제곱 오차를 이용하여, 화질을 평가합니다. 이 때, 수치가 높다고 모두 좋은 것은 아닙니다. 아래 사진을 보면 앵무새 이미지가 더 고화질이라고 나왔지만 사람이 보기에 나비 사진이 더 깔끔하게 느껴지는 것을 볼 수 있습니다.


![psnr_example](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbpq1py%2FbtrIFIRohyu%2FW2VxAALnGYIA48TqcULcK0%2Fimg.png)

이를 개선하기 위해 사람의 시각이 이미지를 받아들이는 과정을 반영한 지표가 SSIM입니다. 휘도(빛의 밝기), 대비(밝기 차이), 구조(상관 관계)를 이용하여 평가합니다. 아래 이미지를 보면 나비 이미지가 더 높은 수치를 가지는 것을 확인할 수 있습니다.

&nbsp;

![ssim_example](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcl6PkR%2FbtqzCqKjn5k%2FseitDBrmj3ufFuJAG4Pol1%2Fimg.png)

&nbsp;

> ## 2. 공식

각 지표의 값들은 하단 공식들을 통해 구할 수 있습니다.

&nbsp;


- MAE

![MAE](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbNOy7E%2FbtqT8JpElnX%2FLk7It3KZh2GO9zI5K8r1xk%2Fimg.png)




- MSE

![MSE](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdhdU5P%2FbtqT18xfjoP%2FWIE89gk9NlNCxrOJIsrv4k%2Fimg.png)

    * 요소
    N : 이미지 수
    y : 원본 이미지
    y^ : 생성 이미지

&nbsp;

- PSNR

![PSNR](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcnamdn%2FbtrINLsJOhe%2FsyvfLkRelLaoPHcASdjjx1%2Fimg.png)

    * 요소
    MAX : 255 = 픽셀 최대 값

&nbsp;

- SSIM

![SSIM-1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbdQ28x%2FbtrIMBdswp4%2F4klTVYTTkfKZhguVx6BC10%2Fimg.png)

![ssim-2](https://github.com/khw927/epozen-dt.github.io/assets/107157737/747fb66b-89f7-4ac2-b479-956e0958ceaf)

    * 요소

    𝑥 : 실제 이미지
    𝑦 : 예측 이미지
    𝜇 : 평균
    𝜎 : 표준 편차
    𝜎_( 𝑥𝑦) : 픽셀 값 변화량
    𝐶_1 : 〖(0.01∗𝐿)〗^2 = 6.5025
    𝐶_2 : 〖(0.03∗𝐿)〗^2 =  58.5225
    𝐶_3 : 𝐶_2/2 = 29.2613
    L : 255 = 픽셀 최대 값
    𝛼, 𝛽, 𝛾 : 각각에 대한 가중치

&nbsp;

------
> 참고

[1] [loss 식](https://bo-10000.tistory.com/44)

[2] [PSNR 설명 참고](https://dbstndi6316.tistory.com/376)

[3] [예시 이미지](https://bskyvision.com/392)
