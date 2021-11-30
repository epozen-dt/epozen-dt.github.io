---

title: "스마트 제조 이상탐지 모델 - NCAE 구현"
date: 2021-11-30

---
> 현재 생산품에 대한 양/불량 판정 알고리즘 구축에 대해서 연구를 진행하고 있습니다. </br>
> 이번 포스팅에서는 **자체 이상탐지 모델인 Neighbor Convolution AutoEncoder 를 구현한 방법**을 공유해보려고 합니다.

</br>

## 목차 </br>
[1.개요](#개요) </br>
[2.Neighbor Convolution Layer 구현](#Neighbor-Convolution-Layer-구현) </br>
[3.추가 확인](#추가-확인) </br>

</br>

## 개요
**Neighbor Convolution AutoEncoder(이하, NCAE)는 기존 Convolutional AE 모델에 Neighbor Convolution 기법을 적용한 모델입니다.**


> **NCAE 모델구성도**
![모델구성1](https://user-images.githubusercontent.com/92897860/143972463-5ff03959-b345-4863-820b-c0651c39b9f7.png)
AutoEncoder 기반의 모델 학습 결과는 이미지 데이터이기 때문에 이미지 데이터 자체만으로는 명확히 구분하기 어렵다.</br>
따라서, 이상치(Anomaly Score)값으로 잔차이미지에 대한 통계치, MSE, 정보엔트로피로 설정하여 양/불량 판정을 수행하였다.

</br>

> **Neighbor Convolution 기본 아이디어**
![아이디어](https://user-images.githubusercontent.com/92897860/143973855-850901a9-db40-4865-a59d-4679a10cd18e.png)
Convolution에서 중앙값을 제외하여 주변 이웃 픽셀들에 대해서만 Convolution을 수행하여 중앙값의 의존성을 제거하고 </br> 이웃 픽셀들의 가중치 학습을 강화하여 효율성을 높일 수 있지 않을까라는 아이디어에서 시작되었다. 

</br>

## Neighbor Convolution Layer 구현
**0. 입력**
  - input ([type]): (N, 300, 300, 1)의 입력 데이터
  - Kaggle의 주조 제품 입력 이미지(300, 300, 1)
  <figure>
     <img src="https://user-images.githubusercontent.com/92897860/143996629-484b319b-1e45-4ebf-8801-af223e8a6fb7.png"  width="200" height="100">
  </figure>

**1. 원본 이미지에 패딩 추가**
  - 행복하길 바래
  ```
  import tensorflow as tf
  ```
**2. 8장의 이미지 생성 **
  ```
  import tensorflow as tf
  ```
**3. 8채널 이미지로 변환**
  ```
  import tensorflow as tf
  ```
**4. pointwise Convolution 수행**
  ```
  import tensorflow as tf
  ```
  
## 추가 확인
* 1채널 이미지여야 한다.
* 가중치 생성
  ```
  import tensorflow as tf
  ```
