---
title: "API 테스트 및 문서화를 위한 Newman과 Redoc 활용"
last_modified_at: 2024-10-21
author: Gun-ha, KANG
---
> 이번 포스팅에서는 **Newman과 ReDoc**를 공유해보려고 합니다.


## **개요**

> Newman

* [Newman](https://www.npmjs.com/package/newman)은 Postman의 CLI Collection Runner
* Jenkins, Travis CI, Docker 와 통합하여 테스트 자동화 수행 가능

> ReDoc

* OpenAPI 문서화 도구  

  * OpenAPI Spec을 기반으로 한 정적 문서 생성
  * 문서화에 초점
  * API 테스트 X


> 정리

*Newman으로 API의 모든 엔드포인트에 대한 자동화된 테스트 수행 (품질 검증)*
 * CI/CD 에서 코드 배포 전 API 테스트 (성공 시, 최신 Redoc 호스팅)

**(공통 사전 작업) Postman**

  * Postman의 Collection Export (.json)
  * Postman의 Environment Export (.json, 필수는 아님)
  * Export 시, ResponseBody를 포함하려면, `Save Response` 수행



# **Newman**

*Postman으로 작성된 API test 자동화 혹은 CI/CD 파이프라인에서 사용*  
*GUI없이, 명령어 한줄로 모든 API를 테스트 가능*  
*Test Report 생성 가능 (html, json, junit 등)*  
*API 품질 검증*   


## **구성**

*docker-compose.yml*

```bash
version: '3.8'

services:
  newman:
    image: postman/newman:6.1.3-alpine
    container_name: newman2json
    command:
      run EMMA.postman_collection.json -k
      -e Staging.postman_environment.json
      -r cli,json
      --reporter-json-export="/etc/newman/reports/newman-report.json"
    volumes:
      - ./volume/collections:/etc/newman
      - ./volume/reports:/etc/newman/reports
    environment:
      NEWMAN_VERSION: 6.1.3
      LC_ALL: ko_KR.UTF-8
      LANG: ko_KR.UTF-8
      LANGUAGE: ko_KR.UTF-8
```

*기본 실행결과*

![1](https://github.com/user-attachments/assets/c1865026-73dd-465b-89e3-d315b88b3886)

*`pm.test(responseBody, true)` 를 통한 상세 결과 출력 (Response 포함)*

![5](https://github.com/user-attachments/assets/255cb2d1-065f-45ec-b0b4-58bae8c175fa)



# **Redoc**

*OpenAPI Spec 기반으로 한 API 문서화를 위한 도구*    
*html 파일로 빌드되어 호스팅 가능*    
*깔끔한 UI, 반응형 디자인*  


## **구성**

```bash
docker/redoc
|-- Dockerfile
|-- docker-compose.yml
|-- convert.js
|-- package.json
|-- package-lock.json (Dockerfile - npm install)
|-- volume
	|-- DummyAPI.postman_collection.json
	|-- output-openapi.yaml (openapi_converter로 인해 생성됨)
	|-- index.html (redoc/cli 로 생성됨)
```

*Dockerfile*

```bash
FROM node:18

WORKDIR /app

COPY package.json package-lock.json ./
COPY convert.js ./

RUN npm install

COPY . .
```


*docker-compose.yml*

  * openapi_converter : convert.js 스크립트를 실행해 openapi로 변환 (postman2openapi)
  * redoc : redocly/cli 를 사용해 변환된 openapi를 html 로 빌드
  * nginx : nginx 를 통해 html을 호스팅

```bash
version: '3.8'

services:
  openapi_converter:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./volume:/app/volume
    command: ["node", "convert.js"]

  redoc:
    image: redocly/cli
    volumes:
      - ./volume:/app/volume
    command: ["build-docs", "/app/volume/output-openapi.yml", "--output", "/app/volume/index.html"]
    depends_on:
      - openapi_converter

  nginx:
    image: nginx:latest
    ports:
      - "8001:80"
    volumes:
      - ./volume:/usr/share/nginx/html
    depends_on:
      - redoc
```

*package.json*

```bash
{
  "name": "postman-to-openapi-converter",
  "version": "1.0.0",
  "main": "convert.js",
  "type": "module",
  "dependencies": {
    "postman-to-openapi": "^3.0.1",
    "js-yaml": "^4.1.0"
  }
}
```

*convert.js*

```bash
import path from 'path';
import postmanToOpenApi from 'postman-to-openapi';

async function main() {
    const postmanCollectionPath = path.join(process.cwd(), 'volume/DummyAPI.postman_collection.json');
    const outputPath = path.join(process.cwd(), 'volume/output-openapi.yml');

    try {
        const result = await postmanToOpenApi(postmanCollectionPath, outputPath, { defaultTag: 'General' });
        console.log(`OpenAPI spec ${result}`);
    } catch (err) {
        console.log(err);
    }
}

main();
```

*호스팅된 API 문서, Redoc을 사용하여 렌더링된 OpenAPI 문서*

![2](https://github.com/user-attachments/assets/2fb38868-a243-4518-804c-12b638a4ffa2)


> **참고**  

* [Postman-to-OpenAPI](https://medium.com/@sakshamsingh5001/create-openapi-spec-from-postman-collection-using-node-js-718856d54327)
* [Redocly/cli](https://hub.docker.com/r/redocly/cli)
* [Redoc/Sample](https://redocly.github.io/redoc/#tag/Events/operation/getSpecialEvent)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
