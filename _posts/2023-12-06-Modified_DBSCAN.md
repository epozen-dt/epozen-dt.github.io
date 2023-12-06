---
title: "시계열 이상탐지 - Modified DBSCAN"
last_modified_at: 2023-12-06
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **Modified DBSCAN**을 공유해보려고 합니다.

## **Modified DBSCAN**

### **개요 [1]**

* 감지하고 싶지 않은 패턴인 Spike, Avalanche, 혹은 Fluctuate와 같은 그래프 패턴이 있을 때, 이를 오탐(False Positive)으로 받고 싶지 않을 때
* 오히려 정상의 범위를 확실하게 모를때에도 유용

![1](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/fa7b6eb3-4185-4290-9517-15cc68eb9589)


### **Density-Based Clustering [2]**

* 점 p(중심점)에서부터 거리 Eps(epsilon) 내에 점이 m(minPts)개 있으면 하나의 군집으로 인식

![2](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/bdd76d88-e38e-436c-b9b9-9b48a25b7fb9)

* **Type1.** 군집, P가 중심점
* **Type2.** P2는 중심점이 되진못하나, P를 중심점으로 하는 군집에 속하기에 border point(경계점)이 됨
* **Type3.** P3 내에 P가 존재, 이 경우 서로 연결되어있다고 가정하고, 하나의 군집이 됨 (밀도에 따른 클러스터 연결)
* **Type4.** 어느 군집에도 속하지 않음, outlier로, 이를 noise point라고 함


### **DBSCAN [3]**

> **아이디어**
* 클러스터는 밀도가 높은 data point들의 집합
* Noise point 주변의 밀도는 매우 낮음
* 클러스터링 알고리즘은 공간(spatial) 데이터베이스에서 class를 식별하는 작업에 적합

> **목적**
* 유효한 클러스터 집합을 찾아내기 위해, 클러스터와 noise point 의 특징을 정량화
* 주어진 데이터를 Core, Border, Noise로 분류(식별)하겠다는 의미

![3](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/019729d0-e8dd-40db-83dc-9fb69a630e03)


### **Definitions**

* data point 를 클러스터에 할당, 클러스터 간 관계 정의
* 임의의 점으로부터 Density-connectivity 될 수 있는 점들을 계속 확장해나가며 클러스터를 구성

![4](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/e569e306-3d3c-4315-82cf-dd5afbcea353)

* Directly density reachable
  + q가 p의 epsilon 반영이내에 존재(reachability, 서로 연결됨)
  + q는 p로부터 d.d.r하며, 그 반대는 안됨(epsilon neighborhood안에 있으나 minPts를 만족하지못해서)
* Density reachable
  + q는 p로부터 d.r하다는 것은 d.d.r한 r과 같은 포인트의 체인으로 연결됨을 의미
* Density connectivity
  + P와 q는 어떤 o라는 점을 공유하며, 그 점으로부터 각 두 점은 d.r한 성질을 갖음
  + 즉, 임의의 점으로부터 Density connectivity 될수 있는 점들을 계속적으로 확장해나가며 클러스터를 구성


### **DBSCAN Clustering**

1. 임의의 점 p 선정
2. p로부터 density-reachable한 모든 q를 찾음
3. p가 core point라면, cluster 형성
4. p가 border point라면, 새로운 점으로 시작
5. (1-4과정 반복)

![8](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/9f8e5771-bdea-42d9-bad7-b14fc3ed9263)

  * 어떠한 점부터 시작해도 군집은 동일

### **Background**

* 목표: 개별 월에 대한 local anomalies 식별
  + global anomalies + local anomalies

![5](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/dcfa9dd6-1c06-4e19-8c31-79b92dc6cc77)


### **Modified DBSCAN [4]**

* DBSCAN 알고리즘 자체는 실질적으로 변경 X 
* 입력 데이터 구조를 수정
* **주어진 1차원 데이터셋에서, 각 데이터 포인트에 대해 월별 좌표를 추가로 할당**

![6](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/f97dbc24-b718-4514-972f-1df678f314dc)


### **Modified DBSCAN 입력 데이터**

* ex. 월별 주기 12 (t, t mod 12)
* 3차원 도메인 (시간, 월에 대한 주기적 변환, 월별 온도 값)으로 투영

![7](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/3591f1a3-f71e-4e0a-afc6-dfc8080e4812)


> **참고 논문**  

* [[1] 그림 출처, Outlier detection in Datadog: A look at the algorithms](https://www.datadoghq.com/blog/outlier-detection-algorithms-at-datadog/)
* [[2] 그림 출처, DBSCAN (밀도 기반 클러스터링)](https://bcho.tistory.com/1205)
* [[3] A Density-Based Algorithm for Discovering Clusters in Large Spatial Databases with Noise](https://www.dbs.ifi.lmu.de/Publikationen/Papers/KDD-96.final.frame.pdf)
* [[4] A Modified DBSCAN Algorithm for Anomaly Detection in Time-series Data with Seasonality](https://www.iajit.org/portal/images/Year2022/No.1/19023.pdf)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
