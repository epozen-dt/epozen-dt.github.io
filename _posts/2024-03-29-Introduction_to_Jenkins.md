---
title: "Jenkins 소개"
last_modified_at: 2023-03-29
author: Do-soo, KIM
---


> 이번 포스팅에서는 Jenkins에 대해서 소개해 볼까 합니다.

### CI(Continuous Integration)/CD(Continuous Deployment/Delivery)
---

Jenkins에 대해서 설명하기 전에 먼저 CI/CD에 대한 설명부터 해야겠군요.

CI/CD는 애플리케이션 개발 단계부터 배포까지 모든 단계를 자동화하여 좀 더 효율적이고 빠르게 사용자에게 배포하는 것입니다.

#### - CI
지속적인 통합, 빌드/테스트 자동화 과정이라고 할 수 있습니다.<br>
즉, 애플리케이션의 버그 수정이나 코드 변경 발생 시마다 주기적으로 빌드 및 테스트 진행하고 Repository에 통합(merge) 하는 과정입니다.

#### - CD
CI에서 Build 및 Test, Release 준비 이후, 프로덕션 환경에 수동 배포하는 과정을 Continuous Delivery, 자동 배포하는 과정을 Continuous Deployment라고 합니다.

### Jenkins
---

#### - 기본 개념

Jenkins이란 위에서 잠깐 설명한 CI/CD를 자동화 해 주는 툴입니다.<br>

JetBrains에서 발표한 2023년 CI Tool 사용 순위에 따르면 Jenkins이 2위 입니다.
그만큼 많이 사용한다고 볼 수 있겠죠?

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/5efe6469-94ba-4e2f-ac1d-a60aa31bc342" width="90%" height="90%" /><br>
    출처: JetBrains
</p>

Jenkins에서 CI/CD 동작 과정은 다음과 같습니다.

1. 개발자가 Local에서 Repository(Git 등)에 해당 코드 반영

2. Jenkins에서 WebHook 등과 같은 방식을 활용하여 소스코드의 변경을 감지

3. Jenkins에 작성된 내용을 기준으로 빌드 및 테스트 등을 통한 배포

<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/a09663e4-3ec0-4530-8d67-1f3d0cb4e1a3" width="90%" height="90%" /><br>
    출처: https://security-gom.tistory.com/60
</p>


Jenkins은 Java Runtime 환경에서 동작하고 다양한 플러그인을 활용해서 각종 자동화 작업을 처리합니다.<br>
그 중 대표적인 플러그인은 다음과 같습니다.

- Credentials Plugin: AWS token, Git access token, secret key, ssh(username, password)등의 정보 저장 및 활용
- Git Plugin: git으로부터 코드를 가져와 빌드
- Pipeline: ipeline으로 CI/CD 구성
- Docker plugin and Docker Pipeline: Docker 및 Docker agent 사용

Jenkins의 가장 큰 특징은 바로 Pipeline이라고 하는 것을 통해 CI/CD를 구축한다는 것입니다.
CI/CD Pipeline은 일련의 플러그인들의 집합해서 구성하는데, 여러 플러그인들이 파이프라인을 통해 흘러가는 과정이 CI/CD 구조 인 셈이죠.


#### - 특징

Jekins의 대표적인 특징은 다음과 같습니다.

- 프로젝트 표준 컴파일 환경에서 컴파일 오류 검출

- 자동화 테스트 수행

- 정적 코드 분석(SonarQube 등)에 의한 코딩 규약 순수 여부 체크

- 프로파일링 툴을 이용한 소스변경에 따른 성능 변화 감지

Jenkins은 Jenkins에서 실행하는 일련의 CI/CD 프로세스 단위인 Job으로 CI/CD를 구성합니다.<br>
즉 일련의 CI/CD 과정은 1개의 Job이 되는 것이죠.

Jenkins에서 Job을 구성하는 방법은 Freestyle과 Pipeline, 이렇게 2가지 입니다.

#### Freestyle

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;웹 기반의 GUI를 통해 구성하기 때문에, 플러그인 사용이 쉽고, Learning Curve가 낮아서 초보자 접근이 용이합니다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;그리고 Pipeline 대비 reference가 많다는 게 장점입니다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;하지만 각각의 과정별 모니터링이 불가능하고, 커스터마이징 역시 어려우며, 병렬 처리가 불가능하다는 것이 단점입니다.

#### Pipeline

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;하나의 스크립트 파일(Jenkinsfile)만으로 CI/CD pipeline 설정 및 흐름 제어가 가능하고,<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;커스터마이징이 쉬우며, 병렬 처리가 가능하다는 것이 장점이지만<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Groovy라는 언어 기반의 스크립트 문법에 대해 학습이 필요하고 Freestyle 대비 reference가 적다는 것이 단점입니다.

이상으로 Jenkins에 대해서 간략하게 소개해 드렸습니다.

다음 포스팅에서는 Jenkins을 직접 설치하는 방법에 대해 소개해 볼까 합니다.


> Reference

- https://velog.io/@bbkyoo/Jenkins
- https://seongwon.dev/DevOps/20220717-CICD%EA%B5%AC%EC%B6%95%EA%B8%B02/