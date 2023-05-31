---
title: "Modern Hopfield Network"
last_modified_at: 2023-05-31
author: Gun-ha, KANG
---

> 이번 포스팅에서는 "Hopfield Networks is All You Need" 논문 중 **Modern Hopfield Network**에 대해 일부 내용을 공유해보려고 합니다.


# **Modern Hopfield Network**

> **기존 홉필드 네트워크의 한계**  

* 패턴을 저장할 수 있는 용량 제한
  + 패턴 저장 용량이 대략 0.15N,
  + 10x10 크기의 이미지에 대해서 15개의 이미지를 넘어서면 수렴이 잘 안됨을 의미
* 스퓨리어스 상태(Spurious state) 존재
  + 고정점으로 수렴하지 않는 상태
* 느린 수렴 속도
  + 상태가 변하지 않을 때까지 진행
  + 이때, 업데이트가 잦음


> **제안**

* 연속 상태를 이용한 에너지 함수의 일반화
* 새로운 업데이트 규칙 제안

![1](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/968e2c1b-932e-4559-a6d4-2d399209bb26)


> **에너지 함수**

* 에너지 함수를 최소화하는 방향으로 패턴 조정
  + $X$: 저장된 패턴
  + $\xi$: 상태 패턴 (입력 쿼리)
  + 이때, 입력 쿼리만 바뀌며, 저장된 패턴은 고정되어있음
* 첫 항: LSE(Log-Sum-Exponential Function)을 활용하여 유사도 계산
* 셋째 항: 엔트로피

![2](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/59901767-14f7-4476-ba0c-fa3405d082d9)

> **연속 상태에 대한 업데이트 규칙**

* 업데이트된 상태는 다음 단계의 입력으로 사용
  + iteration 마다 변하는 값은 $\xi$
  + $\xi$ 와 $X$ 를 기반으로 패턴 간 유사도 계산하고, 이를 확률분포로 변환
* 반복 적용함으로써, 저장된 패턴에 수렴하고 복원됨


![3](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/5ed34ff2-a203-4ea1-98c2-a7df9a52787c)


> **Golbal Convergence(Zangwill): Energy Function**

* 업데이트 규칙 반복 시, 입력 $\xi$ 의 에너지 $E(\xi^t)$가 무한히 반복되면 최소 에너지 상태 $\xi*$ 로 수렴함을 보장
  + 에너지 함수를 최소화한다는 것은 고정점을 찾는 과정

![4_1](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/742136f6-fdaa-4fd2-8cda-ac56a196b3f3)


* CCCP(The Concave-Convex Procedure, 주어진 문제를 부분 문제로 분해하고, 각 부부 문제를 순환적으로 최적화) 를 통해 증명
  + 업데이트 규칙을 통해 최적해에 수렴
  + 이러한 전역 수렴성은 네트워크 일관성 및 안정상태 도달을 보장함

![4_2](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/1b900bb8-d3cb-41ec-8a5e-7f7feeaa25c1)



> **Global Convergence (Stationary Points)**

* 에너지 함수는 여러 개의 지역적 최소값(정지점)을 가짐
  + 최적해의 후보들
  + 고정점은 모든 패턴에 대해 에너지가 최적으로 수렴하는 상태를 의미

![5](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/94b8efbc-1670-4732-ad72-4ea46ac1aae4)


> **Local Covergence to fixed Points**

* 저장된 패턴에 따라, 3가지 고정점 유형 중 하나를 가짐
  + (a) Stored Patterns: 저장된 정보, 특정 패턴에 수렴, 해당 패턴 유지
  + (b) Metastable State: 상대적으로 많은 패턴을 저장 및 유지
  + (c) Global fixed Point: 모든 입력에 대해 동일한 상태로 수렴
* 업데이트 시, 각 패턴은 특정 고정점으로 수렴
  + 모든 패턴이 항상 수렴하진 않음, 실패확률이 존재
* 고정점이란, 업데이트 규칙에 따라 변하지 않는 상태로 해당 패턴에 수렴함을 의미
  + 저장된 패턴은 각각 네트워크 상태공간에서의 고정점으로 간주


![6](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/59c18f09-9422-4056-b7ca-8bf94be61dd2)



> **Exponential Storage Capacity**

* 모즌 저장된 패턴이 반지름 M 을 가진 구 위에 있다고 가정
  + 패턴을 구의 형태로 표현
  + 반지름 M 은 해당 패턴의 크기(norm)으로 고정
  + 주어진 쿼리 ($\xi$) 가 해당 구 안에 있을 때, 그 구의 패턴으로 수렴

![7](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/510b8e66-e339-45ce-b83b-c89db5c385d2)

* 차원에 따라 저장할 수 있는 패턴의 갯수가 있음
  + $N$: 패턴의 갯수
  + $d$: 벡터가 존재하는 차원
* 홉필드 네트워크에서 얼마나 많은 패턴을 저장할 수 있는가를 의미

![8](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/7f4d5bf7-2062-4584-ab26-baedca1e64f2)

* 실패 확률과 차원에 따라 저장 가능한 패턴의 갯수에 대한 하한을 제공
  + 실패 확률: 저장된 패턴 검색X, 왜곡될 확률을 의미

![9](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/fd44e90c-eb6e-4422-971b-bd7e1be20bca)


> **Retrieval With One Update**

* 패턴이 잘 분리된 경우, 한번의 업데이트 후에 수렴
  + 분리도: 저장된 패턴들을 구분하고 유지할 수 있는 능력
  + 새로운 점과 고정점 사이의 거리가 기하급수적으로 작아짐 (분리도 높음)
* (6)과 (7)은 정량화된 거리 수식

![10](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/a7537617-bd80-49cb-aae4-e595ad0ee520)

> **모든 홉필드에서의 어텐션 메커니즘**

* Self-Attention 과의 개념적 유사성을 의미
  + 주어진 쿼리와 저장된 패턴 간의 유사도 기반으로 계산
  + 업데이트 시, 특정 패턴에 더 큰 가중치 부여

![11](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/773777fd-75b2-4a49-a7da-03abaab8f601)

* 참고) 업데이트 규칙과 트랜스포머의 어텐션 메커니즘

![12](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/6d97b78a-2978-48b6-8c68-b4c738f88cd7)


> **참고 자료**  

* [[1] Hopfield Networks is All You Need](https://arxiv.org/abs/2008.02217)  
* [[2] Hopfield Networks is All You Need - Blog](https://ml-jku.github.io/hopfield-layers/)  
* [[3] Hopfield-layers](https://github.com/ml-jku/hopfield-layers)  

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
