---
title: "Docker Registry HTTP API"
last_modified_at: 2023-09-21
author: 최선빈
---

본 포스팅은 Docker Registry HTTP API V2에 대한 내용입니다.

---
&nbsp;

> ## Docker Registry HTTP API V2

&nbsp;

- 도커 이미지를 도커 엔진으로 쉽게 배포하기 위한 프로토콜입니다.
- 도커 이미지 정보 관리 및 배포를 담당하는 도커 레지스트리와 상호 작용합니다.

![Docker REGISTRY](https://4.bp.blogspot.com/-yeD6y92nmvc/W1SvdGaBHUI/AAAAAAAABcs/5EXkFqwbeVUh2guqQDvjJ-xbjf4BFaP1wCLcBGAs/s1600/private%2Bdocker%2Bregistry.jpg)

---
&nbsp;

> ## 도커 이미지

&nbsp;

- 도커 이미지는 일련의 레이어들로 구성합니다.
- 각 레이어는 이미지의 Dockerfile 내 명령을 의미합니다.

![Docker Image](https://cdn.jsdelivr.net/gh/springcloud-community/image-bucket/2022/02/13/f2089fe302fb404db4569cfd22746afc.png)

- 이미지의 각 레이어는 레지스트리에 저장 시, 아래 세 종류의 파일 형식으로 저장됩니다.
- 'docker pull ~' 명령어는 아래 세 가지 레이어들을 다운로드 하는 것 과 동일합니다.

- **Manifest** 레이어 (1개만 존재)
    - 내용: 이미지 메타(Config, Blob 레이어들 대한 정보)를 저장
    - 형식: Application/vnd.docker.distribution.manifest.v2+json

- **Config** 레이어 (1개만 존재)
    - 내용: 이미지의 실행 플랫폼, OS 등에 대한 정보를 저장
    - 형식 Application/vnd.docker.container.image.V1+json

- 1개 이상의 **Blob** 레이어
    - 내용: 이미지 자체에 대한 레이어 blog 형식의 파일로 구성
    - 형식: Application/vnd.docker.image.rootfs.diff.tar.gzip

---

> ## 도커 레지스트리 내부 구조

&nbsp;

- 도커 레지스트리는 트리 구조로 이미지 관련 내용들을 디렉토리에 저장

[Registry] -> [V2] -> [blob] -> [sha256 - 1c - 1c889ce.... - data]
[v2] -> [repositores] -> [이미지명] -> [_layer] -> [sha25 ... link ... link]
[이미지명] -> [_manifests] -> [revision - sha256 - ... - link, tag - latest - curren - link, latest - index - sha256 - ... - link]
[이미지명] -> [_uploads]

---
&nbsp;

> ## 주요 기능 - 이미지 목록 조회

&nbsp;

- RePository 조회 (이미지 목록 조회)
    - curl -X GET [레지스트리 주소]/v2/_catalog
        - 결과: {"repositories":["centos", "ubuntu"]}

- 이미지 태그 조회
    - curl -X GET [레지스트리 주소]/v2/이미지명/tags/list
        - 결과: {"name":"ubuntu","tags":["latest","16.04"]}
---
&nbsp;

> ## 주요 기능 - 이미지 Pull

&nbsp;

- Manifest 레이어 다운로드
    - curl -X GET [레지스트리 주소]/v2/이미지명/manifest/태그명 -ㅗ "Accept":
    - 결과: application/vnd.docker.distribution.manifest.v2+json" > image.manifest

- Config 레이어 다운로드
    - curl -H "Accept: application/vnd.docker.image.rootfs.diff.tar.gzip" -X GET [레지스트리 주소]/v2/이미지명/blobs/sha256:*digest > digest

- Blobs 레이어 다운로드
    - curl -H "Accept: application/vnd.docker.image.rootfs.diff.tar.gzip" -X GET [레지스트리 주소]/v2/이미지명/blobs/sha256:*digest > digest


---
&nbsp;

> ## 주요 기능 - 이미지 Push

&nbsp;

- Blobs 레이어 업로드
    - Location 값 획득: curl -i -d '' -X POST http://[[도메인주소]]/v2/[[레포지토리 이름]]/blobs/uploads/
    - 업로드: curl -v -X PUT -G -H "Content-Type: application/octet-stream" -T "[[blob파일 절대경로]]" --data "digest=sha256:[[레이어의 digest값]]" [[Location값]]

- Config 레이어 업로드
    - Blobs 레이어 업로드 과정처럼 Location 값 획득 후, 업로드

- Manifest 레이어 업로드
    - curl -v -X PUT -G -H "Content-Type: application/vnd.docker.distribution.manifest.v2+json" -T "[[manifest 파일절대경로]]“ http://[[registry 도메인]]/v2/[[repo이름]]/manifests/[[태그명]]
---
&nbsp;

> ## 주요 기능 - 이미지 삭제

&nbsp;

- Manifest 삭제
- 이미지 관련 메타 정보만 사라지고, 일부 blob은 참조되지 않은 상태로 남아있음
    - curl -X DELETE 저장소경로/v2/이미지명/manifests/sha256:digest
    - Garbage Collection

- 시스템에서 더 이상 참조되지 않는 blob(Layer or Manifest)들을 제거할 때 사용
- 주로 Registry 서버의 저장공간 관리 목적으로 사용됨
    - registry 서버 컨테이너 내부접속
    - bin/registry garbage-collect [--dry-run] [--delete-untagged] /path/to/config.yml
---
&nbsp;

> ## 주요 기능 - 정리

&nbsp;

- private Registry에 등록된 이미지 목록은 repository(이미지명)과 tag로 구분되어 있음
    - Registry 내 이미지 목록 조회 기능 구현 시 참고

- ‘docker pull’ / ‘docker push’는 Registry API를 여러 번 호출하는 과정을 통해 구현됨
    - pull / push 는 Docker remote API로 구현되어 있음

- private Registry에 등록된 단일 이미지 자체를 삭제하는 기능은 없음
    - Manifest 삭제를 통해 이미지에 대한 참조 정보 삭제
    - Registry 컨테이너 내부에서 Garbage Collection을 통해 미사용 Blobs 삭제
    - Registry의 Volume에 직접 접근하여 repository 자체를 삭제하는 방법도 있음
    - 삭제 기능 구현 시, (manifest 삭제 + 컨테이너 내부 명령 실행) 방안 고려

---
&nbsp;

> ## Private Registry 보안 적용

&nbsp;

- 인증
    - Registry는 패스워드 기반 인증을 통해 접근제어 가능
    - 인증을 사용하는 Registry 서버 기동 시, 아래와 같은 환경변수 추가
        - REGISTRY_AUTH=htpasswd
        - REGISTRY_AUTH_HTPASSWD_REALM=Registry-Realm
        - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd

- 외부 접속 허용 시, TLS 설정
    - TLS 설정을 사용하는 Registry 서버 기동 시, 아래와 같은 환경변수 추가
    - Registry 서버에 접근하는 클라이언트는 인증서를 갖고 있어야 함
        - REGISTRY_HTTP_ADDR=0.0.0.0:443
        - REGISTRY_HTTP_TLS_CRETIFICATE=/certs/server.crt
        - REGISTRY_HTTP_TLS_KEY=/certs/server.key

---
&nbsp;

> ## 도커 이미지 save & load

&nbsp;

- ’docker save’ 기능을 통해 도커 이미지 압축파일(.tar) 생성 가능
- ‘docker load’ 기능을 통해 도커 이미지 압축파일을 도커 이미지로 업로드 가능
- 클라이언트에서 A서버의 이미지 save 후, B서버로 이미지 load 가능

      #docker save
      image = client_7.images.get('httpd:2')
      f = open('./httpd-2.tar', 'wb')
      for chunk in image.save(named=True):
        f.write(chunk)
      f.close()

      #docker load
      f = open('./httpd-2.tar', 'rb')
      client_9.images.load(f.read())
      f.close()

---
&nbsp;

------
> 참고문헌

[1] [도커 이미지1](https://trylhc.tistory.com/entry/Docker-registry-API-pull)

[2] [도커 이미지2](https://velog.io/@namkun/Image-Layer)

[3] [Docker REGISTRY HTTP API v2](https://velog.io/@namkun/Image-Layer)

[4] [도커 이미지 load](https://docs.docker.com/engine/reference/commandline/load/)

[5] [도커 이미지 save](https://docs.docker.com/engine/reference/commandline/save/)
