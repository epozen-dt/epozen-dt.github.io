---
title: "triplet-loss"
last_modified_at: 2024-04-30
author: 김혜원
---

본 포스팅은 facenet에 사용된 손실함수인 triplet-loss에 대해 공유합니다.

---

triplet-loss는 facenet에서 사용된 손실함수로, 얼굴 인식을 목적으로 한 지도학습에서 사용됩니다. triplet-loss는 Anchor(앵커), Positive(양성), Negative(음성)으로 구성되어 있고, 각각의 의미는 다음과 같습니다.


* Anchor(앵커): 학습 데이터셋에서 어떤 객체의 표현을 학습하기 위해 사용되는 기준점입니다. 예를 들어, 얼굴 인식에서는 특정 사람의 얼굴 이미지를 앵커로 사용할 수 있습니다.
* Positive(양성): 앵커와 동일한 객체를 나타내는 샘플입니다. 즉, 동일한 객체의 다른 표현이 될 수 있습니다. 얼굴 인식의 경우, 동일한 사람의 다른 사진이 양성 샘플이 될 수 있습니다.
* Negative(음성): 앵커와 다른 객체를 나타내는 샘플입니다. 즉, 다른 객체의 표현입니다. 얼굴 인식의 경우, 다른 사람의 사진이 음성 샘플이 될 수 있습니다.

&nbsp;

![image](https://github.com/khw927/epozen-dt.github.io/assets/107157737/ebf665d6-cbe0-4487-8bf1-ead07ff4a08a)

&nbsp;

모델은 triplet-loss를 사용하여 anchor와 positive 사이의 거리를 최소화하고, negative와의 거리를 최대화하는 방향으로 학습합니다. 유클리디안 거리를 사용하며, 다음 식을 통해 값을 구합니다.


![image](https://github.com/khw927/epozen-dt.github.io/assets/107157737/2793d986-09ce-4e13-b136-ba241e17fec8)


&nbsp;


$$\begin{align*} 
&<각 기호의 의미> \\
\\
&A: 앵커에\,입력할\,것 \\
&P: A와\,동일한\,클래스의\,Positive \\ &N: A와\,동일한\,클래스의\,Negative \\
&\alpha : P와\,N사이의\,공백\,(margin) \\
&f: 임베딩\,(embedding) \\
&||\,\cdot\,|| : 두\,벡터\,간의\,유클리드\,거리
\end{align*}$$

&nbsp;

triplet loss는 mini-batch 별로 구하며, min-max를 정할 수 있는 최소한의 샘플들은 항상 존재해야 합니다. 참고한 설명에 따르면 약 1800개의 mini-batch를 사용한다고 합니다. 이 손실함수를 사용함으로써 안면 인식의 정확도를 높일 수 있습니다.

&nbsp;

&nbsp;

&nbsp;

본 포스팅은 여기서 마칩니다.

&nbsp;


------
> 참고

[1] [facenet 논문](https://www.cv-foundation.org/openaccess/content_cvpr_2015/papers/Schroff_FaceNet_A_Unified_2015_CVPR_paper.pdf)

[2] [설명](https://soobarkbar.tistory.com/43)

