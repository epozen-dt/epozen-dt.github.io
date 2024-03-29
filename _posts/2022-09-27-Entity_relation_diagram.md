---
title: "ERD(Entity Relation Diagram) 표기법"
last_modified_at: 2022-09-27
author: Do-soo, KIM
---


데이터베이스 설계 과정 중, 개념적 설계 단계에서 ERD(Entity Relation Diagram)를 작성하게 됩니다.

그럼 데이터베이스 설계 단계를 잠깐 살펴볼까요?

1단계는 요구사항 분석입니다.
이 단계에서는 데이터베이스를 사용해 업무에 어떤 용도로 사용하는지, 또한 사용자들에게 필요한 데이터의 종류와 처리 방법을 파악하게 됩니다.
이렇게 분석한 결과를 토대로 요구사항명세서를 작성하게 됩니다.

2단계는 개념적 설계 단계입니다.
요구사항 분석 단계에서 파악학 사용자의 요구사항을 개념적 데이터 모델을 이용해서 표현하는데 실제 개발에 사용할 데이터베이스 관리 스스템의 종류에 독립적이면서 데이터 요소와 요소 간의 관계를 표현할 때 사용합니다. <br>
바로 이 단계에서 ERD를 작성하게 되는 것이죠.

3단계는 논리적 설계 단계입니다.
개발에 사용될 데이터베이스 관리 시스템에 적합한 논리적 데이터 모델을 이용해 개념적 설계 단계에서 생성한 스키마를 기반으로 논리적 구조를 설계하는 단계입니다.

4단계는 물리적 설계 단계입니다.
논리적 설계 단계에서 생성된 스키마를 기반으로 물리적 구조, 즉 데이터베이스를 저장 장치에 실제로 저장하기 위한 내부 저장 구조 및 접근 경로 등을 설계하는 단계입니다.
따라서 이 단계에서는 저장 장치에 적합한 저장 레코드와 인덱스의 구조 등을 설계하고 저장된 데이터와 인덱스에 빠르게 접근할 수 있는 탐색 기법 등을 정의합니다.

마지막 단계는 구현 단계입니다.
이전 단계에서 도출된 데이터베이스 스키마를 실제 파일로 생성하는 단계입니다. 앞서 산출된 SQL로 작성된 명령문을 실행하여 데이터베이스를 실제로 생성합니다.

여기까지 데이터베이스 설계 단계를 살펴보았고, 다시 ERD에 대해 얘기해 보겠습니다.

Entity(개체)란 현실에 존재하는 개별적으로 식별할 수 있는 물리적 또는 추상적인 개체를 의미합니다. 

각 Entity 는 특징을 나타낼 수 있는 Attribute (속성) 들을 가지고 있습니다. 예를 들어 ‘학생’ 이라는 Entity 는 ‘학번’, ‘학생 이름’ 등의 Attribute 를 가질 수 있고, ‘수업’ 이라는 Entity 는 ‘학수번호’, ‘수업 이름’ 등의 Attribute 를 가질 수 있다.

ER(개체-관계) 모델은 위에서 설명한 Entity 사이의 Relation(관계)을 통해 현실 세계를 표현하기 위한 설계 방식입니다. 예를 들어 ‘학생’ 과 ‘수업’ Entity 끼리는 ‘수강하다’ 라는 관계를 맺을 수 있습니다. 

이는 현재 가장 인기있는 관계형 데이터베이스의 테이블 구조로 사상(Mapping)하기에도 쉬워 인기가 많은 방식입니다. 

따라서 ER 모델을 도식화 하여 표현하는 방법이 ERD라고 할 수 있습니다.

---

## 1. ERD 표기법 분류

ERD 표기법은 크게 Peter-Chen 표기법과 까마귀 발 표기법으로 나눌 수 있습니다.

### Peter-Chen 표기법

Peter Chen 표기법은 흔히 대학교에서 배우는 ERD 표기법으로 실무에서는 보통 사용되지 않는 방법입니다. 아래 그림과 같이 Entity 는 직사각형, Attribute 는 타원, Relation 은 마름모로 도식화하는 표기법입니다.

![image](https://user-images.githubusercontent.com/92565548/192405625-2cbf7f12-5c12-4c80-b1db-e2c4cd3012d0.png)


### 까마귀 발 표기법

까마귀 발(Crow’s Feet) 표기법은 표기 방법이 까마귀의 발을 닮았다고 하여 붙여진 이름입니다.

실무에서 가장 많이 사용되는 방식으로, ERWin 과 같은 여러 CASE(Computer Aided Software Engineering) 툴에서 가장 많이 지원하는 표기법입니다.

IE 표기법, 바커 표기법 등 다양한 형태로 변형이 존재하며, 이 포스팅에서는 실무에서 가장 많이 사용되는 IE 표기법에 대해 설명하려고 합니다.

## 2. IE 표기법

IE 표기법 (Information Engineering Notation) 은 위에서 얘기했듯이 실무에서 가장 많이 사용되는 표기법입니다. 그럼 좀 더 자세히 설명하겠습니다.

#### Entity

아래 그림과 같이 직사각형에 Entity 의 이름을 상단에 표기합니다.

![image](https://user-images.githubusercontent.com/92565548/192405678-e7ff53c5-62b5-490b-a4d1-e9972cd68513.png)

#### Attribute

Entity 이름 하단에 아래 그림과 같이 좌측에 PK, FK 등의 정보를 표기하고, 우측에는 어트리뷰트의 이름을 표기합니다.

![image](https://user-images.githubusercontent.com/92565548/192405734-a8b45c18-48d8-4ffa-b253-2dfb937d700a.png)

#### Relation

각 Entity 간의 관계는 기본적으로 실선과 점선으로 표현될 수 있습니다.

실선은 식별(Identifying) 관계를 나타냅니다. 식별 관계란 부모 Entity 의 기본키 또는 유니크키를 자식 Entity 의 기본키로 사용하는 관계, 즉 자식 Entity 는 부모 Entity 가 존재해야 존재할 수 있습니다.

점선은 비식별(Non-Identifying) 관계를 나타냅니다. 비식별 관계란 부모 Entity 의 기본키 또는 유니크키를 자식 Entity 에서 외래키로 사용하는 관계, 즉 자식 Entity 는 부모의 존재유무와 관계 없이 독립적으로 존재할 수 있습니다.

학생과 성적의 관계를 예로 들어보자면 성적 데이터는 학생이 존재해야 존재할 수 있으므로 이 관계는 식별 관계이며, ERD에서는 실선으로 표시하면 됩니다.

또한, 학생과 과목의 관계에서는 수강하는 학생이 없더라도 과목은 독립적으로 존재할 수 있기 때문에 이 관계는 비식별관계이며, ERD에서는 점선으로 표시하면 됩니다.

#### Mapping Cardinality

개체와 개체간의 Mapping Cardinality(대응수)란, 특정 Entity가 상대 Entity와 관계를 몇 회 맺을 수 있는지를 나타내는 것입니다.

학생과 학급의 소속 관계를 예로 들어보겠습니다.

한 학급에는 여러 학생이 소속될 수 있지만, 한 학생이 여러 학급에 소속될 수는 없습니다. 이때, 학생과 학급의 Cardinality 는 N:1 입니다.

이번에는 학생과 학생의 짝꿍 관계를 예로 들어보겠습니다.

한 학생이 다른 학생과 짝꿍관계를 맺을 수 있는 관계의 경우의 수는 서로 1회 밖에 없습니다. 따라서 이 관계의 Cardinality 는 1:1 입니다.

마지막으로 학생과 동아리 간의 소속 관계를 예로 들면 하나의 학생은 여러 동아리에 소속될 수 있고 하나의 동아리 또한 여러 학생을 소속시킬 수 있습니다. 따라서 이 관계의 Cardinality 는 N:N 입니다.

IE 표기법에서는 이를 아래와 같이 까마귀 발 모양과 같이 나타낼 수 있습니다. 이 모양을 양쪽 Entity 쪽에 각각 표기해주면 됩니다.

![image](https://user-images.githubusercontent.com/92565548/192405758-9005c227-3281-4bde-a2bf-85616781424c.png)


지금까지 설명한 내용을 종합하여 ERD를 작성해 본다면 아래와 같이 작성할 수 있습니다.

![image](https://user-images.githubusercontent.com/92565548/192405806-072a6e8f-02ce-4545-ae38-f23890a19053.png)


이번 포스팅은 여기서 마치겠습니다.

### References

```
https://hudi.blog/entity-relation-diagram/(이미지 출처 포함)
https://iwuooh.com/entry/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%EC%9D%98-%EC%84%A4%EA%B3%84DataBase-Design%EC%9D%98-%EA%B0%9C%EC%9A%94%EC%99%80-%EB%8B%A8%EA%B3%84
```