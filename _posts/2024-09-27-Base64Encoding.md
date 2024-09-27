---
title: "Base64 Encoding"
last_modified_at: 2024-09-27
author: 김혜원
---

Base64 Encoding에 대한 포스팅입니다.

---

&nbsp;

## Base64 Encoding?

Base64 Encoding은 바이너리 데이터를 ASCII 코드를 이용하여 텍스트 형식으로 변환하는 방법입니다. 데이터를 문자열로 변환하여 네트워크 전송이나 데이터 저장 시 사용됩니다.
Base64는 알파벳 대소문자, 숫자, `+`, `/`, `=` 문자를 사용하여 데이터를 Encoding합니다.
Java에서는 여러 라이브러리를 사용하여 Base64 Encoding 및 Decoding을 쉽게 처리할 수 있습니다.


- 자바를 이용한 방법 3가지

    1. 자바 8 기본 라이브러리 (java.util.Base64)

    2. Apache Commons 라이브러리 (org.apache.commons.codec.binary.Base64)

    3. 자바 6이상 기본 라이브러리 (javax.xml.bind.DatatypeConverter)

&nbsp;

## Base64 Encoding/Decoding 예시

아래는 문자열을 Base64 인코딩한 후 복원하는 코드입니다.

```java
import java.util.Base64;

public class Base64Example {
    public static void main(String[] args) {
        String originalString = "It`s Base64";
        
        // Encoding
        String encodedString = Base64.getEncoder().encodeToString(originalString.getBytes());
        System.out.println("Encoded: " + encodedString);
        
        // Decoding
        byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
        String decodedString = new String(decodedBytes);
        System.out.println("Decoded: " + decodedString);
    }
}
```



    <출력 결과>

    Encoded: SXRgcyBCYXNlNjQ=
    Decoded: It`s Base64


&nbsp;



------
> 참고

[1] [[JAVA] Base64 인코딩, 디코딩 3가지 방법](https://veneas.tistory.com/entry/JAVA-Base64-%EC%9D%B8%EC%BD%94%EB%94%A9-%EB%94%94%EC%BD%94%EB%94%A9-3%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95)

