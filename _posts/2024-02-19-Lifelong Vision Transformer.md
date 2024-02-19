---
title: "Lifelong Vision Transformer"
last_modified_at: 2024-02-19
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **Lifelong Vision Transformer**를 공유해보려고 합니다.


# **1. 개요**

1. **Vision Transformer의 한계**
    - CNN 계열보다 inductive bias가 적기에 더 많은 데이터를 필요로 함
    - 데이터 개선을 위해 CoAT와 같이 Conv-attention Module이 제안되기도 했으나, 부족함

2. **지속적인 학습**
     - 새로운 작업을 학습할 때, 이전 작업에서 추출한 샘플을 다시 사용하는 회상(Rehearsal) 적용  
     - 이전 작업에서의 경험을 재생(replay)함으로써 잊어버리기를 완화  

3. **활용 부재**
     - Continual Learning은 주로 CNNs 기반이나, Vision Transformer에서는 활용 X



### **기반 아이디어**

* **QKV 중 (조건, 상황에 의한) K의 변화**
  + Query, Value = 현재 모델의 입력 Query, Value
  + Key = 이전 모델의 Key
    + 외부 Key의 정보를 참조하여 입력 요소의 중요성을 결정

* External Attention 예시[1]
![1-removebg-preview](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/82499949-0f7e-42c7-b3a6-152152d5f3b2)


* **이전 모델에 대한 외부키를 사용함으로써, catastrophic forgetting 극복하고자 함**  


# **2. LVT 모델**

* Inter-task attention + Dual classifier

![2](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/46884700-1b54-4557-814b-6178f3817cac)


### **2.1 Inter-task attention**

* 이전 학습 모델의 Key(외부)를 사용함으로써, 이전모델과의 상호작용을 수행
  + 결과, 입력에 대한 선형 변환을 대체하기에 self-attention 에 비해 파라미터 수를 줄일 수 있음
* self-query 와 학습 가능한 외부 키 $K_W$ 간의 유사성을 계산

![3](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/c41b21bb-1798-4d70-9255-91fa3f64357a)


### **2.2 Dual Classifier**

* **Catastrophic interference 현상 방지**
  + 하나의 분류기 사용 시, 이전에 학습한 데이터에 대한 정보를 덮어쓰는 현상이 있다고 함

1. **주입 분류기(Injection):** 새로운 task에 대한 학습
2. **누적 분류기(Accumulation):** 메모리에 저장된 이전 데이터를 사용하여 학습
    + 목적: 이전에 학습한 데이터에 대한 지식을 유지하는 것


![8](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/0be49dba-eb9c-4f1e-bcd6-84f2b794f662)

* 동일한 분류기를 사용하여 (이전+새로운) 작업 학습을 진행 시, 이는 catastrophic interference 가 발생할 가능성 존재


### **2.3 손실함수**

![4](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/822012cb-c345-4169-bb43-92bd23138071)


### **2.4 메모리 버퍼**

* 예전 데이터(exemplar)를 메모리 버퍼에 저장     
  + 새로운 데이터를 학습할 때, 이를 활용 (forgetting을 막는 전략)
  + 이를 현재 task의 데이터와 함께 사용, 모델을 업데이트

* **버퍼를 어떻게 업데이트하는가**
  + 각 클래스 당 샘플 수 제한
    + Fixed Memory: 최대용량 고정
    + 모든 클래스에 대한 equal representation을 보장하기 위해 클래스별 동일한 sample 수의 exemplar가 저장되는 것을 강요
  + 소프트맥스 결과가 좋은 순으로 샘플 내림차순
    + confidence-aware sampling

![5](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/abd55771-8862-47ff-83f5-ee7fc6123f8f)


# **3. Implementation**

### **3.1 CL(Continual Learning) 시나리오[3]**

* **순차적인 Task Structure 구성을 의미**
* 총 3가지가 존재하나, 해당 논문에서는 2가지만 사용
  + Task Incremental(Task-IL): 현재 작업에 대한 분류 성능 측정
  + Class Incremental(Class-IL): 모든 이전 작업 + 현재 작업에 대한 분류 성능 측정

![9](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/6c0e2fe8-a9ad-4f0a-ad2f-03a4d330aa93)

### **3.2 SetUp**

* 가정) CIFAR 100
  + Class 100개 / 샘플당 학습 500개 / 샘플당 검증 100개 / 5~15split / Memory size 200, 500
  + Class-IL 구성) 클래스 수 순차 증가 (10splits 기준, 클래스 2개씩, 랜덤 혹은 유사도에 따른 분할) 
  + Task-IL 구성) 작업 수 순차 증가 (Class-IL과 동일, 단 평가 시 조금 다름)

![10](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/c5209169-3a28-4989-b095-b49a96d9c4df)

### **3.3 평균 정확도(accuracy)[3]**

* Task-IL: 현재 작업에 대한 정확도 (그림의 빨간 부분) 
* Class-IL: 모든 이전작업 + 현재 작업의 정확도 (그림의 빨간+보라색 부분)

![11](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/f23fb08e-eec2-48e3-82bd-33e674c26971)

### **3.4 평균 망각(forgetting)[3]**

* 특정 작업을 학습한 후, 그 이전 작업들에 대한 정확도 변화를 측정하여 망각으로 사용
* 정확도가 제대로 나와야 망각 또한 제대로 해석할 수 있음
  + 확도가 낮아도, 망각은 낮게 나올수 있음 (그렇다고, 이를 이전 지식을 잘 유지했다고 보기 어려움)
* 낮을수록, 모델이 새로운 작업을 학습하면서도 이전 작업의 지식을 더 잘 보존한다는 것을 의미

![12](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/18bc324e-f789-4d61-ad18-7ef2136ba750)

### **3.5 평가 결과 예시**

* 지금의 경우, 이전 학습한 정보를 많이 잊어버림 (catastrophic forgetting 발생)
* 일반적으로, Class-IL의 성능 난이도가 높음 

![13](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/2a73370a-207e-4fba-89a8-5ebbbf45bdcb)


### **3.6 논문 설정값**

* pretrain 여부는 모름 (안한 것으로 보임)
* SGD optimizer
* 분류 손실 CrossEntropy
* CIFAR100 기준) epoch 50, mini-batch 32, lr 0.1
* 모델의 하이퍼파라미터 최적화를 위해 훈련 데이터의 10%를 사용하여 검증 세트를 만들고, Gride Search를 통해 최적의 설정을 찾음


# **4. Experiment**

> **1. Results (overall accuracy %) on CIFAR100 benchmark**

* 적은 파라미터 갯수로 좋은 성능을 보임
* CL task에서 잘 동작한다는 점

![6](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/d653ca9e-b84c-423c-a272-44b8ca66b9ed)


> **2. Forgetting results (%) on CIFAR100 benchmark**

* LVT의 메모리 크기가 500일 때, 더 적은 잊어버림을 겪는다는 것
* CNN 계열(ResNet18)과 ViT계열(ViT, LeViT, CoaT, CCT)의 비교
  + LeViT, CvT, 및 CCT는 inductive biases를 얻기 위해 CNN 구조를 포함하나, catastrophic forgetting이 발생
* ViT가 연속 학습 작업에 적합 X, 대규모 데이터셋에만 적합한 data hungry 특성


![7](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/4c0d9115-bda8-4c78-ac8a-20886e12824d)



# **5. Conclusion**

> **Conclusion**

* 지속적인 학습을 위한 비전 트랜스포머를 설계한 문헌 최초의 논문
* 더 적은 매개 변수로 효율적인 학습을 수행 (8.9M)
  + 외부 키 사용
* Inter-task attention + Dual-classifier 구조로 CL에 적합하도록 구성
  + CL(Continual Learning)을 위한 Stability-Plasticity dilemma 

> **Critique**

* BERT와 같이 Task-specific token을 사용하여 쿼리를 조정하거나, Query-selected Attention을 통해 Query를 선택하는 방법 등 Q의 변화들이 존재하는데 LVT에서는 K의 변화를 통해 모델의 성능을 향상시켰다는 점



> **참고 논문**  

* [[1] Beyond Self-attention: External Attention using Two Linear Layers for Visual Tasks](https://arxiv.org/abs/2105.02358)
* [[2] Continual Learning with Lifelong Vision Transformer](https://openaccess.thecvf.com/content/CVPR2022/papers/Wang_Continual_Learning_With_Lifelong_Vision_Transformer_CVPR_2022_paper.pdf)
* [[3] Online Continual Learning in Image Classiﬁcation: An Empirical Survey](https://arxiv.org/abs/2101.10423)


---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
