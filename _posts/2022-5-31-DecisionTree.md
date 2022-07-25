---
title: "의사결정 트리 - CART 알고리즘"
last_modified_at: 2022-5-31
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **의사결정 트리 - CART 알고리즘** 에 대한 내용을 공유해보려고 합니다.

※ 목표변수(y)가 범주형일 경우로 진행하며, 재귀적 분기방법을 위주로 작성했습니다.

# **의사결정 트리**

데이터에서 서로 다른 특징과 속성에 기반해 트리형태의 분류 구조를 만드는 것을 의미합니다.  
아래 그림과 같이, Root Node에서부터 Leaf 1~4 까지의 규칙 즉, 의사결정 규칙을 도표화한다고도 할 수 있습니다.

![fig1](https://user-images.githubusercontent.com/92897860/171088823-56f92d81-b532-44ad-8510-4c3ae74da64a.png)


> **특징**
- 의미있는 질문이 영향력이 높으며, 그 기준으로 가지치기를 진행
- 체계적 의사결정 과정이 아닌 이전의 경험이나 주변의 단서를 통해 발생하는 편견을 통해 직관적으로 결정하는 휴리스틱한 학습 방법을 이용


> **분석 절차** 

의사결정트리의 여러 알고리즘이 있지만, 중요한 기본 아이디어 2개는 바뀌지 않습니다.  
부모 노드에서 자식노드로 찾아가는 과정인 **Recursive Partitioning(재귀적 분기)** 와 과적합이 방지를 위해 일부 노드를 제거하는 **Pruning the Tree(가지치기)** 과정이 있습니다. 데이터 유형에 따라서 어떠한 불순도 알고리즘을 선택하냐에 따라 의사결정트리의 알고리즘이 결정되고, 분기하는 과정과 가지치는 방법이 정해지게됩니다.

아래 분석 절차는 일반적인 의사결정트리의 분석 절차를 의미합니다. 

1. 목표변수(y)를 잘 설명하는 설명변수(x_n)를 선택
2. 분리기준에 근거하여 불순도가 최소화되는 분리지점을 선택
3. 트리 생성
4. 가지치기 수행
5. 최종 의사결정 트리 선정.
6. 분류규칙 도출 및 분류(예측)를수행


## **CART(Classification And Regression Trees)**  

CART 알고리즘 데이터 분류와 회귀가 가능한 지도학습 알고리즘입니다.  

매 회 반복마다 지니계수가 가장 작은 특징과 그에 대응하는 최적의 분리점(segmentation point)를 선택하여 분류 진행하게 되며, 이지 분리(Binary Split)를 사용하기 때문에 최종적으로 이진트리의 결과를 도출합니다.  

또한, 분리 기준에 대하여 평면의 특정축에 수직으로 기준을 긋기 때문에 해당 알고리즘 결과에 대해서도 설명력(해석력)을 갖는다고 할 수 있습니다.

> **특징**
- 불순도로 지니계수 사용
- 연속형과 범주형 데이터 모두 적용 가능
- 이지 분리 사용 ( 노드에서 가지를 칠 때, 2 개의 가지로 침)


> **지니 계수(Gini Index)**

특징에 의한 분리가 이진 분류로 나타날 경우, 지니계수를 사용합니다.  
CART에서 지니 계수를 통해 불순도를 측정하게 되며, 불순도가 작아지는 방향으로 분기를 수행하게 됩니다. 
목표변수(y)의 분포를 가장 잘 구별하는 기준인 분리 기준(Splitting Rule)을 설정하기 위해서 불순도라는 지표를 사용한다라고 생각하시면 됩니다.


먼저 지니 계수에 대해 알아보겠습니다.  
아래 그림의 빨간 점선에 해당하는 부분입니다. 지니계수는 최대 0.5(반반 섞여있을 확률, 두 집단이 동일한 경우)를 가지게 됩니다.
어느 한쪽으로 쏠릴 수록 불확실성이 낮고, 반반 섞여있을 경우 불확실성이 높기 때문에 우리는 지니 계수를 계산할 경우, 최소가 되는 방향으로 분기를 수행하게 됩니다.


![fig3](https://user-images.githubusercontent.com/92897860/171093194-d3459db8-622c-4b1b-8b1a-d3772d0f9303.png)



수식은 다음과 같습니다.  
n개의 관측치 중에서 임의로 2개를 선택하였을 때, 선택된 2개가 서로 다른 범주에 속할 확률을 의미합니다. 제곱을 해줌으로써 불확실성에 을 제대로 확인하게 됩니다. 

![fig4](https://user-images.githubusercontent.com/92897860/171092967-fab2d7a5-b13d-4a6e-b560-d31024661211.png)


예시로는 다음과 같습니다.  
LC(충성고객)과 CC(이탈고객)이 존재할 경우, 성별(남,여)라는 조건으로 지니계수를 계산해보겠습니다.


![fig5](https://user-images.githubusercontent.com/92897860/171093849-728089e1-084f-4194-921a-df09571c4433.png) 

![fig6](https://user-images.githubusercontent.com/92897860/171095008-199c89d0-8cad-4da8-8a62-bc9109ab919c.png)


> **재귀적 분기 (Recursive Binary Splitting)**

위에서 설명한 지니계수(불순도) 계산 과정을 사용합니다.
각 영역에 대해서 지니계수를 따로 계산하고 각 영역이 가지는 개체 수를 가지고 가중합을 하는 것을 의미합니다.

위의 예시를 그대로 사용하여 수식을 계산하면, 지니계수의 변화량은 0.167 로 계산되었습니다.  

![fig7](https://user-images.githubusercontent.com/92897860/171095662-11c39434-7d9f-47d7-a4a8-f82bf19f7397.png)

G(분기 이전) - G변화량(조건에 따른 분기 이후) 의 값을 보면,   
0.5 => 0.167 로 불확실성이 감소한 것을 볼 수 있습니다.

**이런식으로, 모든 변수(설명 변수X)에 대해서 (G-G변화량, Information Gain)을 비교하여 지니계수의 감소량이 가장 큰 변수를 기준으로 분리를 수행하게 됩니다. 그 이후는 정지 조건까지 반복을 수행합니다.**



## **참고 자료**  

* [Classification and Regression Trees (CART) : Theory and Application](https://www.semanticscholar.org/paper/Classification-and-Regression-Trees(CART)Theory-and-Timofeev/c77b4aff121e3c2c2c18bef72b36286bb3a6d375)  
* [CART 자료](http://www.math.snu.ac.kr/~hichoi/machinelearning/lecturenotes/CART.pdf)  
* [그림 참조](https://wannabenice.tistory.com/8)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
