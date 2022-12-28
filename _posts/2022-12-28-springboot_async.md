---
title: "스프링부트에서 annotation을 활용한 비동기 method 구현하기"
last_modified_at: 2022-12-28
author: Do-soo, KIM
---

초당 반복작업을 진행해야 하는 테스트 프로그램을 개발하다가 비동기 방식의 method 구현이 필요하였는데<br>
스프링부트에서는 annotation을 사용하여 쉽게 구현할 수 있는 방법이 있어 이번 포스팅에서 소개하고자 합니다.

우선, 기존 Java에서는 보통 아래 코드와 같이 비동기 방식의 method를 구현 하였습니다.

```java
public class testAsync {

    static ExecutorService executorService = Executors.newFixedThreadPool(5);

    public void test(final String message) throws Exception {
        executorService.submit(new Runnable() {
            @Override
            public void run() {
                // 수행할 작업
            }            
        });
    }
}
```

<br>
하지만 이러한 구현 방식은 비동기 method를 정의 할 때마다 Runnable의 run()을 재구현해야 하는 등의 작업을 반복하는 번거로움이 있습니다.<br>
그런데 스프링부트에서는 @Async라는 Annotation을 활용하면 쉽게 비동기 method를 구현할 수 있습니다.

@Async Annotation은 Spring에서 제공하는 Thread Pool을 활용하는 비동기 메소드 지원 Annotation 입니다.

사용 방법은 간단합니다. 아래 코드와 같이 Application Class에 @EnableAsync Annotation을 추가하고,<br>
비동기 method에 @Async Annotation을 붙여주면 됩니다.



```java
@EnableAsync
@SpringBootApplication
public class SpringBootApplication {
    ...
}

public class testAsync {

    @Async
    public void test(final String message) throws Exception {
        ....
    }
}
```

하지만 이렇게만 사용하면 @Async의 기본설정인 SimpleAsyncTaskExecutor를 사용하는 것이므로 Customize하게 구현하고 싶다면<br>
직접 AsyncConfigurerSupport를 상속받는 Class를 만들어서 사용하면 됩니다.

```java
@Configuration
public class AsyncCustomConfig extends AsyncConfigurerSupport {
	
	@Override
	public Executor getAsyncExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(10);
		executor.setMaxPoolSize(10);
		executor.setQueueCapacity(20000);
		executor.setThreadNamePrefix("epz-thread-");
		executor.initialize();

		return executor;
	}
}
```

여기서 설정한 요소들은 다음과 같습니다.

- @Configuration: Spring 설정 관련 Class로 @Component 등록되어 Scanning 됨<br>
- CorePoolSize: 기본 실행 대기하는 Thread의 수<br>
- MaxPoolSize: 동시 동작하는 최대 Thread의 수<br>
- QueueCapacity: MaxPoolSize 초과 요청에서 Thread 생성 요청시, 해당 요청을 Queue에 저장하는데 이때 최대 수용 가능한 Queue의 수<br>
- ThreadNamePrefix : 생성되는 Thread 이름에 붙는 접두사<br>

위와 같이 class를 생성한 후에는 작성한 후 구현할 비동기 method에 @Async Annotation을 붙여서 작성하면 됩니다.

마지막으로, @Async Annotation을 사용할 때 아래와 같은 세 가지 사항을 주의하여야 합니다.<br>
1.	private method는 사용 불가
2.	self-invocation(자가 호출) 불가, 즉 inner method는 사용 불가
3.	QueueCapacity 초과 요청에 대한 비동기 method 호출시 방어 코드 작성

위 주의사항에 대한 내용은 다음 포스팅에서 자세히 다뤄볼까 합니다.<br>
이번 포스팅은 여기서 마치겠습니다.
