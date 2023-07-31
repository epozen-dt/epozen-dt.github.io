---
title: "네이버클라우드 API"
last_modified_at: 2023-07-31
author: Do-soo, KIM
---


이번 포스팅에서는 네이버 클라우드의 API에 대해서 소개하고자 합니다.

클라우드 서비스 제공을 위해서 네이버 클라우드 플랫폼을 이용하시는 분들이 많을텐데요.<br>
네이버 클라우드 플랫폼에서는 서비스와 솔루션을 효과적으로 이용할 수 있도록 API(Application Program Interface, 응용 프로그램 인터페이스)와 SDK(Software Development Kit)를 제공하고 있습니다.<br>
액션에 따라 파라미터 값을 입력하고 등록, 수정, 삭제, 조회 기능을 이용할 수 있으며 서비스와 운영 도구 자동화에도 활용할 수 있습니다.<br>
일반적으로 XML, JSON 형식으로 응답하는 API URL 형태로 크게 기본 API, 호환 API, 연동 API로 구분할 수 있습니다.

제공하는 카테고리는 Platform, Compute, COntainers 등 다양하지만 이 많은 카테고리에 대해서 다 설명하기는 어려워서 본 포스팅에서는 Platform 항목에 대해서만 다뤄 볼 겁니다.

Platform은 네이버 클라우드 플랫폼의 서비스 과금(Billing) 관련 API 레퍼런스, 리전정보 관련 API 레퍼런스와 관련된 카테고리 입니다.

이 포스팅에서는 Platform 카테고리 내의 서비스 과금 관련 API인 Billing API에 대해서 살펴보고 어떻게 사용하는지 살펴보도록 하겠습니다.


### 1. 개요

Billing API는 네이버 클라우드 플랫폼에서 제공하는 서비스의 사용량/이용금액을 조회하는 API로, Restful 형태로 제공되며 GET Method를 통해 XML 또는 JSON 포맷으로 응답하는 형태 입니다.

#### 1.1 공통 설정

- API URL

```
GET https://billingapi.apigw.ntruss.com/billing/v1/
```

- Request Header

| 헤더명 | 설명 |
| ------ | ----- |
| x-ncp-apigw-timestamp | 현재 시간의 timestamp 값 |
| x-ncp-iam-access-key | 네이버 클라우드 플랫폼에서 발급받은 API Key(네이버 클라우드 플랫폼 포털에서 발급 및 확인 가능) |
| x-ncp-apigw-signature-v2 | Request 메시지를 HmacSHA256 알고리즘을 통해 “Secret Key”로 암호화한 후, Base64로 인코딩한 값 |

x-ncp-apigw-signature-v2 값을 생성하는 방법은 네이버 클라우드 플랫폼 가이드에서 아래와 같이 언어별로 코드를 제공해 주고 있습니다.<br>
아래는 Java 코드입니다.

```java
public String makeSignature() {
	String space = " ";					// one space
	String newLine = "\n";					// new line
	String method = "GET";					// method
	String url = "/photos/puppy.jpg?query1=&query2";	// url (include query string)
	String timestamp = "{timestamp}";			// current timestamp (epoch)
	String accessKey = "{accessKey}";			// access key id (from portal or Sub Account)
	String secretKey = "{secretKey}";

	String message = new StringBuilder()
		.append(method)
		.append(space)
		.append(url)
		.append(newLine)
		.append(timestamp)
		.append(newLine)
		.append(accessKey)
		.toString();

	SecretKeySpec signingKey = new SecretKeySpec(secretKey.getBytes("UTF-8"), "HmacSHA256");
	Mac mac = Mac.getInstance("HmacSHA256");
	mac.init(signingKey);

	byte[] rawHmac = mac.doFinal(message.getBytes("UTF-8"));
	String encodeBase64String = Base64.encodeBase64String(rawHmac);

  return encodeBase64String;
}
```

#### 1.2 Operation

| API명 | URI | 설명 | 
| ------ | ----- | ----- |
| getContractSummaryList | /cost/getContractSummaryList | 사용자 계약 요약 리스트 조회 |
| getCostRelationCodeList | /cost/getCostRelationCodeList | 비용연관코드 리스트 조회 |
| getContractUsageList | /cost/getContractUsageList | 계약 사용량 리스트 조회 |
| getContractUsageListByDaily | /cost/getContractUsageListByDaily | 일별 계약 사용량 리스트 조회 |
| getContractDemandCostList | /cost/getContractDemandCostList | 계약 청구 비용 리스트 조회 |
| getProductDemandCostList | /cost/getProductDemandCostList | 상품 청구 비용 리스트 조회 |
| getDemandCostList | /cost/getDemandCostList | 청구 비용 리스트 조회 |

### 2. 사용 방법

이 포스팅에서는 여러 Operation 중, 청구비용 리스트를 조회하는 getDemandCostList API를 사용해 보도록 하겠습니다.<br>
2023년 7월에 이용한 서비스의 청구 비용을 조회해 보겠습니다.

요청 URL은 아래와 같이, 2023년 7월부터 2023년 7월까지 JOSN 포맷으로 응답을 요청합니다.

```
GET https://billingapi.apigw.ntruss.com/billing/v1/cost/getDemandCostList?startMonth=202307&endMonth=202307&responseFormatType=json
```

사용한 요청 파라미터는 아래와 같습니다.<br>
(다양한 파라미터가 있지만, 포스팅에서 사용한 파라미터만 소개한 겁니다.)


| 파라미터명 | 필수여부 | 타입 | 설명 | 
| ------ | ----- | ----- | ----- |
| startMonth | Yes | String |  조회 시작 월 (yyyyMM) |
| endMonth | Yes | String |  조회 마지막 월 (yyyyMM) |
| responseFormatType | No | String | 응답 결과의 포맷 타입<br>Otions: xml or json<br>Default: xml |


조회 결과는 아래 그림과 같습니다.

![image](https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/fe25e0d3-e259-446e-9603-b6a9781961ff)

제공하는 API를 다양하게 활용하면 클라우드 서비스를 효율적으로 관리하는 기능을 만들어 볼 수 있을 것 같습니다.
이번 포스팅은 여기서 마치겠습니다.

### Reference

https://api.ncloud-docs.com/docs/platform-costandusage
