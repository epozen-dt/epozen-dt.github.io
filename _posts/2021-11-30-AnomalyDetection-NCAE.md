---

title: "스마트 제조 이상탐지 모델 - NCAE 구현"
date: 2021-11-30

---
> 현재 생산품에 대한 양/불량 판정 알고리즘 구축에 대해서 연구를 진행하고 있습니다. </br>
> 이번 포스팅에서는 **자체 이상탐지 모델인 Neighbor Convolution AutoEncoder 를 구현한 방법**을 공유해보려고 합니다.



## 목차 </br>
[1.개요](#개요) </br>
[2.NCAE 구성](#NCAE-구성) </br>
[3.Neighbor Convolution Layer 구현](#Neighbor-Convolution-Layer-구현) </br>
[4.구현 확인](#구현-확인) </br>


## 개요
Neighbor Convolution AutoEncoder(이하, NCAE)는 기존 Convolutional AE 모델에 Neighbor Convolution 기법을 적용한 모델입니다.

> **NCAE 모델구성도**
![모델구성1](https://user-images.githubusercontent.com/92897860/143972463-5ff03959-b345-4863-820b-c0651c39b9f7.png)

> **Neighbor Convolution 기본 아이디어**
![아이디어](https://user-images.githubusercontent.com/92897860/143973855-850901a9-db40-4865-a59d-4679a10cd18e.png)


## Neighbor Convolution Layer 구현
* 0. 가중치 생성
  ```
  import tensorflow as tf
  ```
* 1. 원본 이미지에 패딩 추가
  ```
  import tensorflow as tf
  ```
* 2. 8채널 이미지 생성
  ```
  import tensorflow as tf
  ```
* 3. pointwise Convolution 수행
  ```
  import tensorflow as tf
  ```
  
## 구현 확인
