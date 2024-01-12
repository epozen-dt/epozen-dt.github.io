---
title: "시계열 이상탐지 - Unlearning Framework"
last_modified_at: 2024-01-12
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **Unlearning Framework**를 공유해보려고 합니다.

## **Unlearning Framework**

### **개요**

기존의 배치 학습은 시공간 비용 모두에서 낮은 효율성을 보입니다. 즉 모델을 학습하거나 업데이트하는데 소요되는 시간과 학습마다 전체 데이터를 한번에 사용하여 학습하기에 많은 메모리 공간을 필요로 합니다. 게다가 다시 학습한다는 것은 프로그램에 대한 확장성을 떨어뜨립니다.

온라인 학습은 데이터가 도착할 때마다 즉시 모델을 업데이트하므로, 전체 데이터셋을 한 번에 처리하는 것보다는 효율적으로 시간과 공간을 활용할 수 있습니다. 이는 실시간 예측이나 대규모 데이터 스트림에서 유용하며, 새로운 데이터가 도착할 때마다 즉시 반영함으로써 더 빠르게 학습하고 예측할 수 있게 합니다.


### **온라인 학습**

* 오프라인 학습  

  + 한번에 주어지는 데이터로 모델 학습 (=Batch learning)  


* 온라인 학습  

  + 데이터가 subset(mini-batch)형태로 순차적으로 모델 학습  
  + 학습에 사용된 subset데이터는 저장 불가능(다시 사용 X)  

![Screenshot_2](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/fb7ebed4-4fb1-42e0-ba13-1b739c305fc3)

* **입력 데이터의 스트림(stream)으로부터 점진적으로 학습할 수 있는가**  



### **Catastrophic Forgetting**

* 데이터, 시간의 흐름에 따라 끊임 없이 변화(성장)  
* 학습된 기존 모델에 새로운 데이터셋으로 학습 시, 데이터셋 간에 관련이 있더라도 이전 데이터셋에 대한 정보를 대량으로 손실  

![Screenshot_3](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/faa1bdde-bd09-46ab-a33e-ada4e5acaa3a)



### **Confusion Matrix**

* True Positive (TP): 정상으로 제대로 예측  
* True Negative (TN): 이상으로 제대로 예측  
* False Positive (FP): 모델이 정상인데, 이상으로 잘못 예측  
* False Negative (FN): 모델이 이상인데, 정상으로 잘못 예측  

![Screenshot_4](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/07a83aca-26b4-4edc-97c2-3e522208926f)

* **FP 와 FN을 줄이면 모델의 성능이 증가**


### **언러닝, 문제 정의**

* 새롭게 보고된 FN 과 FP 를 예측에서 발생한 오류로 간주, 해당 FN과 FP만 업데이트  
  + 새로운 인스턴스들 중 전체 데이터가 아닌, 오류만 가지고 학습   

![Screenshot_5](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/b30fa659-5bde-4780-9116-2a039e642807)



### **이상탐지의 기본원리**

* 새로운 데이터에 대한 분포를 얻으면, 이 값을 기반으로 해당 데이터의 이상유무 판별 가능

![Screenshot_6](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/eb810ab0-e224-42f6-bf7f-f1d0ba3f97fc)

* 주변 확률을 학습하여 새로운 데이터에 대한 예측을 수행

![Screenshot_7](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/3755557c-0ceb-4bbc-aaac-0fea5eb205ea)


### **이상탐지 모델 (정상 데이터로만 학습)**

* 시간 영향 O, LSTM 기반 이상탐지 수행  
* 시간 영향 X, Autoencoder 기반 이상탐지 수행  

![Screenshot_8](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/87f9f8b5-63cd-4152-afc0-6c185c897b91)



### **언러닝, 업데이트 방식**

![Screenshot_9](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/dd2d45d9-0d5b-4e32-9521-7a0c5f0dcdf4)



### **Implementation**

> Exploding loss 문제 발생

* 최적화의 목표는 손실 함수의 최소화  
* 반대로 동작하는 항이 추가되면, 원하지 않는 결과를 도출  

![Screenshot_10](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/365ac9d0-a869-46cf-9fa5-f5650959ab62)

> Preventing Catastrophic forgetting

* 업데이트된 모델을 기존 모델에 가까이 유지, 이전 데이터에 대한 그들의 성능도 근접하도록  
  + 파라미터를 이전 값 θ⋆에서, 새로운 값 θ로 업데이트  


![Screenshot_11](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/5c423db2-c873-4373-b425-267fcb302919)



### **Evaluation**

> 1. Baseline 과 언러닝(증분 업데이트) 간의 비교

* (3가지) 임계값이 증가함에 따른 FP와 FN 비교  
  + 언러닝의 효과성을 보여줌  

![Screenshot_12](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/7a5476c5-72e1-4461-82c4-f8d8ee91fb50)


> 2. 시간의 지남에 따른 Unlearning 개선 정도

* 모델은 몇 번의 업데이트만으로 크게 개선 (x=10)   
* 시간이 진화함에 따라 천천히 개선됨을 보임  

![Screenshot_13](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/f5024162-e701-4e06-b140-adc429001922)



### **Contribution**

1. 딥러닝 기반 이상 탐지 문제에 대한 lifelong learning을 최초로 검토    
    1. 기존 딥러닝 기반 zero-positive 이상탐지 알고리즘에 일반적으로 적용 가능   

2. Exploding loss 와 Catastrophic forgetting 해결  
    1. Exploding loss, 이상탐지의 고유 특성  
    2. Catastrophic forgetting, 일반적인 lifelong learning 특성  

3. FP와 FN을 크게 줄임  
    1. 이전의 연구자들은 정상데이터, FP만 활용  
    2. 시스템 이상 탐지에서는, 더 적은 FN을 갖는 것이 더 많은 FP를 허용하는 것보다 중요하다고 판단  


### **Conclusion & Critique**

> Conclusion

* 제안된 손실함수를 통해, 새로운 Negative examples의 증분 학습으로도 일반화 가능 

> Critique

* 이상탐지 모델에 대한 자세한 설명은 없음  
* 어떤 딥러닝 모델에도 적용이 가능하나, 별도의 온라인 학습 필요  
  + FN, FP만 업데이트  
  + 언러닝은 특정 패턴이나 예측에 대한 조정 또는 수정이 필요한 경우 사용  
  + lt=-1일때 정상데이터, lt=1일때 비정상 데이터로 가정하면, 온라인 학습 효과를 내지 않을까  
* 어떻게 업데이트 전략을 세울   
  + 어느 시점에 얼마만큼의 데이터로 모델을 업데이트 시키는지 정의하는 것이 중요해보임  



> **참고 논문**  

* [Lifelong Anomaly Detection Through Unlearning](https://zhichen98.github.io/data/3319535.3363226.pdf)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
