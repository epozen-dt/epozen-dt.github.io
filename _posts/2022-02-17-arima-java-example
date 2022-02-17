---
layout: post
title: "시계열 데이터 분석 - ARIMA & S-ARIMA 모형 적용 결과 비교 테스트 (Python vs Java)"
date: 2022-02-17
author: 심건우
---

이번 포스팅에서는 시계열 데이터 분석 도구 중 ARIMA 모형과 S-ARIMA 모형을 Java로 적용해보고, Python으로 적용한 결과와 비교하겠습니다.

## 테스트 조건
  
  - 각 테스트에 적용될 모형의 파라미터는 Python 분석 결과에 사용된 모형의 파라미터와 같습니다.
  - 데이터셋은 비행기 탑승객 데이터(train : 132개 / test: 12개), 날씨 데이터(train : 731 / test : 20)를 사용했습니다.
  - 테스트에 사용되는 Java 라이브러리 이름은 Signaflo, Workday 입니다.
  
## 테스트 과정

  - 데이터들과 모형들의 조합에 따라 테스트 데이터 수만큼 예측 데이터를 구하고, 시각화하여 성능 평가 결과를 비교합니다.
  
  1. 비행기 탑승객 데이터, ARIMA(2,1,2) 모형
  2. 비행기 탑승객 데이터, S-ARIMA(1,1,0,0,1,0,12) 모형
  3. 날씨 데이터, ARIMA(0,1,2) 모형
  4. 날씨 데이터, S-ARIMA(1,1,2,3,1,0,12) 모형
  
## 테스트 코드

  - Signaflo Maven Dependency
  
  ```xml
  <dependency>
    <groupId>com.github.signaflo</groupId>
    <artifactId>timeseries</artifactId>
    <version>0.4</version>
  </dependency>
  ```
  
  - Workday Maven Dependency
  
  ```xml
  <dependency>
    <groupId>com.workday</groupId>
    <artifactId>timeseries-forecast</artifactId>
    <version>1.1.1</version>
  </dependency>
  ```
  
## 테스트 결과
## 결론






