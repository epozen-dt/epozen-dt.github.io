---
title: "Docker remote api - 보안 설정 (TLS)"
last_modified_at: 2022-10-27
author: 심건우
---

이번 포스팅에선 Docker remote API 보안 설정 (TLS)에 대해 다룬다.

# Docker remote API TLS(Transport Layer Security)
 - Docker remote API는 REST API를 기반으로 작동한다.(HTTP)
 - 외부에서 Docker Daemon에 접근 시, HTTP 관련 보안 설정이 필요하다.
 - Docker는 HTTP 관련 보안 설정 방법으로 TLS(HTTPS) 기능을 제공한다.
 - 아래 구조는 서버와 클라이언트 서로 인증이 가능한 Docker TLS 구조이다.

![image](https://user-images.githubusercontent.com/87160438/198226274-4e51feaf-6a16-41c6-a533-43e07aacee4d.png)


[TLS 상세 설명](https://www.ibm.com/docs/ko/ibm-mq/9.1?topic=mechanisms-cryptographic-security-protocols-tls)

# Docker TLS 적용 방법
- 서버에 생성되는 파일 목록

![image](https://user-images.githubusercontent.com/87160438/198226387-788611cd-211e-4b74-bd41-2653c127c05d.png)


- 클라이언트에 생성되는 파일 목록

![image](https://user-images.githubusercontent.com/87160438/198226422-053d97dd-4c28-46e0-9ff9-2633211a470c.png)


## 1. CA(Certificate Authority) 개인키 생성
 - 인증서 내 공개키 디지털 서명 용도, 유출 금지
 - Docker Daemon Host Server에서 진행

```commandline
openssl genrsa -aes256 -out ca-key.pem 4096
```

## 2. CA(Certificate Authority) 공개키 생성
 - 서버/클라이언트 공용
 - 인증서 내 디지털 서명 복호화 시 사용
 - Docker Daemon Host Server에서 진행

```commandline
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
```

## 3. 서버 개인키 생성
 - 클라이언트에서 송신한 암호화된 세션키를 복호화 시 사용
 - Docker Daemon Host Server에서 진행

```commandline
openssl genrsa -out server-key.pem 4096
```

## 4. 서버 CSR(Certificate Signing Request) 생성
 - 서버 인증서에 포함되는 서버 공개키
 - 클라이언트에서 세션키 암호화 시 서버 공개키 사용
 - $HOST, 호스트명은 CA 공개키 생성 과정(2)에서 설정한 "Common Name"과 일치해야 함
 - Docker Daemon Host Server에서 진행

```commandline
openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
```

## 5. 인증서 생성 시 필요한 IP 주소 지정
 - TLS 연결은 DNS 이름뿐만 아니라 IP를 통해서도 가능
 - $HOST, 호스트명은 CA 공개키 생성 과정(2)에서 설정한 "Common Name"과 일치해야 함
 - Docker Daemon Host Server에서 진행

```commandline
echo subjectAltName = DNS:$HOST,IP:xxx.xxx.xxx.xxx >> extfile.cnf
```

## 6. 서버 키를 서버 인증 용도로 설정
 - Docker Daemon Host Server에서 진행

```commandline
echo extendedKeyUsage = serverAuth >> extfile.cnf
```
## 7. 서버 인증서 생성
 - CSR(서버 공개키), 디지털 서명, 기타 설정이 포함된 인증서
 - Docker Daemon Host Server에서 진행

```commandline
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAKey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
```

## 8. 클라이언트 개인키 생성
 - 클라이언트 인증 시 사용
 - 클라이언트에서 생성한 랜덤값을 해당 개인키로 암호화 후 서버로 전송
 - Docker Daemon Host Server에서 진행

```commandline
openssl genrsa -out key.pem 4096
```

## 9. 클라이언트 CSR(Certificate Signing Request) 생성
 - 클라이언트 인증서에 포함되는 클라이언트 공개키
 - 서버에서 클라이언트 인증 시 암호화된 랜덤값 복호화 시 사용
 - Docker Daemon Host Server에서 진행

```commandline
openssl req -subj "/CN=client" -new -key key.pem -out client.csr
```

## 10. 클라이언트 키를 클라이언트 인증 용도로 설정
 - Docker Daemon Host Server에서 진행

```commandline
echo extendedKeyUsage = clientAuth > extfile-client.cnf
```

## 11. 클라이언트 인증서 생성
 - CSR(클라이언트 공개키), 디지털 서명, 기타 설정이 포함된 인증서
 - ca.pem, ca-key.pem은 서버 인증서 생성 시 사용한 것과 동일
 - Docker Daemon Host Server에서 진행

```commandline
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAKey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf
```

## 12. 인증서 생성에 사용된 CSR 및 기타 설정 파일 제거
 - Docker Daemon Host Server에서 진행

```commandline
rm -v client.csr server.csr extfile.cnf extfile-client.cnf
```

## 13. 키 손상 방지를 위해 쓰기 권한 제거, 파일 작성자만 읽기 권한 부여
 - Docker Daemon Host Server에서 진행

```commandline
chmod -v 0400 ca-key.pem key.pem server-key.pem
```

## 14. 인증서 손상 방지를 위해 쓰기 권한 제거
 - Docker Daemon Host Server에서 진행

```commandline
chmod -v 0444 ca.pem server-cert.pem cert.pem
```

## 15. /usr/lib/systemd/system/docker.service의 ExecStart 속성 수정
 - tlsverify 활성화
 - CA(tlscacert), 서버 인증서(tlscert), 서버 개인키(tlskey) 경로 지정
 - Docker Daemon TLS 전용 Port인 2376 추가
 - Docker Daemon Host Server에서 진행

```
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
 --tlsverfiy 
 --tlscacert=ca.pem 경로/ca.pem 
 --tlscert=server-cert.pem 경로/server-cert.pem 
 --tlskey=server-key.pem 경로/server-key.pem 
 -H tcp://0.0.0.0:2376
```

## 16. 변경된 설정 적용 위해 Daemon Reload
 - Docker Daemon Host Server에서 진행

```commandline
systemctl daemon-reload
```

## 17. 변경 설정을 포함한 Docker Restart
 - Docker Daemon Host Server에서 진행

```commandline
service docker restart
```

## 18. 외부 접속 허용을 위해 방화벽 2376 port 추가 및 적용
 - Docker Daemon Host Server에서 진행

```commandline
firewall-cmd --permanent --zone=public --add-port=2376/tcp
```

```commandline
firewall-cmd --reload
```

## 19. 클라이언트 설정 방법 (Java 라이브러리)

```java
DockerClient dockerClient = DefualtDockerClient.builder()
        .uri(URI.create("https://도커 서버 주소:2376"))
        .dockerCertificates(new DockerCertificates(Paths.get("tls 관련 파일들 경로")))
        .build();
```

## 20. 클라이언트 설정 방법 (Python 라이브러리)

```python
tls_config = docker.tls.TLSConfig(ca_cert='ca.pem 경로', 
                                  client_cert=('cert.pem 경로', 'key.pem 경로'))
client = docker.DockerClient(base_url='https://도커 서버 주소:2376',
                             tls=tls_config)
```
