---
title: "카카오톡 메시지 REST API 사용법"
last_modified_at: 2023-11-16
author: 이현지
---

이번 포스트에선 카카오톡 메시지 REST API를 이용한 나에게 보내기 기능을 구현해보겠습니다.

<br><br>
## 1. 카카오톡 계정으로 Kakao Developers 가입

기존 카카오톡 계정으로 Kakao Developers 에 가입합니다.<br>
**https://developers.kakao.com/**

<br><br>
## 2. 애플리케이션 등록 

Kakao Developers 메인 화면 우측 상단 - [내 애플리케이션] - [애플리케이션 추가하기] - 앱 정보 입력 후 [저장]

![애플리케이션 추가 1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQZ93N%2FbtsAm1Xgcyw%2Fi3ZVKezBSVQqYU5ky9B1C1%2Fimg.png)
<br><br>
![애플리케이션 추가 2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcFAGMa%2FbtsAiDi9R3Z%2FYLwbBMK7XzYeoAAkKBm4sk%2Fimg.png)

<br><br>
## 3. 플랫폼 등록

생성된 애플리케이션을 클릭하면 애플리케이션 설정 화면이 표시됩니다.

우측 [요약정보] 탭 - 페이지 하단 [플랫폼 설정하기] - [Web 플랫폼 등록] - 클라이언트 도메인 주소 입력 - [저장]

![풀럇폼 등록 1](https://blog.kakaocdn.net/dn/dvLZE0/btsAfB0cnXm/XNcq7fa8MfUS8MD8bs4pk0/img.png)
<br><br>
![플랫폼 등록 2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FJqqNa%2FbtsArlPaStG%2FVmJNPiFF18hy3YS0oEm6v0%2Fimg.png)

<br><br>
## 4. 카카오 로그인 활성화 및 Redirect URI 입력

우측 [카카오 로그인] 탭 - '활성화 설정' -  'ON', 'Redirect URI' 입력

Redirect URI 는 카카오 로그인 시 리다이렉트 되어 인증 코드를 받을 URI를 입력합니다. 

![카카오 로그인 활성화](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBAOD0%2FbtsAkl92Awz%2F1kekH8LzMPQtqKpmpSusa1%2Fimg.png)

<br><br> 
## 5. 카카오톡 메시지 전송 동의 

우측 [동의 항목] 탭 - '접근권한' - '카카오톡 메시지 전송' - [설정] - '동의 단계' 선택 및 '동의 목적' 입력 - [저장]

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcCnKex%2FbtsAmB5J1Tf%2FFJayZDDCNVRY3PfnPClgH0%2Fimg.png)

<br><br>
## 6. Client Secret 생성 

[보안] - [코드 생성] 클릭 시 코드 발급 - '활성화 상태' - '사용함'으로 변경

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqjdPN%2FbtsAm1J5kKl%2FhF4ZhKsbATaJJDJPwcZm1K%2Fimg.png)

<br><br>
## 7. 코드 작성

### 인증 코드 받기
**REST API 키** : [내 애플리케이션] - [앱설정] - [요약 정보] 탭에서 확인 가능<br>
**Client Secret 보안 키** : [내 애플리케이션] - [앱설정] - [보안] 탭에서 확인 가능 <br>
**redirect URL** : [내 애플리케이션] - [앱설정] - [카카오 로그인] 탭에서 확인 가능 <br>
```java
@Service
public class Authentication {
		
	// 인증 URL
	private static final String AUTH_URL = "https://kauth.kakao.com/oauth/token";
	
	public static String accessToken; // 액세스 토큰
	public static String refreshToken; // 리프레시 토큰
	
	public void getToken(String code)  {
		
		// 인증 요청 헤더 설정
		HttpHeaders header = new HttpHeaders();
		header.set("Content-Type", "application/x-www-form-urlencoded;charset=UTF-8");

		// 인증 및 리다이렉트 정보 
		MultiValueMap<String, String> parameters = new LinkedMultiValueMap<>();
        parameters.add("code", code);
		parameters.add("grant_type", "authorization_code");
		parameters.add("client_id", "REST API 키 입력");
		parameters.add("client_secret", "Client Secret 보안 키 입력");
		parameters.add("redirect_url", "리다이렉트 URL 입력");

		// 인증 HTTP 요청 객체 생성
		HttpEntity<?> requestEntity = new HttpEntity<>(parameters, header);
		// 인증 HTTP 요청
		ResponseEntity<String> response = 
				new RestTemplate().exchange(AUTH_URL, HttpMethod.POST, requestEntity, String.class);
		
		// 응답 
		JSONObject jsonData = new JSONObject(response.getBody());

        accessToken = jsonData.get("access_token").toString();
        refreshToken = jsonData.get("refresh_token").toString();
	}

}
```
<br><br>
### 메시지 전송
```java
@Service
public class Message {
	
	// 기본 템플릿 메시지 보내기 URL
	private static final String MSG_SEND_SERVICE_URL = "https://kapi.kakao.com/v2/api/talk/memo/default/send"; 

	public void sendMessage() {		
		// 액세스 토큰
		String accessToken = Authentication.accessToken;
		// 헤더 설정
		HttpHeaders header = new HttpHeaders();
		header.set("Content-Type", "application/x-www-form-urlencoded;charset=UTF-8");
		header.set("Authorization", "Bearer " + accessToken);

		// (기본) 템플릿 구성
		MultiValueMap<String, String> parameters = new LinkedMultiValueMap<>();	
		JSONObject templateObj = new JSONObject(); 
		templateObj.put("object_type", "text");
		templateObj.put("text", "메시지 내용 입니다.");		
		templateObj.put("button_title", "자세히보기");
		JSONObject linkObj = new JSONObject();
		linkObj.put("web_url", "web url입니다.");
		linkObj.put("mobile_web_url", "mobile web url 입니다.");
		templateObj.put("link", linkObj);
		parameters.add("template_object", templateObj.toString());
		
		// 메시지 전송 HTTP 요청 객체 생성
		HttpEntity<?> messageRequestEntity = new HttpEntity<>(parameters, header);
		// 메시지 전송 HTTP 요청 
		ResponseEntity<String> response = 
                    new RestTemplate().exchange(MSG_SEND_SERVICE_URL, HttpMethod.POST, messageRequestEntity, String.class);
		
		// 응답
		JSONObject jsonData = new JSONObject(response.getBody());

        String resultCode = jsonData.get("result_code").toString();
		if (resultCode.equals("0")) { 
			System.out.println("메시지 전송 성공");
		} else {
			System.out.println("메시지 전송 실패");
		}
	}

}
```
<br><br>
### 컨트롤러
```java
@RestController
public class Controller {

	@Autowired
	Authentication authentication;

	@Autowired
	Message message; 

	@GetMapping("/") // 리다이렉트 주소 (http://localhost:8080)
	public void authAndSendMsg(String code) {		
        authentication.getToken(code); // 인증
        message.sendMessage();  // 메시지 전송
		return "메시지 전송 성공";
	}
	
}
```
<br><br>
## 8. 브라우저를 통한 수동 로그인

애플리케이션 실행 후 브라우저를 통해 다음 URL로 접속하여 카카오톡 계정으로 로그인합니다.<br>
**https://kauth.kakao.com/oauth/authorize?Client_id={REST API 키}?redirect_uri={REDIRECT URI}&response_type=code&scope=talk_message**

<br><br>
![로그인](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb5qjBx%2FbtsAmqcJ7y9%2F2y5QbIJgYyRuTF9irnq7UK%2Fimg.png)
<br><br>
[계속하기] 를 클릭하면 리다이렉트 URL(http://localhost:8080)으로 넘어가며 인증코드(code 파라미터)가 발급되고 메시지 전송 작업이 실행됩니다.
<br><br>
![리다이렉트 화면](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FroX8H%2FbtsAjTzwrG9%2FbzGkPJ13pzXfk8pMOxlpW0%2Fimg.png)

<br><br>
## 9. 전송된 메시지

![전송된 메시지](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcaAni9%2FbtsArcK0PY9%2FhHz9XdbmRB6WmTbkekN2jK%2Fimg.png)

<br><br>
이상으로 카카오톡 메시지 REST API 사용법 포스팅을 마치겠습니다.

<br><br>
 > Reference 
- https://developers.kakao.com/docs/latest/ko/message/rest-api<br>
- https://foot-develop.tistory.com/21<br>
