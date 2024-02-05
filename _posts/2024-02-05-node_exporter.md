---
title: "Prometheus Node Exporter 설치 방법"
last_modified_at: 2024-02-05
author: 이현지
---

본 포스팅은 Node Exporter 설치 방법에 대한 글입니다.

<br><br>

# Prometheus Node Exporter
하드웨어와 UNIX 계열의 커널 관련 metrics를 수집하여 HTTP 엔드포인트로 내보내는 오픈소스 소프트웨어

<br><br>

## 설치 방법
- 압축 파일을 다운 받아 바이너리 파일로 실행
- 도커 이미지로 컨테이너 생성하여 실행
- 도커 컴포즈로 실행

<br><br>

## 바이너리 파일로 실행 
 

1. Prometheus Download 페이지 접속<br>
https://prometheus.io/download/

<br><br>

2. OS에 맞는 파일 - 우클릭 - 링크 주소 복사 <br><br>
![node exporter 바이너리 파일 다운로드](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwXJnS%2FbtsEeFjDBMq%2FuM3nbqyi6lkc2KnkFOZli1%2Fimg.png)

<br><br>

3. wget 명령어로 압축 파일 다운로드 
```bash
wget https://github.com/prometheus/node_exporter/releases/download/vx.x.x/node_exporter-x.x.x.linux-amd64.tar.gz
```
<br><br>

4. tar xzvf 명령어로 압축 파일 압축 해제
```bash
tar xzvf prometheus-x.xx.x.linux-amd64.tar.gz
```
<br><br>

5. 압축 해제된 파일이 있는 경로로 이동 후 바이너리 파일 실행
<br>
```bqsh
$ ./node_exporter
```
<br><br>

## 컨테이너로 실행

```ini
$ docker run -d --name node-exporter \
--net="host" \
--pid="host" \ 
-v "/:/host:ro,rslave" \ 
quay.io/prometheus/node-exporter:latest \
--path.rootfs=/host
```
<br>
Node Exporter를 컨테이너로 실행할 경우, 몇 가지 옵션을 설정해줘야 Node Exporter 컨테이너 자체가 아닌 호스트를 모니터링 할 수 있습니다. 
<br><br>


```bash
$ docker run -d --name node-exporter \
```

Docker를 사용하여 컨테이너를 'node-exporter'라는 이름으로 백그라운드(-d 옵션)에서 실행합니다.

```bash
--net="host" \
```

컨테이너가 호스트 네트워크를 사용하도록 설정합니다.
<br>이렇게 하면 컨테이너는 호스트의 네트워크 스택을 그대로 사용할 수 있습니다.

```bash
--pid="host" \
```

호스트의 PID 네임스페이스를 공유하여 컨테이너 내부에서 호스트의 프로세스 ID를 볼 수 있습니다.

```bash
-v "/:/host:ro,rslave" \ 
```

호스트의 루트 디렉토리("/")를 컨테이너의 "/host" 디렉토리에 읽기 전용(ro)으로 마운트합니다.<br>
'rslave' 옵션은 호스트에서 마운트 된 변경 사항이 컨테이너에 전파되도록 설정합니다.

```bash
quay.io/prometheus/node-exporter:latest \
```

컨테이너가 실행할 이미지와 이미지 버전을 명시합니다. 

```bash
--path.rootfs=/host
```

Node Exporter에게 시스템의 루트 파일 시스템이 '/host'에 마운트되어 있다는 것을 알려주는 옵션입니다.
이 옵션을 통해 Node Exporter는 호스트 시스템의 메트릭을 수집할 수 있습니다.

<br><br>

## 도커 컴포즈로 실행

```ini
version: '3.8'

services: 

  node_exporter: 
    image: quay.io/prometheus/node-exporter:latest 
    container_name: node_exporter 
    command:
      - '--path.rootfs=/host' 
    network_mode: host
    pid: host
    restart: unless-stopped 
    volumes: 
      - '/:/host:ro,rslave'
```
<br><br>

## 포트 
Node Exporter는 기본적으로 9100 포트에서 실행됩니다.

<br><br>

## 수집된 metrics 확인

Node Exporter 실행 후 아래 URL 접속 시 metrics 정보를 확인할 수 있습니다.

_Node Exporter가 설치된 IP주소:9100/metrics_



<br><br>
 > Reference 
- https://github.com/prometheus/node_exporter<br>
- https://yoo11052.tistory.com/204<br>