---
layout: post
title: "VAR - 다변량 시계열 분석"
date: 2022-4-22
author: 이한솔
---

이 포스팅에서는 다변량 시계열 분석 모형인 **VAR(Vector AutoRegression)** 구현 방법에 대해 알아보겠습니다.

> **VAR 이란** 변수간에 나타나는 상관관계와 인과관계를 추정할 수 있는 다변량 시계열 모형입니다.

---

## **목차**
1. 데이터셋
2. 데이터 로드
3. 피어슨 상관 관계 분석
4. 정상성 확인
   1. ACF 시각화 확인
   2. ADF
5. 차분
6. 모델 선정 (P parameter) 및 학습
7. 예측 결과

---

## **1. 데이터셋**
데이터셋은 ETT (Electricity Transformer Temperature, 변압기 오일 온도)을 사용하였습니다. 

ETT는 변압기 오일 온도 예측 및 극한 부하 용량 조사를 위한 데이터셋입니다. <br>
컬럼은 다음과 같습니다.

date<br>
HUFL : High UseFul Load <br>
HULL : High UseLess Load <br>
MUFL : Middle UseFul Load <br>
MULL : Middle UseLess Load <br>
LUFL : Low UseFul Load <br>
LULL : Low UseLess Load <br>
**OT : Oil Temperature** (Target) 

---

## **2. 데이터 로드**
먼저 csv 파일로 되어있는 데이터를 불러옵니다. 표본은 500개의 데이터로 설정하였습니다.
```python
ETT = pd.read_csv('/content/gdrive/MyDrive/VAR_ETT/ETTh1.csv')
ETT['date'] = pd.to_datetime(ETT['date'])
ETT = ETT[1000:1500]
```
<img src="https://user-images.githubusercontent.com/96156882/170914666-3d0bfc85-a69c-4eb4-931e-b496d220640d.png" width="700">

결측치가 있는지 확인합니다. <br>
모든 컬럼에 결측치가 없는 것으로 확인되었습니다.
```python
ETT.info()
```
RangeIndex: 17420 entries, 0 to 17419 <br>
Data columns (total 8 columns): <br>
<img src="https://user-images.githubusercontent.com/96156882/170914427-109bdb7f-6c2b-4803-b51c-e57642aa9570.png" width="400">

---

## **3. 피어슨 상관 관계 분석**
> **피어슨 상관 관계**는 두 변수가 어떤 상관 관계를 가지는지를 분석합니다. <br>
> 1에 가까울수록 관계가 깊고 0에 가까울수록 관계가 적음을 의미합니다.
```python
heat_map = sns.heatmap(ETT.corr(method='pearson'), annot=True, fmt='.2f', linewidths=2)
heat_map.set_xticklabels(heat_map.get_xticklabels(), rotation=45);
```
<img src="https://user-images.githubusercontent.com/96156882/164161250-f774591f-094e-41d3-a869-16d2f6fd4351.png" width="400">

이를 통해 변수들과 Target값(OT)의 관계가 적음을 알 수 있습니다.

---

## **4. 정상성 확인**
VAR 모형을 사용하기 위해서는 시계열 데이터가 정상성인지 확인해주고, 정상성이 아니라면 차분을 해주어야 합니다.
제가 정상성을 확인한 방법은 **ACF 시각화, ADF 단위근 검정** 두가지 입니다.

**4-1. ACF 시각화**
   > ACF는 자기상관함수(AutoCorrelation Function)로 시차에 따른 일련의 자기 상관을 의미합니다. 정상 시계열일 경우 빠르게 0에 수렴하며, 추세가 없어야 합니다.

임의로 OT 데이터를 시각화 한 결과, 정상성이 아님을 알 수 있습니다.
```python
fig, ax = plt.subplots(1,2,figsize=(20,5))
fig.suptitle('Raw Data')
sm.graphics.tsa.plot_acf(ETT['OT'].values.squeeze(), lags=40, ax=ax[0]);
```
<img src="https://user-images.githubusercontent.com/96156882/164163076-90bf1c44-90a5-4a75-8a0a-09c4066c037c.png" width="400">

**4-2.ADF 단위근 검정**
   > ADF(Augmented Dickey Fuller) 검정은 단위근이 존재하는지 확인하는 검정입니다. 단위근은 시계열 데이터에서 예측할 수 없는 결과를 나타낼 수 있기 때문에, 단위근이 존재하지 않는 데이터가 정상성을 만족하는 것으로 보고 있습니다.

ADF 검정 함수는 다음과 같습니다.
```python
def adfuller_test(series, signif=0.05, name='', verbose=False):

    r = adfuller(series, autolag='AIC')
    output = {'test_statistic':round(r[0], 4), 'pvalue':round(r[1], 4), 'n_lags':round(r[2], 4), 'n_obs':r[3]}
    p_value = output['pvalue'] 
    def adjust(val, length= 6): return str(val).ljust(length)

    print(f'    Augmented Dickey-Fuller Test on "{name}"', "\n   ", '-'*47)
    print(f' Null Hypothesis: Data has unit root. Non-Stationary.')

    if p_value <= signif:
        print(f" => P-Value = {p_value}. Rejecting Null Hypothesis.")
        print(f" => Series is Stationary.")
    else:
        print(f" => P-Value = {p_value}. Weak evidence to reject the Null Hypothesis.")
        print(f" => Series is Non-Stationary.")
```
ETT 데이터를 적용해보겠습니다.
```python
for name, column in ETT.iteritems():
    adfuller_test(column, name=column.name)
    print('\n')
```    
    Augmented Dickey-Fuller Test on "OT" 
        ----------------------------------------------- 
    Null Hypothesis: Data has unit root. Non-Stationary. 
    => P-Value = 0.7266. Weak evidence to reject the Null Hypothesis. 
    => Series is Non-Stationary. 

위처럼 HUFL의 출력값을 포함해 모든 변수가 **비정상성**으로 검정되었습니다. <br>

---

## **5. 차분**
비정상성 데이터를 정상성 데이터로 만들어주기 위해서는 차분이 필요합니다.
> 차분(differencing)은 연이은 관측값들의 차이를 계산하는 것으로 시계열의 수준에서 나타나는 변화를 제거하여 시계열의 평균 변화를 일정하게 만드는데 도움이 될 수 있습니다.

이렇게 간단하게 1차 차분을 해준 뒤
```python
df_differenced = ETT.diff().dropna()
```
정상성인지 ACF, ADF를 통해 다시 한번 확인해주도록 하겠습니다.

**ACF 시각화**

<img src="https://user-images.githubusercontent.com/96156882/164166438-797a9ccc-a1bd-4ad2-839d-459c23a45cd7.png" width="400">

**ADF 단위근 검정**

     Augmented Dickey-Fuller Test on "OT" 
    ----------------------------------------------- 
     Null Hypothesis: Data has unit root. Non-Stationary. 
    => P-Value = 0.0. Rejecting Null Hypothesis. 
    => Series is Stationary. 
 
모두 정상성 데이터가 된 것을 확인 할 수 있습니다.

---

## **6. 모델 선정 (P parameter)및 학습**
최적의 모델을 찾기 위해서는 시차길이(P parameter)를 설정 해줘야 합니다. 일반적으로 AIC(Akaike's Information Criterion)가 낮은 parameter를 선택합니다.

**모델 선정**

```python
model = VAR(df_train)
sorted_order = model.select_order(maxlags=10)
print(sorted_order.summary())
```
![Screenshot_1](https://user-images.githubusercontent.com/96156882/164415863-15b41788-cbc5-4948-9279-a621e79de6cf.png)

P parameter를 10으로 설정하고 학습시켜줍니다.<br>

**학습**
```python
fitted = model.fit(10)
fitted.summary()
```

---

## **7. 예측 결과**

예측에 대한 초기 값을 지정해주고
```python
lag_order = 10 # 모델 시차 정보(p)

# 예측 위해 데이터 input
forecast_input = df_test.values[-lag_order:]
forecast_input
```
예측한 결과입니다.
```python
ETT_forecast = ETT[['HUFL','HULL','MUFL','MULL','LUFL','LULL','OT']]

#------ forecast 한 값
fc = fitted.forecast(y=forecast_input, steps=n_test) 
df_forecast = pd.DataFrame(fc, index=ETT_forecast.index[-n_test:], columns=ETT_forecast.columns + '_1d')
df_forecast[:5]
```

![Screenshot_1](https://user-images.githubusercontent.com/96156882/164418895-47ecd5a5-e498-440d-8d8c-0a5c204d6161.png)

**차분 데이터 복구** <br>
정상성 데이터로 바꿔주기 위해 차분을 했기 때문에 차분 값을 다시 되돌려줍니다.
```python
def invert_transformation(real_data, df_forecast, second_diff=True):
    df_fc = df_forecast.copy()
    columns = real_data.columns
    for col in columns:        
        # Roll back 1st Diff
       df_fc[str(col)+'_forecast'] = real_data[col].iloc[-n_test-1] + df_fc[str(col)+'_1d'].cumsum()
    return df_fc

df_results = invert_transformation(ETT, df_forecast, second_diff=False)        
```

**결과 시각화** <br>
예측결과를 시각화 해보겠습니다.

![Screenshot_1](https://user-images.githubusercontent.com/96156882/165193577-d7d85e14-f893-4221-9e0f-219e78d4994e.png)


## **결론**
ETT 데이터셋으로 VAR 예측을 해본 결과 다소 아쉬운 예측 결과를 보였습니다. 또한 표본 추출 개수, train/test 데이터 비율에 따라 결과가 달라진다는 한계점이 있었습니다.

이번 포스팅은 여기까지입니다. 감사합니다.
