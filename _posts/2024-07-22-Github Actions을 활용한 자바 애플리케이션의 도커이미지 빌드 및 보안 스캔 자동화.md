---
title: "GitHub Actions로 자바 애플리케이션의 도커이미지 빌드 및 보안 스캔 자동화"
last_modified_at: 2024-07-22
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **GitHub Actions를 활용하여 자바 애플리케이션의 Docker 이미지를 자동으로 빌드하고, Trivy를 사용해 보안 스캔을 수행하는 방법을 설명**를 공유해보려고 합니다.


# **개요**

* **CI/CD 파이프라인 구축:** GitHub Actions를 활용하여 애플리케이션의 Docker 이미지를 자동으로 빌드하고 배포
* **보안 점검:** Trivy를 사용하여 Docker 이미지의 보안 취약점을 점검하고, 안전한 배포를 보장


### **사전 설정**

**1. GitHub Repo 설정**
  
*깃허브  레포 > Actions 탭 > Set up this workflow 클릭*
  * `./github/workflows/<workflow_name>.yml` 생성


**2. DockerHub에 Personal access tokens 설정**

* Generate new token
	* Scope (Acess permissions): Read & Write

**3. GitHub Repo > Settings 탭 > Secrets and variable 탭 > Action 탭**

* Repository secrets

	* DOCKERHUB_USERNAME (사용자 설정)
		* 도커허브의 유저이름
	* DOCKERHUB_TOKEN (사용자 설정)
		* 도커허브의 토큰값



### **Workflow File**

*다음은 자바 애플리케이션의 week1 모듈에 대해 Docker 이미지를 빌드하고, Trivy를 사용하여 보안 스캔을 수행하는 GitHub Actions의 예시*


```bash
name: Java App Docker Build and Image Scan

on:
  push:
    branches:
      - master
    paths:
      - 'week1/**'
  pull_request:
    branches:
      - master
    paths:
      - 'week1/**'
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Check out repository
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        distribution: 'adopt'
        java-version: '17'
        
    - name: Cache Maven packages
      uses: actions/cache@v4
      with:
        path: ~/.m2/repository
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: |
          ${{ runner.os }}-maven-

    - name: Build Maven Project
      run: mvn -B clean install -pl week1

    - name: Build Docker Image
      run: |
        cd ./week1/C:/apps/demo/week1/
        docker build -t gunnha/my-app-week1:latest .

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Push Docker Image
      run: docker push gunnha/my-app-week1:latest

    - name: Install Trivy
      run: |
        sudo apt-get update
        sudo apt-get install -y gnupg curl
        curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.18.3

    - name: Scan Docker Image with Trivy
      run: trivy image gunnha/my-app-week1:latest
``` 

**Workflow 옵션**

* 트리거 조건 : master 브랜치로의 push, pull_request와 week1/** 경로에 변경이 있을때
* 워크플로우 수동 실행 추가
* Job으로 build를 정의
  * uubuntu-latest 가상머신에서 실행됨
  * Step들이 순차적으로 실행
    * `actions/checkout@v3`
      * 저장소의 코드 체크아웃 (현재 레포를 가져옴)
    * `actions/step-java@v3`
      * JDK 17버전을 설정
    * `actions/cache@v3`
      * Maven 패키지 캐싱 (디스크 캐싱을 통해 빌드 속도 향상)
    * `Maven 빌드`
    * `docker build`
    * `docker/login-ation@v2` 
      * DockerHub 로그인
    * `docker push`
    * `Trivy Install`
    * `Trivy Image Scan`  


**실행 결과**

![1](https://github.com/user-attachments/assets/d835e854-7bef-4d9b-9f53-94ec0bbe7bbe)


**주의 사항**

* Dockerfile 위치 : week1 디렉토리 내에 위치
  * src>main>docker
* 현재는 maven 빌드 시 -pl week1을 사용하여 해당 모듈만 빌드되도록 설정
  * 즉, 전체 프로젝트가 아닌 특정 모듈만 빌드



> **참고**  

* [Trivy (Github)](https://github.com/aquasecurity/trivy)



---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
