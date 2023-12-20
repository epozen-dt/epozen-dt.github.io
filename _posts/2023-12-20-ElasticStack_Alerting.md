---
title: "Elastic Stack의 Alerting 기능"
last_modified_at: 2023-12-20
author: Do-soo, KIM
---


> 이번 포스팅에서는 Elastic Stack이 제공하는 Alerting 기능에 대해서 정리해 보려고 합니다.

### 개요
---

Elastic Stack은 Elastic에서 제공하는 검색, 분석, 시각화를 위한 오픈 소스 도구 모음입니다.
핵심적인 요소로는 Elasticsearch, Logstash, Kibana, Beats 정도로 이야기해 볼 수 있습니다.

Alerting은 시스템이나 응용 프로그램에서 특정 이벤트가 발생했을 때 사용자에게 경고를 보내는 기능을 말합니다. 
Elastic Stack 내에서도 이러한 Alerting 기능을 사용할 수 있습니다.
Elastic Stack에서의 Alerting은 주로 Elasticsearch와 Kibana의 기능을 활용하여 이루어집니다.



### 특징
---

Elastic Stack Alerting은 Elasticsearch 데이터에 대해 Kibana를 통해 “Rule”이라는 객체로 실시간 경고 및 알림 설정을 정의하여 기능을 수행합니다.

#### Rule 구성 요소


- Condition<br>
&emsp;Alert을 생성할 때 사용되는 데이터 패턴이나 조건 정의

- Action<br>
&emsp;Alerting이 감지한 이벤트에 대한 대응 조치로 어떤 작업을 수행할지 정의

- Schedule<br>
&emsp;각 Rule이 얼마나 자주 수행될 지에 대한 일정 정의

### Alerting 적용 방법

본 포스팅에서는 Elastic Cloud에서 사용하는 방법을 소개하고 있습니다. Elastic Cloud는 시험판 버전으로 14일 동안 무료 사용이 가능합니다.

#### Rule 생성

Elastic Cloud --> Alerting and Insights --> Rules 메뉴 선택 --> Create rule 버튼 클릭

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/6e51c738-f980-4eac-aaa7-5174a53fffc2" width="100%" height="100%" />
</p>

검사명 지정 및 검사종류 템플릿 선택(템플릿에 따라 검사 대상 자동 설정됨)

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/11934fae-981a-4aea-b601-fffe3f435378" width="50%" height="50%" />
</p>

Condition(임계점) 설정

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/f39cbbe7-0efc-472c-9a4d-30821387664a" width="50%" height="50%" />
</p>

Schedule(검사 주기) 설정

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/f6cc3b19-9691-40a8-b4be-fac569865c27" width="50%" height="50%" />
</p>

Action(알림전송방법) 설정<br>
&emsp;필자는 Email을 선택해 보았습니다. Alerting 결과를 Email로 받겠다는 뜻이죠.

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/e935e7cc-5482-4e90-bbec-b33a266d5398" width="50%" height="50%" />
</p>

Email 수신 정보 설정

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/946e487c-9a67-494d-acbd-323b06e0756e" width="50%" height="50%" />
</p>

여기까지 Rule 생성 과정을 마쳤습니다.

#### Alerting 수신 확인

그럼, Rule에서 정의한 메일 주소로 Alering 결과를 수신했는지 확인해 볼까요?

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/0c901a4a-777c-41ca-a075-1c41b2c521df" width="80%" height="80%" />
</p>

짜잔! 이렇게 메일이 수신되었네요. 그럼 성공입니다.!!!

### Reference
---
- https://www.elastic.co/guide/en/kibana/8.11/alerting-getting-started.html



