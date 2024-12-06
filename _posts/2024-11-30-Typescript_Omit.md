---
title: "TypeScript"
last_modified_at: 2024-10-31
author: 권수연
---

### 📝 **TypeScript 유틸리티 타입과 활용**

TypeScript는 **유틸리티 타입(Utility Types)**을 제공하여 기존 타입을 변형하거나 새로운 타입을 쉽게 생성할 수 있도록 지원합니다. 특히, `Omit`, `Pick`, `Partial` 같은 타입은 복잡한 객체 지향 프로그래밍과 프론트엔드 상태 관리 라이브러리에서 매우 유용하게 쓰입니다.

<br>

---

<br>

## 🌟 **`Omit` 타입 개요**

`Omit`은 기존 객체 타입에서 **특정 속성을 제외한 타입**을 생성합니다. 불필요한 속성을 제거하거나 일부 속성을 숨겨야 할 때 유용합니다.

```tsx
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

### **사용법**

- `T`: 변형할 원본 타입.
- `K`: 제외할 속성의 키(Key)들.

`Omit`은 내부적으로 `Pick`과 `Exclude`를 사용하여, 지정된 속성을 제외한 새로운 타입을 생성합니다.

---

<br>

### 📜 **`Omit` 사용 예제**

### 기본 예제

```tsx
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Omit<User, "password">;

const user: PublicUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
};
```

여기서 `PublicUser`는 `User` 타입에서 `password` 속성만 제외한 타입으로 만들어집니다. 이 방식은 **민감한 정보나 필요하지 않은 속성을 제외하는 데** 매우 유용합니다.

---

<br>

## 🛠 **TypeScript 유틸리티 타입 확장**

`Omit` 외에도 다양한 유틸리티 타입들이 있습니다. 이들을 조합하거나 단독으로 사용하면 더욱 강력한 타입 시스템을 구축할 수 있습니다.

---

<br>

### 1. **`Pick` 타입**

`Pick`은 기존 객체 타입에서 **특정 속성만 선택**하여 새로운 타입을 생성합니다.

```tsx
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

### 사용 예제

```tsx
type UserBasicInfo = Pick<User, "id" | "name">;

const userBasic: UserBasicInfo = {
  id: 1,
  name: "Alice",
};
```

`Pick`은 UI에 특정 데이터만 필요한 경우나 **요청/응답의 데이터 형태를 줄이고자 할 때** 유용합니다.

---

<br>

### 2. **`Partial` 타입**

`Partial`은 모든 속성을 **선택적(optional)**으로 만드는 타입입니다. 주로 업데이트용 객체를 정의할 때 유용합니다.

```tsx
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

### 사용 예제

```tsx
type UpdateUser = Partial<User>;

const updateUser: UpdateUser = {
  email: "new_email@example.com",
};
```

`Partial`은 특정 속성만 업데이트가 필요한 경우에 활용됩니다.

---

<br>

### 3. **`Required` 타입**

`Required`는 모든 속성을 **필수(non-optional)**로 만드는 타입입니다. 주로 선택적 속성을 강제할 때 사용됩니다.

```tsx
type Required<T> = {
  [P in keyof T]-?: T[P];
};
```

### 사용 예제

```tsx
type FullUser = Required<Partial<User>>;

const fullUser: FullUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  password: "secure",
};
```

---

### 4. **`Readonly` 타입**

`Readonly`는 모든 속성을 읽기 전용으로 만들어, 객체를 불변(Immutable) 상태로 유지할 수 있습니다.

```tsx
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

### 사용 예제

```tsx
type ReadonlyUser = Readonly<User>;

const readonlyUser: ReadonlyUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  password: "secret",
};

// readonlyUser.name = "Bob"; // Error: 읽기 전용 속성입니다.
```

---

### 5. **`Record` 타입**

`Record`는 키(Key)와 값(Value)의 타입을 지정하여 객체 타입을 생성합니다.

```tsx
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

### 사용 예제

```tsx
type PostStatus = "draft" | "published" | "archived";

type PostDirectory = Record<PostStatus, string>;

const postDirectory: PostDirectory = {
  draft: "/drafts",
  published: "/published",
  archived: "/archives",
};
```

`Record`는 **동적으로 키를 지정해야 하는 경우**에 적합합니다.

---

### 6. **`Exclude` 타입**

`Exclude`는 두 타입에서 **제외할 타입**을 명시하여 새로운 타입을 생성합니다.

```tsx
type Exclude<T, U> = T extends U ? never : T;
```

### 사용 예제

```tsx
type AllActions = "create" | "update" | "delete" | "view";
type AdminActions = Exclude<AllActions, "view">;

const action: AdminActions = "create"; // OK
```

---

### 7. **`Extract` 타입**

`Extract`는 두 타입 간 **공통 타입**을 추출합니다.

```tsx
type Extract<T, U> = T extends U ? T : never;
```

### 사용 예제

```tsx
type CommonActions = Extract<AllActions, "create" | "view">;

const action: CommonActions = "create"; // OK
```

---

### 8. **`NonNullable` 타입**

`NonNullable`는 `null`과 `undefined`를 제거한 타입을 생성합니다.

```tsx
type NonNullable<T> = T extends null | undefined ? never : T;
```

### 사용 예제

```tsx
type Name = string | null | undefined;
type DefinedName = NonNullable<Name>;

const name: DefinedName = "Alice"; // OK
```

---

## 🔄 **유틸리티 타입 조합**

TypeScript 유틸리티 타입은 조합하여 더욱 강력한 타입을 만들 수 있습니다.

### 예제: API 응답 처리

```tsx
type ApiResponse<T> = Partial<Readonly<T>>;

type UserResponse = ApiResponse<Omit<User, "password">>;

const response: UserResponse = {
  id: 1,
  name: "Alice",
};
```

이 코드는 API 응답에서 비밀번호를 제외하고 읽기 전용 및 선택적 속성을 추가합니다.

---

## 📚 **결론**

TypeScript의 유틸리티 타입은 복잡한 타입 정의를 간결하게 만들고, 반복되는 코드 작성의 부담을 줄여줍니다. `Omit`, `Pick`, `Partial` 같은 타입을 적절히 활용하면 **유연하고 유지보수 가능한 코드 작성**이 가능하며, 더욱 강력한 타입 시스템을 구축할 수 있습니다.

> **참고**

- [블로그](https://velog.io/@productuidev/%EB%94%94%EB%B2%84%EA%B9%85-%EC%9E%98-%ED%95%98%EA%B8%B0-feat.-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%8F%84%EA%B5%AC-debugger)

---
