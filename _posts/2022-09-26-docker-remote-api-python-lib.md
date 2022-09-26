---
title: "Docker remote api - Python 클라이언트 라이브러리"
last_modified_at: 2022-09-26
author: 심건우
---

이번 포스팅에선 이전 포스팅에서 java로 접근했던 Docker remote api를 python 클라이언트로 접근해보았다.

## Docker remote API Python SDK
 Docker 공식 remote api 관련 sdk이다.
  - Apache License 2.0
  - Docker 측에서 직접 테스트 완료
  - Docker 측에서 버전 관리
  - pip를 통한 간단 설치(pip install docker)

[참고](https://docs.docker.com/engine/api/sdk/)

## 구현 방식 비교 (python vs java)
 아래 코드는 두 라이브러리 간 컨테이너 생성 및 환경 변수 설정 시, 차이를 비교한 것이다.
  - python은 dictionary에서 key를 통해 값을 가져오고, java는 builder 패턴을 사용

![image](https://user-images.githubusercontent.com/87160438/192229718-9e07b65d-7c0d-4b7e-b0b0-457176ced2e3.png)


## 코드 예시
 - docker import

```python
import docker
```

 - 컨테이너 목록 조회

```python
def test_get_container_list(self):
 """
 docker ps
 :return:
 """
 # 클라이언트 연결
 client = docker.DockerClient(base_url = '도커 host url')
 
 # 컨테이너 목록 출력
 for container in client.containers.list():
  print(container.attrs)
 
 # 클라이언트 연결 종료
 client.close()
```


 - 도커 이미지 pull

```python
def test_pull_image(self):
 """
 docker pull
 :return:
 """
 # 클라이언트 연결
 client = docker.DockerClient(base_url = '도커 host url')
 
 # docker image pull, mongodb
 client.images.pull("mongo:5.0")
 
 # 클라이언트 연결 종료
 client.close()
```


 - 컨테이너 환경 변수 설정 및 실행

```python
def test_run_container(self):
 """
 docker run
 :return:
 """
 # 클라이언트 연결
 client = docker.DockerClient(base_url = '도커 host url')
 
 # 컨테이너 환경 설정 및 실행
 client.containers.run(image="mongo:5.0",
                       restart_policy={'Name' : 'always'},
                       ports={'27017/tcp': ('', 27017)},
                       volumes=["/home/gunwu/8.mongo/mongodb:/data/db"],
                       environment=[
                           "MONGO_INITDB_ROOT_PASSWORD=''",
                           "MONGO_INITDB_DATABASE=''",
                           "MONGO_INITDB_ROOT_USERNAME=''"
                       ],
                       name='epozen-test',
                       detach=True)
 # 클라이언트 연결 종료
 client.close()                       
```


 - 컨테이너 중지 및 삭제

```python
def test_stop_and_remove_container(self):
 # 클라이언트 연결
 client = docker.DockerClient(base_url = '도커 host url')
 
 # 컨테이너 중지
 client.containers.get('epozen-test').stop()
 
 # 컨테이너 삭제
 client.containers.get('epozen-test').remove()
 
 # 클라이언트 연결 종료
 client.close()                       
```
