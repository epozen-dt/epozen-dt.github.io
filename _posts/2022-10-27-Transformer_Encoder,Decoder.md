---
title: "Transformer - Encoder,Decoder"
last_modified_at: 2022-10-27
author: 권재우
---

이번 포스팅은 자연어 처리 모델인 트랜스포머(transformer)의 Encoder와 Decoder의 세부 기술에 대해서 포스팅하겠습니다.


# 트랜스포머(transformer)

- Transformer 모델 구조

<img src="https://user-images.githubusercontent.com/70212461/197941760-31164b3d-63de-4a0a-bb0f-109448ca93f6.png" width="400" height="500">


위 그림과 같이 Transformer모델은 크게 Encoder와 Decoder로 구분됩니다.  
아래는 이전 transformer에 대한 포스팅 링크입니다. 해당 기술의 설명은 아래 링크에서 자세한 설명을 확인할 수 있습니다.

- Inputs

[Embedding 설명 바로가기](https://epozen-dt.github.io/Transformer_embedding/)

[Positional-Encoding 설명 바로가기](https://epozen-dt.github.io/Transformer_positional-encoding/)

- Attention

[self-attention 설명 바로가기](https://epozen-dt.github.io/Transformer-Self-Attention/)

오늘 포스팅에서는 Attention을 제외하고 Encoder부분에서는 Normalize, Feed-Forward를 설명하고 Decoder에서는 Masked Multi-Head Attention, ouput 기술을 설명하겠습니다. 

# Encoder - Add & Normalize(계층정규화)

이 과정은 Residual Connection이라고도 불리고 모델 학습을 돕기 위해 Vision분야에서 자주 사용하는 기법입니다. 

Encoder에서 Attention에서 나온 출력값은 Normalize과정을 거치게 됩니다.

![ecd2](https://user-images.githubusercontent.com/102636902/197944055-9482f136-8bb9-4748-a2a9-f3225ac49752.png)

위 그림과 같이 계층 정규화 과정은 Self-Attention과정 전,후 값인 입력값(X)과 출력값(Z)을 합(sum)함으로써 한 계층에서 과도하게 데이터가 변경되는 부분을 방지할 수 있습니다.

이 기술은 기울기 소실문제를 줄이기 위한 솔루션입니다.

# Encoder - Feed_Forward

Add & Norm을 거친 결과는 Feed-Forward 네트워크를 통과하게 됩니다.

![ecd3](https://user-images.githubusercontent.com/102636902/197947272-77757118-e416-4822-9c2d-d4e42e444c3c.png)

Feed-Forward 네트워크는 2개의 MLP 레이어와 ReLU 함수로 구성되어 있는 신경망 레이어로써 가중치를 학습하는 공간이라고 생각하면 됩니다. 

# Decoder 

위에서 설명한 Encoder과정은 입력과 출력을 N번 반복해서 실행하게 됩니다.  
마지막으로 나온 출력값 K,V는 Decoder의 입력값으로 각각 들어가게 됩니다.  
(*논문에서는 N=6으로 설정)

<img src="https://user-images.githubusercontent.com/102636902/197947534-471159a0-f510-454d-8e57-e0782888acb4.png" width="700" height="400">

이제 Decoder를 살펴보겠습니다.

# Decoder - Input

![ecd5](https://user-images.githubusercontent.com/102636902/197948149-7375d2b4-75d0-42ac-9144-2a4df4137d1a.png)

Decoder도 Encoder의 입력값처럼 Embedding, Positional Encoding과정을 거치게 됩니다. 하지만 차이점이 있는데, Encoder의 입력값으로는 문장의 토큰이 한번에 들어왔지만 Decoder의 입력값은 RNN입력처럼 오른쪽으로 이동하면서 단어가 하나씩 입력되는 형식입니다.

입력값의 마지막은 eos(end of Sentence)로 처리해줍니다.

# Decoder - Masked Multihead_Attention

이전에 설명한 Self-Attention방식에서 Masking작업만 추가한 것으로 생각하면 됩니다.


<img src="https://user-images.githubusercontent.com/102636902/197951054-875f4e93-424a-49e5-a6e2-fbb4ae4dda17.png" width="200" height="400">

Masking 작업을 하는 이유로는 Decoder의 입력값으로 Atttention 과정을 수행하는데 이 때 정답이 입력되기 때문에 입력을 masking함으로써 과적합으로 인한 성능저하를 막기 위함입니다.

<img src="https://user-images.githubusercontent.com/102636902/197951059-a57546f2-5f5c-4e38-a702-114801fd9684.png" width="600" height="300">

Attention 기법으로 단어간 유사도를 계산한 결과에서 뒤쪽 부분을 0(Zero Weight)으로 만들어 주는 방법입니다.

이렇게 출력된 가중치를 Encoder에서 나온 출력값과 비교하면서 학습하는 과정을 거치게 됩니다.

# Decoder - output

디코더로 출력된 벡터를 단어로 바꿔주는 작업으로 Linear, Softmax방법을 사용합니다.

<img src="https://user-images.githubusercontent.com/102636902/197951969-cb1c8caa-c708-4722-bef1-d4d67a1e1e92.png" width="200" height="300">

디코더의 출력값으로 나온 값을 선형 레이어로 변환하고 변환된 레이어에 Softmax함수를 사용해서 확률값으로 바꿉니다.

이 확률값 중 가장 높은 확률을 가지는 단어가 출력되는 방법입니다.

<img src="https://user-images.githubusercontent.com/102636902/197951977-694f6054-82de-4811-91a8-aaf05c595ad8.png" width="500" height="300">

이것으로 오늘 Transoformer의 Encoder, Decoder 포스팅을 마치겠습니다.


## 출처

- https://arxiv.org/abs/1706.03762
- https://data-science-blog.com/blog/2021/04/07/multi-head-attention-mechanism/
- https://dhpark1212.tistory.com/entry/Transformer
- https://zeuskwon-ds.tistory.com/88
- https://jalammar.github.io/illustrated-transformer/

- https://d2l.ai/chapter_attention-mechanisms-and-transformers/transformer.html
