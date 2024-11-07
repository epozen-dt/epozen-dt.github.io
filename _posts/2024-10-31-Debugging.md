---
title: "Debugging"
last_modified_at: 2024-10-31
author: 권수연
---

# **Debugging의 개요**

**Debugging** 은 코드에서 발생하는 버그나 문제를 찾고 해결하는 과정으로, 개발자가 문제의 원인을 분석하고 수정할 수 있도록 도와주는 필수적인 기술입니다. 프론트엔드 개발에서는 브라우저 환경, 다양한 디바이스, 네트워크 요청, 비동기 코드 등 다양한 요소가 문제를 야기할 수 있어 디버깅 기술이 특히 중요합니다.

---

## **디버깅 기본 전략**

- **문제 재현**: 문제가 발생한 상황을 명확하게 재현하는 것이 중요합니다. 특정 브라우저에서만 발생하거나, 특정 사용자 시나리오에서만 발생하는 경우가 있을 수 있으므로 상황을 구체화합니다.
- **원인 추적**: 문제의 원인을 찾기 위해 범위를 좁혀나갑니다. 최근 수정한 코드, 연관된 모듈, 네트워크 요청 등을 하나씩 검토하며 문제가 발생하는 지점을 파악합니다.
- **가설 검증**: 문제 원인에 대한 가설을 세우고, 이를 검증하는 과정을 통해 실제 원인을 찾아냅니다. 필요할 경우 코드에 로그를 추가하거나 브레이크포인트를 설정해 실행 흐름을 추적합니다.

---

### 1. **기본 디버깅 전략과 코드 예시**

- **문제 재현**: 동일한 문제를 반복해서 재현할 수 있는 환경을 설정합니다.
- **가설 세우기**: 코드의 어느 부분이 문제를 일으켰는지 가설을 세우고 테스트합니다.
- **문제 해결 후 리팩토링**: 버그를 수정한 뒤, 더 나은 코드로 개선할 수 있는 부분이 있는지 검토합니다.

**예시**: 배열의 값을 두 배로 만드는 함수에 문제가 발생한 경우

```javascript
function doubleArrayValues(arr) {
  return arr.map((num) => num * 2);
}

// 테스트 시도
const result = doubleArrayValues([1, 2, "3", 4]);
console.log(result); // 예상: [2, 4, 6, 8], 실제: [2, 4, NaN, 8]
```

문제는 `'3'`이 문자열로 전달되어 `NaN`이 반환되는 것입니다. 이를 수정하려면 숫자 여부를 검사하고 변환하는 코드를 추가해야 합니다:

```javascript
function doubleArrayValues(arr) {
  return arr.map((num) =>
    typeof num === "number" ? num * 2 : parseInt(num) * 2
  );
}
```

---

### 2. **브라우저 개발자 도구 활용**

- **Console 로그 디버깅**: 실행 흐름을 따라가며 `console.log`를 사용해 상태를 출력합니다.

**예시**: 네트워크 요청의 응답을 확인하는 코드

```javascript
fetch("/api/data")
  .then((response) => response.json())
  .then((data) => {
    console.log("API 응답 데이터:", data); // 로그로 데이터 확인
  })
  .catch((error) => {
    console.error("API 요청 실패:", error);
  });
```

- **브레이크포인트 디버깅**: `Sources` 패널에서 코드 라인에 브레이크포인트를 설정하고 변수의 값을 실시간으로 확인합니다.

---

### 3. **네트워크 패널로 API 요청 검사**

- **API 요청이 실패하는 경우 상태 코드와 에러 메시지 확인**

**예시**: 네트워크 요청 실패 시 에러 메시지를 사용자에게 보여주는 코드

```javascript
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP 오류: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("데이터 로드 실패:", error);
    alert("데이터를 불러오는 중 오류가 발생했습니다. 다시 시도해 주세요.");
  }
}
```

---

### 4. **비동기 코드 디버깅**

- **Promise 체인의 에러 처리**: `.catch()`로 에러를 처리하고, `finally`를 사용해 리소스를 정리합니다.

**예시**: 파일 업로드 시 에러가 발생하는 경우의 디버깅 코드

```javascript
async function uploadFile(file) {
  try {
    const formData = new FormData();
    formData.append("file", file);

    const response = await fetch("/api/upload", {
      method: "POST",
      body: formData,
    });

    if (!response.ok) {
      throw new Error(`업로드 실패: ${response.statusText}`);
    }

    const result = await response.json();
    console.log("파일 업로드 성공:", result);
  } catch (error) {
    console.error("파일 업로드 중 오류:", error);
  } finally {
    console.log("업로드 시도 완료");
  }
}
```

---

### 5. **성능 디버깅**

- **렌더링 성능 최적화**: 불필요한 컴포넌트 리렌더링을 피하기 위해 React의 `useMemo`, `useCallback` 등의 Hook을 사용합니다.

**예시**: `useMemo`를 사용해 계산 비용이 큰 작업의 결과를 메모이제이션

```javascript
import React, { useMemo } from "react";

const ExpensiveComponent = ({ data }) => {
  const processedData = useMemo(() => {
    // 계산 비용이 큰 작업
    return data.map((item) => item.value * 2);
  }, [data]);

  return (
    <div>
      {processedData.map((value, index) => (
        <p key={index}>{value}</p>
      ))}
    </div>
  );
};
```

- **네트워크 최적화**: `lazy loading`이나 `code splitting`을 통해 불필요한 리소스 로드를 방지합니다.

**예시**: React의 `React.lazy`와 `Suspense`를 사용한 코드 스플리팅

```javascript
import React, { Suspense } from "react";

const LazyComponent = React.lazy(() => import("./SomeComponent"));

const App = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <LazyComponent />
  </Suspense>
);
```

---

이와 같은 코드를 활용해 디버깅 작업을 효율적으로 수행할 수 있습니다.

> **참고**

- [블로그](https://velog.io/@productuidev/%EB%94%94%EB%B2%84%EA%B9%85-%EC%9E%98-%ED%95%98%EA%B8%B0-feat.-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%8F%84%EA%B5%AC-debugger)

---
