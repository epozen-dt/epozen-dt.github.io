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
[4.이상치 판별 기준 설정](#이상치-판별-기준-설정) </br>
[5.성능지표 결과](#성능지표-결과) </br>


## 개요
Neighbor Convolution AutoEncoder(이하, NCAE)는 기존 Convolutional AE 모델에 새롭게 제안하는 Neighbor Convolution 기법을 적용한 모델입니다. </br>
또한, 이상탐지 모델이기 때문에 이상치(Anomaly Score)에 따른 정상과 비정상 데이터를 구분하는 기준선이 필요합니다. 하지만, 



## NCAE 구성



* 코드 작성
  + 라이브러리 선언
  ```
  import tensorflow as tf
  ```
