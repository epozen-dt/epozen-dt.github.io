---
title: "스프링부트에서 annotation을 활용한 비동기 method 구현하기 - 두 번째"
last_modified_at: 2023-01-26
author: Do-soo, KIM
---

지난 포스팅에 이어,<br>
이번 포스팅에서는 스프링부트에서 @Async Annotation을 사용할 때의 주의사항에 대해 확인해 보겠습니다.

### 1.	private method는 사용 불가
<br>
@Async는 AOP가 적용되어 Spring Context에서 등록된 Bean Object의 method가 호출 될 때 Spring이 확인할 수 있고 @Async가 적용된 method의 경우 Spring이 method를 가로채 다른 Thread에서 실행 시켜주는 방식으로 동작합니다.

이 때문에 Spring이 해당 @Async method를 가로챈 후 다른 Class에서 호출이 가능해야 하므로, private method는 사용할 수 없는 것입니다.


### 2.	self-invocation(자가 호출) 불가, 즉 inner method는 사용 불가
<br>
Spring Context에 등록된 Bean의 method의 호출이어야 Proxy 적용이 가능하므로, inner method의 호출은 Proxy 영향을 받지 않기에 self-invocation이 불가능합니다.

아래 샘플 코드는 동일 Class 내에 있는 method를 비동기 방식으로 사용하는 것입니다.<br>

```java
@Controller
public Class TestController {

    @Async
    public void asyncMethod(int i) {
        try {
            Thread.sleep(500);
            log.info("[AsyncMethod]"+"-"+i);
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }

    @GetMapping("async")
    public String testAsync() {
        log.info("TEST ASYNC");
        for(int i=0; i<50; i++) {
            asyncMethod(i);
        }
        return "";
    }
}
```

이 샘플코드를 실행해 보면 method가 비동기 방식으로 호출되지 않습니다.<br>
즉, 자가 호출에서는 @Async 사용이 불가능 하다는 의미입니다.

하지만 이 샘플 코드를 아래와 같이 바꿔보면 다릅니다.<br>
@Service annotation을 사용하여 Bean 등록된 Service를 통해 주입하는 방식으로 바꾸어 본 것입니다.<br>

```java
@Service
public class TestService {
    @Async
    public void asyncMethod(int i) {
        try {
            Thread.sleep(500);
            log.info("[epz-thread]"+"-"+i);
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }
}

@AllArgsConstructor
@Controller
public Class TestController {

    private TestService testService;

    @GetMapping("async")
    public String testAsync() {
        log.info("TEST ASYNC");
        for(int i=0; i<50; i++) {
            testService.asyncMethod(i);
        }
        return "";
    }
    
}

```

이 샘플 코드를 실행해 보면 method 호출 순서와 상관없이 비동기 방식으로 호출 됩니다.


### 3.	QueueCapacity 초과 요청에 대한 비동기 method 호출시 방어 코드 작성
<br>
AsyncConfig에서 PoolSize와 QueueCapacity 값을 줄여서, 위 샘플 코드를 실행해 보면 다른 결과가 나오게 됩니다.

TaskRejectedException이 발생하게 됩니다.
값을 줄인 PoolSize와 QueueCapacity 보다 초과된 요청에 의해서 발생하는 Exception 입니다. 
따라서 이 Exception을 handling하는 방어 코드를 추가해 주어야 합니다.


```java
@AllArgsConstructor
@Controller
public Class TestController {

    private TestService testService;

    @GetMapping("async")
    public String testAsync() {
        log.info("TEST ASYNC");
        try {
            for(int i=0; i<50; i++) {
                testService.asyncMethod(i);
        } catch (TaskRejectedException e) {
            // ....
        }
        return "";
    }

```

위와 같이, handling 코드를 작성해 주면 됩니다.

이상으로 스프링부트에서 비동기 method를 구현하는 방법에 대한 포스팅을 마무리 하겠습니다.

### Reference

https://dzone.com/articles/effective-advice-on-spring-async-part-1<br>
https://velog.io/@gillog/Spring-Async-Annotation%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%A9%94%EC%86%8C%EB%93%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0
