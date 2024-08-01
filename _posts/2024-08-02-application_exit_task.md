---
title: "Trie 자료 구조 구현하기"
last_modified_at: 2024-08-02
author: 이현지
---

본 포스팅에서는 자바 애플리케이션 종료 시 특정 작업을 수행하는 방법에 대해 알아보겠습니다.
 <br><br>


애플리케이션 종료 시 주로 수행되는 작업들로는 리소스 정리, 데이터 저장, 로그 기록 등이 있습니다.

이러한 작업들을 수행할 수 있도록 자바에선 Runtime.addShutdownHook()이라는 메서드를 제공하고 있습니다.
<br>
 
# java.lang.Runtime.addShutdownHook(Thread t) 
JVM(Java Virtual Machine)이 정상 혹은 비정상적으로 종료될 때 실행될 스레드를 지정 가능
<br>
 
## 사용방법 
### 1. Thread 상속을 통한 shutdown hook 생성
```java
public class ShutDownHook extends Thread {

	@Override
	public void run() {
		System.out.println("종료 되었습니다.");
	}
	
}
```
<br>

### 2. shutdown hook 등록
```java
@SpringBootApplication
public class ShutDownHookApplication {

	public static void main(String[] args) {		
		// shutdown hook 등록
		Runtime.getRuntime().addShutdownHook(new ShutDownHook());	            
		SpringApplication.run(ShutDownHookApplication.class, args);
	}

}
```
<br>

스프링 프로젝트의 경우 이렇게 Application 클래스의 main 메서드 내에 지정해주면 애플리케이션 전체 종료 시 shutdown hook이 실행됩니다.
<br>

그런데 애플리케이션 종료 후 한번만 실행 되어야 할 shutdown hook이 두번 실행 되는 경우가 있습니다.<br>

![애플리케이션 종료 이미지](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkLRHV%2FbtsG0vejUU1%2F26iZFieQlnRceqMIrMkLkk%2Fimg.png)
<br>
이건 spring boot devtools 라는 의존성이 추가 돼 있어서 생기는 문제입니다.
<br>

### spring boot devtools

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```
<br>
devtools는 개발 편의성을 위해 소스 파일이 변경될 때마다 JVM 재시작 없이 스프링 컨테이너만 자동 재실행해주는 역할을 합니다.<br><br>
shutdown hook이 두 번 실행되는 이유는 바로 이 devtools로 인해 main 메서드가 두 번 실행 되기 때문인데요<br><br>
자바 프로그램 실행 시 JVM은 프로그램의 진입점인 main 메서드를 호출합니다.<br>
이 main 메서드는 기본 스레드인 main 스레드에서 실행되고, 스프링 컨테이너도 원래는 main 스레드에서 실행됩니다.<br>

그런데 devtools는 JVM 재시작 없이 스프링 컨테이너를 재실행하기 위해 main 스레드가 아닌 restartedMain이라는 별도의 스레드에서 스프링 컨테이너를 실행합니다. 그리고 이 때 스프링 컨테이너를 실행하기 위해 main 메서드를 한번 더 호출합니다.<br>

따라서 프로그램 종료 시, 두 번 호출된 main 메서드들이 종료되면서 각 main 메서드 내부의 shutdown hook도 두 번 실행되는 것입니다.<br>

그래서 이 문제를 해결하기 위해선 devtools 의존성을 제거하면 됩니다. 
<br>

### shutdown hook 여러 개 등록 가능
하나의 JVM에 여러 개의 shutdown hook을 등록하는 것도 가능합니다.<br>
단 다중 스레드처럼 shutdown hook이 실행되는 순서는 지정할 수 없습니다.
<br>

```java
public class ShutdownHookTest {

	static class HookThread1 extends Thread {
		@Override
		public void run() {
			System.out.println("first hook");
		}		
	}
    
	static class HookThread2 extends Thread {
		@Override
		public void run() {
			System.out.println("second hook");
		}		
	}

	public static void main(String[] args) {
		Runtime.getRuntime().addShutdownHook(new HookThread1());
		Runtime.getRuntime().addShutdownHook(new HookThread2());
		System.out.println("application exited.");
	}

}
```
<br>

### 실행 결과 
shutdown hook등록된 순서와 실행되는 순서는 무관하다는 것을 알 수 있습니다.
<br>

![hook 순서 무관 이미지](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbT64wO%2FbtsG3lB0QNc%2FQq8ZlVlPpKJLCKcFz2pCg1%2Fimg.png)
<br>

## 사용 시 주의할 점 

●  shutdown hook은 최대한 짧게 작성 돼야 합니다.
<br>

사용자 로그오프 또는 시스템 종료로 인해 가상 머신이 종료될 경우 shutdown hook이 실행되기 전이나 실행되는 도중에 JVM이 종료될 수 있습니다.
<br>

●  shutdown hook이 프로그램 종료 시 반드시 수행된다는 걸 보장할 수 없습니다.

<br>

다음과 같이 프로세스가 종료되는 경우에는 shutdown hook이 정상적으로 실행됩니다.

- System.exit() 에 의한 종료 
- 사용자 인터럽트(ctrl+c)에 의한 JVM 종료
- 사용자 로그오프 또는 시스템 셧다운에 의한 종료
- SIGTERM 신호에 의한 종료
- handled되지 않은 Exception 발생으로 인한 프로세스 종료

<br>

그러나 다음과 같은 경우에는 shutdown hook이 실행되지 않고 프로세스가 종료되어 버리므로 주의해야 합니다.

- Runtime.halt() 에 의한 종료
- SIGKILL에 의한 종료 (kill -9 명령어 같은)을 받는 경우 
- JVM에 문제가 발생하여 종료되는 경우

<br><br>

---

<br><br>

# Spring의 Application Context Event 
Spring에서 기본 제공하는 Application Context Event를 이용하는 방법도 있습니다.
<br>
Spring Framework에서는 어플리케이션의 생명주기에 따른 다양한 이벤트를 제공하며,
개발자는 Spring의 이벤트 리스닝 메커니즘을 통해 특정 이벤트 발생 시 원하는 동작을 정의할 수 있습니다.
<br>

## Spring 에서 기본 제공하는 이벤트
![스프링 이벤트 종류](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtVL8v%2FbtsG3n00Rkh%2F1uh1EIdDCRfzyzHKlC3OC0%2Fimg.png)
<br>
이 중 애플리케이션 컨텍스트가 종료될 때 발생하는 이벤트는 ContextClosedEvent 입니다.
<br>

## ContextClosedEvent 사용 방법
### 1.이벤트 리스너 생성
다음과 같이 클래스 전체에서 ApplicationListener<ContextClosedEvent>를 상속 받거나

```java
@Service
public class ContextClosedEventTest implements ApplicationListener<ContextClosedEvent> {

	@Override
	public void test(ContextClosedEvent event) {
		
		System.out.println("애플리케이션 종료 시 실행됨");
		System.out.println("애플리케이션 종료 이벤트 발생 시간(timestamp) : " + event.getTimestamp());
		
	}
	
}
```
<br>

@EventListener 어노테이션을 이용하여 구현 가능합니다.

```java
@Service
public class ContextClosedEventTest {

    @EventListener
    public void test(ContextClosedEvent event) throws Exception {
	
    	System.out.println("애플리케이션 종료 시 실행됨");
        System.out.println("애플리케이션 종료 이벤트 발생 시간(timestamp) : " + event.getTimestamp());
    
    }
    	
}
```
<br>

### 애플리케이션 실행 결과

![결과](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FGaIZb%2FbtsG3pLdooE%2FMag7jfkZ5A3jH6PRbGPoXK%2Fimg.png)

 
<br><br>

이상으로 애플리케이션 종료 시 특정 작업을 수행하는 방법들에 대해 알아보았습니다.


<br><br>
 > Reference 
- https://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html<br>
- https://blog.seulgi.kim/2014/06/java-shutdown-hook.html<br>
- https://seriouskang.tistory.com/10<br>
- https://blog.seulgi.kim/2014/06/java-shutdown-hook.html<br>
- https://tyboss.tistory.com/entry/Java-Runtime-%EC%9D%98-addShutdownHook-%EC%82%AC%EC%9A%A9<br>
- https://www.inflearn.com/questions/115655/%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%9D%B4-2%EB%B2%88%EC%8B%A4%ED%96%89-%EB%90%A9%EB%8B%88%EB%8B%A4<br>
- https://velog.io/@probsno/Spring%EC%97%90%EC%84%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8%EB%A5%BC-%EC%B2%98%EB%A6%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95<br>
