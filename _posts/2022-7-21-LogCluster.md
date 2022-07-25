---
title: "로그 자동 파싱 - LogCluster"
last_modified_at: 2022-7-21
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **로그 자동 파싱 기술 중 오픈소스 알고리즘인 LogCluster**에 관한 내용을 공유해보려고 합니다.

---

> **배경**

로그는 소프트웨어의 이벤트를 기록함으로써, 소프트웨어의 동작 상태를 파악하고 문제가 발생했을 때 이 동작 파악을 통해서 문제를 찾고 해결하기 위해서 디자인되었으며,
시스템의 개발과 유지보수에 중요한 역할을 합니다. 최근에는 소프트웨어 스택이 OS, 미들웨어, 사용자 애플리케이션 등 다중*대형화되면서 여러 대의 서버에 로그를 기록하거나 MSA로 인해 서버 컴포넌트가 분산됨에 따라 로그 수집 포인트 또한, 다양해짐에 따라 단순 기록에서 벗어나 사용자 액티비티 수집하는 등 로그가 가지는 가치가 더 증가하게 되었습니다.

> **Log Parsing**

로그를 분석하기 위해서는 먼저 로그 메시지에서 원하는 정보를 추출할 수 있어야 합니다. 
하지만, 대량의 로그 데이터에 대한 처리와 구조화되지 않은 형태의 로그는 로그 분석에 많은 에러를 가져다 줍니다.
이러한 문제를 해결하기 위해서 Log Parsing 을 수행합니다.

Log Parsing을 통해서 아래와 같은 Structured Log 형태로 변환하여 로그에서 원하는 정보를 추출하게 됩니다.

![그림1](https://user-images.githubusercontent.com/92897860/179671327-4d3d8c08-113e-42c2-ab4d-34f2586c6fec.png)


> **Log Parsing 종류**

로그 파싱으로는 3가지로 나뉠수 있습니다. 본 포스팅에서는 이 중 자동 로그 파싱 알고리즘(Automated Log Parser) 중 LogCluster에 대해 알아볼 생각입니다. 

- **Custom Log Parser** (ex. Regex, Grok)
- **Automated Log Parser** (ex. Drain, LogCluster)
- **Deep Learning Log Parser** (ex. NuLog)


> **Automated Log Parser**

말 그대로, 정규식을 일일이 지정하는게 아니라 구조화되어있지 않은 로그 데이터를 주면 알아서 자동으로 로그 템플릿, 파라미터 등을 추출해 주는 알고리즘을 말합니다.


# **LogCluster**

텍스트 기반의 이벤트 로그에서 자주 발생하는 라인 패턴과 특이치(outlier)를 찾는 자동 로그 파싱 알고리즘입니다. 로그 클러스터는 로그 클러스터링 문제를 패턴 마이닝 문제로 봅니다. 따라서, 빈도 기반 로그 파일의 로그에서 특정 동작을 특징짓는 패턴 즉, 시스템의 정상적인 동작 로그의 패턴을 찾아 클러스터를 생성하는 알고리즘이라고 볼 수 있습니다.


> **1. 사전 정보**

- 15년도, SLCT(Simple Logfile Clustering Tool) 알고리즘의 개선 버전으로 LogCluster가 등장했습니다.
- Perl 기반의 Tool로 오픈소스를 제공합니다. 그 외의 언어로는 제공하지 않는 것으로 확인됩니다. (ex. java, python X)  
- 마지막 Release 버전은 19년도 3월 20일입니다.   

![그림3](https://user-images.githubusercontent.com/92897860/179672521-56dc2f3f-4f7a-4fe7-84fd-ac49907b86d3.png)

LogHub 라는 여러 유형의 로그 데이터들을 모은 저장소 데이터로 성능을 비교한 결과입니다. 최적의 옵션을 사용한 결과는 아니지만, 다른 알고리즘에 비해서 낮은 값을 보였습니다.

![그림4](https://user-images.githubusercontent.com/92897860/179672527-0bfe6224-2a20-49a4-ab85-cea826689a08.png)


### **2. LogCluster 알고리즘**

> **가정**

알고리즘은 각 이벤트가 이벤트 로그의 한 줄로 설명되고 각 라인 패턴은 유사한 이벤트 그룹을 나타낸다고 가정합니다.

> **목표**

빠르고 데이터에 몇 가지 경로만 통과하고 원래 데이터 공간의 하위공간에 존재하는 클러스터를 탐지하는 알고리즘 설계를 목표로 합니다.


> **프로세스 과정**

해당 알고리즘은 3단계로 구성되어 진행됩니다.  

1. 첫 단계는 데이터를 스캔하면서 자주 발생(사용)하는 단어의 집합(데이터 요약 정보)을 만드는 과정입니다. 로그 데이터에서 자주 식별되는 단어들의 집합을 의미합니다.

2. 두번째 단계는 수집된 단어의 집합(데이터 요약 정보)을 사용하여 클러스터 후보를 만드는 과정입니다. 단어 집합 식별 이후, 알고리즘은 한 번의 데이터셋 스캔을 통해 모든 클러스터 후보를 구축합니다. 라인별로 식별되어 라인에서 하나 이상의 빈도가 높은 단어가 확인되면 클러스터 후보로 선정이 됩니다. 후보들은 후보 테이블에 보관되며, 클러스터 후보가 없는 경우 지원값이 1인 테이블에 삽입되며 그렇지 않은 경우 지원 값이 증가하여 다시 식별하는 과정 등을 거칩니다.

3. 세번째 단계는 클러스터 후보 집합에서 클러스터를 선정하는 과정입니다. 후보 테이블 을 검사하고, 지원 값이 지원 임계값(s)보다 크거나 같은 모든 경우를 알고리즘에 의해 클러스터로 선정이 됩니다. 각 클러스터는 특정 라인 패턴에 해당합니다.
  
첫 단계는 Apriori 알고리즘과 접근 방식이 매우 유사합니다. 다만, 차이로는 LogCluster 알고리즘은 모든 클러스터 후보를 한번에 생성합니다. 즉, 자주 발생하는 빈도 높은 단어의 세트의 하위 세트를 제외한 최대 빈도 단어 세트만 고려하기 때문에 데이터를 2번 만 스캔하게 되기에 속도가 빠르다고 할 수 있습니다.  

지원 임계값을 적게 잡고 패턴 마이닝을 수행하면 SLCT와 같이 LogCluster가 과적합되는 경향이 있습니다. 이러한 문제를 해결하기 위해 LogCluster는 보다 일반적인 클러스터 후보의 지원을 늘리고 클러스터를 결합하기 위해 두 가지 선택적 휴리스틱을 사용하였습니다. 자세한 내용은 논문을 참고하시기 바랍니다.


### **3. 예시**

이벤트 로그에 대해서 LogCluster 3단계를 수행하게 되면 아래 그림과 같은 과정을 거쳐 클러스터는 아래 그림과 같은 로그 템플릿 형태로 도출하게 됩니다.   

DMZ-link 는 하나의 단어이므로 *(1,1)로 표현하며, HQ Link는 두 단어의 되어있기 때문에 *(1,2)로 표현이 됩니다. 이는 SLCT와 LogCluster를 차이이기도 합니다.

![그림2](https://user-images.githubusercontent.com/92897860/179671794-6ae75abe-8d59-4fe3-b5fd-1b89fcbdcbf7.png)


### **4. 테스트 결과**

> **테스트 환경**

Perl 기반의 오픈소스 Tool 사용했습니다.  
데이터로는 Loghub의 Zookeeper 로그 데이터를 사용했습니다.

> **로그 템플릿 추출 결과 확인**

입력 데이터에서 클러스터링을 감지한 후, 클러스터에 해당하는 라인 패턴을 출력하여 결과를 확인해보았습니다. 옵션으로는 여러 가지가 있지만, 특이치를 텍스트 파일로 출력하는 옵션과 기본 필수 옵션인 지원 임계값(s)를 설정하였습니다. 지원 임계값이란, 각 클러스터의 최소 라인 수를 의미힙나다. 앞서 알고리즘 프로세스 과정에서 설명한바와 같이 후보클러스터 중 해당 지원 임계값보다 크거나 같은 로그 라인만 클러스터로 구성되게 됩니다. 

- logcluster.pl --input="../logs/Zookeeper/Zookeeper_2k.log" --support=100 --debug --outlier="zoo_outlier.txt" --lfilter="(/|)(\d+\.){3}\d+(:\d+)?"

입력데이터, 최소 클러스터 행 100줄, 특이치 출력파일, debug 과정을 LogCluster로 실행했습니다. 아래 그림과 같이, 빈번한 단어 집합 구성과 후보 클러스터 및 클러스터 구축이 되는 것을 볼 수 있습니다.

![그림5](https://user-images.githubusercontent.com/92897860/180113808-914ed163-97ff-4af1-96e6-cbf3b4777ce8.png)

> **특이치 결과 확인**

특이치를 출력했을 때, 다음과 같이 출력됩니다. 특이치에 대한 상세 설명은 없고 로그 행만 출력합니다.  
속도로는 2000건의 Zookeeper 로그 기준 1건 당 0.0002초 걸렸습니다.  

![그림6](https://user-images.githubusercontent.com/92897860/180113819-16424c3e-9af5-4733-80e9-c01c94b21ac4.png)


### **5. 정리**

로그 자동 파싱 알고리즘 중 하나인 LogCluster에 대해 알아보았습니다.  
다만, 자료 찾기가 쉽지 않고 offline 모드이며 perl 기반 언어로만 되어있고, 지원임계값을 낮게 설정하면 클러스터 갯수(템플릿)가 많아지는 문제가 있었습니다.

실제 많이 사용하는 SQL 데이터(Binding 변수를 포함하는), System Log, 서버 간의 통신 내역 등 로그에서 실질적으로 자주 사용되는 데이터에 대한 테스트 결과를 확인해야 정확히 나올 것 같습니다.



> **참고 논문 및 자료**  

* [LogCluster - A Data Clustering and Pattern Mining Algorithm for Event Logs](https://dl.ifip.org/db/conf/cnsm/cnsm2015/1570161213.pdf) 
* [A Data Clustering Algorithm for Mining Patterns From Event Logs](https://ristov.github.io/publications/slct-ipom03-web.pdf)   
* [Loghub: A Large Collection of System Log Datasets towards Automated Log Analytics](https://arxiv.org/abs/2008.06448)   
* [LogCluster Github](https://github.com/ristov/logcluster)   

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
