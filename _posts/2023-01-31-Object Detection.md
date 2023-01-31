---
title: "Obeject Detection 이란?"
last_modified_at: 2023-01-31
author: 김상민
---

-------------

이번 포스팅에서는 **Object Detection(객체 탐지)**에 관한 기본적인 내용을 공유해보려고 합니다. 

---------------

> ## Object Detection 이란?
![1](https://user-images.githubusercontent.com/102953592/215691714-a62818e3-4f38-48f4-a041-aa9e8fe00ff0.JPG)


Object Detection이란 컴퓨터 비전과 이미지 처리와 관련된 컴퓨터 기술로서, 여러 물체에 대해 어떤 물체인지 분류하는 Classification 문제와 그 물체가 어디 있는지 박스를 통해 바운딩 박스(Bounding box) 위치 정보를 나타내는 Localization 문제를 둘 다 해내야 하는 분야를 뜻합니다. 

Classification : 특정 이미지가 어떤 객체인지 클래스를 판단하는 분류를 하며 주로 CNN 계열의 모델을 사용합니다.
Classification + Localization : Localization이란, 단 1개의 객체 위치를 바운딩 박스(Bounding Box)로 지정해 찾는 것을 의미합니다. 
Object Detection : 여러 개의 객체들에 대한 위치를 여러 개의 바운딩 박스로 지정해서 찾습니다. 이 때 동시에 여러 개의 객체들이 무엇을 의미하는지 클래스 분류도 수행힙니다.
Segmentation : Pixel-level로 Object Detection을 하는 것을 의미합니다. 그림에서 Object Detection일 때 바운딩 박스와 비교해보면 시각적으로 차이점이 무엇인지 확연히 알 수 있습니다.

![3](https://user-images.githubusercontent.com/102953592/215691963-e0a8cb5f-5598-4f11-a7cf-c8e2615913c2.JPG)

객체 탐지는 위와 같이 물체의 위치(바운딩 박스)를 찾는 '예측'과 물체를 분류하는 '분류' 작업을 함께 수행합니다.


> ## Bounding Box 이란?
![2](https://user-images.githubusercontent.com/102953592/215691914-e572c5cd-b289-4396-85fe-e57f2bbd9c1f.JPG)

객체 탐지에서 박스의 형태로 나타내는 위치를 의미합니다. 이미지 상에서 수직/수평 방향을 향한 직사각형의 모양의 박스입니다.


> ## 손실 함수 (Loss Function)
![5](https://user-images.githubusercontent.com/102953592/215692205-5a439cac-1746-416d-a1a3-7496cf3c7e72.JPG)

 1) Classification loss : 제대로 된 분류를 위한 손실 함수
 2) Localization loss : 해당 Grid 안에 물체가 있을지 없을지에 대한 손실 함수
 3) Confidence loss : 바운딩 박스의 Center Point(X, Y)와 Width, Height를 잘 찾기 위한 회귀 손실 함수


> ## 모델 연혁
![6](https://user-images.githubusercontent.com/102953592/215692239-c7367640-595e-4068-aafd-2f883a6f1880.JPG)

-One Stage : 
 Classification, Regional Proposal을 동시에 수행하여 결과를 얻는 방법
 이미지를 모델에 입력 후, Conv Layer를 사용하여 이미지 특징을 추출
 Example) YOLO, SSD

-Two Stage :
 Classification, Regional Proposal을 순차적으로 수행하여 결과를 얻는 방법
 비교적 One-Stage Detector는 빠르지만 정확도가 낮고, Two-Stage Detector는 느리지만 정확도가 높음
 Example) R-CNN, Fast R-CNN, Faster R-CNN


> ## 성능 평가 매트릭스
- IoU(Intersection Over Union)
![4](https://user-images.githubusercontent.com/102953592/215692082-90ea2272-1ba8-44d4-8428-99278c3fefc9.JPG)

실제 객체가 있는 바운딩 박스(Ground Truth)와 모델이 예측한 바운딩 박스(Predicted)가 포함하고 있는 영역을 계산합니다.

- mAP(mean-Average Precision)
Precision, Recall은 Trade-off 관계가 존재합니다. 즉, 두 개의 값을 모두 최상위 값으로 끌어올릴 수 없습니다. 어느 한 값으로 알고리즘의 성능을 판단하기는 어렵고, 두 값을 종합해서 알고리즘을 평가하기 위한 것이 Average Precision입니다.


이상으로 Object Detection에 대해 간단히 알아보았습니다.

------------------------

> **참고 자료**  
* [github](https://github.com/sgrvinod/a-PyTorch-Tutorial-to-Object-Detection)
* [Blog](https://pseudo-lab.github.io/Tutorial-Book/chapters/object-detection/Ch1-Object-Detection.html)
* [Blog](https://techblog-history-younghunjo1.tistory.com/178)
