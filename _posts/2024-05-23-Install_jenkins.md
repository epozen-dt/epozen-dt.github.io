---
title: "Jenkins 설치 및 시작하기"
last_modified_at: 2024-05-23
author: Do-soo, KIM
---


> 이번 포스팅에서는 Jenkins을 설치하여 시작해 보도록 하겠습니다.

### 설치하기
---

Jenkins를 설치하는 방법은 여러 가지가 있겠지만, 필자는 docker compose를 사용하여 설치해 보겠습니다.

#### - docker-compose.yml 파일 작성

다음과 같이 docker-compose.yml 파일을 작성합니다. <br>
일부 내용들은 각자 필요하신 값으로 수정해서 사용하시면 됩니다.

```yml
version: '3.8'
services:
  jenkins:
    container_name: dt_jenkins
    image: jenkins/jenkins:lts
    ports:
      - "8081:8080“
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - ./.ssh:/.ssh
    user: root
    environment:
      - TZ=Asia/Seoul
restart: unless-stopped

```



#### - 실행

아래 명령어로 실행합니다.

```
docker-compose up -d
```



### 시작하기
---

이제 실행이 되었으니 한번 시작해 볼까요?

하지만, 본격적으로 시작하기 전에 몇 가지 귀찮은 일을 더 해야 합니다.<br>
차근차근 해 보도록 하겠습니다.

#### - 연결 설정

웹브라우저에서 http://{설치한 ip주소}:8081로 접속합니다.<br>
그럼 아래와 같은 접속화면이 보일 겁니다.

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/5aadcf93-263b-40cc-8de1-1fd8b657e246" width="70%" height="70%" /><br>
</p>



관리자 비밀번호는 입력해야 하는데, 비밀번호는 실행된 jenkins 도커 컨테이너 내부로 진입하여 확인하면 됩니다.

아래 명령어를 실행하면 됩니다.

```
docker exec -it {container_id} /bin/bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

다음은 플러그인을 설치합니다.

Default로 추천되는 플러그인을 우선 설치하고, 필요한 플러그인은 나중에 추가 설치하면 됩니다.

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/cab48bb1-ae5a-4029-a6ed-adf88f634203" width="70%" height="70%" /><br>
</p>

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/ebda776f-49df-4077-9698-637e8ea5e072" width="70%" height="70%" /><br>
</p>


admin 계정을 생성합니다.

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/4b6556de-cbe7-40f2-9212-e5e5e8f967bc" width="70%" height="70%" /><br>
</p>

인스턴스를 설정합니다.

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/049549d1-f3e1-4eaf-ae4e-be79b13f6da0" width="70%" height="70%" /><br>
</p>

설정이 완료되면, 드디어 jenkins를 시작할 수 있습니다.

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/e0e4815d-4e58-4b47-80e9-0d23e4a60441" width="70%" height="70%" /><br>
</p>

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/95a29645-68af-451d-b060-87bd36515f8f" width="90%" height="90%" /><br>
</p>


#### - 관리자 권한 설정

jenkins를 사용할 수 있도록 관리자 권한을 설정합니다.

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/bb32fae5-71df-4a0b-81a9-452b92bc4def" width="90%" height="90%" /><br>
</p>

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/ac2567ca-9f05-4a0e-8ba7-90ecbcc79a9e" width="90%" height="90%" /><br>
</p>

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/66d037c9-787f-4345-a5d6-1edcc0751a88" width="90%" height="90%" /><br>
</p>

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/6a6eb15c-dd68-4529-a6a3-cfee6ee0a813" width="50%" height="50%" /><br>
</p>

<p align="left">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/88d36c95-d68b-4471-bcd6-701ffa3a1b1b" width="90%" height="90%" /><br>
</p>

여기까지 관리자 권한 설정이 끝났습니다.


그럼, 다음 포스팅에서는 Jenkins를 직접 사용하는 방법을 소개하겠습니다.

