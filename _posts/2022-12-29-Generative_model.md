---
title: "Generative_model"
last_modified_at: 2022-12-29
author: 김혜원
---

이번 포스팅은 생성모델 중 자주 사용되는 모델들에 대한 개요입니다.

---------------
> ## 생성모델이란?

&nbsp;

생성모델은 랜덤 변수로부터 모델이 학습 데이터의 분포 자체를 학습하여 그와 유사한 분포를 가지는 데이터를 생성하는 모델입니다. 생성모델의 종류에는 GAN(Generative Adversarial Network), VAE(Variational Auto-Encoder), Normalizing flow model 등이 있습니다. 각 모델 구조는 아래 그림과 같습니다.

&nbsp;

* GAN

![GAN](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWlCwU%2FbtrHKdREkbz%2FbsysnUDZzbB5AgGhhG0ZTK%2Fimg.png)

&nbsp;


GAN은 생성기(Generator)와 판별기(Discriminator)로 이루어져 있습니다. 생성기는 노이즈 변수로부터 최대한 실제 데이터와 유사한 데이터를 생성하여 판별기를 속이려 하고, 판별기는 실제 데이터와 가짜 데이터를 구별합니다. 이러한 경쟁을 통해 학습을 하면서 생성기가 실제와 매우 유사한 데이터를 생성하게 됩니다. 하지만 GAN에는 Mode-collapse 라는 한계점이 존재합니다. Mode-collapse(아래 그림)는 모델이 국소점에 빠져 더이상 학습이 진행되지않는 상태로, 생성기가 만든 데이터가 실제와 매우 흡사한 경우 혹은 생성기가 너무 쉬운 데이터만 생성하는 경우 등 판별기가 가짜 데이터를 실제 데이터라고 판별하는 것을 말합니다.

&nbsp;

![mode-collapse](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEVS1X%2FbtrHL1XJQvG%2FM0lZABrbGSuUAXWz2Q41PK%2Fimg.png)

&nbsp;

* VAE
![VAE](https://user-images.githubusercontent.com/24144491/50323466-18d03700-051d-11e9-82ed-afb1b6e2666a.png)

&nbsp;

VAE는 Encoder-Decoder 구조로 Encoder의 출력에서 평균과 표준편차를 각각 구한 후 랜덤 노이즈와 연산하여 구한 잠재변수 Z_i를 Decoder의 입력으로 이미지를 생성합니다. 이 때, 예측 이미지는 입력 이미지와 똑같이 생긴 이미지가 생성됩니다.

&nbsp;

* Normalizing flow 
![NF](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWuLzZ%2FbtqOCdSfy7h%2FQw8uu14U79cpBBRAzpHV01%2Fimg.png)

&nbsp;

Normalizing flow는 flow-based generative model이라고도 부르는데, 어떤 쉬운 분포 z에 f 함수를 연산하여 복잡한 확률분포 p(x)를 만듦으로서 새로운 이미지를 생성합니다.

위의 Normalizing flow와 VAE를 합쳐서 만들어진 모델이 Diffusion model 입니다.

&nbsp;

이 포스팅은 생성모델들의 구조에 대해 간략하게 정리한 것이므로 자세한 사항은 논문을 참고하여 주세요.



------
> 참고문헌

* [GAN](https://process-mining.tistory.com/169)
* [GAN_논문](https://arxiv.org/abs/1406.2661)
* [VAE](https://taeu.github.io/paper/deeplearning-paper-vae/)
* [VAE_논문](https://arxiv.org/abs/1312.6114)
* [Normalizing_Flow](https://judy-son.tistory.com/12)
* [Normalizing_Flow_논문](https://arxiv.org/abs/1505.05770)