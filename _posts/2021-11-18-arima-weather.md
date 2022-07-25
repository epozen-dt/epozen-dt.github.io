---
title: "시계열 데이터 분석 - ARIMA 모형을 활용한 날씨 예측"
last_modified_at: 2021-11-29
author: 심건우
---

이번 포스팅에서는 시계열 데이터 분석 도구 중 ARIMA 모형을 활용하여 날씨 예측을 해보겠습니다.


## 시계열 데이터

시계열 데이터란 시간의 흐름에 따라 관찰된 값을 말합니다.

시계열 데이터 분석의 주 목적은 미래의 값을 예측하거나 데이터의 특성을 파악하는 것이라 할 수 있습니다.

그 예로, 날씨 예측, 매출액 예측 등이 있고 데이터의 경향, 주기, 계절성 파악 등이 있습니다.

시계열 데이터를 분석하는 방법은 장기 예측과 단기 예측으로 분류할 수 있습니다.

- 장기 예측 : 회귀분석 등
- 단기 예측 : Box-Jenkins, 지수평활법, 시계열 분해 등

이번 포스팅에서 사용할 ARIMA 모형은 Box-Jenkins 방법에 해당됩니다.

분석 방법 소개에 앞서, 시계열 데이터의 요소와 정상성에 대해 짚고 가겠습니다.

시계열 데이터 요소에는 다음과 같은 것들이 있습니다.

- 추세 (trend) : 장기적으로 나타나는 변동 패턴
- 계절성 (Seasonal) : 주, 월, 분기 등 이미 알려진 시간의 주기로 나타나는 패턴
- 랜덤 요소 (random/residual/remainder)

![image](https://user-images.githubusercontent.com/87160438/142582177-268ea17f-b25c-418b-b9db-dc7bc450e9cd.png)



## 정상성

시계열 데이터에서의 정상성이란, 데이터의 특징이 해당 데이터의 관측 시간과 무관함을 말합니다.

자세히 말하면, 계절성과 추세가 보이지 않고, 데이터의 평균이나 분산의 증감이 없어야 정상성을 갖는 데이터라 할 수 있습니다.

![image](https://user-images.githubusercontent.com/87160438/142582319-7e42c74b-8bec-4be3-9212-0e6c80263032.png)

출처 : https://otexts.com/fppkr/stationarity.html

정상성을 만족하지 못하는 데이터는 분석이나 예측이 어렵기 때문에 정상성을 만족하게 변형해줘야 합니다.

눈으로만 봐선, 데이터가 정상성을 갖는지, 아닌지 구분하기 어렵습니다. 이러한 어려움은 정량적으로 정상성을 판단하는 방법을 사용하면 해소될 수 있습니다.

정상성 판단은 아래와 같은 방법들을 통해 할 수 있습니다.

1. ACF(자기상관함수, AutoCorrelation Function)

   - 시차가 커질수록 ACF 값은 0에 수렴한다.
   - 정상성을 갖는 시계열 데이터는 상대적으로 빠르게 0에 수렴한다.
   - 비정상성을 갖는 시계열 데이터는 상대적으로 천천히 감소하고, 종종 큰 양의 값을 갖는다.

    ![image](https://user-images.githubusercontent.com/87160438/142583130-f3abfc73-ac0b-4bdd-9125-9b1a8464b983.png)

2. ADF 검정 (Augmented Dickey-Fuller Test)

   - 시계열 데이터가 정상성을 갖는지 확인하는 단위근 검정 중 하나이다.
     (단위근 검정 : 확률 변수가 안정적인지 불안정적인지 확인하는 검정)
   - 귀무 가설 : 데이터에 단위근이 존재한다 -> 데이터가 정상성을 만족하지 않는다.
   - 대립 가설 : 데이터가 정상성을 만족한다.
   - p-value 가 0.05 보다 클 경우, 귀무 가설 기각이 불가하다. -> 정상성을 만족하지 못 한다.
     (p-value : 귀무가설이 맞다고 가정할 때 얻은 결과보다 극단적인 결과가 실제로 관측될 확률)
   - python 으로 작성한 ADF 검정 예시
   
   
      ```python
      def adf_test(df):
        result = adfuller(df.values)
        p_value = result[1]
        return p_value
      ```

3. KPSS 검정
   
   - 시계열 데이터가 평균 또는 선형 추세 주변에 고정되어 있는지 또는 단위근으로 인해 고정되어 있지 않은지 검정하는 방법
   - 귀무 가설 : 데이터가 정상성을 만족한다.
   - 대립 가설 : 데이터가 정상성을 만족하지 않는다.
   - p-value 가 0.05 보다 작을 경우, 귀무 가설을 기각한다. -> 정상성을 만족하지 못 한다.
   - python 으로 작성한 KPSS 검정 예시
   
   
      ```python
      def kpss_test(df):
        statistic, p_value, n_lags, critical_values = kpss(df.values)
        return p_value
      ```



위 방법들을 거친 데이터가 정상성을 띄지 않는 경우 정상성을 띄게끔 변환해주는 작업이 필요합니다. 변환 방법에는 다음과 같은 것들이 있습니다.

  - 분산의 변화가 일정하지 않은 경우 -> 로그 변환
  - 평균의 변화가 일정하지 않은 경우(추세, 계절성이 존재하는 경우) -> 차분(Differencing)
    - 1차 차분 후, 정상성을 띄지 않는 경우 -> 차분 반복
    - 차분 예시 ([세계 2차 대전 날씨 데이터, Kaggle](https://www.kaggle.com/smid80/weatherww2))
    

![image](https://user-images.githubusercontent.com/87160438/142587529-d454e100-c5d9-434a-8535-aa88a9fe51e8.png)



## ARIMA

본 포스팅에선 Box-Jenkins 방법 중 ARIMA 모형으로 날씨 예측을 시도해 본 것에 대해 다룹니다.

그 전에 Box-Jenkins, ARIMA 모형 순으로 간단하게 설명하겠습니다.

Box-Jenkins 방법이란, 확률적인 시계열 모형을 주어진 데이터에 적용 시, 모형의 적합성 여부에 대해 판단하는 것이라 할 수 있습니다.

아래는 그 흐름도 입니다.

![image](https://user-images.githubusercontent.com/87160438/142790789-2d2ccbf0-480b-4d4a-bb83-2027c09767ee.png)

ARIMA(Auto Regressive Integrated Moving Average)는 확률에 기반하여 시계열 데이터를 분석하고 예측하는 모형이라 할 수 있습니다.

ARIMA 는 AR 모형과 MA 모형을 합친 모형이라 할 수 있습니다.

여기서 AR 모형은 AR(p)로 표기할 수 있고, p 만큼의 과거값과 현재값의 선형조합을 통해 다음 값을 예측하는 모형이라 할 수 있습니다.

공식은 아래와 같습니다.

![image](https://user-images.githubusercontent.com/87160438/142591455-ff7f4539-fd5e-45a1-a2e9-5dd47b4011a4.png)

MA 모형은 MA(q)로 표기할 수 있고, q 만큼의 과거 예측 오차를 이용하여 다음 값을 예측하는 모형이라 할 수 있습니다.

공식은 아래와 같습니다.

![image](https://user-images.githubusercontent.com/87160438/142591552-e74bdd69-42e4-43a8-b234-da48697931bd.png)

위 두 모형을 합친 것과 차분 횟수를 포함시킨 것을 ARIMA 모형이라 할 수 있고, 표기는 ARIMA(p, d, q) 입니다.

즉, d차 차분한 시계열 데이터에 AR(p) 모형과 MA(q) 모형을 적용한 것이라 할 수 있습니다.

ARIMA 모형을 적용하려는 데이터는 반드시 정상성을 띄어야 한다는 것이 전제 조건입니다.

공식은 아래와 같습니다.

![image](https://user-images.githubusercontent.com/87160438/142594200-bcd56050-4f48-43b0-9235-9cfa8ea5fb9f.png)

본 포스팅에서 언급한 데이터인 날씨 데이터는 계절성을 띈다고 할 수 있습니다.

이러한 계절성을 갖는 데이터를 다룰 수 있는 ARIMA 모형의 일종으로 S-ARIMA 모형이 있습니다.

S-ARIMA(p, d, q)(P, D, Q, m)으로 표기할 수 있고, 일반 ARIMA 모형에 계절성 관련 항을 포함한 모형이라 할 수 있습니다.

단순히, 추가적인 계절성 항을 비계절성 항에 곱한 것이고, m은 매년 관측값의 개수입니다.

![image](https://user-images.githubusercontent.com/87160438/142594580-c7c6fc15-4cd1-4915-89fe-01c96bd35c71.png)

공식은 아래와 같습니다.

![image](https://user-images.githubusercontent.com/87160438/142594599-74098f3b-4397-48c4-93c7-544295d501db.png)

공식 출처 : https://otexts.com/fppkr/arima.html


## 날씨 예측 예제

예제는 Google Colab에서 Python을 사용했습니다.

먼저 관련 모듈을 설치하겠습니다.


```python
!pip install pmdarima
```

Colab에서 구글 드라이브와 연결하기 위한 설정을 지정합니다.


```python
from google.colab import drive
drive.mount('/gdrive', force_remount=True) # 구글 드라이브 경로
```

csv 형식의 데이터를 다루기 위한 라이브러리를 import 합니다.


```python
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
```

csv 형식의 날씨 데이터를 로드합니다.


```python
weather_station_location = pd.read_csv('/gdrive/MyDrive/Colab/arima/Weather Station Locations.csv') # csv 형식의 데이터
weather = pd.read_csv('/gdrive/MyDrive/Colab/arima/Summary of Weather.csv')
weather_station_location = weather_station_location.loc[:,['WBAN', 'NAME', 'STATE/COUNTRY ID', 'Latitude', 'Longitude']]
weather = weather.loc[:, ['STA', 'Date', 'MeanTemp']]
```

데이터 특징을 파악하기 위해 시각화합니다.


```python
weather_station_id = weather_station_location[weather_station_location.NAME == 'BINDUKURI'].WBAN
weather_bin = weather[weather.STA == int(weather_station_id)]
weather_bin = weather_bin.drop(columns=['STA'])
weather_bin.columns = ['date', 'temp']
weather = weather_bin
weather['date'] = pd.to_datetime(weather['date'])
weather.index = weather['date']
weather.set_index('date', inplace=True)
print(weather['temp'].size)
weather.plot()
```

시각화 결과입니다.

![image](https://user-images.githubusercontent.com/87160438/142818473-5401140f-0b3e-48bb-bc23-ef89a08e8d29.png)

계절성을 보이는 것 같습니다. 좀 더 자세히 알아보기 위해 시계열 분해를 진행합니다.


```python
from statsmodels.tsa.seasonal import seasonal_decompose

decomposition = seasonal_decompose(weather['temp'], period=12)
fig = decomposition.plot()
```

겨울에는 온도가 낮아지고, 여름에는 온도가 높아지는 듯한 계절성을 볼 수 있습니다.

![image](https://user-images.githubusercontent.com/87160438/142818635-9abf591d-5229-4cc6-b070-7c7dc58b04e3.png)

학습 데이터셋과 테스트 데이터셋을 구성합니다.


```python
test_num = 20 # 테스트 데이터 수
train, test = weather[:weather['temp'].size - test_num], weather[weather['temp'].size - test_num:]
```



### 데이터 정상화

데이터의 특징을 파악했고 계절성이 있다는 것을 알았습니다.

눈으로 파악한 특징 검증을 위해, 정량적 검증을 진행합니다.

먼저 ACF 그래프를 적용해봅니다.


```python
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

print(plot_acf(weather))
```

결과를 보았을 때, 시차에 따라 ACF 값이 천천히 감소하는 것을 볼 수 있습니다. 이는 데이터가 정상성을 띄지 않음을 의미합니다.

![image](https://user-images.githubusercontent.com/87160438/142820133-38411d4b-ffad-47d1-bcec-3c7b8567e66b.png)



다음으로 ADF 안정성 테스트와 KPSS 안정성 테스트를 동시에 진행하여 교차 검증합니다.

ADF 안정성 테스트란, 정상성을 판단하는 방법 중 하나입니다. p-value가 0.05보다 큰 경우 정상성을 띄지 않음을 의미합니다.

KPSS 안정성 테스트란, 시계열 데이터가 평균 또는 선형 추세 주변에 고정되어 있는지 또는 단위근으로 인해 고정되어 있지 않은지 확인하는 방법입니다. p-value가 0.05보다 작은 경우 정상성을 띄지 않음을 의미합니다.

아래는 두 테스트를 동시에 적용하여 정상성을 판단하고, 정상성을 띌 때까지 차분을 수행하여 차분 횟수를 구하는 코드입니다.


```python
from statsmodels.tsa.stattools import adfuller
from statsmodels.tsa.stattools import kpss

def adf_test(df):
  result = adfuller(df.values)
  p_value = result[1]
  return p_value
  
def kpss_test(df):
  statistic, p_value, n_lags, critical_values = kpss(df.values)
  return p_value

def check_stationary(df):
  if (adf_test(df) > 0.05):
    result = False
  else:
    if (kpss_test(df) < 0.05):
      result = False
    else:
      result = True
  print(f'adf_test p-value : {adf_test(df)}')
  print(f'kpss_test p-value : {kpss_test(df)}')
  return result

diff_1 = weather['temp']
d_n = 0
while check_stationary(diff_1) == False:
  d_n += 1
  diff_1 = diff_1.diff().dropna()
  # diff_1 = diff_1.diff(periods=12).dropna() # 계절성 차분
print(f'\n*** differencing count : {d_n}')
```

아래 결과를 보면, 차분을 1회 수행해야 데이터가 정상성을 띄는 것을 알 수 있습니다.

![image](https://user-images.githubusercontent.com/87160438/142821324-f98f9bd6-ce0c-4c73-a293-3975a09b6c8e.png)

아래 그래프는 차분 전과 후 데이터를 비교한 것입니다.

![image](https://user-images.githubusercontent.com/87160438/142821736-f1b321b3-6247-4d65-94fe-dc872b559e25.png)



### ARIMA 모델 파라미터 설정

파라미터는 auto_arima라는 라이브러리를 통해 설정합니다.

또, ARIMA 모형과 S-ARIMA 모형을 적용했을 때의 차이점 파악을 위해 동시에 모델을 구성합니다.



먼저, auto_arima를 사용하기 위한 pmdarima를 import 합니다.


```python
from pmdarima.arima import auto_arima
```



ARIMA 모형을 구성합니다.


```python
  model_arima = auto_arima(train['temp'], # 데이터
                          seasonal=False, # 계절성 여부
                          start_p=0, d=d_n, start_q=0, # ARIMA(p, d, q)(P, D, Q)[m] 값 설정
                          max_p=3, max_d=3, max_q=3,
                          supress_warnings=True,
                          stepwise=True,
                          trace=True)
```

![image](https://user-images.githubusercontent.com/87160438/142822477-2a2c8720-f20c-4fdc-a80b-2bc4c41cf66c.png)



S-ARIMA 모형을 구성합니다.


```python
model_sarima = auto_arima(train['temp'], # 데이터
                         seasonal=True, # 계절성 여부
                         start_p=0, d=d_n, start_q=0, # ARIMA(p, d, q)(P, D, Q)[m] 값 설정
                         max_p=3, max_d=3, max_q=3,
                         start_P=0, D=d_n, start_Q=0,
                         max_P=3, max_D=3, max_Q=3,
                         m=12, # 계절성 주기 -> 연도 별 날씨 데이터 특성 고려, 12 로 지정
                         supress_warnings=True,
                         stepwise=True,
                         trace=True)
```

![image](https://user-images.githubusercontent.com/87160438/142822571-290568fe-4fa1-4721-8449-efb3ea89b9a9.png)



테스트 데이터를 각각 구성한 모델에 적용하고 결과를 비교합니다.



결과 시각화를 위한 matplotlib 라이브러리를 import 합니다.


```python
import matplotlib.pyplot as plt
```



ARIMA 모형 적용 결과입니다.


```python
pred = pd.DataFrame(model_arima.predict(n_periods=test_num), index=test.index)
pred.columns = ['temp']

plt.figure(figsize=(15,4))
plt.plot(train[-(test_num * 2):], label='train')
plt.plot(test, label='test')
plt.plot(pred, label='pred')
plt.legend(loc='Left corner')
plt.show()
```

![image](https://user-images.githubusercontent.com/87160438/142823863-6abd4f0b-8622-40b3-8c73-b56e17c411e1.png)



S-ARIMA 모형 적용 결과입니다.

```python
pred_s = pd.DataFrame(model_sarima.predict(n_periods=test_num), index=test.index)
pred_s.columns = ['temp']

plt.figure(figsize=(15,4))
plt.plot(train[-(test_num * 2):], label='train')
plt.plot(test, label='test')
plt.plot(pred_s, label='pred')
plt.legend(loc='Left corner')
plt.show()
```

![image](https://user-images.githubusercontent.com/87160438/142824015-55e5518f-283d-4031-bc32-c65a7354b818.png)

ARIMA는 단순 추세 정도만 파악 가능했고, S-ARIMA는 비교적 상세하게 파악 가능했습니다.



### 성능 평가

단순 그래프로만으론 성능 평가가 어렵기에 회귀 모델에 많이 사용되는 아래 4가지 성능 평가 방법을 도입했습니다.

  - R2 score (결정 계수) : 실제 값의 분산 대비 예측 값의 분산 비율 
     (1에 가까울수록 좋은 모델, 0에 가까울 수록 나쁜 모델, 음수인 경우 잘못 평가되었음을 의미)
  - 피어슨 상관계수 : 두 값의 선형 상관 관계를 계량화한 수치 (1에 가까울수록 좋은 모델)
  - RMSE : 평균 제곱근 오차 (0에 가까울 수록 좋은 모델)
  - MAPE : 평균 절대 백분율 오차 (0에 가까울 수록 좋음, 단위 %)



아래는 4가지 방법을 적용한 코드입니다.


```python
import numpy as np
from sklearn.metrics import r2_score, mean_squared_error

def MAPE(test, pred):
	return np.mean(np.abs((test - pred) / test)) * 100 
  
def scoring(test, pred):
  r2 = r2_score(test, pred) # r2 점수
  corr = np.corrcoef(test, pred)[0, 1] # 피어슨 상관계수
  mape = MAPE(test, pred) # mean absolute percentage error
  rmse = mean_squared_error(test, pred, squared=False) # root mean squared error
  df = pd.DataFrame({'R2' : round(r2, 3),
                     'Corr' : round(corr, 3),
                     'RMSE' : round(rmse, 3),
                     'MAPE' : round(mape, 3)},
                    index = [0])
  return df
```



ARIMA 모형 성능 평가 결과입니다.

![image](https://user-images.githubusercontent.com/87160438/142824749-11842967-d057-4fb4-8cfc-9d492c6dcf35.png)



S-ARIMA 모형 성능 평가 결과입니다.

![image](https://user-images.githubusercontent.com/87160438/142824770-79abedbc-9a94-4265-9907-b7ebecb20b57.png)



위 결과를 통해, 본 포스팅에서 사용한 날씨 데이터에는 S-ARIMA 모형을 적용하는 것이 더 좋은 결과를 나타냄을 알 수 있습니다.



마지막으로, 본 포스팅에서 진행한 내용과 알게 된 것들에 대해 정리하겠습니다.

  1. ACF, ADF, KPSS 등을 정상성 판단 도구로 사용했습니다.

    -> 정량적 도구를 사용했기에, 육안으로 판단하는 것보다 비교적 정확한 판단을 내릴 수 있었습니다.
  2. 데이터 정상화를 위해 차분을 사용했습니다.

    -> 데이터 특성에 따라 다른 방법이 요구될 수 있습니다.
  3. pmdarima의 auto_arima를 사용했습니다.

    -> 최적의 파라미터를 설정할 수 있었습니다.
  4. 회귀 모델에 많이 사용되는 r2_score, 피어슨 상관계수, RMSE, MAPE를 성능평가지표로 사용했습니다.

    -> 결과에 객관성을 부여할 수 있었습니다.
  5. 계절성을 보이는 데이터에 ARIMA 모형과 S-ARIMA 모형을 적용하여 결과를 비교했습니다.

    -> 계절성을 보이는 데이터는 S-ARIMA 모형을 적용하는 것이 더 정확한 결과를 도출함을 알 수 있었습니다.



이상입니다.
