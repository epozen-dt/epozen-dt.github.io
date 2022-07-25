---
title: "의사결정 트리 - CART 알고리즘 II"
last_modified_at: 2022-6-29
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **의사결정 트리 - CART 알고리즘의 가지치기(Prunning)** 에 대한 내용을 공유해보려고 합니다.  

## **CART(Classification And Regression Trees)**    

지난번 설명에 이어 다시 정리를 해보면, 의사결정 트리의 형성과정은 크게 성장(Growing), 가지치기(Prunning), 타당성 평가와 예측으로 이루어집니다.
각 마디에서 최적의 분리규칙(CART의 경우, 이진분리)을 적절하게 찾아 트리를 성장시키는 과정과 적절한 정지규칙을 만족하면 중단하는 것을 말합니다.


> **가지치기 (Prunning)**

분리 기준에 따라 트리를 끝까지 분리 했을 때, 모델은 일반화되지 못하는 경우가 발생하기도 합니다. Fig1과 같이 과적합인 상태가 되기도 합니다. 따라서, 먼저 full tree를 생성한 뒤 가지치기를 통해 분할되었던 영역을 병합하여 노드를 잘라내는 과정이 필요합니다.  

* full tree란, Terminal node의 순도가 100% 상태인 트리를 말합니다.

![1](https://user-images.githubusercontent.com/92897860/176336227-448260e0-9d5c-41f0-8252-4131a4644cf4.png)


> **가지치기 방식**

가지치기는 크게 2가지로 사전 가지치기(Pre-Pruning)과 사후 가지치기(Post-Pruning) 방식이 존재합니다.

사전 가지치기는 말 그대로 사전에 정지규칙을 설정하여 사용합니다. 즉, 사전에 Max Depth를 설정하면 그 Max Depth만큼만 분리를 수행하게 됩니다.

사후 가지치기는 사전 가지치기와 다르게 full tree를 만든 이후에 노드를 제거(병합)하는 방식입니다.


> **사후 가지치기 알고리즘**

Ruduced Error Prunning, Rule Post Pruning 등 여러 형태의 가지치기가 존재하는데, CART에서는 Cost Complexity Pruning이라는 비용 복잡도 제거 가지치기를 사용합니다.
의사결정 트리에서의 가지치기란 분할되었던 영역을 병합하는 의미가 큰데, 이러한 병합의 기준으로 비용 복잡도를 사용한다고 보시면 되겠습니다.


> **비용 복잡도 함수**

트리가 커질 때마다 비용 복잡도가 증가하므로, 최대 크기의 트리(fulee tree)에서 그 비용 복잡도를 최소화하는 방향으로 가지치기를 하는 방식입니다.

수식은 다음과 같습니다. 

  - R(T): 학습 데이터에 대한 오분류율(오분류된 객체의 비중)
  - α : 가중치(tuning Parameter), 정규화 파라미터
  - f(T) : Leaf Node의 갯수


![2](https://user-images.githubusercontent.com/92897860/176351439-b1c73f1f-0dce-4873-811d-97a42d6e3ead.png)


비용 복잡도를 낮추기 위해서 2가지를 고려해야됩니다.  

- 같은 에러율이라면, 훨씬 더 단순한 형태의 트리를 선택
- 같은 leaf node 수라면, 에러율이 낮은 것을 선택


### **정리**  

학습 데이터를 가지고 Recursive Binary Splitting(Recursive Partioning)을 수행하여 최대 크기의 트리를 만들고, Terminal node가 100%에 가까워질때까지 분리를 합니다.

그 후에 데이터에 적합한 depth와 node를 구하기 위해서 Cross-validation을 통해 모든 subtree를 만들고 모델을 고르는 것은 비효율적이기 때문에, 만들어진 트리에 비용 복잡도 가지치기를 적용하여 Best Subtree를 모델로 고르게 됩니다. 

해당 모델은 새로운 데이터에 대해서 full tree에 비해 예측성이 더 높게 나오게 됩니다.  


## **참고 자료**  

* [CART 관련 연구보고서](https://www.kiri.or.kr/pdf/%EC%97%B0%EA%B5%AC%EC%9E%90%EB%A3%8C/%EC%97%B0%EA%B5%AC%EB%B3%B4%EA%B3%A0%EC%84%9C/nre2018-16_04.pdf)
* [Fig 1 참조](https://soobarkbar.tistory.com/17)  
* [수식 참조](http://mlwiki.org/index.php/Cost-Complexity_Pruning)  

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
