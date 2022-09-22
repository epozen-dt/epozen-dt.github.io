---
title: "텐서보드 (TensorBoard)"
last_modified_at: 2022-9-22
author: 김상민
---

-------------

이번 포스팅에서는 **텐서보드 (TensorBoard)**에 관한 내용을 공유해보려고 합니다. 간단한 예시를 통해 실습해보도록 하겠습니다.

---------------

> ## TensorBoard 시작하기
기계 학습에서 무언가를 개선하려면 종종 측정할 수 있어야 합니다. TensorBoard는 기계 학습 워크플로 중에 필요한 측정 및 시각화를 제공하기 위한 도구입니다. 손실 및 정확도와 같은 실험 메트릭을 추적하고, 모델 그래프를 시각화하고, 임베딩을 저차원 공간에 투영하는 등의 작업을 수행할 수 있습니다.

   - 손실 및 정확도와 같은 측정학목 추적 및 시각화
   - 모델 그래프(작업 및 레이어) 시각화
   - 시간의 경과에 따라 달라지는 가중치, 편향, 기타 텐서의 히스토그램 확인
   - 저차원 공간에 임베딩 투영
   - 이미지, 텍스트, 오디오 데이터 표시
   - TensorFlow 프로그램 프로파일링
   - 그 외 다양한 도구


그렇다면 간단한 MNIST를 통한 예제로 실습해보겠습니다.


> ## TensorBoard 시작하기
```Python
from keras.datasets import mnist
from tensorflow.keras.utils import to_categorical
from tensorflow.python.keras.callbacks import TensorBoard
from time import time
from keras import layers
from keras import models
```


> ## 데이터 로드 및 타입 변경
```Python
(train_images, train_labels), (test_images, test_labels) = mnist.load_data() 

train_images = train_images.reshape((60000, 28, 28, 1))
train_images = train_images.astype('float32') / 255
test_images = test_images.reshape((10000, 28, 28, 1))
test_images = test_images.astype('float32') / 255
```


> ## 라벨 One-hot 인코딩
```Python
train_labels = to_categorical(train_labels)
test_labels = to_categorical(test_labels)
```


> ## 모델 생성
```Python
model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(28, 28, 1), padding='same'))
model.add(layers.MaxPooling2D(2, 2))
model.add(layers.Conv2D(64, (3, 3), activation='relu'))
model.add(layers.MaxPooling2D(2, 2))
model.add(layers.Conv2D(32, (3, 3), activation='relu'))

model.add(layers.Flatten())
model.add(layers.Dense(64, activation='relu'))
model.add(layers.Dense(10, activation='softmax'))

print(model.summary())
```


> ## 텐서 보드 생성 및 경로 설정
   - log 기록을 위해 디렉토리 위치 설정합니다.
```Python
tensorboard = TensorBoard(log_dir = f"{base_dir}/log/{time()}")
```


> ## 모델 컴파일
```Python
model.compile(optimizer='rmsprop',
              loss='categorical_crossentropy',
              metrics=['accuracy'])
```


> ## 모델 훈련
```Python
model.fit(train_images, train_labels, epochs=5, batch_size=64, callbacks=[tensorboard])
```


> ## 모델 테스트
```Python
test_loss, test_acc = model.evaluate(test_images, test_labels)
```


> ## 디렉토리 이동 및 텐서보드 실행
  - 현재 경로로 이동하고 tensorboad -- logdir=(현재디렉토리)를 실행시킵니다.
![cmd창](https://user-images.githubusercontent.com/60912905/191175474-12d9da01-6c44-48fd-bc27-9e74fe416cb8.JPG)


> ## 페이지 접속
  - cmd창에서 나오는 http://localhost:6006/으로 접속합니다.


> ## 텐서보드 결과확인
  - 실행 결과를 확인합니다.
![텐서보드](https://user-images.githubusercontent.com/60912905/191176037-23f790e0-4044-4590-add6-c692a61b74f6.JPG)


------------------------

> **참고 자료**  
* [TensorFlow](https://www.tensorflow.org/tensorboard/get_started?hl=ko)
