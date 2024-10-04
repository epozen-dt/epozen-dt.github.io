---
title: "TanStack React Table 조사"
last_modified_at: 2024-09-30
author: Hwa-Yong, KANG
---

> 이번 포스팅에서는 TanStack React Table 에 대해서 조사하고, 사용 방법에 대해서 작성해 보도록 하겠습니다.

### TanStack React Table 조사

**1. TanStack Table 이란**
  * Headless UI 라이브러리
    - Headless UI : UI 엘리먼트와의 상호 작용을 위한 로직, 상태, 처리 API만들 제공하며, 그외에 어떠한 마크업이나 스타일, 선탑재 기능 등을 제공하지 않는 라이브러리를 의미
  * 설치 방법
    - npm install @tanstack/react-table
  * Column 정의
    ```typescript
    const columnHelper = createColumnHelper();
    const columns = [
      columnHelper.accessor("name", { header: "이름" }),
      columnHelper.accessor("phone", { header: "휴대폰" }),
      columnHelper.accessor("birth", { header: "생년월일" }),
      columnHelper.accessor("register_date", { header: "등록일" }),
      columnHelper.accessor("last_edit_date", { header: "최종수정일" }),
    ];
    ```
  * Column Defs : 각 컬럼과 그 컬럼의 데이터 모델, 화면 구성 등을 설정하는데 사용하는 객체
    - Column Defs는 Table Data가 테이블 안에서 어떤 식으로 그려질지를 담는 ‘컬럼 정의’ 객체
    - Sorting, Filtering, Grouping 등 기초 데이터 모델에 사용되며, 테이블 안에 들어가는 데이터 모델을 규격화
    - 헤더그룹, 헤더, 푸터를 생성할 수 있음
    - 단순 표시 목적의 컬럼 뿐 아니라 버튼, 체크박스, 확장 기능을 가진 컬럼을 정의
    - 컬럼은 다음의 세 가지 종류로 분류
      * Accessor Columns
        - 가공 가능한 데이터 모델을 표시하기 위한 컬럼
        - Sorting, Filtering, Grouping이 가능
      * Display Columns
        - 버튼이나 체크 박스 등 데이터와 상관없이 동일한 엘리먼트를 렌더링 하는 용도로 사용
        - Sorting, Filtering, Grouping이 불가능
      * Grouping Columns
        - 컬럼끼리 묶어서 표시하기 위한 컬럼으로 Header와 Footer에 주로 사용
        - Sorting, Filtering, Grouping이 불가능
  * Column Helpers : Column Defs 객체를 함수형으로 선언 할 수 있게 해줌
    - Column Helper를 통해 컬럼 설정값을 담아 호출하면 Column Defs 객체를 반환
    - 예시
    ```typescript
    const column = columnHelper.accessor("phone", 
      {
        header: "이름",
        cell: ({ renderValue }) =>
          renderValue().replace(/(\d{3})(\d{3,4})(\d{4})/, "$1-$2-$3")
      }
    )
    ```
  * Table
    - Table 생성 : 각종 state와 API를 모두 포함하는 코어 테이블 객체
    - useReactTable 훅을 이용하여 테이블 객체 생성
      - 테이블 객체는 useReactTable 훅을 통해 정의
    - 예시
    ```typescript
    import { getCoreRowModel, useReactTable } from "@tanstack/react-table";

    // 테이블에 표시할 데이터를 state로 정의
    const [data, setData] = useState([...defaultData]);

    // 각 columnHelper(혹은 Column Defs)들을 담은 배열
    const columns = [...]

    // useReactTable로 테이블 구조 정의
    const table = useReactTable({
      data,
      columns,
      getCoreRowModel: getCoreRowModel(),
    });
    ```
  * Table 출력 예시
    - getHeaderGroups와 getFooterGroups, getRowModel은 테이블 객체에 정의된 컬럼들을 출력할 때 사용할 배열을 반환
    - 예시
    ```typescript
    return (
      <table>
        <thead>
          {table.getHeaderGroups().map((headerGroup) => (
            <tr key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <th key={header.id}>
                  {header.isPlaceholder
                    ? null
                    : flexRender(
                      header.column.columnDef.header,
                      header.getContext()
                  )}
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody>
          {table.getRowModel().rows.map((row) => (
            <tr key={row.id}>
              {row.getVisibleCells().map((cell) => (
                <td key={cell.id}>
                  {flexRender(cell.column.columnDef.cell, cell.getContext())}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    )
    ```

> **참고**  
* [TanStack-React-Table-v8-기본편](https://velog.io/@kemezz/TanStack-React-Table-v8-%EA%B8%B0%EB%B3%B8%ED%8E%B8)

