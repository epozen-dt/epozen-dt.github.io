# 1. 생성자 패턴 (Constructor Pattern)
- 생성자를 이용하여 클래스의 인스턴스를 생성
- 객체 생성 시 필요한 모든 필드를 생성자 매개변수로 전달

## 예시 
```java
public class User {
	private String name;
    private int age; 
    
    public User(String name, int age) {
    	this.name = name; 
        this.age = age;
    }
}
```

## 장점
- 간단하고 직관적
- 객체 생성 시 누락 없이 모든 필드 초기화 가능

## 단점
- 매개변수가 많으면 가독성이 떨어짐
- 매개변수 값이 바뀌어서 입력될 경우, 같은 타입이면 컴파일러 오류가 발생하지 않음

```java
// User user = new User("이름", "나이"); 일 경우
User user = new User("최강수", "41");
User user = new User("45", "강현식"); 
// ↑ 이름과 나이가 바뀌어도 컴파일 오류 발생하지 않음
```
<br><br>

# 2. 정적 메소드 패턴 (Static Method Pattern)
- 정적 메소드를 사용하여 객체를 생성함

## 예시 
```java
public class User {
	private String name; 
    private int age;
    
    private User(String name, int age) {
    	this.name = name;
        this.age = age;
    }
    
    public static User createUser(String name, int age) {
    	return new User(name, age);
    }
}
```

## 장점
- 객체 생성 로직을 캡슐화하여 클라이언트 코드를 단순화하고, 동일한 객체 생성 로직을 여러 곳에서 재사용 할 수 있음 (중복 감소, 일관성 유지)
- 반환되는 인스턴스가 어떤 역할을 하는지 객체 생성 메소드명에 명시할 수 있음
- 인터페이스나 추상클래스를 이용하여 반환타입을 제한할 수 있음

## 단점
생성자 패턴과 마찬가지로 매개변수 값이 잘못 바뀔 경우 타입만 맞으면 컴파일러 오류로 잡아낼 수 없음

<br><br>

# 3. 수정자 패턴 (Setter Pattern)
- setter를 이용하여 객체의 필드를 개별적으로 설정 가능
- 생성자로 기본값을 설정하고, 수정자를 통해 값을 변경함

## 예시 
```java
public class User {
	private String name;
    private int age; 
    
    public User() {}

	public void setName(String name) {
    	this.name = name;
    }
    
    public void setAge(int age) {
    	this.age = age;
    }
}
```

## 장점
- 객체를 선택적, 단계적으로 초기화할 수 있음

## 단점
- setter가 많을 경우 클라이언트 코드에서 객체를 설정하는 코드가 길고 복잡해짐
- setter를 통해 객체의 상태를 변경할 수 있으므로 객체의 불변성 및 일관성을 유지하기 어려움<br>
-> 멀티스레드 환경에서 문제를 일으킬 수 있음
- 단계적으로 객체 초기화가 가능하기 때문에 필요한 모든 속성이 설정되지 않은 상태에서 객체가 사용될 우려가 있음

<br><br>

# 4. 빌더 패턴 (Builder Pattern)
- 빌더 클래스를 사용하여 객체를 생성함
- 복잡한 객체 생성 시 유용함

```java
public class User {
	// 필수 필드
	private String id; 
    private String pwd;
	// 옵션 필드
	private String name; 
    private int age;
    
    /** 빌더만 인스턴스 생성 가능하도록 생성자를 private로 선언 */
    private User(UserBuilder builder) {
    	this.id = builder.id; 
        this.pwd = builder.pwd;
    	this.name = builder.name; 
        this.age = builder.age;
    }

	/** 빌더 클래스 */
	public static class UserBuilder {
        // 필수 필드
        private id; 
        private pwd;
        // 옵션 필드
        private String name;
        private int age;

       // 필수 필드는 Builder 생성자로 정의 
        public UserBuilder(String id, String pwd) {
        	this.id = id; 
            this.pwd = pwd;
            return this;
        }
        
        // 옵션 필드는 메서드로 정의
        public UserBuilder setName(String name) {
        	this.name = name;
            return this;
        }
        
        public UserBuilder setAge(int age) {
        	this.age = age;
            return this;
        }
        
        // 빌더 클래스의 build() 메서드를 통해서만 User 인스턴스 생성 가능
        public User build() {
        	return new User(this);
        }    
	}
}
```

## 인스턴스 생성

```java
public class Sample {
	
    public void test() {
    	// 필수 필드는 빌더클래스의 생성자로 정의
    	User user = User.UserBuilder("testId", "testPwd")
        				// 옵션 필드는 빌더 클래스의 메서드로 정의
                        // .setName("Sam")
                        // .setAge(10)
                        .build();
    }

}
```

## @Builder 어노테이션
@Builder 어노테이션을 이용하면 빌더 클래스를 직접 작성하지 않고 빌더 패턴 적용 가능

### 예시1) 클래스 위에 @Builder 를 붙이는 경우
```java
import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class User {
    private String name;
    private int age;
}
```
### 예시2) 생성자 위에 @Builder 를 붙이는 경우
```java
import lombok.Builder;
import lombok.Getter;

@Getter
public class User {
    private String name;
    private int age;

    @Builder
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
예시 1과 2 모두  builder() 메소드로 객체를 생성
```java
public class Main {
    public static void main(String[] args) {
        User user = User.builder()
                        .name("Harry")
                        .age(25)
                        .build();

        System.out.println("Name: " + user.getName());
        System.out.println("Age: " + user.getAge());
    }
}
```

## 장점

- 클라이언트 코드의 가독성 향상
- 필수 필드만 선택적으로 초기화 가능
- 필수 필드에 대해선 컴파일러로 오류 체크 가능

## 단점
- 추가적인 객체(builder)를 생성해야 하므로 성능에 미세한 오버헤드 발생 가능
- 간단한 객체는 단순한 생성자나 수정자를 사용하는 것이 더 나음
- 객체의 필수 속성을 체크하거나 제약 조건을 처리하는 코드가 빌더 내부에서 복잡해질 수 있음
- 여러 빌더 인스턴스가 동시에 사용될 경우, 각각의 빌더가 동일한 객체상태를 유지하기 어려우므로 추가적인 설정이 필요함

<br><br>

이상으로 다양한 Java 객체 생성 패턴 및 장단점에 대한 포스팅을 마치겠습니다.

<br><br>
> Reference
- https://mangkyu.tistory.com/163
- https://velog.io/@a01021039107/4.-%EA%B0%9D%EC%B2%B4-%EC%83%9D%EC%84%B1%EC%97%90-%EC%83%9D%EC%84%B1%EC%9E%90-vs-%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4-%EB%AC%B4%EC%97%87%EC%9D%84-%EC%8D%A8%EC%95%BC-%ED%95%A0%EA%B9%8C