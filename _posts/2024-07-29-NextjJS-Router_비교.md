---
title: "Next.js Router 비교"
last_modified_at: 2024-07-29
author: Hwa-Yong, KANG
---

> 이번 포스팅에서는 Next.js 에서 사용되는 Routing (APP/Page Router) 에 대해서 비교해 보도록 하겠습니다.


### Routing

* Next의 Docs 가 APP/Page Router 에 따라 두 가지 버전으로 나뉠 만큼, Next의 모든 기술은 Router를 기반으로 동작
* App Router 는 13.4.0 버전부터 안정와 되어 많은 개발자가 Page Router -> App Router 로 마이그레이션 작업을 진행하고 있음
* Next.js 는 파일 시스템을 기반으로 Routing을 구현


**1. Built-on**
  * App Router : 서버 중심 라우팅
    - react server components를 기반으로 구축
    - 서버 데이터 가져오기에 중점되어 있음
    - 경로 이동 시, 페이지를 다시 렌더링 하지 않고, SPA처럼 URL만 업데이트 하고 next는 변경된 세크먼트만 렌더링
  * Page Router : 클라이언트 중심 라우팅
  
**2. Location**
  * App Router
    - app 디렉토리 내에 root layout 을 필수로 포함해야 함
    - root layout 뿐 아니라 각각의 화면에 대한 Layout 을 만들어 compose 할 수 있음
    - 데이터 패칭 또한 동시에 가능

  * Page Router
    - 전역 공유 Layout을 지정하기 위해 _app 파일을 사용
    - 여러 layout을 compose 할 수 없음
    - data fetching과 component를 함께 배치 할 수 없음

**3. Layout**
  * App Router
    - app 디렉토리 내에 root layout 을 필수로 포함해야 함
    - root layout 뿐 아니라 각각의 화면에 대한 Layout 을 만들어 compose 할 수 있음
    - 데이터 패칭 또한 동시에 가능

  * Page Router
    - 전역 공유 Layout을 지정하기 위해 _app 파일을 사용
    - 여러 layout을 compose 할 수 없음
    - data fetching과 component를 함께 배치 할 수 없음

**4. Advanced**
  * App Router
    - Parallel Router : 동일한 layout에서 하나 이상의 페이지를 동시 또는 조건부로 렌더링 할 수 있음
    - Intercepting Router : 현재 페이지의 컨텍스트를 유지하면서 현재 레이아웃 내에서 경로를 로드 할 수 있음


### Rendering

* Next는 기본적으로 모든 경로의 페이지에 대한 HTML 파일을 사전 렌더링 함

**1. SSG (Static Site Generation)**
  * App Router
    - Server / Client Component 에 따라 다르게 동작
      - a) Server Component 는 렌더링 되어 HTML을 생성 : 관련 javascript 코드가 클라이언트에 전송되지 않는다
      - b) Client Component 는 HTML 및 JSON을 미리 렌더하고 서버에 캐싱된다 : 캐싱 결과는 클라이언트로 전송되어 Hydration 됨

  * Page Router
    - App Router 의 Client Component와 동일하게 동작

**2. ISR (Incremental Static Regeneration)**
  * App Router
    - 공식 문서에서 언급이 사라짐
  
  * Page Router
    - 주기적으로 정적페이지를 재생성 : getStaticProp의 반환값에 revalidate 필드를 추가하면 됨
  
**3. SSR (Server Side Rendering)**
  * App Router
    - 정적 렌더링 중에 동적기능/동적 fetch(), searchParams prop 등이 감지되면 해당 경로를 Dynamic Rendering 대상으로 판단
      - a) 동적기능 : cookies(), headers()
      - b) 동적 fetch() : no-store, revalidate

  * Page Router
    - 페이지에 접글할 때 마다 필요한 데이터를 가져오고 서버에서 렌더링 함
    - getServerSideProps 를 사용

### Data Fetching

**1. Method**
  * App Router
    - getServerSideProps, getStaticProps, getInitialProps와 같은 Method는 더 이상 사용하지 않는다
    - react server components 기반이기 때문에 일반적인 방법으로 서버 데이터를 가져옴
      - a) 서버 데이터베이스 리소스에 직접 접근 가능
      - b) 민감한 정보 클라이언트에 노출 X
      - c) 빌드시에 데이터 패칭이 이루어지고 캐싱됨
      - d) 성능이 향상
    - 클라이언트 측에서 데이터를 가져오는 것도 가능 함

  * Page Router
      - getServerSideProps : 페이지 접근할 때 마다 서버 사이드 렌더링에 필요한 데이터를 가져올 때 사용
                                    (최신 데이터가 필요할 경우에 사용)
      - getStaticProps : next build 시 정적 페이지를 생성 할 때 필요한 데이터를 가져올 때 사용
      - getInitialProps : 이미 서버에 있는 데이터를 이용해서 서버 사이드 렌더링을 할 때 사용

> **참고**  

* [[nextjs] 13.4.0부터 안정화된 App Router. Pages Router와 비교](https://velog.io/@jjunyjjuny/nextjs-13.4.0%EB%B6%80%ED%84%B0-%EC%95%88%EC%A0%95%ED%99%94%EB%90%9C-App-Router.-Pages-Router%EC%99%80-%EB%B9%84%EA%B5%90)

