---
title: "Java Interface: ScheduledExecutorService"
last_modified_at: 2023-10-25
author: Do-soo, KIM
---


> 이번 포스팅에서는 Java Interface인 ScheduledExecutorService에 대해서 정리해 보려고 합니다.


### 1. 개요

이번에 진행하는 프로젝트에서 어떤 이벤트에 의해서 타이머를 실행하고, 특정 시간이 되면 타이머를 종료하는 기능 구현이 필요했습니다.

SpringBoot에서는 @Scheduled로 편하게 사용하는 방법이 있지만, 이 방법을 사용하면 특정 이벤트에 의해 실행할 수 있는 구현이 어렵고, 딱 한번만 실행이 필요한 저에게는 비효율적이라고 생각해서 다른 방법을 찾아보던 중에 ExecutorService를 사용하면 좋겠다고 생각해서 적용해 보았습니다. <br>
ExecutorService를 상속하는 ScheduledExecutorService interface를 사용하면 TimeUnit을 사용하여 미리 정의된 지연 후에 작업을 한번 수행할 수 있습니다.<br>
ExecutorService는 각각 고유한 특징과 기능을 가진 ThreadPoolExecutor, ScheduledThreadPoolExecutor, ForkJoinPool을 포함한 여러 구현 클래스를 제공합니다.<br>
<p align="center">
    <img src="https://github.com/epozen-dt/epozen-dt.github.io/assets/92565548/79dca518-06be-473d-a8a2-a497ecc6daf2" width="50%" height="50%" />
    <br>출처: https://www.geeksforgeeks.org/scheduledexecutorservice-interface-in-java/
</p>

그럼 ScheduledExecutorService를 어떻게 사용하는지 보겠습니다.

### 2. 사용 예제

ScheduledExecutorService 를 인스턴스화하는 가장 좋은 방법은 Executors 클래스의 팩토리 메서드를 사용하는 것입니다.<br>
이 포스팅에서는 일단 스레드를 1개로 하여 newScheduledThreadPool를 사용해 보겠습니다.

```java
public class ScheduledExecutorRunnable {

    public static void main(String[] args) {

    	ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);
       	Runnable task2 = () -> System.out.println("Running task2...");

        task1();
		executorService.schedule(task2, 5, TimeUnit.SECONDS);
		task3();

        executorService.shutdown();

    }

    public static void task1() {
        System.out.println("Running task1...");
    }

    public static void task3() {
        System.out.println("Running task3...");
    }
}
```

일반적으로 ExecutorService는 처리할 작업이 없을 때 자동으로 소멸되지 않기 때문에, 종료를 해줘야 합니다.<br>
그래서 위 코드에서 보듯이, 작업 완료 후에 shutdown() 메서드를 사용하여 종료하는 코드가 추가되었습니다 .<br>
shutdown() 메서드는 ExecutorService를 즉시 종료하지 않고, 새 작업 수락을 중지하고 실행 중인 모든 스레드가 현재 작업을 완료한 후 종료합니다.

위 코드를 실행해 보면, task1과 task3이 수행되고 5초 후 task2가 수행됩니다.<br>
이렇게 지정한 시간만큼 delay 후에 예약한 작업을 수행하도록 구현할 수 있음을 확인해 보았습니다.


> Reference

- JavaDoc - ScheduledExecutorService<br>
- https://colevelup.tistory.com/33<br>
- https://mkyong.com/java/java-scheduledexecutorservice-examples/<br>



