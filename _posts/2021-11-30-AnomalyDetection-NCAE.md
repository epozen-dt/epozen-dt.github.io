---

title: "스마트 제조 이상탐지 모델 - NCAE 구현"
date: 2021-11-30

---
> 현재 생산품에 대한 양/불량 판정 알고리즘 구축에 대해서 연구를 진행하고 있습니다. </br>
> 이번 포스팅에서는 **자체 이상탐지 모델인 Neighbor Convolution AutoEncoder 를 구현한 방법**을 공유해보려고 합니다.

</br>

## 목차 </br>
[1.개요](#개요) </br>
[2.Neighbor Convolution Layer 구현](#Neighbor-Convolution-Layer-구현) </br>
[3.추가 확인](#추가-확인)

</br>

## 개요
**Neighbor Convolution AutoEncoder(이하, NCAE)는 기존 Convolutional AE 모델에 Neighbor Convolution 기법을 적용한 모델입니다.**


> **NCAE 모델구성도**
![모델구성1](https://user-images.githubusercontent.com/92897860/143972463-5ff03959-b345-4863-820b-c0651c39b9f7.png)
AutoEncoder 기반의 모델 학습 결과는 이미지 데이터이기 때문에 이미지 데이터 자체만으로는 명확히 구분하기 어렵다.</br>
따라서, 이상치(Anomaly Score)값으로 잔차이미지에 대한 통계치, MSE, 정보엔트로피로 설정하여 양/불량 판정을 수행하였다.

</br>

> **Neighbor Convolution 기본 아이디어**
![아이디어](https://user-images.githubusercontent.com/92897860/143973855-850901a9-db40-4865-a59d-4679a10cd18e.png)
Convolution에서 중앙값을 제외하여 주변 이웃 픽셀들에 대해서만 Convolution을 수행하여 중앙값의 의존성을 제거하고 </br> 이웃 픽셀들의 가중치 학습을 강화하여 효율성을 높일 수 있지 않을까라는 아이디어에서 시작되었다. 

</br>

## Neighbor Convolution Layer 구현
Layer 클래스를 상속받아 상요자 정의 클래스(custom layer)를 생성하여 구현하였습니다. </br>
기본 아이디어를 실제로 적용한 코드(*일부, 순방향 연산에 관한 코드*)로, 아래와 같이 순서대로 진행합니다.


**0. 입력**
  - input ([type]): (N, 300, 300, 1)의 입력 데이터
  - 1채널, GrayScale 이미지만 가능하도록 구현함
  - Kaggle의 주조 제품 입력 이미지(300, 300, 1)
  <figure>
     <img src="https://user-images.githubusercontent.com/92897860/143996629-484b319b-1e45-4ebf-8801-af223e8a6fb7.png"  width="200" height="100">
  </figure>
  
**1. 원본 이미지에 패딩 추가**
  - 원본 이미지 주변에 1픽셀씩 추가, 이때 픽셀값은 0
  - 300 크기의 이미지 => 302 크기의 이미지
  ```
  x_padding = ZeroPadding2D(padding=1)(x)
  ```
  
**2. 8장의 이미지 생성**
  - 좌측 상단을 기준으로 좌표 변경을 통해 300 크기의 8장의 이미지 생성
  ```
  img1 = Cropping2D(cropping=((0, 2), (0, 2)))(x_padding)
  img2 = Cropping2D(cropping=((0, 2), (1, 1)))(x_padding)
  img3 = Cropping2D(cropping=((0, 2), (2, 0)))(x_padding)
  img4 = Cropping2D(cropping=((1, 1), (0, 2)))(x_padding)
  img5 = Cropping2D(cropping=((1, 1), (2, 0)))(x_padding)
  img6 = Cropping2D(cropping=((2, 0), (0, 2)))(x_padding)
  img7 = Cropping2D(cropping=((2, 0), (1, 1)))(x_padding)
  img8 = Cropping2D(cropping=((2, 0), (2, 0)))(x_padding)
  ```
  
**3. 8채널 이미지로 변환**
  - 8장의 개별 이미지를 8채널 이미지로 변환
  ```
  x_total = K.concatenate([img1 ,img2, img3, img4, img5, img6, img7, img8], 3)
  ```
  
**4. pointwise Convolution 수행**
  - build()에서 설정한 가중치 커널과 편향 추가하여 Convolution 연산 파라미터 계산을 수행하도록 설정
  - Keras.Backend의 경우, conv2d안에 Activation 과정이 포함되지 않기 때문에 따로 ReLU 적용 수행
  ```
  pw_conv2d = K.conv2d(x=x_total, kernel=self.kernel, strides=self.strides, padding='same')
  pw_conv2d = K.bias_add(pw_conv2d, self.bias)
  pw_conv2d = K.relu(pw_conv2d)
  ```
  
## 추가 확인
* 가중치 생성 과정
  - 실제 Convolution 계산을 수행하기 위해서 가중치를 정의
  - 순방향 연산 수행 전에 내부적으로 실행되는 build()라는 함수에서 구현 
  - 가중치는 모두 Keras.Conv2D의 디폴트 항목에 맞게 설정
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

  
  
## 전체 코드  

