---
title: "Java Application에서 PyTorch 사용을 위한 libtorch 윈도우/리눅스 설정 가이드"
last_modified_at: 2024-07-19
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **Java Application에서 PyTorch 사용을 위한 libtorch 윈도우/리눅스 설정 가이드**를 공유해보려고 합니다.


# **개요**

**Java Application에서의 PyTorch 사용 (3가지)**

  * (1) libtorch 설정을 통한 PyTorch 사용  
    + PyTorch의 JIT 컴파일러를 활용하여 모델을 TorchScript로 변환  
    + 변환된 모델을 Java 바인딩을 제공하는 라이브러리를 사용하여 Java에서 사용  
      + 이때, libtorch 가 필수
  * (2) Deep Java Library (DJL)  
  * (3) ONNX Runtime  

이 중 libtorch 설정을 통한 PyTorch 사용을 위해 필요한 사전 설정 방법 사용

* 장/단점
  * 직접적인 PyTorch C++ API 접근으로 높은 성능
  * 단, 시스템 환경에 따라 설정이 다르며, 좀 복잡

***※ 테스트: libtorch-CPU (Not use GPU, CUDA) 버전 사용***
  * *현 시간 기준, 2.3.0(stable) 은 dll 인식이 잘 안되서 2.2.2 를 사용했음*

## **1. Window**

* **1.1 libtorch 다운로드 및 설치**
  * 릴리즈(배포용) 버전 사용
    * 디버깅 정보 삽입X, 코드를 최적화하여 실행파일 크기를 최대한 줄여준 버전

![1](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/f2a7f742-9f0d-4990-9981-51dd21998e91)

* **1.2 윈도우의 시스템환경변수에 등록**
  * ex. C:\Program Files\libtorch\lib

![2](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/cadb3202-f5d9-4551-90ba-71bd7bd201fc)

* **1.3 PC 재부팅**


## **2. Linux**

*리눅스는 윈도우에 비해, 버전 호환성이 더 까다로운 느낌*

**예시. 서버 디렉토리 구조**

```bash
/path/to/app/4.pytorch
├── Dockerfile
├── docker-compose.yml
├── target
│   ├── pytorch-java-1.0-SNAPSHOT.jar
│   └── lib/
├── ext
│   ├── model
│   │   └── lstm_v2.pt
│   └── example
│       ├── main.cpp
│       └── CMakeLists.txt
``` 

**예시. 컨테이너 내부 디렉토리 구조**

```bash
/app
├── pytorch-java-1.0-SNAPSHOT.jar
└── lib/

/home/libtorch
├── 설치 파일들
├── CMakeLists.txt  # 빌드 테스트용, 없어도 무방
└── example
    └── main.cpp  # 빌드 테스트용, 없어도 무방
``` 

* **2.1 libtorch URL 복붙**

  * c++11 이전에 컴파일된 라이브러리들과의 호환성을 지원하는 pre-cxx11 ABI 사용

![3](https://github.com/gunnha/epozen-dt.github.io/assets/92897860/15dfd459-cab6-4349-91fc-c42d40d436af)

* **2.2 Dockerfile 및 docker-compose.yml 작성**

  * openjdk:8  
    + Debian GNU/Linux 11 (bullseye)  
    + 패키지 매니저: apt 사용  
  * openjdk:17  
    + Red Hat Enterprise Linux(RHEL) 8.5  
    + 패키지 매니저: microdnf 사용  

![5](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/06eeae72-4f90-430c-8cbf-da5e7b2d5131)


* **선택. libtorch Build Test를 원하면,**


![6](https://github.com/epozen-dt/epozen-dt.github.io/assets/92897860/3dab1235-77c1-45a1-b6d4-75f08749b942)


## **결론**

libtorch 설정을 통해 java에서 PyTorch 모델이 사용가능함을 확인



> **참고**  

* [PyTorch KR](https://pytorch.kr/get-started/locally/)


---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
