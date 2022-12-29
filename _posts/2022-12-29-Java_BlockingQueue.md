---
title: "Java Blocking Queue"
last_modified_at: 2022-12-29
author: 이한솔
---

이번 시간에는 Java Blocking Queue에 대해 알아보도록 하겠습니다.

---
> **Blocing Queue**는 큐가 꽉 찼을 때의 삽입 시도 or 큐가 비어 있을 때의 추출 시도를 차단하는 큐입니다.

종류
- ArrayBlockingQueue
- LinkedBlockingQueue
- PriorityBlockingQueue
- DelayQueue
- SynchronousQueue

큐의 크기 : ArrayBlockingQueue만 필수로 지정해야하며, 다른 큐의 경우 생략 시 Integer.MAX_VALUE로 자동 지정됩니다.

---

## **Blocking Queue 메소드**
<br>

삽입, 제거 메소드는 기능에 따라 다음과 같습니다. <br>
![image](https://user-images.githubusercontent.com/96156882/209918834-534d8065-c4d4-4eba-9758-330ab1b6b584.png)

### 1. 삽입 메소드
- add(e) : 큐에 요소 추가 (예외 O) <br>
Exception : IllegalStateException
- offer(e) : 큐에 요소 추가 (예외 X) <br>
Return : 성공 시 true, 실패 시 false를 return
- put(e) : 여유 공간이 생길 때까지 대기 후 요소 추가
- offer(e, timeout, unit) : 지정 시간 대기 후 요소 추가 <br>
(e:추가할 요소 / timeout:큐 내에서 대기할 시간 / unit:시간 초과 매개 변수 시간 단위)
ex) queue.offer(객체, 5000, TimeUnit.MILLISECONDS);

### 2. 제거 메소드
- remove() : 큐의 요소 제거 (예외 O) <br>
Exception : NoSuchElementException 
- poll() : 큐의 요소 제거 (예외 X)
- Return : 큐가 비어 있으면 null을 return
- take() : 큐가 생길 때까지 대기 후 요소 제거
- poll(timeout, unit) : 지정 시간 대기 후 요소 제거 <br>
(timeout:큐 내에서 대기할 시간 / unit:시간 초과 매개 변수 시간 단위)

---
## 1. ArrayBlockingQueue

고정 array를 기반으로 하며, 생성 후 크기 변경 불가합니다.

생성 방법은 다음과 같습니다. <br>
BlockingQueue<Integer> queue = new ArrayBlockingQueue<Integer>(capacity, fair); <br>
	- parameters <br>
	  capacity : 큐의 크기 <br>
	  fair : true일 경우 차단된 스레드에 대한 큐가 FIFO 순서로 처리.
  false이면 순서가 지정되지 않음

<br>

## 2. LinkedBlockingQueue 
Linked list를 기반으로 하며, array 기반 큐보다 시간당 처리량(throughput)이 많습니다.

생성 방법은 다음과 같습니다. <br>
BlockingQueue<Integer> queue = new LinkedBlockingQueue<Integer> (capacity); <br>
	- parameter <br>
	  capacity : 큐의 크기. 생략할 경우 Integer.MAX_VALUE로 자동 지정.

<br>

## 3. PriorityBlockingQueue
PriorityBlockingQueue는 힙(이진 트리 구조) 정렬 방식을 지닙니다. 높은 우선순위의 요소를 먼저 꺼내서 처리하는 구조이며, 자원 고갈시 OutOfMemoryError 발생으로 인해 삽입이 실패할 수 있습니다.

생성 방법은 다음과 같습니다. <br>
BlockingQueue<Integer> priorityBlockingQueue = new PriorityBlockingQueue<>(); <br>

priorityBlockingQueue는 기본적으로 같은 우선순위를 가지는 요소에 대해서는 순서를 보장하지 않습니다. <br>
만약 순서를 정하고 싶은 경우 Comparable Interface를 implements하는 Class를 생성한 후, compareTo() 오버라이드 메소드를 통해 사용자 지정이 가능합니다.

	ex) BlockingQueue<prio> priorityBlockingQueue = new PriorityBlockingQueue<>();
	      prio.add(new prio(…););
	      class prio implements Comparable<prio>{
	      	@Override
    		public int compareTo(prio o) { 사용자 지정 }
	      }

<br>

## 4. DelayQueue
큐가 대기하는 시간을 지정하여 처리합니다. 대기가 끝난 요소만 제거 가능하며, 큐의 head는 대기가 가장 오래 지난 요소입니다.

생성 방법은 다음과 같습니다. <br>
BlockingQueue<Delayed> queue = new DelayQueue<>(); <br>
	- 이때, 요소로는 Delayed 인터페이스의 구현체만 들어갈 수 있습니다.
  
Delayed 인터페이스 <br>
	- compareTo() 오버라이드 메소드 : 대기 시간을 비교해 큐에 추가 <br>
	- getDelay() 오버라이드 메소드 : 해당 요소가 대기 열에서 제거될 수 있는지 확인. 반환 값이 0 이하일 경우 대기 열에서 제거 가능 

<br>

## 5. SynchronousQueue
삽입 메소드를 호출하면 다른 스레드에서 제거 메소드가 호출 될 때까지 대기하는 큐입니다.따라서 서로 대칭되는 큐가 없으면 대기합니다.<br>
이 큐에는 저장되는 데이터가 없어 내부 용량을 지니지 않고, 
스레드가 없으면 제거에 사용할 수 없는 요소가 없습니다.

---

## Blocking Queue 정리
![image](https://user-images.githubusercontent.com/96156882/209922591-82232635-6942-4f59-859e-bfb6fdacb065.png)

