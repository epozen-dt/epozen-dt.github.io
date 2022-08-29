---
title: "Transformer - Self Attention"
last_modified_at: 2022-8-29
author: 권재우
---

이번 포스팅은 자연어 처리 모델인 트랜스포머(transformer)의 Attention 매커니즘에 대해 포스팅하겠습니다.


# 트랜스포머(transformer)란?
트랜스포머(Transformer)는 2017년 구글이 발표한 논문인 "Attention is all you need"에서 나온 모델로 기존의 seq2seq의 구조인 인코더-디코더를 따르면서도, 논문의 이름처럼 어텐션(Attention)만으로 구현한 모델입니다. 이 모델은 RNN을 사용하지 않고, 인코더-디코더 구조를 설계하였음에도 번역 성능에서도 RNN보다 우수한 성능을 보여주었습니다.

- Transformer 모델 구조

![selfa5](https://user-images.githubusercontent.com/70212461/187125087-b32b7f59-3d00-499e-8825-cf872d4b904e.png)  

# seq1seq 모델의 한계
기존의 seq2seq 모델은 인코더-디코더 구조로 구성되어져 있었습니다. 여기서 인코더는 입력 시퀀스를 하나의 벡터 표현으로 압축하고, 디코더는 이 벡터 표현을 통해서 출력 시퀀스를 만들어냈습니다. 하지만 이러한 구조는 인코더가 입력 시퀀스를 하나의 벡터로 압축하는 과정에서 입력 시퀀스의 정보가 일부 손실된다는 단점이 있는데, 이 말은 한마디로 입력과 출력이 멀어질수록 연관 관계가 적어진다는 뜻입니다.(기울기 소실 문제) 

이러한 단점을 보정하기 위해 어텐션이 사용되었습니다

즉, 기존 시계열 모델인 RNN 모델과는 전혀 다른 Attention 매커니즘을 가집니다.

Transformer에서 나온 Self-Attention 매커니즘에 대해서 알아보겠습니다.

# Self-Attention이란?
먼저 Attention 함수는 주어진 Query에 대해 모든 Key와의 유사도를 구하고 이후 해당 유사도를 가중치로 하여 Value에 반영해주는 방법입니다. 

Transformer 모델에서는 Self라는 단어가 붙은 Attention매커니즘 방식을 사용하는데 이 Self-Attention은 유사도 측정을 자기 자신에게 수행한다는 의미입니다. 

![selfa2](https://user-images.githubusercontent.com/70212461/187122216-8e171160-2a37-49f0-83ae-8d1ccf510a5d.png)

**X는 입력단어 시퀀스이고 입력단어에 가중치 행렬을 곱해서 Q(Query), K(Key), V(Value) 세 가지 값을 계산합니다. **

Query, Key, Value가 모두 동일한 입력 시퀀스 X에서 생성되는 경우 이를 **Self-Attention**이라고 합니다. 

- 예시문장 

![selfa1](https://user-images.githubusercontent.com/70212461/187122138-44fcd426-9ca0-4e88-a778-bb9a8b3b7bff.png)

위 예시 문장을 번역하면 "그 동물은 길을 건너지 않다. 왜냐하면 그것은 너무 피곤했기 때문이다."입니다. 여기서 그것(it)에 해당하는 것은 사람눈에는 당연히 동물(animal)이겠지만, 기계는 길(street)인지, 동물인지 알기 쉽지 않습니다. 그래서 셀프 어텐션을 통해 단어 문장 내의 단어들 끼리 유사도를 구함으로써 그것(it)이 동물(animal)과 연관되었을 확률이 높다는 것을 찾아냅니다.

# Self-Attention 함수식
그럼 Self-Attention이 어떻게 단어간의 유사도를 나타낼수 있는지 논문에 나와있는 함수식을 통해 설명하겠습니다.


![selfa3](https://user-images.githubusercontent.com/70212461/187123569-f86cde84-49eb-4dce-80a1-de42caf2805b.png)

![self5](https://user-images.githubusercontent.com/70212461/187124728-3372448d-d4a8-4716-97f8-8553ff9599b8.png)


위 함수식이 Self-Attention의 함수식으로 단어간의 유사도를 계산하는 방법입니다. 각 단어에 대한 Q, K, V 값을 구하고 K를 Transpose를 해서 Q값과 내적한 값으로 각 단어간 유사도를 구합니다 그리고 root(dk)와 softmax함수를 통해 출력된 스코어를 정규화해주고 백분율화 합니다. 이 score 값을 V에 곱해줌으로써 입력 값과 같은 차원의 값을 출력할 수 있게 됩니다. 

![selfa4](https://user-images.githubusercontent.com/70212461/187123576-5e9a0d31-5c60-4726-9c10-e76d8ad4d867.png)

# Multhead-Attention
Transformer의 Encoder, Decoder에 사용되는 Multhead-Attention은 위에서 설명한 Self-Attention을 병렬로 처리하는 방법입니다. 

![selfa6](https://user-images.githubusercontent.com/70212461/187126212-21a5661b-75a9-4c83-a393-96db023b256b.png)

입력값을 head의 수만큼 나누고, 각 head별로 Self-Attention 수행하고 나온 출력값에 Concat방식으로 입력값과 동일한 출력값을 내보내게 됩니다. 


<img src="https://user-images.githubusercontent.com/70212461/187126637-731b3ad8-c8c4-43c9-90aa-9a873d7f5621.png" width="500" height="500">

위 방식을 통해 Self-Attention의 병렬화 작업이 가능하게 됩니다. 


## 출처
- https://moondol-ai.tistory.com/460
- https://blogs.oracle.com/ai-and-datascience/post/multi-head-self-attention-in-nlp
- https://wikidocs.net/31379
- https://data-science-blog.com/blog/2021/04/07/multi-head-attention-mechanism/
