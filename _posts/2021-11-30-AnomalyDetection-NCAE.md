---
title: "스마트 제조 이상탐지 모델 - NCAE 구현"
last_modified_at: 2021-11-30
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **자체 이상탐지 모델인 Neighbor Convolution AutoEncoder 를 구현한 방법**을 공유해보려고 합니다.

## 목차

[1. 개요](#개요)  
[2. Neighbor Convolution Layer 구현](#neighbor-convolution-layer-구현)  
[3. 이상치 적용](#이상치-적용)  
[4. 성능지표 적용](#성능지표-적용)

## 개요

**Neighbor Convolution AutoEncoder(이하, NCAE)는 <u>기존 Convolutional AE 모델에 Neighbor Convolution 기법을 적용</u>한 모델**입니다.

> **Neighbor Convolution AutoEncoder 모델구성도**  
> ![모델구성1](https://user-images.githubusercontent.com/92897860/143972463-5ff03959-b345-4863-820b-c0651c39b9f7.png)  
> AutoEncoder 기반의 <u>모델 학습 결과는 이미지 데이터이기 때문에 이미지 데이터 자체만으로는 명확히 구분하기 어렵습니다.</u>
> 따라서, 이상치(Anomaly Score)값으로 <u>잔차이미지에 대한 통계치, MSE, 정보엔트로피로 설정</u>하여 양/불량 판정을 수행하도록 모델을 구성하였습니다.

> **Neighbor Convolution 기본 아이디어**
> ![아이디어](https://user-images.githubusercontent.com/92897860/143973855-850901a9-db40-4865-a59d-4679a10cd18e.png)
> Convolution에서 <u>중앙값을 제외하여 주변 이웃 픽셀들에 대해서만 Convolution을 수행하여 중앙값의 의존성을 제거하고 이웃 픽셀들의 가중치 학습을 강화</u>하여 효율성을 높일 수 있지 않을까라는 아이디어에서 시작되었습니다.

> ![비교](https://user-images.githubusercontent.com/92897860/147439731-00770b40-662d-41c0-9dd6-b73052f7de0a.png)
> 다음과 같이, 중앙값을 제외한 나머지 픽셀들에 대해 Convolution을 수행합니다.

## Neighbor Convolution Layer 구현

Layer 클래스를 상속받아 사용자 정의 클래스(custom layer)를 생성하여 구현하였습니다.

**0. 입력**

- input ([type]): (N, img_size, img_size, channel)의 입력 이미지 데이터
- 1채널, GrayScale 이미지만 가능하도록 구현하였기 때문에 주의해야 됩니다.
- Kaggle의 주조 제품 입력 이미지(300, 300, 1)
  ![입력이미지](https://user-images.githubusercontent.com/92897860/143996629-484b319b-1e45-4ebf-8801-af223e8a6fb7.png)

**1. 원본 이미지에 패딩 추가**

- 패딩을 1픽셀씩 추가하며, 이때 추가된 픽셀값은 0 이 됩니다.
- 해당 작업은 이웃한 픽셀값을 사용하기 위한 작업입니다.
- ex. 3 픽셀 이미지는 5 픽셀 이미지로 됩니다.
  ![패딩](https://user-images.githubusercontent.com/92897860/147438229-133c9a88-87e6-4a55-8efd-044703d2bd93.png)

**2. 8장의 이미지 생성**

- 입력이미지와 동일한 크기의 8장의 이미지 생성합니다.
  ![8장](https://user-images.githubusercontent.com/92897860/147438526-89daac10-730f-4e59-bfc3-114ff596f606.png)

**3. 8채널 이미지로 변환**

- 8장의 개별 이미지를 8채널 이미지로 변환합니다.
  ![채널](https://user-images.githubusercontent.com/92897860/147439055-dad5506c-d0a5-4a3f-93e3-5da3da6d32ba.png)

**4. pointwise Convolution 수행**

- 기본 Convolution 연산은 공간과 채널방향의 Convolution을 수행한다면, Pointwise Convolution은 1x1 커널로 고정된 연산을 수행하며 채널 방향에 대해서만 Convolution을 수행합니다.
- 입력의 특징을 더 적은 채널로 압축시켜 연산량 감소 목적으로 보통 사용하지만, 본 내용에서는 1개의 채널로 압축시키는 역할로 사용
- build()에서 설정한 가중치 커널과 편향 추가하여 Convolution 연산 파라미터 계산을 수행하도록 설정합니다.
- Keras.Backend의 경우, conv2d안에 Activation 과정이 포함되지 않기 때문에 따로 ReLU 적용합니다.
  ![pwconv](https://user-images.githubusercontent.com/92897860/147440077-e7543d4a-2597-48fd-b14a-daff4ddc3bb0.png)

### 추가 확인

**1. 가중치 생성 과정**

- 실제 Convolution 계산을 수행하기 위해서 가중치를 정의해야 됩니다.
- 순방향 연산 수행 전에 내부적으로 실행되는 build()라는 함수에서 구현되어야 순방향 연산 시 가중치를 주입할 수 있습니다.
- 가중치는 모두 Keras.Conv2D의 디폴트 항목에 맞게 설정하였습니다.

```
# 8 채널 & 이미지 크기
# 파라미터 계산에 필요한 설정
self.channels = input_shape[-1] * 8
self.img_size = input_shape[1]

# 가중치 형태 : (커널 사이즈, 커널 사이즈, 채널, 필터) 정의
self.shape = (self.kernel_size, self.kernel_size) + (self.channels, self.units)
# 가중치 정의
self.kernel = self.add_weight(name='kernel', shape=self.shape,
                              dtype='float32',
                              initializer='glorot_uniform', # 가중치 행렬의 디폴트 initializer
                              trainable=True)  # 훈련 가능한 가중치 설정 (역전파 고려)

# 편향 정의
self.bias = self.add_weight(name='bias', shape=(self.units,),
                            dtype='float32',
                            initializer='zeros',  # 디폴트 initializer
                            trainable=True)

super(NeighborConv2D, self).build(input_shape)
```

**2. Convolution 가중치(W_c), 편향(B_c), 파라미터 계산(P_c)**

```
# I(이미지 크기) = 300
# K(커널 사이즈) = 1
# N(전체 커널 갯수) = 256
# S(Stride) = 1, 디폴트
# P(패딩) = 0
# C(채널 갯수) = 8

# 계산식
W_c = K^2 * C * N
B_c = N
P_c = W_c + B_c

# 계산수행 결과 => 2048(W_c) + 256(B_c) = 2304
```

**3. 파라미터값 확인**

- 위에서 수행한 계산 결과와 실제 모델에 파라미터 계산 수가 맞으면 순방향은 문제없다는 것을 알수 있습니다.
  ![Screenshot_1](https://user-images.githubusercontent.com/92897860/144004480-904cabb8-7a1b-486c-a5a2-7407c99eaac9.png)

## 이상치 적용

앞서 얘기한 대로 모델의 구성도를 보면 결국 출력은 AutoEncoder와 동일하기 때문에 이미지 데이터를 출력하게 됩니다.
따라서, 잔차 이미지(원본 - 생성 이미지)에 대한 통계치와 정보 엔트로피 혹은 MSE 값을 이상치로 사용하게 됩니다.

**1. 잔차 이미지 생성**

- 각 픽셀간의 차이(원본-생성)로, 학습이 잘될수록 흑백에 가깝게 이미지가 도출됩니다.
  ![Screenshot_11](https://user-images.githubusercontent.com/92897860/144008353-567a3967-3609-4251-9eec-ceb6e2340dce.png)

**2. 통계치 적용**

- 정량적 분석으로 대표값을 추출할 수 있을 만한 통계 수치를 활용하였습니다.
- 해당 기술 통계 분석을 이미지 데이터의 간 픽셀값(이산)에 대입하여 정상 잔차와 비정상 잔차 이미지 간 수치 비교(ex. 산점도, )를 통해 이상치 기준선을 구합니다.
- 적용 항목으로는 아래의 코드와 같습니다.

  ```
  # 기술 통계 + 정보 엔트로피 계산 함수
  def descriptive_statistics(img):
     # 리스트로 변환 (왜도, 첨도)
     img_tmp = img.reshape(-1)
     img_list = img_tmp.tolist()

     # 평균
     avg = img.mean()
     # 표준 편차
     std = img.std()
     # 분산
     var = img.var()

     # 첨도
     kurto = kurtosis(img_list)
     # 왜도
     skewness = skew(img_list)

     # 범위
     img_range = img.max() - img.min()
     # 최소값
     _min = img.min()
     # 최대값
     _max = img.max()
     # 누적 합
     cumsum = img.cumsum()[-1]

     # 정보 엔트로피
     h_x = np.nan_to_num(entropy(img_list))

     return avg, std, var, kurto, skewness, img_range, _min, _max, cumsum, h_x
  ```

  **3. MSE & 정보 엔트로피 적용**

- MSE(mean square error)는 평균제곱오차로 이미지의 유사도를 비교하는 방법으로, 두 이미지의 각 픽셀간의 차이를 평균내어 유사도를 계산합니다.
- 정보 엔트로피는 얻은 정보가 의미있는 정보인지(정보량)을 판단하는 척도로, 이를 해당 데이터에 적용하여 정상으로만 학습된 모델에 넣었을 때, 정상 데이터가 들어온다면 놀라운 일이 아니므로 0에 가깝게 출력할 것이고, 결함 데이터라면 새로운(놀라운) 데이터이기에 1에 가깝게 출력되지 않을까라는 아이디어에서 적용하였습니다.

## 성능지표 적용

모델의 성능 평가 척도로는 정확도, AUC ROC, AUC PRC, 조화평균 등 총 4개를 기준으로 평가하였습니다.

- **성능 지표 결과 (ROC, PRC)**

![Screenshot_12](https://user-images.githubusercontent.com/92897860/144008649-43ddfc55-e455-45d8-b773-4e223e8003a3.png)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
