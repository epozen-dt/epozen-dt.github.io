---
title: "데이터 스케일링 (Data Scaling)"
last_modified_at: 2022-11-28
author: 김상민
---

-------------

이번 포스팅에서는 **데이터 스케일링(Data Scaling)**에 관한 내용을 공유해보려고 합니다. 

---------------

![scaling](https://user-images.githubusercontent.com/102953592/204191566-97876d5a-28dc-4e0b-b8aa-7867cabd165e.png)
> ## 스케일링(Scaling) 이란?
데이터 전처리 과정 중 하나로 데이터의 값의 범위를 조정하는 것입니다.
너무 값이 크거나 작은 경우에 모델 알고리즘 학습 과정에서 0으로 수렴하거나 무한으로 발산을 막기 위해 스케일링을 사용합니다.


그렇다면 간단한 예제로 실습해보겠습니다.


> ## StandardScaler
![StandardScaler](https://user-images.githubusercontent.com/102953592/204192050-e348d1d7-5eea-4888-96d0-32cbc4115dfc.JPG)
특성들의 정규분포(평균을 0, 분산은 1)로 스케일링 합니다. 
- 이상치에 민감
- 회귀보다 분류에 주로 이용

``` python
from sklearn.preprocessing import StandardScaler

# 변형 객체 생성
std_scaler = StandardScaler()

# 훈련데이터의 모수 분포 저장
std_scaler.fit(X_train)

# 훈련 데이터 스케일링
X_train_scaled = std_scaler.transform(X_train)

# 테스트 데이터의 스케일링
X_test_scaled = std_scaler.transform(X_test)

# 스케일링 된 결과 값으로 본래 값을 연산
X_origin = std_scaler.inverse_transform(X_train_scaled)
```


> ## MinMaxScaler
![MinMaxScaler](https://user-images.githubusercontent.com/102953592/204192102-b17eb0e5-f78b-48bd-bc7f-c3d84bdeeec8.JPG)
가장은 값을 0, 가장 큰값을 1로 스케일링 합니다.
- 이상치에 민감
- 분류보다 회귀에 주로 이용

``` python
from sklearn.preprocessing import MinMaxScaler

# 변형 객체 생성
minmax_scaler = MinMaxScaler()

# 훈련데이터의 모수 분포 저장
minmax_scaler.fit(X_train)

# 훈련 데이터 스케일링
X_train_scaled = minmax_scaler.transform(X_train)

# 테스트 데이터의 스케일링
X_test_scaled = minmax_scaler.transform(X_test)

# 스케일링 된 결과 값으로 본래 값을 연산
X_origin = minmax_scaler.inverse_transform(X_train_scaled)
```


> ## MaxAbsScaler
![MaxAbsScaler](https://user-images.githubusercontent.com/102953592/204192123-b77cece9-f95e-4885-8089-41d92c7c93c1.JPG)
데이터의 절대 값을 0~1 범위로 제한합니다. 즉, 데이터의 값은 -1에서 +1로 스케일링 합니다.
- 이상치에 민감
- 분류보다 회귀에 주로 이용

``` python
from sklearn.preprocessing import MaxAbsScaler

# 변형 객체 생성
maxabs_scaler = MaxAbsScaler()

# 훈련데이터의 모수 분포 저장
maxabs_scaler.fit(X_train)

# 훈련 데이터 스케일링
X_train_scaled = maxabs_scaler.transform(X_train)

# 테스트 데이터의 스케일링
X_test_scaled = maxabs_scaler.transform(X_test)

# 스케일링 된 결과 값으로 본래 값을 연산
X_origin = maxabs_scaler.inverse_transform(X_train_scaled)
```


> ## RobustScaler
![RobustScaler](https://user-images.githubusercontent.com/102953592/204192142-30952e85-d5cd-40f4-97c0-637484fc137c.JPG)
평균과 분산 대신에 중간 값(median)과 사분위 값을 사용합니다.
사분위수를 사용하기 때문에 아주 동떨어진 데이터, 즉, outlier 제거에 효과적입니다.
- 이상치 영향을 최소화
- 분류보다 회귀에 주로 이용

``` python
from sklearn.preprocessing import RobustScaler

# 변형 객체 생성
robust_scaler = RobustScaler()

# 훈련데이터의 모수 분포 저장
robust_scaler.fit(X_train)

# 훈련 데이터 스케일링
X_train_scaled = robust_scaler.transform(X_train)

# 테스트 데이터의 스케일링
X_test_scaled = robust_scaler.transform(X_test)

# 스케일링 된 결과 값으로 본래 값을 연산
X_origin = robust_scaler.inverse_transform(X_train_scaled)
```


> ## Normalizer
![Normalizer](https://user-images.githubusercontent.com/102953592/204192165-3e50ed38-8bf1-417d-a63c-69bca8e73880.JPG)
앞의 4가지 스케일러는 각 특성(열, columns)의 통계치를 이용하지만,  Normalizer 의 경우 각 샘플(행, row)마다 적용되는 방식입니다.
한 행의 모든 특성들 사이의 유클리드 거리(L2 norm)가 1이 되도록 스케일링 합니다.
- 모델(특히나 딥러닝) 내 학습 벡터에 적용

``` python
from sklearn.preprocessing import Normalizer

# 변형 객체 생성
normal_scaler = Normalizer()

# 훈련데이터의 모수 분포 저장
normal_scaler.fit(X_train)

# 훈련 데이터 스케일링
X_train_scaled = normal_scaler.transform(X_train)

# 테스트 데이터의 스케일링
X_test_scaled = normal_scaler.transform(X_test)

# 스케일링 된 결과 값으로 본래 값을 연산
X_origin = normal_scaler.inverse_transform(X_train_scaled)
```


이상으로 5가지 스케일링 종류 및 간단한 예제와 함께 알아보았습니다.

------------------------

> **참고 자료**  
* [sklearn](https://scikit-learn.org/stable/modules/preprocessing.html)
* [Blog](https://wooono.tistory.com/96)
* [Image](https://towardsai.net/p/data-science/how-when-and-why-should-you-normalize-standardize-rescale-your-data-3f083def38ff)