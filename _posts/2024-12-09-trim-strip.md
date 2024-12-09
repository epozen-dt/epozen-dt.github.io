---
title: "trim() vs strip()"
last_modified_at: 2024-12-09
author: 조평연
---

본 포스팅은 trim() vs strip() 을 알아보는 내용입니다.

로그파일에서 로그를 파싱하려고 정규표현식으로 파싱하는데  매치 부분에서 매치가 되지 않는걸
한참을 찾아보다 문자열 마지막에 커서를 맨앞으로 보내는 개행 (\r) 문이 들어가는걸 발견했습니다.

테스트 케이스에서는 문자를 직접 작성하다보니 문제가 없었는데
실제 서버에서 로그파일이 찍힐때는 해당 개행이 들어가는 것이였습니다.

개발을 할때 최종 데이터를 반환하고 그걸 찍어보고 확인해보는 작업도 필요하단걸 느꼈습니다.

# 1. trim() vs strip()
- trim() : 문자열에 양변에 있는 공백을 제거한 결과값을 반환
- strip() 은 JDK11에서 부터 지원하는 양변에 공백 + 유니코드 전반의 whitespace 문자까지 제거 가능

# 2. 예제
```java
public class StringTrimStripExample {
    public static void main(String[] args) {
        // 양변에 있는 공백문자 삭제
        // 아스키코드에 대해서는 모두 다 작동 잘함
        String str1 = "        \t\t\n\rhello        \t\t\n\r";
        System.out.println("Before: \"" + str1 + "\"");
        System.out.println("After trim: \"" + str1.trim() + "\"");
        System.out.println("After strip: \"" + str1.strip() + "\"");
        System.out.println();
 
        // 유니코드에 대해서는 trim() 작동하지 않음
        // 유니코드에 대해서는 strip()만 작동 (JDK11+)
        String str2 = '\u2001' + "String    with    space" + '\u2001';
        System.out.println("Before: \"" + str2 + "\"");
        System.out.println("After trim: \"" + str2.trim() + "\"");
        System.out.println("After strip: \"" + str2.strip() + "\"");
    }
}
```

# 3. 결과
```java
Before: "        		

hello        		

"
After trim: "hello"
After strip: "hello"

Before: " String    with    space "
After trim: " String    with    space "
After strip: "String    with    space"
```

# 4. 참고문헌
- https://docs.oracle.com/en/java/javase/17/