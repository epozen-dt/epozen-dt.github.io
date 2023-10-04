---
title: "Attention UNet"
last_modified_at: 2023-08-31
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **Attention UNet**을 공유해보려고 합니다.

# **개요**

> **배경**  

의료 이미지 분석에서 시작했습니다. 방사선 전문의들에게 있어 이미지를 해석하는 것은 특정 병을 찾기 위해 정확히 어디를 봐야하는지 아는 것에서 비롯하며, 이에 입력값의 특정 부분에 집중하는 attention 의 개념을 가져와 적용했습니다. 


## **Attention U-Net**

### **Attention U-Net segmentation Model**

Attention 의 개념을 U-Net 모델에 적용한 것으로, skip-connection 부분에 Attention Gate를 적용합니다.   
Latent vector 를 Query / Skip connection 을 Key 로 Attention 을 사용한 방법입니다.

* 가장 낮은 수준의 피쳐맵에서는 게이팅 사용 X
  + 높은 차원 공간에서 입력 데이터를 나타내지 않음
* 각 skip connection 에 대한 게이팅 신호가 여러 이미징 스케일에서 정보를 집계

![0](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/4878d529-d6f7-4a5e-a51e-f609fd6cd907)


### **Attention Gate (AG)**

Attention Gate가 중요하다고 판단하는 이미지의 영역들을 확률적으로 제안하며, 이를 통해 특정 영역에 Attention 할 수 있게 됩니다.  

* 이미지 그리드 기반의 Gating을 통해 attention coefficients를 지역적으로 특정
* g: 게이팅 신호 (이전 레이어의 벡터)
* xl: skip-connection

![1](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/4f7d2413-09be-432c-9893-9dbd0f24e812)

각기 다른 에포크(3,6,10,60,150)에서의 어텐션 계수를 보여주는 그림입니다. 췌장, 신장, 비장에 대한 학습과정을 시각화한 그림으로 보입니다. 모델이 시간이 지남에 따라, 특정 부분에 집중하도록 학습되며 제대로 식별하려고 하는 모습을 볼 수 있습니다.

![5](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/1ac16f1c-af17-4d11-96f5-ab2d39d81c71)

### **Attention Gates for Image Analysis**

gating signal g는 모든 이미지 픽셀에 대한 벡터가 아닌, 이미지 그리드 신호입니다.

gating signal g는 xl의 연산(conv->maxpooling->upconv)로부터 온것이고, 다시 xl을 사용하여 attention을 구하므로 self-attention의 관점으로 볼 수 있습니다.

초기 레이어에서는 원본 이미지의 고해상도 특징을 추출, 레이어가 깊어질수록 특징 맵의 크기가 작아지며, 이는 이미지의 고수용 영역 정보를 더 넓은 범위에 적용하기 위해 특징 맵의 크기를 축소하여 문맥 정보를 효과적으로 포착하려는 것으로 보입니다.

![2](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/5dbcfd81-eb4a-44ae-8ca9-80128ff0efc9)

이미지가 아닌, Filter에 대한 연산을 진행했다는 점이 중요한 것 같습니다.

![7](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/4c49ad6f-2d4b-4f42-a278-d63124c112b1)

왼쪽 그림은  (a~c), (f~h) 는 복부 CT의 여러 관점의 view를 의미합니다. (d~e), (i~j)은 skip-connection 의 gating 이전과 이후에 대한 그림으로 게이팅을 통해 특정 부분에 어텐션을 수행하고 있다는 것을 알 수 있습니다.

오른쪽 그림은 (a) Ground truth / (c) UNet / (d) Ours(Attention-UNet) 입니다.
가장 잘 포착하는 것을 볼 수 있습니다.

![3](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/de0f13e7-5113-4d0e-aa86-0249f5bd0362)


### **컨볼루션 매개 변수 업데이트 규칙**

Convolution 의 역전파과정에서 attention 이 적용된 형태의 체인룰을 나타낸 것입니다.
일반적으론 출력부터 입력까지의 모든 단계의 미분값을 곱해 최종 입력에 대한 gradient를 계산한다면, 이 식은 어텐션 가중치($\alpha_i^l$)과 이전 레이어의 출력함수와 곱하여 최종 gradient를 계산합니다. 이렇게 곱함으로써, 특정 입력의 중요성을 조절하는 것 같습니다.

![4](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/41cbf534-b610-4bf9-8c58-70fcbd66fd92)


### **Segmentation Experiment**
  
segmetation task에서 좋은 성능을 보였습니다.

![6](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/a89fb9f8-7c96-4d11-87cb-97107813df64)


### **Contribution**
  
정리하면 attention gating module을 제안하며, 이 모듈은 CNN 아키텍처와 통합하여 사용될 수 있다는 점과 이때 계산되는 비용이 최소한으로 유지된다는 점이 이 논문의 큰 강점인것 같습니다.

* 로컬 영역에 특화된 attention coefficients 를 가능하게 하는 그리드 기반 게이팅을 제안
* 의료 영상 작업에 적용된 FeedForward CNN 모델에서 Soft-attention 기술이 처음 사용된 사례
* 기존 U-Net 모델의 확장으로 heuristic한 작업 없이도 픽셀에 대한 모델의 민감도를 향상시킴





> **참고 논문**  

* [Attention U-Net: Learning Where to Look for the Pancreas](https://arxiv.org/abs/1804.03999)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
