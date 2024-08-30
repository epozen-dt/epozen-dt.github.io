---
title: "다양한 Java 객체 생성 패턴 및 장단점"
last_modified_at: 2024-08-31
author: 이현지
---

본 포스팅에서는 여러 가지 Java 객체 생성 패턴 및 장단점에 대해 알아보겠습니다.
<br><br>


# 1. 생성자 패턴 (Constructor Pattern)

- 클래스의 인스턴스를 생성할 때 생성자를 사용하는 전통적인 방법

- 객체 생성 시 필요한 모든 필드를 생성자 매개변수로 전달함

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

- 객체 생성 시 모든 필드를 빠짐 없이 초기화할 수 있음

 

## 단점

- 매개변수가 많으면 가독성이 떨어짐

  -> 같은 타입의 매개변수 값이 바뀌었을 경우 컴파일러 오류가 발생하지 않음

<br><br>

# 2. 정적 메소드 패턴 (Static Method Pattern)

- 정적 메소드를 사용하여 객체를 생성함

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

- 객체 생성 로직을 캡슐화함

  -> 클라이언트 코드 단순화, 유지보수 용이, 동일한 객체 생성 로직을 여러 곳에서 재사용 가능

- 객체 생성 메소드의 이름을 통해  반환되는 인스턴스가 어떤 역할을 할 지 명시할 수 있음

- 상위 클래스의 하위클래스를 반환하도록 반환타입을 제한할 수 있음

 

## 단점

생성자 패턴과 마찬가지로 같은 타입의 매개변수 값이 바뀌었을 경우 컴파일러 오류가 발생하지 않음

 
<br><br>

# 3. 수정자 패턴 (Setter Pattern)

- setter를 이용하여 객체의 필드를 개별적으로 설정

- 생성자로 기본값을 설정하고, 수정자를 통해 값을 변경함

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

- 객체를 단계적으로 초기화할 수 있음

- 필드를 선택적으로 초기화할 수 있음

 

## 단점

- setter가 많을 경우 클라이언트 코드에서 객체를 설정하는 코드가 길어지고 복잡해져서 가독성이 저하됨

- setter를 통해 객체의 상태를 변경할 수 있으므로 객체의 불변성 및 일관성을 유지하기 어려움

  -> 멀티스레드 환경에서 문제를 일으킬 수 있음

- setter를 통해 의존성을 주입할 경우, 필요한 모든 속성이 설정되지 않은 상태에서 객체가 사용될 수 있음

  -> 런타임 오류 발생

<br><br>

# 4. 빌더 패턴 (Builder Pattern)

- 복잡한 객체 생성 시 유용함

- 빌더 클래스를 사용하여 객체의 다양한 속성을 설정하고 객체를 생성함

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
    	this.id = id; 
        this.pwd = pwd;
    	this.name = name; 
        this.age = age;
    }

	/** 빌더 클래스 */
	public static class UserBuilder {
        // 필수 필드
        private id; 
        private pwd;
        // 옵션 필드
        private String name;
        private int age;
        
        public Builder setName(String id, String pwd) {
        	this.id = id; 
            this.pwd = pwd;
            return this;
        }
        
        public UserBuilder setName(String name) {
        	this.name = name;
            return this;
        }
        
        public UserBuilder setAge(int age) {
        	this.age = age;
            return this;
        }
        
        // 빌더 클래스의 build() 메서드를 통해서만 인스턴스 생성 가능
        public User build() {
        	return new User(this);
        }    
	}
}
``` 

### 인스턴스 생성

```java
public class Sample {
	
    public void test() {
    	// 필수 필드는 User의 생성자로 설정
    	User user = User.UserBuilder("testId", "testPwd")
        				// 옵션 필드는 set()로 설정
                        // .setName("Sam")
                        // .setAge(10)
                        .build();
    }

}
``` 

### @Builder 어노테이션

@Builder 어노테이션을 이용하면 별도의 빌더 클래스를 작성할 필요 없이 간단하게 빌더 패턴을 적용할 수 있습니다.

 

#### 예시1) 클래스 위에 @Builder 를 붙이는 경우

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

#### 예시2) 생성자 위에 @Builder 를 붙이는 경우

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

예시 1과 2 모두  builder() 메소드로 객체를 생성합니다.

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

- 필수 필드만 초기화 가능

- 필수 필드에 대해선 컴파일러로 오류 체크 가능

 

## 단점

- 객체를 생성하기 위해 추가적인 객체(builder)를 생성해야 하므로 성능에 미세한 오버헤드 발생 가능

- 간단한 객체 생성 시에는 단순한 생성자나 수정자를 사용하는 것이 더 나음

- 객체의 필수 속성을 체크하거나 제약 조건을 처리하는 것이 builder 내부에서 복잡해질 수 있음

- 여러 빌더 인스턴스가 동시에 사용될 경우, 각각의 빌더가 동일한 객체상태를 유지하기 어려우므로 추가적인 설정이 필요할 수 있음

 
<br><br>

이상으로 다양한 Java 객체 생성 패턴 및 장단점에 대한 글을 마치겠습니다.

<br><br>
 > Reference 
- https://mangkyu.tistory.com/163
- https://velog.io/@a01021039107/4.-%EA%B0%9D%EC%B2%B4-%EC%83%9D%EC%84%B1%EC%97%90-%EC%83%9D%EC%84%B1%EC%9E%90-vs-%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4-%EB%AC%B4%EC%97%87%EC%9D%84-%EC%8D%A8%EC%95%BC-%ED%95%A0%EA%B9%8C