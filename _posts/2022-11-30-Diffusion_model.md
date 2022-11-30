---
title: "Diffusion_model"
last_modified_at: 2022-11-30
author: 김혜원
---

이번 포스팅에서는 Diffusion model에 대하여 공유하려고 합니다.


---

>## Diffusion model?

&nbsp;

Diffusion model(확산 모델)은 생성모델의 일종으로 image generation, image-to-image translation, image segmentation 등 다양한 목적으로 사용합니다. [[1]](https://arxiv.org/abs/2209.04747) 모델의 구조는 markov chain 형태이며, diffusion process와 reverse process로 구성되어 있습니다.

    <Markov chain>
    Markov chain은 markov 특징을 가진 이산 확률 과정입니다. markov 특징이란 이전 상태와 상관 없이 '현재' 상태만 고려하여 미래 상태를 결정하는 것을 말합니다. 다시 말해, 이산적인 시간(1초, 2초 등)에서의 확률이 변화하는 것을 이용하여 미래 상태 값을 예측하는 것입니다.
&nbsp;

![Diffusion_process](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvwAn1%2FbtrNwkjFAn3%2FAvT141LiMsckI2XpEZtYSK%2Fimg.png)

&nbsp;

 각각의 이미지(X)는 한 단계의 chain이 되며, 현재의 이미지를 보고 다음 이미지를 생성합니다. 
 
 diffusion process는 원본 이미지(X0)로부터 gaussian noise를 조금씩 랜덤하게 추가하여 완전한 노이즈 이미지(XT)까지 생성하는 과정입니다. gaussian noise는 정규분포를 가지는 잡음으로 0~1 사이의 값을 사람이 설정합니다.
 
reverse process는 생성된 노이즈 이미지에서 denoising하는 방법을 학습하는 과정으로, 위 그림과 같이 표현합니다. 노이즈 값은 추정치이기 때문에 현재 이미지의 가우시안 분포를 기반으로 근사치를 구하여 다음 이미지를 생성합니다. 이 과정을 반복하여 원본 이미지를 찾습니다.

&nbsp;

    <각 process의 input & output>
    diffusion process : original image(input) / noise image(output)
    reverse process : noise image(input) / original image(output)

&nbsp;


원문에서는 CIFAR-10 데이터 세트로 FID(Frechet Inception Destance) score 3.17을 달성했기 때문에 작은 데이터세트로도 좋은 성능을 도출할 수 있다고 주장합니다[[2]](https://arxiv.org/abs/2006.11239). 순차적으로 노이즈를 추가하는 과정때문에 속도는 느리지만 다양한 이미지를 생성할 수 있고, 여러 개의 latent variable을 활용했다는 점에서 GAN(Generative Adversarial Networks)보다 좋은 평을 받기때문에 활용도가 좋습니다. 이 포스팅은 여기서 마치겠습니다.

&nbsp;

---
> 참고문헌

&nbsp;

[1] [Diffusion Models in vision : A Servay](https://arxiv.org/abs/2209.04747)

[2] [Denoising Difussion Probabilistic Models](https://arxiv.org/abs/2006.11239)

[3] [Diffusion model 설명1](https://process-mining.tistory.com/182)

[4] [Diffusion model 설명2](https://theaisummer.com/diffusion-models/#approximating-the-reverse-process-with-a-neural-network)


