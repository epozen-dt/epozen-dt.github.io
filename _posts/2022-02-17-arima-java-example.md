---
title: "시계열 데이터 분석 - ARIMA & S-ARIMA 모형 적용 결과 비교 테스트 (Python vs Java)"
last_modified_at: 2022-02-24
author: 심건우
---

이번 포스팅에서는 시계열 데이터 분석 도구 중 ARIMA 모형과 S-ARIMA 모형을 Java로 적용해보고, Python으로 적용한 결과와 비교하겠습니다.

## 테스트 조건
  
  - 각 테스트에 적용될 모형의 파라미터는 Python 분석 결과에 사용된 모형의 파라미터와 같습니다.
  - 데이터셋은 비행기 탑승객 데이터(train : 132개 / test: 12개), 날씨 데이터(train : 731 / test : 20)를 사용했습니다.
  - 테스트에 사용되는 Java 라이브러리 이름은 Signaflo 입니다.
  - Spring Boot 환경에서 Maven Dependency를 사용했습니다.
  
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

  - Signaflo, 비행기 탑승객 데이터 ARIMA & S-ARIMA 모형 테스트 코드


```java
@SpringBootTest
public class SignafloArimaTest {
    @Test
    public void airPassengersTest() throws IOException, CsvException {
        // 데이터 경로
        String filePath = "D://arima/AirPassengers.csv";
        ArrayList<Double> passengers = new ArrayList<>();
        // 데이터 읽기
        CSVReader reader = new CSVReader(new FileReader(filePath));
        reader.skip(1);
        String[] nextLine = null;
        while ((nextLine = reader.readNext()) != null) {
            passengers.add(Double.parseDouble(nextLine[1]));
        }
        // 예측할 데이터 수
        int forecastSize = 12;
        // train 데이터
        List<Double> train = passengers.subList(0, passengers.size() - forecastSize);
        // test 데이터
        List<Double> test = passengers.subList(passengers.size() - forecastSize, passengers.size());
        System.out.println(train.size());
        System.out.println(test.size());
        // 파라미터 p, d, q
        ArimaOrder arimaOrder = ArimaOrder.order(2,1,2);
        // 파라미터 m
        TimeSeries timeSeries = Ts.newMonthlySeries(DoubleFunctions.arrayFrom(train));
        // 파라미터 p, d, q, P, D, Q
        ArimaOrder seasonalArimaOrder = ArimaOrder.order(1,1,0,0,1,0);
        // ARIMA 모형
        Arima arima = Arima.model(timeSeries, arimaOrder);
        // S-ARIMA 모형
        Arima seasonalArima = Arima.model(timeSeries, seasonalArimaOrder);
        // ARIMA 예측 결과
        Forecast forecast = arima.forecast(forecastSize);
        // S-ARIMA 예측 결과
        Forecast seasonalForecast = seasonalArima.forecast(forecastSize);
        List<Double> pred = forecast.pointEstimates().asList();
        List<Double> pred_s = seasonalForecast.pointEstimates().asList();
        System.out.println("ARIMA Result : " + pred);
        System.out.println("S-ARIMA Result : " + pred_s);
        double[] testData = ArrayUtils.toPrimitive(test.toArray(new Double[test.size()]));
        double[] predData = ArrayUtils.toPrimitive(pred.toArray(new Double[test.size()]));
        double[] predSeasonalData = ArrayUtils.toPrimitive(pred_s.toArray(new Double[test.size()]));
        // 피어슨 상관계수
        double corr = new PearsonsCorrelation().correlation(testData, predData);
        double corr_s = new PearsonsCorrelation().correlation(testData, predSeasonalData);
        // RMSE
        double rmse = RMSE(testData, predData);
        double rmse_s = RMSE(testData, predSeasonalData);
        // MAPE
        double mape = MAPE(testData, predData);
        double mape_s = MAPE(testData, predSeasonalData);
        System.out.println(corr);
        System.out.println(corr_s);
        System.out.println(rmse);
        System.out.println(rmse_s);
        System.out.println(mape);
        System.out.println(mape_s);
    }
    public double RMSE(double[] test, double[] pred) {
        double sum = 0.0;
        for (int i = 0; i < test.length; i++) {
            double diff = test[i] - pred[i];
            sum = sum + diff * diff;
        }
        double mse = sum / test.length;
        return Math.sqrt(mse);
    }

    public double MAPE(double[] test, double[] pred) {
        double sum = 0.0;
        for (int i = 0; i < test.length; i++) {
            double absoluteError = Math.abs(test[i] - pred[i]);
            sum = sum + (absoluteError / test[i]);
        }
        return (sum / test.length) * 100;
    }
}
```

  - Signaflo, 날씨 데이터 ARIMA & S-ARIMA 모형 테스트 코드


```java
@SpringBootTest
public class SignafloArimaTest {
    @Test
    public void weatherTest() throws IOException, CsvValidationException {
        // 데이터 경로
        String filePath = "D://arima/WeatherData.csv";
        ArrayList<Double> weather = new ArrayList<>();
        // 데이터 읽기
        CSVReader reader = new CSVReader(new FileReader(filePath));
        String[] nextLine = null;
        while ((nextLine = reader.readNext()) != null) {
            weather.add(Double.parseDouble(nextLine[0]));
        }
        // 예측할 데이터 수
        int forecastSize = 20;
        // train 데이터
        List<Double> train = weather.subList(0, weather.size() - forecastSize);
        // test 데이터
        List<Double> test = weather.subList(weather.size() - forecastSize, weather.size());
        System.out.println(train.size());
        System.out.println(test.size());
        // 파라미터 p, d, q
        ArimaOrder arimaOrder = ArimaOrder.order(0,1,2);
        // 파라미터 m
        TimeSeries timeSeries = Ts.newMonthlySeries(DoubleFunctions.arrayFrom(train));
        // 파라미터 p, d, q, P, D, Q
        ArimaOrder seasonalArimaOrder = ArimaOrder.order(1,1,2,3,1,0);
        // ARIMA 모형
        Arima arima = Arima.model(timeSeries, arimaOrder);
        // S-ARIMA 모형
        Arima seasonalArima = Arima.model(timeSeries, seasonalArimaOrder);
        // ARIMA 예측 결과
        Forecast forecast = arima.forecast(forecastSize);
        // S-ARIMA 예측 결과
        Forecast seasonalForecast = seasonalArima.forecast(forecastSize);
        List<Double> pred = forecast.pointEstimates().asList();
        List<Double> pred_s = seasonalForecast.pointEstimates().asList();
        System.out.println("ARIMA Result : " + pred);
        System.out.println("S-ARIMA Result : " + pred_s);
        double[] testData = ArrayUtils.toPrimitive(test.toArray(new Double[test.size()]));
        double[] predData = ArrayUtils.toPrimitive(pred.toArray(new Double[test.size()]));
        double[] predSeasonalData = ArrayUtils.toPrimitive(pred_s.toArray(new Double[test.size()]));
        // 피어슨 상관계수
        double corr = new PearsonsCorrelation().correlation(testData, predData);
        double corr_s = new PearsonsCorrelation().correlation(testData, predSeasonalData);
        // RMSE
        double rmse = RMSE(testData, predData);
        double rmse_s = RMSE(testData, predSeasonalData);
        // MAPE
        double mape = MAPE(testData, predData);
        double mape_s = MAPE(testData, predSeasonalData);
        System.out.println(corr);
        System.out.println(corr_s);
        System.out.println(rmse);
        System.out.println(rmse_s);
        System.out.println(mape);
        System.out.println(mape_s);
    }

    public double RMSE(double[] test, double[] pred) {
        double sum = 0.0;
        for (int i = 0; i < test.length; i++) {
            double diff = test[i] - pred[i];
            sum = sum + diff * diff;
        }
        double mse = sum / test.length;
        return Math.sqrt(mse);
    }

    public double MAPE(double[] test, double[] pred) {
        double sum = 0.0;
        for (int i = 0; i < test.length; i++) {
            double absoluteError = Math.abs(test[i] - pred[i]);
            sum = sum + (absoluteError / test[i]);
        }
        return (sum / test.length) * 100;
    }
}
```


## 테스트 결과
  1. 비행기 탑승객 데이터, ARIMA(2,1,2) 모형

![image](https://user-images.githubusercontent.com/87160438/154433091-65aba3ed-b7ee-4664-a907-1fa293818757.png)


  2. 비행기 탑승객 데이터, S-ARIMA(1,1,0,0,1,0,12) 모형

![image](https://user-images.githubusercontent.com/87160438/154433264-9cf03ac6-56c4-41bc-9834-cb6bd1ac4eb1.png)


  3. 날씨 데이터, ARIMA(0,1,2) 모형

![image](https://user-images.githubusercontent.com/87160438/154433402-05935097-4b38-4027-a2aa-204912ce94af.png)


  4. 날씨 데이터, S-ARIMA(1,1,2,3,1,0,12) 모형

![image](https://user-images.githubusercontent.com/87160438/154433541-c0c93b05-74fb-43c5-92cd-e3620be074c8.png)


## 결론

여기까지가 두 환경에서 ARIMA & S-ARIMA 모형 적용한 결과입니다.

Java Signaflo 라이브러리 적용 결과는 Python 적용 결과와 거의 유사했습니다.

각자 상황에 맞게 적용하면 좋을 것 같습니다.

감사합니다.
