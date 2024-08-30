---
title: "Observer Pattern과 알림 시스템"
last_modified_at: 2024-08-31
author: 이현지
---

본 포스팅은 Observer Pattern과 알림시스템에 대한 글입니다.
<br><br>

# Observer Pattern
Observer 패턴은 객체 지향 프로그래밍에서 사용되는 디자인 패턴입니다.

특정 객체의 상태가 변하면 다른 객체들이 변화된 정보를 갱신할 수 있도록 통지하며,<br>
하나의 Subject와 여러 Observer들이 일대다 관계를 이룬다는 특징이 있습니다.

 

 

## 주요 구성 요소 

### Subject (관찰 대상)
상태 변화를 감지하여 Observer들에게 통지하는 객체

### Observer (관찰자)
Subject의 상태 변화를 통지받아 필요한 작업 수행

 
<br>

## 동작 과정

1. Subject에 여러 Observer들이 등록됨 (등록 해제 가능)
2. Subject의 상태가 변하면 자신의 상태가 변했음을 Observer들에게 알림
3. 각 Observer들은 Subject의 상태 변화를 통지받고 필요한 작업을 수행

<br> 

## 예시
유튜브 구독 시스템

<br>

![예시](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FMkxgw%2FbtsIp8f5g2A%2FkzS9TkYyAooDNyCO4jn5C0%2Fimg.png)

<br>

구독자(Observer)들은 유튜브 채널(Subject)을 구독하거나 구독 해제 가능
구독 중인 유튜브 채널(Subject)에 새로운 영상이 업로드되면 구독자(Observer)들에게 알림 발생
구독자(Observer)들은 새 영상에 대한 알림을 받고 새 영상을 시청함
 

## 구조
<br>

![구조](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fmbc31%2FbtsIoiyea0R%2FQKQFDsVhokIpOKC8Ev0Wr0%2Fimg.png)

<br>

## 장점

### 객체 간의 결합도 감소
Subject와 Observer가 독립적으로 존재하고 Subject는 Observer에게 자신의 상태 변화를 알리기만 하면 되기 때문에 Subject는 Observer가 특정 인터페이스를 구현한다는 것만 알고 있으며, Observer가 어떻게 동작하는지 모름
### 뛰어난 확장성
새로운 Observer를 추가하더라도 Subject의 코드를 수정할 필요가 없으므로 시스템의 유지보수 및 확장이 용이함
### 실시간 업데이트
Subject의 상태 변화가 즉시 Observer들에게 전달됨

## 단점 
### 성능 문제 발생 가능
너무 많은 Observer들이 등록될 경우 모든 Observer에게 상태 변화를 통지하는 데 시간이 많이 소요될 수 있음
### 순환 참조 문제 발생 가능
Subject와 Observer가 서로를 참조하는 경우 무한 루프에 빠질 수 있으므로 이를 방지하기 위한 추가적인 설계가 필요함


## 예시 코드

### Sample1

#### Subject
```java
public interface Subject {

	/* Observer 등록 */
	void registerObserver(Observer observer);
	/* Observer 삭제 */
	void removeObserver(Observer observer);
	/* Observer들에게 변경사항 통지 */
	void notifyObservers(String msg);

}

import java.util.ArrayList;
import java.util.List;

public class Subject1 implements Subject {

	private List<Observer> observers;
	
	public TestSubject() {
		observers = new ArrayList<Observer>();
	}

	@Override
	public void registerObserver(Observer observer) {
		observers.add(observer);
	}

	@Override
	public void removeObserver(Observer observer) {
		observers.remove(observer);
	}

	@Override
	public void notifyObservers(String msg) {
		for (Observer observer : observers) {
			observer.update(msg);
		}
	}
	
}
```

#### Observer
```java
public interface Observer {

	/* Subject의 상태를 업데이트 받는 메소드 */
	void update(String msg);

}

public class Observer1 implements Observer {

	@Override
	public void update(String msg) { 
		System.out.println("Observer1 : " + msg);
	}

}

public class Observer2 implements Observer {

	@Override
	public void update(String msg) { 
		System.out.println("Observer2 : " + msg);
	}

}
``` 

실행
```java
public class Sample {
	
	public static void main(String[] args) {
	
		Observer observer1 = new Observer1(); 
		Observer observer2 = new Observer2();
		
		Subject subject1 = new Subject1();
		
		// Subject에 Observer 등록 (= Observer들이 Subject를 구독)
		subject1.registerObserver(observer1);
		subject1.registerObserver(observer2);

		// testSubject의 변경사항을 Observer들에게 통지
		String msg = "subject1에 새 영상이 업로드 되었습니다.";
		testSubject.notifyObservers(msg);
        
	}
	
}
``` 

<br><br>

### Sample2

Observer Pattern을 이용하여 아주 간단한 주가 변동 알림 시스템을 구현해봅시다. 
 
```java
import java.util.ArrayList;
import java.util.List;

/**
 *	주가를 관찰하는 주체
 */
class Stock {

	/** 옵저버 리스트 */
    private List<Observer> observers = new ArrayList<>();
    /** 주가 */
    private float price;

	/** 옵저버 등록 */
    public void addObserver(Observer observer) {
        observers.add(observer);
    }

	/** 옵저버 등록 해제 */
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

	/** 주가 변경 및 옵저버들에게 알림 */
    public void setPrice(float price) {
        this.price = price;
        notifyObservers();
    }

	/** 옵저버들에게 주가 변동 사항 알림 */
    private void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(price);
        }
    }
}

/** 
 * 옵저버 인터페이스
 */
interface Observer {
    void update(float price);
}

/** 
 * 주식 가격을 모니터링하는 관찰자
 */
class StockObserver implements Observer {
    
    private String name;

    public StockObserver(String name) {
        this.name = name;
    }

    @Override
    public void update(float price) {
        System.out.println(name + "의 주식 가격이 변경되었습니다.: " + price);
    }
}

/**
  * 메인 클래스
  */
public class ObserverPatternExample {
    public static void main(String[] args) {
    
        Stock stock = new Stock();

        StockObserver observer1 = new StockObserver("관찰자 1");
        StockObserver observer2 = new StockObserver("관찰자 2");

        stock.addObserver(observer1);
        stock.addObserver(observer2);

        // 주식 가격 변경
        stock.setPrice(100.0f); 
        // -> 관찰자 1(또는 2)의 주식 가격이 변경되었습니다.: 100.0
        stock.setPrice(105.5f);
        // -> 관찰자 1(또는 2)의 주식 가격이 변경되었습니다.: 100.0
    }
}
``` 

<br><br>

이상으로 ObserverPatter과 알림 시스템에 대한 글을 마치겠습니다. 