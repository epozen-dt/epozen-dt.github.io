---
title: "쿠버네티스 테스트하기 #2: 쿠버네티스 클러스터에 어플리케이션 배포하기"
last_modified_at: 2022-01-24
author: Do-soo, KIM

---

이번 포스팅은 지난 포스팅에 이어 쿠버네티스 클러스터에 어플리케이션을 배포하는 방법을 소개하겠습니다.
지난 포스팅을 안 보신 분은 이 포스팅을 보시고 테스트할 수 없으니 [쿠버네티스 테스트하기 #1: 쿠버네티스 클러스터에 어플리케이션 배포하기](https://epozen-dt.github.io/2022/01/13/k8s_cluster.html)로 가서 꼭 한번 따라해 보시고 오세요.


### 1. 도커 이미지 생성 및 실행하기

먼저 도커 이미지로 생성할 어플리케이션을 작성해 보겠습니다.
간단한 node.js 어플리케이션입니다.

```
[root@k8s-master ~]# vi hello.js
console.log('Start WebServer');

var http = require('http');

http.createServer(function (request, response) {
      response.writeHead(200, {'Content-Type' : 'text/plain'});
      response.write('Hello nodejs');
      response.end();
}).listen(8889);
```

이제 도커 파일을 작성합니다.
도커 파일은 도커 이미지를 생성하기 위한 스크립트(설정파일) 입니다. 여러가지 명령어를 토대로 Dockerfile을 작성한 후 빌드하면 Docker는 Dockerfile에 나열된 명령문을 차례대로 수행하며 도커 이미지를 생성해 줍니다. 

도커 파일을 작성 할 땐 실제 파일의 이름을 'Dockerfile'로 해야 합니다.

```
[root@k8s-master ~]# vi Dockerfile
FROM node:12
RUN npm install
COPY . .
EXPOSE 8889
CMD ["node","hello.js"]
```

그럼 도커 이미지를 빌드해 보겠습니다. 꼭 그래야 하는 것은 아니지만 편의 상 이미지 이름은 '[docker registry 주소]/[이미지명]:[태그명]' 형태로 지정해 줍니다.
뒤에서 다시 나오겠지만, 이 포스팅에서는 master node를 docker registry로 사용할 겁니다.(docker registry의 기본 포트는 5000입니다.)
docker registry는 도커 이미지 파일들을 보관하는 docker hub라고 생각하시면 됩니다.

```
[root@k8s-master ~]# docker build . -t 192.168.10.101:5000/nodejs:v1
```

그럼, 생성된 도커 이미지를 실행해 보겠습니다.

```
[root@k8s-master ~]# docker run -d -p 8889:8889 192.168.10.101:5000/nodejs:v1
```
제대로 실행되었는지 확인해 볼까요?
웹브라우저를 띄워 접속해 봅니다.

![image](https://user-images.githubusercontent.com/92565548/150733603-0f686553-7289-4104-b346-c32d35a7449b.png)


### 2. k8s 클러스터에 어플리케이션 배포하기

작성한 어플리케이션을 도커 이미지로 만들어 실행해 보았습니다.
이제 이 도커 이미지를 k8s 클러스터에 배포해 보겠습니다.
도커 이미지를 배포하기 전에 저장소를 먼저 설정해야 겠죠? 이 저장소를 우리는 이전부터 docker registry라고 부르고 있습니다. 

이미지를 pull한 다음, 실행하면 설정이 완료됩니다.
master node에서 이 작업을 진행하고 있기 때문에 앞에서 언급했듯이 master node를 docker registry로 사용하게 됩니다.

```
[root@k8s-master ~]# docker pull registry

[root@k8s-master ~]# docker run -dit --name docker-registry -p 5000:5000 registry
```

그럼 이제 아까 만들어 둔 도커 이미지를 docker registry로 올려줘야 합니다. k8s 클러스터에서 공유할 수 있도록 말이죠....

```
[root@k8s-master ~]# docker push 192.168.10.101:5000/nodejs:v1
```

이제 deployment를 진행해서 pod를 생성해 보겠습니다.

deployment를 위한 파일을 작성하고 실행하면 됩니다.

```
[root@k8s-master ~]# vi hello-deploy.yaml

[root@k8s-master ~]# kubectl create -f hello-deploy.yaml
```

pod가 제대로 생성되었는지, 확인해 봅니다.

![image](https://user-images.githubusercontent.com/92565548/150733703-1f5be82a-bac2-4435-8d33-a1ace76157ac.png)

![image](https://user-images.githubusercontent.com/92565548/150733774-773718a2-932e-4e5e-b0fc-11473df8e3bb.png)

이제 서비스를 등록합니다.
서비스를 등록하는 이유는 작성한 Deployment를 외부에서 접근하기 위해서 입니다. 

NodePort 서비스로 노드의 포트를 여는 방법과 포트 포워딩을 활용하는 방법, 2가지가 있는데 이 포스팅에서는 NodePort 서비스를 이용하는 방법으로 테스트 해 보겠습니다.

서비스 등록 파일을 작성하고, 실행하면 됩니다.

```
[root@k8s-master ~]# vi hello-svc.yaml

[root@k8s-master ~]# kubectl create -f hello-svc.yaml
```

서비스가 제대로 등록되었는지 확인해 봅니다.

![image](https://user-images.githubusercontent.com/92565548/150733872-67b47f22-f4b5-41c7-ab41-c2dc7669582b.png)

자, 배포가 완료되었으니 어플리케이션을 확인해 볼까요?

![image](https://user-images.githubusercontent.com/92565548/150733948-6d19739d-9e5b-402b-b388-037b605dd36f.png)

잘 되시나요? 혹시 안 되시는 분들은 포스팅을 다시 한번 천천히 따라해 보세요.

그럼 이번 포스팅은 여기서 마치겠습니다.































