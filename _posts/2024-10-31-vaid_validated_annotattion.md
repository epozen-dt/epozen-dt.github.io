---
title: "@Valid, @Validated를 이용한 데이터 유효성 검사"
last_modified_at: 2024-10-31
author: 이현지
---

<br>

이번 포스트에서는 Spring Framework에서 데이터 유효성 검사 수행 시 사용되는 @Valid, @Validate 에 대해 알아보겠습니다.

<br>

# @Valid
- 자바 표준 스펙 (Bean Validation 사양(JRS-303) 에 정의됨)
- 자바의 표준 유효성 검사 기능을 활용하여 객체의 필드에 대한 유효성을 검증
- 주로 RequestBody를 검증할 때 사용함
- 검증 오류 시 MethodArgumentNotValidException 을 발생시킴

<br>

### 유효성 검사 어노테이션
#### @NotNull
필드가 null이 아닌지 검사
<br>
예: @NotNull(message = "이름은 필수입니다.") private String name;

#### @NotEmpty
문자열이 null이 아니고 비어 있지 않은지 검사합니다.
<br>
예: @NotEmpty(message = "이메일은 필수입니다.") private String email;

#### @NotBlank
문자열이 null이 아니고 공백이 아닌지 검사합니다.
<br>
예: @NotBlank(message = "비밀번호는 필수입니다.") private String password;

#### @Size
문자열, 배열, 컬렉션의 크기를 제한합니다.
<br>
예: @Size(min = 3, max = 15, message = "이름은 3자 이상 15자 이하입니다.") private String name;

#### @Min
숫자가 지정된 최소값 이상인지 검사합니다.
<br>
예: @Min(value = 18, message = "최소 나이는 18세입니다.") private int age;

#### @Max
숫자가 지정된 최대값 이하인지 검사합니다.
<br>
예: @Max(value = 100, message = "최대 나이는 100세입니다.") private int age;

#### @Email
유효한 이메일 주소 형식인지 검사합니다.
<br>
예: @Email(message = "유효한 이메일 주소를 입력하세요.") private String email;

#### @Pattern
정규 표현식에 따라 문자열이 특정 형식인지 검사합니다.
<br>
예: @Pattern(regexp = "^[A-Za-z0-9]{3,15}$", message = "사용자 이름은 3자 이상 15자 이하의 영문자 또는 숫자여야 합니다.") private String username;

#### @Future
날짜가 현재보다 미래인지 검사합니다.
<br>
예: @Future(message = "날짜는 미래여야 합니다.") private LocalDate eventDate;

#### @Past
날짜가 현재보다 과거인지 검사합니다.
<br>
예: @Past(message = "날짜는 과거여야 합니다.") private LocalDate birthDate;
 
<br>

### 사용 예시
#### DTO 내 필드에 유효성 검사 어노테이션 추가
```java
@Getter
@Setter
public class UserDTO {

	@NotNull(message="이름을 입력하세요.")
	private String name;

	@NotBlank(message="아이디를 입력해주세요. 빈 문자열 null, 공백 입력 불가")
	private String id;

	@NotEmpty(message="비밀번호를 입력해주세요. 빈 문자열, null 입력 불가")
	private String pwd;

	@Email(message="유효한  이메일 주소를 입력해주세요.")
	private String emailAddress;
    
}
```
#### @RequestBody로 UserDTO를 받을 때 @Valid 어노테이션도 추가해야 유효성 검증이 수행됨
```java
@RestController
pubic class Controller {
	
    @PostMapping("/register/user")
    public ResponseEntity<String> registerUser(
                                            @Valid @RequestBody UserDTO userDTO
                                            ) throws Exception {   	        
        return ResponseEntity.ok("회원 등록 성공");
    }
    
}
```
#### ExceptionHandler로 MethodArgumentNotValidException 예외 처리 가능

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseMessage<String> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
    return ResponseMessage.badRequest("오류 메시지");
}
```

<br><br>

# @Validated
- 스프링 프레임워크에서 제공하는 어노테이션
- @Valid가 수행하는 기능 외에 몇 가지 추가적인 기능을 가지고 있음
1) 유효성 검사 그룹을 지정하여 특정 조건에 따라 다르게 유효성 검사 수행 가능
2) Controller 외 다른 계층에서도 유효성 검증 가능
- 검증 오류 시 ConstraintViolationException 을 발생시킴

<br>

Request Body의 경우 @Valid 어노테이션을 추가함으로써 검증할 수 있었습니다. 
<br>
그렇다면 <b>쿼리 스트링</b>과 <b>쿼리 파라미터</b>는 어떻게 처리할 수 있을까요?

#### 쿼리 스트링 (Query String)
- URL의 일부로, ? 이후에 오는 모든 키-값 쌍을 포함하는 문자열<br>
https://example.com/search?name=Bob&age=20 에서 name=Bob&age=20 를 가리킵니다.
<br>
#### 쿼리 파라미터 (Query Parameter)
- 쿼리 스트링을 구성하는 개별적인 키-값 쌍 즉, 쿼리 스트링을 구성하는 요소<br>
https://example.com/search?name=Bob&age=20 에서 name=Bob, age=20 각각의 키 값 쌍이 쿼리스트링입니다.

### 사용 예시
#### Controller에서 @Validated를 사용하여 쿼리 파라미터를 검증할 경우 
1)  @Validated 를 클래스 레벨에 선언
2) 파라미터 앞에 검증 어노테이션 추가 (ex. @NotNull)

```java
@RestController
@validated
public class Controller {
	
    @GetMapping("search/user/{id}")
    public RequestEntity<String> find(@PathVariable @NotNull String id) {    	
        return ResponseEntity.ok("조회 성공");
    }
}
```

#### Controller 가 아닌 Service에서 @Validated를 사용할 경우
1) 클래스 레벨에 @Validated를 선언
2) 파라미터에 @Valid 어노테이션을 추가

```java
@Validated
@Service
public class Service {
	
    public void registerUser(@Valid UserDTO userDTO) throws Exception {
    	// 회원 등록 로직
    }
}
```

#### 예외처리
```java 
@ExceptionHandler(ConstraintViolationException.class)
public ResponseMessage<String> handleMethodConstraintViolationException(ConstraintViolationException e) {
    return ResponseMessage.badRequest("");
}
``` 
<br>
 

이상으로 포스팅을 마치겠습니다.<br>
 
> Reference
- https://medium.com/sjk5766/valid-vs-validated-%EC%A0%95%EB%A6%AC-5665043cd64b