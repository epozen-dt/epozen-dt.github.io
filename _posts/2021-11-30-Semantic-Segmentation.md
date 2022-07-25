---
title: "Sematic Segmentation"
last_modified_at: 2021-11-30
author: "김수민"
---

이번 포스팅에서는 Sematic Segmentation에 대해 알아보겠습니다.

------
## 목차
1. [Semantic Segmentation ](#semantic-segmentation) 
2. [Semantic Segmentation Models](#semantic-segmentation-models)
  - [FCN(Fully Convolutional Network for Semantic Segmentation)](#fcnfully-convolutional-network-for-semantic-segmentation)
  - [DeepLab Models](#deeplab-models)
3. [References](#references)

------

## Semantic Segmentation

우선 Object Segmentation이란 Object Detection을 통해 검출된 객체의 형상을 따라 영역을 표시하는 기법으로, 이미지 각 픽셀을 분류하는 기술입니다.

Semantic Segmentation은 이러한 Object Segmentation의 한 종류로 이미지 내 물체들을 같은 클래스 단위로 분할하는 기법입니다.

|      | Image  Segmentation                                          | Semantic  Segmentation                                       | Instance  Segmentation                                       |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 정의 | 이미지의 영역을 분할하고 알고리즘을 이용해 다시 분할된  영역을 합치는 기술 | 같은 class인 object들은 같은 영역 또는     색으로 분할하는 기술 | 같은 class더라도 서로 다른 instance들을   구분해주는 기술    |
| 예시 | ![image](https://user-images.githubusercontent.com/87166420/144014048-7848c79e-815b-430f-a34b-ce0eb8a047e8.png) | ![image](https://user-images.githubusercontent.com/87166420/144014114-0263a37a-5c66-43e2-b2d5-26603072ae4e.png) | ![image](https://user-images.githubusercontent.com/87166420/144014138-2a19d22e-6171-45af-8dfc-eb2e0d6e273b.png) |

즉, 이미지의 각 픽셀이 어느 클래스에 속하는지 예측해 이미지 내 서로 다른 종류의 물체들을 깔끔하게 분할해내는 것이 목표인 기술입니다.

이미지 내 모든 픽셀에 대해 예측을 진행하기 때문에 Dense Prediction이라고 부르기도 합니다.

------

## Semantic Segmentation Models

보통 Semantic Segmentation모델의 Input은 RGB이미지 혹은 흑백 이미지를, Output은 각 픽셀 별 어느 class에 속하는지를 나타내는 Segmentation Map으로 구성되어 있습니다.

![image](https://user-images.githubusercontent.com/87166420/144014340-0a27e1d0-de50-4608-a7c6-bad26782b99d.png)

다양한 구조의 모델들이 있지만, 그 중 대표적으로 사용되는 모델은 Encoder-Decoder 형태의 FCN(Fully Convolutional Network for Semantic Segmentation) 기반의 모델입니다.

------

### FCN(Fully Convolutional Network for Semantic Segmentation)

FCN 모델은 2015년에 발표된 모델로, DownSampling 과정의 Encoder, Upsampling 과정의 Decoder 구조를 통해 Segmentation Map을 구하게 됩니다.

#### DownSampling 과정

DownSampling 과정은 기존의 CNN 모델과 같이 Convolution Layer과 Pooling을 사용한 특징추출을 수행합니다.

위치 정보의 소실을 막고, 다양한 크기의 입력 이미지 허용을 위해 CNN에서 사용되는 Fully Connected Layer 대신, 1x1 사이즈의 Convolution Layer을 사용합니다.

최종적으로 각  Convolution Layer을 거치고 나서 얻게 되는 마지막 Segmentation Map의 개수는 훈련된 클래스의 개수와 동일합니다.
![image](https://user-images.githubusercontent.com/87166420/144014395-c32aaedd-3d33-4f9b-aa16-5d9399f70480.png)



이후 만들어진 특성맵을 사용해 원래 이미지의 크기로 복원해주는 Upsampling 과정을 수행합니다.

#### UpSampling 과정

Upsampling 과정에서는 이미지의 모들 픽셀에 대한 클래스 예측을 위해 대략적인 Segmentation Map들의 크기를 Deconvolution을 통해 원래 이미지의 크기로 다시 복원합니다.

Deconvolution 과정은 Input 이미지에 임의의 padding을 넣어 Convolution 과정을 거쳐 원하는 사이즈의 output을 생성해내는 과정입니다.

다음은 deconvolution에 대한 예시 입니다.

![image](https://user-images.githubusercontent.com/87166420/144014433-3eb9a809-7c57-410e-a65b-08f722765e85.png)

[예시 1]은 2x2 이미지가 들어가 4x4 이미지로 나오는 과정을 보여줍니다. [예시 2]는 2x2 이미지를 stride=2, kernel_size=3으로 설정해 5x5 이미지로 생성하는 과정입니다.

tensorflow에서는 tf.nn.conv2d_transpose, tf.layers.conv2d_transpose, tf.contrib.layers.conv2d_transpose 함수로 구현 가능합니다.

#### FCN의 한계

FCN(Fully Convolutional Network for Semantic Segmentation) 모델은 정해진 Receptive Field를 사용하기 때문에 작은 물체들은 무시되거나 이상하게 인식될 수 있으며 큰 물체를 작은 물체로 인식하거나 일정하지 않은 결과가 나올 수 있습니다.

​	*Receptive Field : 각 레이어의 필터가 한 번에 보는 특정 영역(넓을 수록 특징 추출에 용이함)

또한 Pooling을 거치면서 해상도가 줄어든 것을  Upsampling을 통해서 복원하기 때문에 결과가 정확하지 않을 수 있다는 한계를 가지고 있습니다.

![image](https://user-images.githubusercontent.com/87166420/144014517-aff4fb23-ab4a-4f81-a986-8626720259d8.png)

------

### DeepLab Models

이러한 한계점을 개선하기 위해 다양한 모델들이 만들어졌는데, 이들 중 FCN구조와 다른 방식으로 성능을 개선한 DeepLab 모델에 대해 알아보겠습니다.

DeepLab은 버전이 향상되면서 성능 개선이 계속해서 이루어지고 있는 모델입니다.

 tensorflow에서 공식적으로 코드를 공개하고 있으며, 다양한 데이터셋과 여러 encoder 구조로 학습해둔 pre-trained model을 제공하고 있어 모델 구축 시 참고하기 좋은 모델일 것이라고 생각됩니다.

#### DeepLab v1~v3

DeepLab 초기 모델에서는 Dilated Convolution(Atrous Convolution)을 사용해 Receptive Field를 키워 특징을 추출한다는 특징을 갖습니다. 

Dilated Convolution은 일반적인 Convolution 사이에 공간을 넣어 구멍이 뚫린 듯한 구조로, 같은 연산량으로 더 큰 특징을 추출 할 수 있습니다.

이때 필터의 픽셀 간 거리를 나타내는 dilation rate(확장 비율)이라는 새로운 변수를 활용하게 됩니다.

다음 예제와 같이 필터 사이에 간격을 주면 넓은 영역을 볼 수 있고, 해상도의 손실 없이 Receptive Field 크기를 확장할 수 있습니다.

![image](https://user-images.githubusercontent.com/87166420/144014587-6a726f0f-17f7-44db-9861-6aac6d254370.png)

Receptive Field의 크기가 커지고, dilation Rate(확장 비율) 조정 시, 다양한 scale에 대한 대응이 가능해집니다.

Pooling을 이용해 Receptive Field의 확장 효과를 얻는 것 보다 표현력이 더 좋은 양질의 Feature Map을 얻을 수 있어 Segmentation 성능을 향상 시켰습니다.

![image](https://user-images.githubusercontent.com/87166420/144014608-37fd0784-b8d0-4233-8c8a-141e65087f27.png)



이후 이를 좀 더 활용해 multi-scale context를 적용하기 위해 ASPP(Atrous Spatial Pyramid Pooling) 기법을 적용하게 됩니다.

해당 기법은 특징 맵으로 부터 rate가 다른 Atrous Convolution을 병렬로 적용한 후 이를 다시 합쳐 하나의 특징맵을 도출하는 방식입니다.

이러한 방법들은 보다 정확한 Semantic Segmentation을 수행할 수 있도록 도우며 이후에 나온 DeepLab모델에서는 ASPP를 기본 모듈로 계속 사용하게 됩니다.

![image](https://user-images.githubusercontent.com/87166420/144014818-a9aac0cc-48bc-4b89-8da1-4ca64add83b5.png)

#### DeepLab v3+

DeepLab에서 가장 최신에 발표한 모델로 기존의 ASPP 모듈과 Encoder-Decoder 구조를 합친 형태를 갖고 있는 모델입니다. 

Encoder-Decoder 구조에서 Encoder 부분에서 점진적으로 Spatial dimension을 줄여가며 고차원의 semantic 정보를 추출하고, 이 추출된 정보를 Decoder 에서 Encoder에서 손실된 spatial 정보를 점진적으로 복원해 정교한 boundary Segmentation을 수행합니다.

ASPP구조는 multi-scale context 정보를 인코딩 할 수 있도록 하고, Encoder-Decoder 구조는 객체 경계를 더 섬세하게 추출해 낼 수 있다는 장점을 합쳐 Semantic Segmentation 성능을 보다 향상시켰습니다.

DeepLab v3+ 의 구조는 다음과 같습니다.

![img](https://images.velog.io/images/skhim520/post/beffeebf-a1a0-4200-9e3f-2b7ccf77af04/image.png)

우선 기존의 DeepLab 시리즈의 모델들에서와 같이 Astrous Convolution을 사용한 Encoder 과정을 수행합니다. 

해당 Encoder 구조는 DeepLab v3를 그대로 가져다 놓은 것입니다.

Decoder 과정에서는 기존의 DeepLab 시리즈에서는 Encoding한 결과를 단순히 키워 Input 사이즈와 같게 만들은 반면, 

v3+에서는 encoder 중간 과정에서의 특징맵을 추출해 Encoding한 결과와 합치는 과정을 통해 보다 Encoder-Decoder 구조와 유사하게 변경되었습니다.![img](https://images.velog.io/images/skhim520/post/a30ad15d-1cc0-4413-94c7-5952f1a26b7b/image.png)

또한 전 과정에서 Depthwise Separable Convolution을 활용해 모델의 파라미터 수를 대폭 줄여 용량 감소와 속도향상을 시켰습니다.

Encoder 과정에서는 기존에 ResNet을 활용했었는데, Xception으로 대체되었습니다. Xception은 Depthwise Separable Convolution을 적극 활용한 구조입니다.

이때 Depthwise Separable Convolution이란 기존 Convolution연산에서 channel 축을 필터가 한번에 연산하는 방식 대신,

아래 그림과 같이 모든 channel 축을 분리시킨 후에 channel 축 길이를 항상 1로 갖는 여러개의 convolution 필터로 대체시킨 연산인 Depthwise Convolution을 수행해 그 결과에 대한 convolution 필터를 적용하는 것 입니다.

![depthwise_separable_conv](https://bloglunit.files.wordpress.com/2018/07/depthwise_separable_conv.png?w=300&h=435)

​                        																	   	Depthwise separable convolution. [출처](https://eli.thegreenplace.net/2018/depthwise-separable-convolutions-for-machine-learning/)

즉, Convolution Filter가 Spatial 차원과 Channel 차원을 동시에 처리하던 것을 따로 분리시켜 각각 처리하는 것 입니다.

이 과정에서 여러 개의 필터가 spatial 차원 처리에 필요한 파라미터를 하나로 공유해 필요한 파라미터의 수를 대폭 줄이는 결과를 얻게 됩니다.



현재 DeepLab 시리즈에서는 Deeplab v3+ 가 가장 최신 논문이자, 가장 높은 Semantic Segmentation 성능을 보이고 있습니다.

------

본 포스팅에서는 Semantic Segmentation의 정의와 이를 위한 대표적인 두 모델인 FCN, DeepLab 모델에 대해 살펴보았습니다.

이외에도 U-Net 등 다양한 모델들이 있어 추후 포스팅을 통해 알아보도록 하겠습니다.

------

### References

https://medium.com/hyunjulie/1%ED%8E%B8-semantic-segmentation-%EC%B2%AB%EA%B1%B8%EC%9D%8C-4180367ec9cb

https://velog.io/@skhim520/DeepLab-v3

https://bskyvision.com/491

https://reniew.github.io/18/

https://blog.lunit.io/2018/07/02/deeplab-v3-encoder-decoder-with-atrous-separable-convolution-for-semantic-image-segmentation/
