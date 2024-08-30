---
title: "Next.js API Route 비교 및 API 라이브러리 (Axios) 적용"
last_modified_at: 2024-08-30
author: Hwa-Yong, KANG
---

> 이번 포스팅에서는 Next.js 에서 사용되는 Routing (APP/Page Router) 에서 API Route 를 비교하고, API 라이브러리인 axios 의 사용 방법에 대해서 작성해 보도록 하겠습니다.

### API Route 비교

**1. APP Router**
  * 구조
    - /app/api/ 디렉토리 내에 Route를 정의
    - 디렉토리 내에서 route.js or route.ts 파일 이외에 다른 이름으로 생성 시, 오류 발생
  * 명시적인 HTTP 메서드 이름 사용
    - App Router 방식에서는 파일 내에 직접 HTTP 메서드 이름(GET, POST, PATCH, PUT, DELETE)을 사용하여 함수 정의
    - 다른 이름으로 HTTP 메서드 이름 정의 시, 오류 발생
    - 예시
    ```typescript
    export async function GET(req: NextRequest, res: NextResponse) {
      // GET 요청을 처리하는 로직
    }
    
    export async function POST(req: NextRequest, res: NextResponse) {
      // POST 요청을 처리하는 로직
    }
    ```
  * 응답 객체 생성 방식
    - NextResponse 와 표준 Response 객체를 사용하여 HTTP 응답을 생성하고 반환
    - NextResponse.json(data) 를 사용하면 자동으로 Content-Type 헤더가 application/json 으로 설정 된다
    - Response 객체는 Fetch API의 일부로, HTTP 응답을 나타내며, new Response(body, option) 을 통해 생성
    - 예시
    ```typescript
    // 정상 Case
    return new Response(JSON.stringify(data), {
      status: 200,
      headers: { “Content-Type”: “application/json” }
    }

    // 오류 Case
    return new Response(JSON.stringify({ error: “오류 메시지” }), {
      status: 400,
      headers: { “Content-Type”: “application/json” }
    }
    ```
  
**2. Pages Router**
  * 구조
    - /pages/api/ 디렉토리 내에 API Endpoint를 정의
    - 디렉토리 내에서 파일명은 자유롭게 지정 할 수 있다 (예 : /pages/api/test.ts)
  * Handler 함수 내에서 HTTP 메서드 분기 처리
    - App Router 방식에서는 주로 Handler 라는 이름의 함수를 사용하여 모든 요청을 처리
    - 함수 내부에서 req.method를 확인하여 HTTP 메서드에 따라 다른 로직을 실행
    - 예시
    ```typescript
    export default async function handler(req: NextRequest, res: NextResponse) {
      // req로부터 요청 데이터 접근
      if (req.method === “GET”) {
        // GET 요청을 처리하는 로직
      } else if (req.method === “POST”) {
        // POST 요청을 처리하는 로직
      }
    };
    ```
  * 응답 객체 생성 방식
    - res.status().json() 메서드 체이닝을 사용하여 HTTP 상태 코드와 JSON 응답을 클라이언트에 전송
    - 에러 상황을 처리하기 위해 res 객체의 메서드를 사용하여 직관적으로 응답을 구성하고 전송
    - 예시
    ```typescript
    // 정상 Case
    res.status(200).json(data);

    // 오류 Case
    res.status(400).json({ error: “에러 메시지” });
    ```

### API 라이브러리 설명

**1. axios**
  * axios 란?
    - axios 라이브러리는 브라우저, Node.js 를 위한 Promise API를 활용하는 HTTP 비동기 통신 라이브러리

  * 사용 가능한 HTTP Methods (GET, POST, DELETE, PUT, PATCH) 및 기본 사용 방법
    - GET : axios.get(”url주소”, {옵션})
    - POST : axios.post(”url주소”, {data, header})
    - DELETE : axios.delete(”url주소”)
    - PUT : axios.put(”url주소”, {data, header})

  * EMMA Project에서 적용 예
    ```typescript
    try {
      const getQueueEndpointUrl = `/api/v2/status/endpoint/queues`;
      const getParamsVal = {
        group: groupVal,
        brokerService: bsVal,
        vpn: vpnVal,
        sort: sortVal,
        sortType: sortTypeVal,
        searchParam: searchVal
      };

      await axios.get(getQueueEndpointUrl, {
        params: getParamsVal,
        headers: headerInfo,
        paramsSerializer: (params) => {
          return qs.stringify(params, { arrayFormat: "repeat" });
        },
      })
      .then(function (response) {
        const resCode = response.status;
        if (resCode === 200) {
          const dataVal = response.data.data;
          setTableData(dataVal);
        }
      })
      .catch(function (error) {
        console.log(error);
      });
    } catch (err) {
      console.error("Error fetching table data:", err);
    }
    ```
> **참고**  

* [Axios 라이브러리 개념 정리](https://velog.io/@hyunn/Axios-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC)

