---
title: "Deformable DETR"
last_modified_at: 2023-07-31
author: Gun-ha, KANG
---

> 이번 포스팅에서는 *Deformable DETR**에 대한 내용을 공유해보려고 합니다.


# **Deformable DETR**


> **개요**  

* **DETR 의 한계**

  + Slow Convergence
    + 이미지의 모든 픽셀에 대한 Attention 연산 수행으로 인해 Attention Weight가 처음에는 Uniform한 분포를 가지게 되고, 특정 영역에 집중하고 수렴하는 데에 오랜 시간이 걸림

  + Limited Feature Spatial Resolution
    + DETR은 고정 크기의 feature map으로 인코딩하므로 작고 미세한 객체들은 낮은 spatial resolution으로 인해 제대로 표현되지 않으며, 탐지 성능 저하 및 객체의 크기가 feature map의 cell 크기보다 작을 때 정보 손실이 발생하여 작은 객체 탐지가 어려움



* **Deformable DETR 제안**

  + Deformable attention mechanism
    + 픽셀들의 위치를 조정 가능하게 하여, 객체의 형태나 크기에 잘 적응하도록 설계
    + 작은 객체 더 정확히 탐지
    + Limited Feature Spatial Resolution 문제 개선
    + 정확한 객체 경계 인식 및 분리가 가능해짐



## **Method**

> **Overview**
 
* Deformable Attention Module 제안
  + Key Points Attention은 어텐션을 계산할 때 여러 해상도 의 특징맵에서 몇 개의 핵심 특징점들로 attention 수행
    + Deformable Convolution 처럼 sampling location 을 정해서 attention 수행  

* Multi-Scale 의 feature map 활용 (FPN 효과)
  + 작은 객체부터 큰 객체까지 효과적 처리

![ex1](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/bc1032e6-34a0-4b69-b4db-82f5b614728d)


> **Deformable Attention Module**

* 모든 픽셀에 대한 attention을 수행하는 대신, 독립적인 linear layer에 통과시켜 Sampling offset, attention weights를 얻게 되며 이들을 이용하여 attention 연산을 수행
* linear layer를 통해 구한 attention weight를 이용
  + 주어진 feature map 에 대한 중요도 정보 계산
* 해당 attention weight를 앞서 구한 sampled points(픽셀들)의 feature들과 가중합하여 attention value를 계산


![ex2](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/f42a5419-cb89-4578-a4fe-1012d0cdb5bc)


* Equation) Deformable Attention Module

![ex3](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/60ffe0a1-449a-4f8f-8d90-e7e5da8e96c4)

* Equation) Multi-scale Deformable Attention Module

![ex4](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/6cb45729-a5c0-431f-ad9b-21ada2faa948)



## **Experiments**

> **DETR 과의 비교**

* 성능은 유사하나, 훨씬 적은 학습 시간이 소요됨 
  + (Training GPU hours) 7000 --> 325


![ex5](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/90a2f5bf-4b29-467a-92fb-387b7758638f)



> **참고 자료**  

* [[1] Deformable DETR: Deformable Transformers for End-to-End Object Detection](https://arxiv.org/abs/2010.04159)  

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>









