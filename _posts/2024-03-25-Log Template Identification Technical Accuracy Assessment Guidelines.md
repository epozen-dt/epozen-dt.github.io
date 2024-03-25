---
title: "로그 템플릿 식별 기술의 정확성 평가지침"
last_modified_at: 2024-03-25
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **향상된 로그 분석을 위한 지침: 로그 메시지 템플릿 식별 기술의 정확성 평가**를 공유해보려고 합니다.


# **1. 개요**

템플릿 식별 기술의 정확도를 평가하는 데 도움이 되는 3가지 지침을 제안

* **(1) 적절한 정확도 측정 기준의 사용**
    * 기존, 서로 다른 측정 기준을 사용하여 평가
    * 자동화된 로그 템플릿 식별 기술은 다양하나, 이들간의 평가와 순위 또한 중요
      * AEL, Drain, IPLoM, LenMa, LFA, LKE, LogCluster, LogMine, LogSig, MoLFI, SHISO, SLCT, Spell 등
    * **TA(Template Accuracy) 메트릭 제안**
      + 포함하여, GA(Grouping Accuracy), PA(Parsing Accuracy) 설명
* **(2) 오라클 템플릿(Ground Truth) 수정**
    * 일부 오라클 템플릿이 잘못되었음
* **(3) 잘못 식별된 템플릿 분석**
    * 일부 템플릿을 잘못 식별하는 이유를 분석하는 것 또한 중요
    * 식별된 템플릿을 3가지 유형으로 분류 (OG, UG, MX)


---

## **1. 적절한 정확도 측정 기준의 사용**

### **GA (Grouping Accuracy) 와 PA (Parsing Accuracy)**

* **GA:** 각 로그 메시지가 올바른 템플릿 그룹에 속하는가
* **PA:** 식별된 템플릿 내의 변수가 정확하게 파싱되었는가

![1](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/d88edb66-0532-4698-9797-b6be82db8619)

* GA 관점에선, 메시지지가 어떤 템플릿으로 식별되었느냐는 중요하지 않음
  * 따라서, "x is 3.5"라는 메시지가 "x is <*>" 또는 "x is <*>.<*>"  어디든 올바르게 그룹화된다면 모두 올바른 것으로 간주
  
### **TA (Template Accuracy)**

아이디어는 템플릿 식별을 로그 메시지의 집합에서 메시지 템플릿을 식별하는 정보 검색과정으로 간주해야 한다는 것. 즉, 로그 데이터 내에서 특정 패턴이나 구조를 가진 템플릿(문서)를 찾아내는 과정이며, 검색결과의 품질을 측정하기 위한 지표로 재현율과 정밀도 등을 사용.

* **GA와 PA는 로그에 포함된 메시지 수에 민감**
  * 반복되는 메시지가 많을 경우, 올바른 판단(평가) 불가

* PTA (Precision TA) 와 RTA (Recall TA)
  * 템플릿 단위의 정확도 평가

![2](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/51b7293a-dc10-4dc9-a8c8-8a8d0d4258f3)

### **적절한 메트릭 선택 기준**

**메트릭의 선택은 시나리오에 따라 달라질 수 있음**

* (1) 로그 메시지의 변수 부분이 사용되는지 여부
  * ex. 이상탐지의 경우, 변수부분은 사용하지 않음 => GA 메트릭 사용
* (2) 메시지의 중요도가 그 빈도에 비례하는 지 여부 
  * ex. "404 Not Found" 와 같이 일반적인 누락 페이지지만 자주 발생하면 심각한 오류를 암시함 => PA 메트릭 사용
  * ex. 자주 발생하진 않으나, 보안 관련 에러 => RTA나 PTA 메트릭 사용 


## **2. 오라클 템플릿 수정**

일부 잘못 식별된 오라클 템플릿(Ground Truth)의 수정  
이러한 규칙들은 로그 템플릿 식별 과정에서 오라클 템플릿의 정확도를 높이기 위한 전처리 단계

### **Heuristic-based rules**

* 오라클 템플릿에 대한 깊은 분석 후, 일부 템플릿들이 로그 메시지에서 변하지 않는 고정된 부분을 하드코딩하는 경향이 있다는 점을 발견
  * LogHub에서 사용된 Hadoop 시스템의 오라클 템플릿 예시 중 "nodeBlacklistingEnabled:true"는 실제 로그 메시지에서 "true" 값을 하드코딩하는 방식이 적절하지 않다는 것을 확인 
  * Hadoop의 소스 코드 검토를 통해, 실제 로그 메시지는 "LOG.info("nodeBlacklistingEnabled:" + nodeBlacklistingEnabled)" 형태로 되어 있으며, 이를 바탕으로 올바른 오라클 템플릿은 "nodeBlacklistingEnabled:<*>"

![3](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/064c0949-3dd3-416d-8060-d8aeea6343d2)

* 규칙에 의거하여, 오라클 템플릿을 수정
  * DG (DiGit), BL (BooLean), PS (Path String)과 같은 고정 토큰을 <*>로 대체
  * US (User-defined String) 처럼 사용자가 특정 문자열을 대체 지정
  * MT (Mixed Token), CV (Consecutive Variables), DV (Dot-separated Variables) 과 같이 변수부분을 포함하는 토큰 (ex. v<*>, <*><*>, <*>.<*>)을 단순히 <*>로 대체  

![5](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/94d7c165-9056-4bfd-8584-811580fd0df6)

* 제안된 오라클 템플릿 수정은 오라클 템플릿이 수동으로 결정될 때 사용되도록 의도되었음(예: 소스 코드에 접근할 수 없을 때

### **오라클 템플릿 수정이 정확도에 미치는 영향**

* 잔차값 (수정된 오라클 템플릿의 정확도 - 기존 오라클 템플릿의 정확도)을 가지고 평가 진행  
* 오라클 템플릿 수정을 고려하지 않았을 때와 고려했을 때의 로그 식별 기술의 순위
  * 오라클 템플릿 수정 전후로 Drain 기술이 모든 메트릭에서 1위를 차지

![6](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/c947cd51-74a9-42e9-af2b-a39180a1d76b)

* 모든 템플릿 식별 기술에 대해, 오라클 템플릿 수정이 제한적인 영향을 미친다는 것이 확인
  * 수정된 템플릿이 전체의 약 28%에 달하는데도, 전반적인 성능 개선은 기대만큼 크지 않았음
* 그러나, 특정 기술의 순위에는 영향을 줄 수 있음을 보여줌
  * 만약, 기술들이 템플릿을 더 정확하게 식별했다면, 수정의 영향이 더 클 수 있음


## **3. 잘못 식별된 템플릿 분석**

단순히 다양한 로그 템플릿 식별 기술을 그들의 정확도 점수에 따라 순위를 매기는 것만으로는 충분하지 않을 수 있음  
따라서, 잘못 식별된 템플릿의 분석을 수행하여, 어떤 방식으로 템플릿이 잘못 식별되었는지를 더 깊이 분석하고 이해하는 것을 제안

### **3가지 유형으로 분류**

* 잘못 식별된 템플릿 = 잘못 일반화된 템플릿
* Over-Generalized(OG), Under-Generalized (UG), MiXed (MX)
  * OG도 UG도 아닌 경우 MX로 분류

![4](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/ad980a0a-5971-4f7a-afe3-5c0871dafafb)

* 템플릿이 로그 메시지를 일반화하는 방식에서 큰 차이를 보임
  * UG: 특정 상황을 충분히 반영하지 못하는 경우
  * OG(과도하게 일반화): 너무 많은 정보를 포괄하여 중요한 구분을 잃어버리는 경우
    * 와일드카드가 쓸데없이 많을수록 OG에 가까움

![7](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/f67850b9-66a0-4ea0-9d4b-53b23af22521)

* 각 기술이 로그 메시지를 식별하면서 어떤 종류의 오류를 범하는지
  * LogCluster는 주로 UG 템플릿(72.9%)을 생성함으로써 템플릿을 과소 일반화하는 경향이 있음
  * LFA는 대부분의 경우(52.7%)에서 OG 템플릿을 생성하여, 주된 문제가 과도한 일반화임을 보여줌

* 이러한 결과는 각 기술의 주요 한계를 드러내고, 이를 통해 알고리즘을 개선할 방법을 제시 (인사이트 제공)
  * 예를 들어, 고정 토큰을 변수로 변환하는 것을 줄이면 LFA의 OG 문제를 개선할 수 있음



## **요약**

* 메시지 템플릿 식별 기술의 정확도를 평가하기 위한 3가지 지침을 제공  
* 실제 로그 데이터셋을 사용하여 14가지 로그 템플릿 식별 기술을 평가  
* 올바른 성능평가를 위해, 메트릭을 계속 정제할 필요성이 존재  




> **참고 논문**  

* [[1] Guidelines for Assessing the Accuracy of Log Message Template Identification Techniques](https://ieeexplore.ieee.org/document/9793976)


---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
