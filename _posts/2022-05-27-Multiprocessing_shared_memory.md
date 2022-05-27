---
layout: post
title: "멀티 프로세스 데이터 공유하는 방법"
date: 2022-5-27
author: 이한솔
---

이 포스팅에서는 Python을 통해 멀티 프로세스 사용 시 데이터 공유하는 방법에 대해 알아보겠습니다.

> **멀티 프로세스란** 하나의 응용 프로그램을 여러 개의 프로세스로 구성하여 각 프로세스가 하나의 작업(task)을 처리하도록 하는 것입니다.

멀티 프로세스는 여러 자식 프로세스 중 하나에 문제가 발생하면, 다른 프로세스에 영향없이 그 프로세스만 죽기 때문에 안정적으로 프로그램을 운용할 수 있다는 장점이 있습니다.

하지만 각 독립된 메모리 영역을 할당받았기 때문에 공유하는 메모리가 없습니다.<br> 이에 **데이터를 공유하는 방법 세가지**를 알려드리도록 하겠습니다.

---

## **목차**
1. Value
2. Array
3. Dict

---
비교를 위해 간단하게 1부터 1억까지 더하는 프로그램을 만들어보도록 하겠습니다.
```python
def sum(start, end):
    result = 0
    for i in range(start, end):
        result += i
    print(result)
    return result

if __name__ == '__main__':
    start_time = time.time()

    s1 = sum(1, 50000000)
    s2 = sum(50000000, 100000000)

    print("s1+s2=",s1+s2)
    print("걸린 시간 : ", time.time()-start_time)

<결과창>
1249999975000000
3749999975000000
s1+s2= 4999999950000000
걸린 시간 :  5.427370071411133
```

다음은 멀티프로세스를 이용해 더하는 프로그램입니다.
```python
from multiprocessing import Process

def sum(start, end):
    result = 0
    for i in range(start, end):
        result += i
    print(result)
    return result

if __name__ == '__main__':
    start_time = time.time()

    s1 = Process(target=sum, args=(1, 50000000))
    s2 = Process(target=sum, args=(50000000, 100000000))

    s1.start()
    s2.start()

    s1.join()
    s2.join()

    print("걸린 시간 : ", time.time()-start_time)

<결과창>
3749999975000000
1249999975000000
걸린 시간 :  2.9453952312469482
```
멀티프로세스를 이용해 시간은 줄었지만 두 프로세스 결과를 더할 수는 없습니다.

---

## **1. Value**
데이터는 Value, Array, Dict를 사용하여 공유 메모리에 저장 될 수 있습니다.<br>
Value를 이용해 두 프로세스 결과를 더해보도록 하겠습니다.

```python
from multiprocessing import Process, Value, Array
import time


def sum(start, end, s_num):
    result = 0
    for i in range(start, end):
        result += i
    s_num.value = result
    print(result)

if __name__ == '__main__':
    start_time = time.time()

    s1_num = Value('d', 0.0)  # 'd'는 부동 소수점을 나타냄
    s2_num = Value('d', 0.0)
    s1 = Process(target=sum, args=(1, 50000000, s1_num))
    s2 = Process(target=sum, args=(50000000, 100000000, s2_num))

    s1.start()
    s2.start()

    s1.join()
    s2.join()

    print("s1+s2=",s_num1.value + s_num2.value)
    print("걸린 시간 : ", time.time() - start_time)

<결과창>
3749999975000000
1249999975000000
s1+s2= 4999999950000000.0
걸린 시간 :  2.7269644737243652
```
---
## **2. Array**
다음은 Array를 이용해 프로세스의 pid와 연산 결과를 저장해 보도록 하겠습니다.
```python
import multiprocessing as mp
from multiprocessing import Process, Value, Array
import time


def sum(start, end, arr):
    result = 0
    arr[0] = mp.current_process().pid
    for i in range(start, end):
        result += i
    arr[1] = result
    print(arr[:])

if __name__ == '__main__':
    start_time = time.time()

    arr1 = Array('d', range(2))
    arr2 = Array('d', range(2))
    s1 = Process(target=sum, args=(1, 50000000, arr1))
    s2 = Process(target=sum, args=(50000000, 100000000, arr2))

    s1.start()
    s2.start()

    s1.join()
    s2.join()

    print("s1+s2=",arr1[1] + arr2[1])
    print("걸린 시간 : ", time.time() - start_time)

<결과창>
[33540.0, 3749999975000000.0]
[3808.0, 1249999975000000.0]
s1+s2= 4999999950000000.0
걸린 시간 :  2.8744490146636963
```

---

## **2. Dict**
공유 데이터는 Manager()를 이용해 Dictionary로도 저장될 수 있습니다.
```python

```
---

## **결론**
이번 포스팅에서는 멀티프로세스에서 공유 메모리를 이용해 데이터를 공유하는 방법을 알아보았습니다.

이번 포스팅은 여기까지입니다. 감사합니다.
