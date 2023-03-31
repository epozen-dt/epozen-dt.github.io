---
title: "HSI_Analysis_Process"
last_modified_at: 2023-03-31
author: 김혜원
---

이번 포스팅은 초분광 이미지 분석절차에 대해 공유합니다.

---
&nbsp;

> 초분광 이미지 분석 절차

초분광 이미지는 여러 스펙트럼이 혼합되어 있습니다. 따라서 어떤 스펙트럼으로 구성되어 있는지 알기 위해서는 분석 과정이 필요합니다. 초분광 이미지를 분석하는 절차는 다음과 같습니다.

&nbsp;

    1. 데이터 수집
    2. 초분광 이미지 보정
    3. 분광 혼합 분석
        1) 차원 축소
        2) 엔드 멤버 추출
        3) 유사도 계산
        4) 이미지에 엔드멤버 표시

&nbsp;


먼저 초분광 카메라로 촬영하여 데이터를 수집합니다. 이 때 촬영된 이미지는 대기 상태에 따라 왜곡된 상태로 저장될 가능성이 높습니다. 따라서 대기 상태를 제거한 빛의 반사값을 구해야 합니다. 보정 과정은 픽셀 값을 휘도 값으로 변경한 후 반사 값으로 변경하여 이미지에 적용합니다[[1]](http://koreascience.or.kr/article/JAKO201914260901738.page)[[2]](http://koreascience.or.kr/article/JAKO201422354180255.page)[[3]](https://m.blog.naver.com/rsmilee/220617353744).

&nbsp;

    <휘도 값 구하기>
    * 𝐿=𝐶_1∗𝐷𝑁+𝐶_2
      𝐿 : 휘도 값
      𝐶_1 : gain value
      𝐶_2 : offset value
      𝐷𝑁 : 픽셀 값
        ※ Gain value 와 offset value는 영상 정보에서 얻을 수 있음 (헤더 파일 명시)

    <반사 값 구하기>
    * 𝑅𝑒𝑓=(𝜋(𝐷𝑁−𝐿))/(𝑇_𝑣 (𝐸_0∗𝑇_𝑧∗𝑐𝑜𝑠𝜃+𝐸_𝑑))
      𝐸_0 : 대기 최상층 에너지
      𝐸_𝑑 : 에너지 감소율
      𝑇_𝑣 : 상향 투과율 (지표면부터 센서까지)
      𝑇_𝑧 : 하향 투과율 (대기부터 지표면까지)
      𝜃 : 태양 천정각
    
    - 각 요소 구하는 식 (참고)
    * 𝐸_0=𝐸_𝑠𝑢𝑛/𝑑^2
      𝐸_𝑠𝑢𝑛 : 태양 표면에너지 (5.961×〖10〗^7  𝑊/𝑚^2)
      𝑑 : 태양과 지구 사이 거리
    * 𝑇_𝑧=(𝐸/𝐸_0 )^sin⁡ℎ 
      𝐸 : 측정할 때의 일사량
    * sin⁡〖ℎ=𝑐𝑜𝑠∅∙𝑐𝑜𝑠𝛿∙𝑐𝑜𝑠𝜔+𝑠𝑖𝑛∅∙𝑠𝑖𝑛𝛿〗
      ∅ : 측정 위치의 위도
      𝛿 : 일 적위 23.45 sin⁡(360×(284𝐽_𝑑𝑎𝑦)/365)
        ※ J_day: 줄리안데이
    * cosω=−𝑡𝑎𝑛∅𝑡𝑎𝑛𝛿
      𝜔 : 일몰 시간각 
      𝜃=90°−태양 고도각
        ※ 태양 고도각은 영상 촬영 시 따로 재야 함


&nbsp;

보정된 이미지는 차원의 저주 문제를 피하기 위해 차원을 축소 시킵니다. 축소된 차원에서 엔드 멤버(어떤 물질)를 추출하여 기존에 알려진 물질들의 스펙트럼과 유사도를 계산하여 엔드멤버를 이미지에 표시합니다.
차원 축소는 주성분 분석을 이용하여 특이값 분해[[4]](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE02380592&language=ko_KR&hasTopBanner=true)를 합니다. 특이값을 구하는 과정은 고유값 분해와 같은 방식으로 이루어집니다. 차이점은 고유값 분해는 정방 행렬(MXM)에 대해서만 가능하지만 특이값 분해는 직사각 행렬(MXN)에 적용할 수 있다는 점입니다. 이 과정에서 구한 특이값 중 앞부분 일부를 이용해서 차원을 축소시킵니다.

축소된 차원을 가지는 이미지에 PPI(Pixel Purity Index) 알고리즘[[5]](https://kr.mathworks.com/help/images/ref/ppi.html)을 적용하여 엔드멤버를 추출합니다.


&nbsp;

    
    <PPI 수행 과정>
    1. 초기 엔드멤버 설정 : 전체 픽셀의 평균값
    2. 랜덤으로 이미지 중심을 관통하는 벡터 1개 선정
    3. 이미지의 모든 픽셀을 벡터에 투영
    4. 꼬리에 해당하는 픽셀(벡터 시작점)을 엔드멤버로 기록
    5. 해당 벡터에 직교하고, 이미지 중심을 관통하는 벡터를 선정
    6. 2-5 반복 -> 새로운 엔드멤버가 없을 때까지


&nbsp;

추출한 엔드멤버는 SAM(Spectral Angle Mapper) 알고리즘을 적용합니다. 이 알고리즘은 엔드멤버(테스트 스펙트럼)와 분광 라이브러리(참조 스펙트럼)를 비교하여 유사도를 구합니다. 두 스펙트럼 사이의 각도가 작을수록 유사도가 높은 것입니다. 유사도에 따라 비슷한 엔드멤버끼리 묶어서 분류하고, 이를 이미지에 표시합니다[[6]](http://koreascience.or.kr/ks-full-text-html-image/JAKO201905260913439:1/image.image).

&nbsp;

![SAM](https://www.e-jcs.org/upload//thumbnails/JCS-2020-36-2-01f4.jpg)

&nbsp;




-------
> 참고문헌

[1] [대기 보정](http://koreascience.or.kr/article/JAKO201914260901738.page)

[2] [대기 투과율](http://koreascience.or.kr/article/JAKO201422354180255.page)

[3] [태양 복사 조도](https://m.blog.naver.com/rsmilee/220617353744)

[4] [특이값 분해](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE02380592&language=ko_KR&hasTopBanner=true)

[5] [PPI](https://kr.mathworks.com/help/images/ref/ppi.html)

[6] [분류 예시](http://koreascience.or.kr/ks-full-text-html-image/JAKO201905260913439:1/image.image)