---
title: "Transformer - Embedding"
last_modified_at: 2022-8-30
author: 이한솔
---
Transformer는 2017년 NIPS를 통해 발표 되었으며, 번역 부분 SOTA를 차지한 우수한 모델입니다.

Transformer의 입력 방식은 Embedding vector에 Positional Encoding을 더해서 입력하게 되는데, 이 포스팅에서는 Embedding vector를 만드는 Embedding 방식에 대해 알아보겠습니다.

---

# **목차**

1. Embedding
   1) 희소 표현
   2) 밀집 표현
2. Embedding layer
3. Word2Vec
   1) CBOW
   2) Skip-gram

<br>

---

# **Embedding**

사람이 쓰는 자연어를 기계가 이해하고, 효율적으로 처리할 수 있도록 단어를 벡터(숫자)로 바꿔주는 것을 벡터화라고 합니다.

> Embedding은 단어를 밀집 벡터로 만들어주는 기술입니다.

밀집 벡터가 무엇인지 알기 위해서는 벡터화 방법에 대해 알아야 합니다.<br>벡터화 방법에는 희소 표현과 밀집 표현이 있습니다.

<img src="https://user-images.githubusercontent.com/96156882/187332647-0b8ced50-b380-47db-b794-285ad458a913.png" width="600">

### **1) 희소 표현**

희소 표현의 대표적인 방식은 One-hot encoding으로 표현하고 싶은 단어의 인덱스에 1의 값, 나머지 인덱스에는 0으로 표현합니다.<br>One-hot encoding은 단어 개수가 늘어날 수록 벡터의 차원이 계속 늘어난다는 것과 단어 유사도 표현이 불가하다는 문제점이 있습니다.

### **2) 밀집 표현**

밀집 표현은 사용자가 설정한 값으로 단어 벡터의 차원을 맞추며 실수값으로 구성되어 있습니다.

밀집 표현 방법은 다음과 같습니다.

1. 임베딩 층을 랜덤 초기화하여 처음부터 학습하는 Embedding layer
2. 사전에 학습된 임베딩 벡터들을 가져와 사용하는 pre-trained word embedding
<br>ex) Word2Vec, Glove, Elmo 등

<br>

---

# **Embedding layer**

Embedding layer는 단어를 랜덤하게 밀집 벡터로 변환한 뒤 인공 신경망의 가중치를 학습하는 방법입니다.

keras의 embedding layer와 pytorch의 nn.embedding을 통해 쉽게 사용할 수 있습니다.

임베딩 층 사용 방법은 단어를 정수로 인코딩 한 뒤 정수를 인덱스로 가지는 테이블로부터 임베딩 벡터 값을 가져옵니다.<br>
이 밀집 벡터는 학습 과정에서 가중치과 학습되는 것과 같은 방식으로 훈련합니다.

<img src="https://user-images.githubusercontent.com/96156882/187333483-010cd231-b7d7-4b85-9f41-c899526d65e7.png" width="500">

<br>

---

# **Word2Vec**

Word2Vec은 2013년 구글에서 발표한 Pre-trained word embedding 방법입니다.

단어 간 유사성을 고려하기 위해 단어의 의미를 벡터화하는 것으로 비슷한 위치에서 등장하는 단어들은 비슷한 의미를 가진다는 가정하에 수행됩니다.

Word2Vec은 2가지 방식이 있습니다.
1. CBOW
2. Skip-gram
   
<br>

### **1) CBOW**
CBOW는 주변에 있는 단어들을 입력으로 중간에 있는 단어들을 예측합니다.

예를 들어 "You say goodbye and I hello" 문장이 있을 때, say를 예측하기 위해 you와 goodbye를 입력값으로 넣어주는 것입니다.

<img src="https://user-images.githubusercontent.com/96156882/187334860-6311795f-7e61-4119-be4f-4267c8727a14.png" width="600">

CBOW의 학습 방법은 다음과 같습니다.

1. one hot vector 형태의 입력 값을 받습니다.
2. 입력 값을 랜덤으로 생성된 가중치 Win과 곱합니다.
3. 2번의 결과인 결과 벡터들의 평균인 벡터를 구합니다.
4. 구해진 평균 벡터는 두번째 가중치 Wout과 곱하고, Softmax 함수를 취해서 각 단어가 나올 확률을 계산해 예측 값을 구합니다.
5. 예측값과 Label 값의 loss를 cross entropy loss로 계산합니다.
6. Back Propagation을 통해 loss값이 줄어들도록 Win과 Wout을 업데이트 합니다.

이때, 업데이트된 Win을 Embedding vector로 사용하는 것입니다.

<br>

### **2) Skip-gram**
Skip-gram은 중간에 있는 단어들을 입력으로 주변 단어들을 예측하는 방법입니다.<br>메커니즘은 CBOW와 같으나, 은닉층에서 벡터들의 평균을 구하는 과정은 제외됩니다.

<img src="https://user-images.githubusercontent.com/96156882/187568117-90c6b13f-8a2b-4443-950f-b50a720dd0b5.png" width="600">

---

이번 포스팅에서는 Transformer의 입력을 위해 Embedding vector를 만드는 Embedding 방식에 대해 알아보았습니다.<br>
다음 포스팅에서는 Positional Encoding에 대해 알아보도록 하겠습니다.

---

### 참고자료
- 딥 러닝을 이용한 자연어 처리 입문 - 유원준, 안상준 (https://wikidocs.net/64779) 
- 밑바닥 부터 시작하는 딥러닝2 - 사이토 고키, 한빛출판네트워크

