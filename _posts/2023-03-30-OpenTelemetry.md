---
title: "OpenTelemetry 소개"
last_modified_at: 2023-03-30
author: Do-soo, KIM
---


이번 포스팅에서는 OpenTelemetry에 대해서 소개하고자 합니다.

OpenTelemetry는 Traces, Metrics, Logs와 같은 원격 측정 데이터를 계측, 생성, 수집하여 내보내기 위한 observability 오픈 소스 프레임워크입니다.

OpenTelemerty는 흔히 줄여서 'OTel'이라고 부르기도 하므로, 이후부터는 OTel이라고 표기하겠습니다.


### 1. 개요
분산된 시스템이 확장됨에 따라 개발자 입장에서는 자신의 서비스가 어떤 서비스에 의존적이고 의존되고 있는지 파악하기 어려워졌고, 결론적으로 어떠한 영향을 주고 받는지 보기 어려워졌습니다.<br>
특히, 서비스 배포 또는 중단하는 경우에 그렇습니다. 

이러한 문제를 해결하기 위해 어떻게 해야할까요?

일단, 시스템에 대한 정보가 부족하므로 시스템을 계측(instrument)해서 관찰가능(observable)하게 만들어야 합니다.

즉, 애플리케이션 코드에서 trace, metric, log 등과 같은 telemetry 데이터를 계측해야 하고, 이 계측된 데이터들을 Observability back-end로 전송해야 합니다.
자체 호스팅 오픈 소스 도구(예: Jaeger 및 Zipkin)에서 상용 SaaS 제품에 이르기까지 수많은 Observability 백엔드가 있습니다.

과거에는 Observability 백엔드마다 데이터를 도구로 내보내는 자체 계측 라이브러리와 에이전트가 있기 때문에 코드가 계측되는 방식이 다양했습니다.

이는 Observability 백엔드로 데이터를 전송하기 위한 표준화된 데이터 형식이 없음을 의미했습니다. 또한 회사가 Observability 백엔드를 전환하기로 선택한 경우 선택한 새 도구에 원격 측정 데이터를 내보낼 수 있도록 코드를 다시 계측하고 새 에이전트를 구성해야 합니다.

표준화의 필요성을 인식한 클라우드 커뮤니티가 함께 모여 OpenTracing(Cloud Native Computing Foundation(CNCF) 프로젝트)과 OpenCensus(Google 오픈 소스 커뮤니티 프로젝트)라는 두 개의 오픈 소스 프로젝트가 탄생했습니다.

OpenTracing은 Observability 백엔드로 원격 측정 데이터를 전송하기 위해 벤더 중립적인 API를 제공했지만, 사양을 충족하기 위해 개발자가 자체 라이브러리를 구현하는 데 의존해야 하는 것이 문제였습니다. 

또한 OpenCensus는 개발자가 코드를 계측하고 지원되는 백엔드 중 하나로 보내는 데 사용할 수 있는 언어별 라이브러리 세트를 제공했습니다.

그래서, 단일된 표준을 갖기 위해 2019년 5월에 OpenCensus와 OpenTracing이 병합되어 OTel이 되었습니다. OTel은 CNCF 인큐베이팅 프로젝트로서 OpenTracing과 OpenCens의 장점을 모두 취하게 되었습니다.<br>
이렇게 탄생한 OTel의 목표는 데이터를 수집, 변환 및 Observability 백엔드(예: 오픈 소스 또는 상용 공급업체)로 전송하기 위한 표준화된 공급업체 독립적인 SDK, API 및 도구 세트를 제공하는 것입니다.


### 2.  관련 용어

그럼 OTel에서 사용되는 용어에 대해서 간단히 살펴보고 가겠습니다.

#### Observability

Observability란 내부 동작 상황을 알 수 없는 시스템에 대하여 이해할 수 있도록 돕는 것, 정보를 의미합니다.
즉, 어떠한 문제가 발생하였을 때 Observability를 통해서 왜 이런 일이 발생했는지 유추해볼 수 있습니다.

너무 설명이 추상적인가요? 그럼, Observability 개념을 제외하고 생각해봅시다. 애플리케이션에서 어떠한 문제가 발생하였을 때, 어떻게 원인을 찾을 수 있을까요?

쉽게 생각한다면 시스템 Log를 보는 방법이 있습니다. 다시 말하면 Log도 Observability 범주에 속합니다. 더 넓게 보자면 trace, metrics, log, span 등과 같은 정보들. 즉, 애플리케이션의 상태를 계측할 수 있는 정보들을 의미합니다.

#### Telemetry

시스템의 동작에 대해 표현하는 데이터들을 의미합니다. 이러한 데이터는 Trace, Metric, Log, Span 같은 형태로 제공될 수 있습니다.


#### Reliability

'서비스가 사용자가 기대하는 것들을 제대로 100% 수행하고 있는가?'를 의미합니다.

예를 들어, 사용자가 검은색 바지에 대해 '장바구니에 추가'버튼을 눌렀다고 합시다. 근데 장바구니엔 담기지 않거나 다른 색상의 바지가 담겼다면, 이는 'Reliability하다' 또는 'Reliability 한 시스템'이라고 할 수 없습니다.
 

#### Metric

인프라 또는 애플리케이션에 대한 일정 기간동안의 숫자 데이터의 집계(aggregations)를 의미합니다. 예를 들면, CPU 사용률, Memory 사용률, 시스템 오류율, 초당 시스템 요청율 등이 있습니다.

#### Log

서비스 또는 구성 요소에서 출력하는 timestamp를 포함한 메시지를 의미합니다. Trace와 달리 특정 사용자의 요청이라던가, Transaction과 반드시 연관되어 있지는 않습니다. 왠만한 모든 애플리케이션에서 찾아볼 수 있으며, 개발자/운영자에게 시스템 동작을 이해하는데 도움을 줍니다.

예를 들어, 형태는 다음과 같습니다.

```
[2021-02-23T13:26:23.505892 #22473]  INFO -- : [6459ffe1-ea53-4044-aaa3-bf902868f730] Started GET "/" for ::1 at 2021-02-23 13:26:23 -0800
```

단순히 애플리케이션에서 어떠한 동작을 할 때마다 메시지 남기기를 원하는 경우엔 Log를 사용하면 됩니다.
 
하지만 Log의 경우에 호출된 위치 같은 실행에 대한 Context 정보가 부족하기 때문에 코드 실행 프로세스를 추적하기 유용하지 않습니다. 더 욱이 MSA 같은 분산구조에서는 이를 활용해 실행 추적하는 것은 불가능에 가깝습니다.

Span의 일부로 포함되어 사용하면, 더 유용하게 사용할 수 있습니다. 

#### Span

작업, 작업의 단위를 의미합니다. 요청에 대한 작업을 추적해서 작업이 실행된 시간동안 발생한 일에 대해 정보를 가집니다.

이름, 시간 데이터, 구조화된 로그 메시지, 작업에 대한 메타데이터 등이 포함됩니다. 예를 들어, 형태는 다음과 같습니다.

```
{
  "trace_id": "7bba9f33312b3dbb8b2c2c62bb7abe2d",
  "parent_id": "",
  "span_id": "086e83747d0e381e",
  "name": "/v1/sys/health",
  "start_time": "2021-10-22 16:04:01.209458162 +0000 UTC",
  "end_time": "2021-10-22 16:04:01.209514132 +0000 UTC",
  "status_code": "STATUS_CODE_OK",
  "status_message": "",
  "attributes": {
    "net.transport": "IP.TCP",
    "net.peer.ip": "172.17.0.1",
    "net.peer.port": "51820",
    "net.host.ip": "10.177.2.152",
    "net.host.port": "26040",
    "http.method": "GET",
    "http.target": "/v1/sys/health",
    "http.server_name": "mortar-gateway",
    "http.route": "/v1/sys/health",
    "http.user_agent": "Consul Health Check",
    "http.scheme": "http",
    "http.host": "10.177.2.152:26040",
    "http.flavor": "1.1"
  },
  "events": {
    "name": "",
    "message": "OK",
    "timestamp": "2021-10-22 16:04:01.209512872 +0000 UTC"
  }
}
```

 
#### Distributed Trace (Trace)

Microservice와 같은 다중 서비스(Multi-service) 아키텍처를 통해 요청 작업이 이뤄질 때, 그 실행 경로를 기록 및 추적하는 것을 의미합니다.

분산 시스템에서는 이슈, 성능문제 원인 등을 찾아내기가 어렵기 때문에, Distributed trace는 필수적입니다. 복잡한 분산 시스템에서 애플리케이션/시스템의 가시성을 향상시키고, 디버그에 도움을 주어 시스템을 분석 및 이해하는데 도움을 줍니다.

Trace는 하나 이상의 Span으로 구성됩니다. 첫 번째 Span은 Root span을 의미하며, root span은 요청의 처음부터 끝까지를 의미합니다. 하위 span들은 root span 중 일어나는 경로들로 실행 경로들을 표현해서 심층적인 context를 제공합니다.


### 3.  구성요소

OTel의 구성요소는 아래 그림과 같습니다.

![image](https://user-images.githubusercontent.com/92565548/228731591-5d7f5b50-95e7-40af-a258-9c23cb65fa82.png)<br>
(이미지 출처: https://opentelemetry.io/img/otel_diagram.png)<br>

크게 나눠보면 데이터를 수집하는 에이전트와 에이전트의 데이터를 받아서 가공하여 다른 곳으로 전송하는 컬렉터로 구성됩니다.<br>
OTel의 취지에 맞게 에이전트를 이용하여 데이터를 수집하고 컬렉터로 간단한 처리를 한 뒤, 데이터를 저장하고자 하는 목적지로 전송만 담당합니다.<br>
그리고 그 외의 영역인 데이터의 통계, 시각화와 같은 다양한 특색화 된 구현은 수 많은 모니터링 벤더에게 위임하는 구조입니다.

이상으로, OTel에 대해서 알아보았습니다.

다음 포스팅에서는 OTel을 직접 사용하는 방법을 소개하도록 하겠습니다.

### Reference

```
https://opentelemetry.io/
https://binux.tistory.com/150
https://jennifersoft.com/ko/blog/opentelemetry/
```