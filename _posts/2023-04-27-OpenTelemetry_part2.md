---
title: "OpenTelemetry Agent 사용하기"
last_modified_at: 2023-04-27
author: Do-soo, KIM
---


이번 포스팅에서는 지난 포스팅에 이어, OTel 사용법에 대해서 소개하고자 합니다.

[지난 포스팅 보기(OpenTelemetry 소개)](https://epozen-dt.github.io/OpenTelemetry/)

지난 포스팅에서 OTel은 Agent와 Collector로 구성된다고 하였는데, 이번 포스팅에서는 단순하게 백엔드 처리 없어 Agent 로그 파일만 추출하는 위한 방법을 소개하기 위해 Collector 사용법은 생략하고 Agent를 사용하는 방법만 소개하겠습니다.<br>

이 포스팅에서 소개하는 것은 Windows 환경에서 사용하는 방법이지만, 리눅스 같은 OS에도 동일하게 적용됩니다.


### 1. Agent 다운로드

먼저, 아래 링크를 통해 Agent를 다운로드 합니다.<br>
[https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases](https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases)

다운로드 할 버전의 Assets 항목에서 Opentelemetry-javaagent.jar 파일을 다운로드 합니다.<br>
![image](https://user-images.githubusercontent.com/92565548/234736934-95240132-3900-4674-88f3-a49020745561.png)



### 2.  Application 실행

이제, OTel Agent로 모니터링 할 Application을 아래와 같이 실행합니다.

```
java -javaagent:<opentelemetry-javaagent.jar 경로> -Dotel.traces.exporter=logging -Dotel.metrics.exporter=logging -jar <appication 경로>
```

-Dotel.traces.exporter=logging은 Tracing 로그 출력하겠다는 것이고, -Dotel.metrics.exporter=logging은 Metrics 로그를 출력하겠다는 옵션입니다.

그리고 javaagent라는 옵션이 보이시죠?<br>
javaagent는 JVM의 다양한 이벤트를 전달받거나 정보 질의, 바이트코드 제어 등을 특정 API를 통하여 수행할 수 있는 Java Application입니다.<br>
Java Application 런타임 시 바이트 조작이 가능하도록 하여, JVM의 실행 지점인 main method를 가로챌 수 있습니다. 

Application을 실행하게 되면, 아래와 같이 Agent 로그가 콘솔에 출력됩니다.

ex> tracing log<br>
```
[otel.javaagent 2023-03-02 10:04:25:407 +0900] [main] INFO io.opentelemetry.exporter.logging.LoggingSpanExporter - 'UPDATE alert.test_user_info' : 328fb12164015920a2cf6c49ee2e4069 ef31dfa5ccb1e085 CLIENT [tracer: io.opentelemetry.jdbc:1.23.0-alpha] AttributesMap{data={db.user=alert, thread.id=1, db.connection_string=postgresql://192.168.123.100:5432, net.peer.name=192.168.123.100, db.system=postgresql, db.statement=update test_user_info set name=? where id=?, db.sql.table=test_user_info, db.operation=UPDATE, net.peer.port=5432, db.name=alert, thread.name=main}, capacity=128, totalAddedValues=11}
[otel.javaagent 2023-03-02 10:04:25:853 +0900] [main] INFO io.opentelemetry.exporter.logging.LoggingSpanExporter - 'SELECT alert.test_user_info' : 11a51b1073ddaac284ec67303e7e2210 c116c29364ba9fc6 CLIENT [tracer: io.opentelemetry.jdbc:1.23.0-alpha] AttributesMap{data={db.user=alert, thread.id=1, db.connection_string=postgresql://192.168.123.100:5432, net.peer.name=192.168.123.100, db.system=postgresql, db.statement=select id, name from test_user_info where id = ?, db.sql.table=test_user_info, db.operation=SELECT, net.peer.port=5432, db.name=alert, thread.name=main}, capacity=128, totalAddedValues=11}
```
ex> metrics log<br>
```
[otel.javaagent 2023-03-02 10:05:20:816 +0900] [PeriodicMetricReader-1] INFO io.opentelemetry.exporter.logging.LoggingMetricExporter - Received a collection of 15 metrics for export.
[otel.javaagent 2023-03-02 10:05:20:817 +0900] [PeriodicMetricReader-1] INFO io.opentelemetry.exporter.logging.LoggingMetricExporter - metric: ImmutableMetricData{resource=Resource{schemaUrl=https://opentelemetry.io/schemas/1.18.0, attributes={host.arch="amd64", host.name="dskim-pc", os.description="Windows 10 10.0", os.type="windows", process.command_line="D:\java-1.8.0-openjdk-1.8.0.232-2\jre\bin\java.exe -javaagent:D:\project\OpenTelemetry\opentelemetry-javaagent.jar -Dotel.traces.exporter=logging -Dotel.metrics.exporter=logging", process.executable.path="D:\java-1.8.0-openjdk-1.8.0.232-2\jre\bin\java.exe", process.pid=41060, process.runtime.description=" OpenJDK 64-Bit Server VM 25.232-b09", process.runtime.name="OpenJDK Runtime Environment", process.runtime.version="1.8.0_232-b09", service.name="agenttester-0.1", telemetry.auto.version="1.23.0", telemetry.sdk.language="java", telemetry.sdk.name="opentelemetry", telemetry.sdk.version="1.23.1"}}, instrumentationScopeInfo=InstrumentationScopeInfo{name=io.opentelemetry.runtime-metrics, version=1.23.0-alpha, schemaUrl=null, attributes={}}, name=process.runtime.jvm.system.cpu.utilization, description=Recent cpu utilization for the whole system, unit=1, type=DOUBLE_GAUGE, data=ImmutableGaugeData{points=[ImmutableDoublePointData{startEpochNanos=1677719059965000000, epochNanos=1677719119983000000, attributes={}, value=0.0, exemplars=[]}]}}
[otel.javaagent 2023-03-02 10:05:20:819 +0900] [PeriodicMetricReader-1] INFO io.opentelemetry.exporter.logging.LoggingMetricExporter - metric: ImmutableMetricData{resource=Resource{schemaUrl=https://opentelemetry.io/schemas/1.18.0, attributes={host.arch="amd64", host.name="dskim-pc", os.description="Windows 10 10.0", os.type="windows", process.command_line="D:\java-1.8.0-openjdk-1.8.0.232-2\jre\bin\java.exe -javaagent:D:\project\OpenTelemetry\opentelemetry-javaagent.jar -Dotel.traces.exporter=logging -Dotel.metrics.exporter=logging", process.executable.path="D:\java-1.8.0-openjdk-1.8.0.232-2\jre\bin\java.exe", process.pid=41060, process.runtime.description=" OpenJDK 64-Bit Server VM 25.232-b09", process.runtime.name="OpenJDK Runtime Environment", process.runtime.version="1.8.0_232-b09", service.name="agenttester-0.1", telemetry.auto.version="1.23.0", telemetry.sdk.language="java", telemetry.sdk.name="opentelemetry", telemetry.sdk.version="1.23.1"}}, instrumentationScopeInfo=InstrumentationScopeInfo{name=io.opentelemetry.runtime-metrics, version=1.23.0-alpha, schemaUrl=null, attributes={}}, name=process.runtime.jvm.memory.committed, description=Measure of memory committed, unit=By, type=LONG_SUM, data=ImmutableSumData{points=[ImmutableLongPointData{startEpochNanos=1677719059965000000, epochNanos=1677719119983000000, attributes={pool="PS Survivor Space", type="heap"}, value=11010048, exemplars=[]}, ImmutableLongPointData{startEpochNanos=1677719059965000000, epochNanos=1677719119983000000, attributes={pool="Code Cache", type="non_heap"}, value=6684672, exemplars=[]}, ImmutableLongPointData{startEpochNanos=1677719059965000000, epochNanos=1677719119983000000, attributes={pool="Metaspace", type="non_heap"}, value=24903680, exemplars=[]}, ImmutableLongPointData{startEpochNanos=1677719059965000000, epochNanos=1677719119983000000, attributes={pool="PS Old Gen", type="heap"}, value=90701824, exemplars=[]}, ImmutableLongPointData{startEpochNanos=1677719059965000000, epochNanos=1677719119983000000, attributes={pool="PS Eden Space", type="heap"}, value=66584576, exemplars=[]}, ImmutableLongPointData{startEpochNanos=1677719059965000000, epochNanos=1677719119983000000, attributes={pool="Compressed Class Space", type="non_heap"}, value=3407872, exemplars=[]}], monotonic=false, aggregationTemporality=CUMULATIVE}}

```

만약, 콘솔로 출력되는 로그를 파일로 생성하고 싶다면, 아래와 같이 io.opentelemetry.javaagent.slf4j.simpleLogger.logFile 옵션을 붙여 Application을 실행하면 됩니다.

```
java -javaagent:<opentelemetry-javaagent.jar 경로> -Dotel.traces.exporter=logging -Dotel.metrics.exporter=logging -Dio.opentelemetry.javaagent.slf4j.simpleLogger.logFile=<로그파일명> -jar <appication 경로>
```


이 로그를 통해 Application을 모니터링 할 수 있습니다.<br>
앞에서 잠깐 언급했던 Collector를 사용하면 Kafka나 OpenSearch등 다양한 백엔드로 agent 로그 데이터를 보낼 수 있습니다.

### Reference

```
https://opentelemetry.io/
```