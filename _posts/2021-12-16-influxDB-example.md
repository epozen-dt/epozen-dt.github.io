---

title: "InfluxDB 예제 - Java Client를 활용한 간단한 Write & Read"
date: 2021-12-16

---

이번 포스팅에서는 InfluxDB에서 지원하는 Java Client를 활용하여 데이터를 삽입하고 조회하는 방법을 간단한 예제를 통해 알아보겠습니다.


## 시계열 데이터

![image](https://user-images.githubusercontent.com/87160438/142582177-268ea17f-b25c-418b-b9db-dc7bc450e9cd.png)

      ```python
      def adf_test(df):
        result = adfuller(df.values)
        p_value = result[1]
        return p_value
      ```
