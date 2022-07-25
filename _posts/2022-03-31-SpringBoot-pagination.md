---
title: "SpringBoot - 페이징 처리 구현하기"
last_modified_at: 2022-03-31
author: 김수민
---

이번 포스팅에서는 SpringBoot를 사용한 화면에서 목록을 보여줄 때 페이징 처리하는 방법에 대해 알아보겠습니다.

---

게시판 등 목록을 보여줄때, 우리는 모든 목록 속 데이터를 무한하게 스크롤을 내리도록 화면을 작성할 수 없습니다. 따라서 일정 갯수 만큼 잘라 페이지를 나누어 출력하게 됩니다.

지금부터 MVC 구조의 페이징 처리 구현 과정을 데이터 구조부터 화면 출력까지 진행해 보겠습니다.

PostgreSQL 기반의 쿼리문을 사용했으며, 화면은 jsp대신 html에서 thymleaf를 사용한 예제입니다.

---

## 1. 데이터 구조

공통적으로 페이징을 처리하기 위해서는 다음의 값들을 필요로 합니다.

- 현재 페이지 번호
- 페이지 당 출력할 데이터 수
- 화면 하단에 표시할 페이지 리스트



또한 이 값들을 사용해서 총 페이지를 계산하고 이후 화면에서 이전/다음 페이지로 이동하는 버튼에 사용할 값들을 계산해 전달하기 위한 다음의 항목들이 필요합니다.

- 전체 데이터 수
- 전체 페이지 수
- 페이지 리스트의 첫 번호
- 페이지 리스트의 마지막 번호
- db에서 사용할 index
- 이전 페이지 존재 여부
- 다음 페이지 존재 여부

전체 항목에 대해 하나의 객체로 만드는 것은, 페이징 처리 시 요청되는 파라미터에 비해 너무 큰 객체를 생성하기 때문에 따로 분리하여 각각 Criteria.java / PaginationInfo.java로 나누어 생성하도록 하겠습니다.

Criteria에서는 현재 페이지를 1, 페이지 당 데이터 수는 10, 페이지 표출 목록 사이즈는 10을 기본으로 잡고 진행할 예정이므로, 생성자를 사용해 다음과 같이 구현합니다.

```java
public static class Criteria {

        private int prsntPageNo;    // 현 페이지 번호
        private int cntntPerPage;   // 페이지당 출력할 데이터 갯수
        private int pageSize;       // 하단에 출력할 페이지 사이즈

        public Criteria(){
            this.prsntPageNo = 1;
            this.cntntPerPage = 10;
            this.pageSize = 10;
        }
 /** 생략 **/  
}
```

Pagination에서는 Criteria를 필수로 입력받아 내부 항목들에 대해 계산해 진행할 것이므로 다음과 같이 생성자를 사용해 입력받도록 합니다.

```java
public static class PaginationInfo {

        private Criteria criteria;

        private int totCnt;             // 전체 데이터 개수
        private int totPageCnt;         // 전체 페이지 개수
        private int firstPageNo;        // 페이지 리스트 첫 번호
        private int lastPageNo;         // 페이지 리스트 마지막 번호
        private int firstIdx;           // db 조건절 시작 index
        private boolean prev;           // 이전 페이지 존재 여부
        private boolean next;           // 다음 페이지 존재 여부

        public PaginationInfo(Criteria criteria){
            if(criteria.getPrsntPageNo()<1){
                criteria.setPrsntPageNo(1);
            }
            this.criteria = criteria;
        }
/** 생략 **/
}
```

Pagination에서의 가장 큰 기능은, 전체 데이터 수와 Criteria를 입력받으면 나머지 데이터들의 값을 설정해 주는 것입니다.

이에 전체 데이터 수가 입력되면 자동적으로 계산 할 수 있도록, setTotCnt()에서 paginate()를 호출해 값들을 셋팅할 수 있도록 구현합니다.

```java
public void setTotCnt(int totCnt){
    this.totCnt = totCnt;

    if(totCnt > 0){
        paginate();
    }
}
```

이제 페이징 계산 함수 내부의 내용을 살펴보겠습니다.

1. 전체 페이지 수 계산

```java
/* 현재 페이지 번호가 전체 페이지 수보다 크면 현재 페이지 번호에 전체 페이지 수를 저장 */
totPageCnt = ((totCnt - 1) / criteria.getCntntPerPage()) + 1;
if (criteria.getPrsntPageNo() > totPageCnt) {
    criteria.setPrsntPageNo(totPageCnt);
}
```

2. 페이지 리스트 첫 페이지/마지막 페이지 번호

```java
firstPageNo = ((criteria.getPrsntPageNo() - 1) / criteria.getPageSize()) * criteria.getPageSize() + 1;

lastPageNo = firstPageNo + criteria.getPageSize() - 1;
            if (lastPageNo > totPageCnt) {
                lastPageNo = totPageCnt;
            }
```

3. 쿼리문에서 사용할 값

```java
firstIdx = (criteria.getPrsntPageNo() - 1) * criteria.getCntntPerPage();
```

4. 이전/다음 페이지 존재 확인

```java
prev = firstPageNo != 1;
next = (lastPageNo * criteria.getCntntPerPage()) < totCnt;
```

---

## 2. DAO / Mapper(xml)

우리가 페이징을 하기 위해 필요한 쿼리문은 2가지 입니다.

첫 번째, 전체 데이터 수를 가져오는 쿼리입니다.

sql.Session.selectOne("xml파일명.id명")을 사용해 mybatis의 mapper에서 데이터를 가져옵니다.

해당 쿼리는 다음과 같습니다.

```sql
<select id="getTotCnt" resultType="int">
    SELECT COUNT(*)
    FROM DB명
</select>
```



두 번째로는 목록을 불러오는데 페이지에 표출될 범위로만 가져오는 쿼리문 입니다.

```sql
   <select id="getList" resultType="AlrtHstVO">
       <![CDATA[
      SELECT 목록 내용,
      FROM DB명
      WHERE 조건절
      LIMIT #{cntntPerPage} OFFSET #{startPage}
]]>
   </select>
```

여기서 LIMIT 부분을 추가해 목록 페이징을 수행합니다.

LIMIT문은 LIMIT '페이지 표출 데이터 수' OFFSET '시작 index' 입니다.

여기서의 startPage는 Criteria에서 getStartPage()를 선언해두어 startpage값을 가져올 수 있습니다.

해당 함수를 적용하는 이유는, db에서의 인덱싱은 0부터 시작하는데 반해, 현재 페이지 번호는 1부터 시작하기 때문에 -1씩 수행하는 계산 과정이 필요하기 때문입니다.

```java
public int getStartPage(){
    return (prsntPageNo - 1) * cntntPerPage;
}
```

---

## 3. Controller / Service

Controller은 화면 주소를 입력받으면 데이터를 추가해 화면으로 연결해주는 클래스입니다.

따라서 데이터 처리에 관한 내용은 Service에서 수행하게 됩니다.

Service의 내용을 보면, 다음과 같이 전체 데이터 수를 getTotCnt를 사용해 가져오고,

해당 값을 PaginationInfo에 주어 페이징 처리 객체를 생성해 셋팅하여 검색 결과를 return해 줍니다.

```java
@Override
    public List<목록결과VO> selectList(Common params) throws Exception {
        int totCnt = alrtHstDao.getTotCnt();    // 전체 페이지 계산을 위한 count 값

        Pagination.PaginationInfo paginationInfo = new Pagination.PaginationInfo(params);   // 페이징 처리를 위한 객체
        paginationInfo.setTotCnt(totCnt);

        params.setPaginationInfo(paginationInfo);
        return 목록출력DAO.selectList(params);   // 전체 목록 return
    }
```

그러면 이제 Controller에서 해당 주소가 호출되면 Service를 불러와 데이터를 가져오고, 화면에 전송해 줍니다.

```java
@RequestMapping(value="/lst")
public String getList(@ModelAttribute("params") Common params, Model model) throws Exception {
    List<AlrtHstVO> alrtList = Service명.selectList(params);
    model.addAttribute("demoboardList", alrtList);
    return "화면명";
}
```

---

## 4. HTML

마지막으로 화면 구현 모습입니다.

html에서 thymleaf를 사용해 값들을 제어합니다.

![image-20220311174044870](C:\Users\epozen\AppData\Roaming\Typora\typora-user-images\image-20220311174044870.png)

다음과 같이 현재 페이지를 표시하고, 첫 페이지로, 마지막 페이지로, 이전페이지, 다음 페이지로 이동하기 위해 구현합니다.

우선 1~10까지 목록의 크기만큼 페이지 배열을 만들기 위한 부분을 보며 사용된 타임리프에 대해 설명하겠습니다.입니다.

```html
<!-- 페이징 -->
<div class="paging_area"  th:if="${params != null and params.paginationInfo.totCnt > 0}" th:object="${params.paginationInfo}" th:with="info=${params.paginationInfo}">
    <div class="paginate">
        <a th:each="pageNo : *{#numbers.sequence( firstPageNo, lastPageNo )}" th:class="${pageNo == params.prsntPageNo} ? 'on'" href="javascript:void(0)" th:onclick="movePage([[ ${#request.requestURI} ]], [[ ${params.setPageInfo(pageNo)} ]])">
          [[${pageNo}]]
        </a>
    </div>
</div>
<!--// 페이징 -->
```

th:if문은 if문을 사용하기 위한 내용입니다. 파라미터가 있고 총 데이터 수가 0보다 큰 경우 해당 모습이 보이도록 구현되어 있습니다.

우리가 눈여겨 봐야 할 부분은 th:each 부분 입니다.

th:each는 반복문을 수행합니다. (첫 페이지 번호, 마지막 페이지 번호) 까지의 값들을 숫자로 pageNo라는 변수에 넣는다는 이야기 입니다.

[[${pageNo}]]를 a태그 값으로 넣어 해당 번호들로 a태그를 만드는 것 입니다.

해당 a태그를 클릭하게 되면 th:onclick으로 movePage(uri, queryString)이라고 선언한 자바스크립트로 넘어갑니다.

해당 내용은 uri와 queryString을 합쳐 url을 만들어 이동하도록 하는 것, 즉 누른 페이지로 이동하도록 구현되어 있습니다.

th:class 내용은 현재 페이지 번호와 반복문으로 표출되는 pageNo들 중 같은 값에 on표시를 해주도록 하는 것 입니다.



다음은 버튼들을 모두 구현한 페이징 부분 코드 입니다.

맨앞, 이전 버튼은 표현되어 있는 페이지 목록 앞의 번호가 있는 경우 외에는 blind 처리를,

맨뒤, 다음 버튼은 표현되어 있는 페이지 목록 뒤의 번호가 있는 경우 외에는 blind 처리를 하도록 구현되어 있습니다.

또한 각각의 버튼들은 누르면 다음 페이지 목록들(1~10에서 누른 경우 11~20이 표현되도록), 이전 페이지 목록들을 보일 수 있도록 firstPageNo를 조작하도록 했습니다.

자세한 타임리프 내용들은 생략하도록 하겠습니다.

```html
<!-- 페이징 -->
<div class="paging_area"  th:if="${params != null and params.paginationInfo.totCnt > 0}" th:object="${params.paginationInfo}" th:with="info=${params.paginationInfo}">
    <div class="paginate">
        <!-- 맨앞, 이전 버튼 -->
        <a href="javascript:void(0)" th:if="*{prev==true}" th:onclick="movePage([[ ${#request.requestURI} ]], [[ ${params.setPageInfo(1)} ]])" title="맨앞">
            <i class="i_first"><span class="blind">맨앞</span></i>
        </a>
        <a href="javascript:void(0)" th:if="*{prev == true}" th:onclick="movePage([[ ${#request.requestURI} ]], [[ ${params.setPageInfo(info.firstPageNo - 1)} ]])" title="이전">
            <i class="i_prev"><span class="blind">이전</span></i>
        </a>
        <a th:each="pageNo : *{#numbers.sequence( firstPageNo, lastPageNo )}" th:class="${pageNo == params.prsntPageNo} ? 'on'" href="javascript:void(0)" th:onclick="movePage([[ ${#request.requestURI} ]], [[ ${params.setPageInfo(pageNo)} ]])">
          [[${pageNo}]]
        </a>
        <!-- 다음, 맨뒤 버튼 -->
        <a href="javascript:void(0)" th:if="*{next == true}" th:onclick="movePage([[ ${#request.requestURI} ]], [[ ${params.setPageInfo(info.lastPageNo + 1)} ]])" title="다음">
            <i class="i_next"><span class="blind">다음</span></i>
        </a>
        <a href="javascript:void(0)" th:if="*{next == true}" th:onclick="movePage([[ ${#request.requestURI} ]], [[ ${params.setPageInfo(info.totPageCnt)} ]])" title="맨뒤">
            <i class="i_last"><span class="blind">맨뒤</span></i>
        </a>
    </div>
</div>
<!--// 페이징 -->
```

---

이번 시간에는 MVC 구조로 페이징 처리하는 방법에 대해 알아보았습니다.

