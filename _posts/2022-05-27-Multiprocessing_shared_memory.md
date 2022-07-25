---
title: "멀티 프로세스 데이터 공유하는 방법"
last_modified_at: 2022-5-27
author: 이한솔
---

이 포스팅에서는 Python에서 멀티 프로세스 사용 시 데이터를 공유하는 방법에 대해 알아보겠습니다.

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

    p1 = Process(target=sum, args=(1, 50000000))
    p2 = Process(target=sum, args=(50000000, 100000000))

    p1.start()
    p2.start()

    p1.join()
    p2.join()

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
Value를 이용해 두 프로세스 결과를 더해보도록 하겠습니다.<br>
Value를 이용해 값을 args로 넘겨줍니다. 이때 사용되는 'd' 는 부동 소수점을 나타내고, 'i' 는 부호 있는 정수를 나타냅니다.

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

    s1_num = Value('d', 0.0)
    s2_num = Value('d', 0.0)
    p1 = Process(target=sum, args=(1, 50000000, s1_num))
    p2 = Process(target=sum, args=(50000000, 100000000, s2_num))

    p1.start()
    p2.start()

    p1.join()
    p2.join()

    print("s1+s2=",s_num1.value + s_num2.value)
    print("걸린 시간 : ", time.time() - start_time)

<결과창>
3749999975000000
1249999975000000
s1+s2= 4999999950000000.0
걸린 시간 :  2.7269644737243652
```
두 결과를 value로 받아 값을 더한 결과입니다.

---
## **2. Array**
다음은 Array를 이용해 저장해 보도록 하겠습니다.
args에 index값을 전달해 arr에 연이어 값을 저장해보겠습니다.
```python
import multiprocessing as mp
from multiprocessing import Process, Value, Array
import time


def sum(start, end, index, arr):
    result = 0
    for i in range(start, end):
        result += i
    arr[index] = result
    print(arr[:])

if __name__ == '__main__':
    start_time = time.time()

    arr = Array('d', range(2))
    s1 = Process(target=sum, args=(1, 50000000, 0, arr))
    s2 = Process(target=sum, args=(50000000, 100000000, 1, arr))

    s1.start()
    s2.start()

    s1.join()
    s2.join()

    print("s1+s2=",arr[0] + arr[1])
    print("걸린 시간 : ", time.time() - start_time)

<결과창>
[1249999975000000.0, 3749999975000000.0]
s1+s2= 4999999950000000.0
걸린 시간 :  2.8744490146636963
```
연산 결과를 array로 받아 연산 결과값을 더한 결과입니다.

---

## **3. Dict**
공유 데이터는 Manager를 이용해 Dictionary로도 저장될 수 있습니다.<br>
중첩 딕셔너리에 각 프로세스의 pid, 걸린 시간, 결과값을 출력하도록 해보겠습니다.
```python
import multiprocessing as mp
from multiprocessing import Process, Manager
import time


def sum(start, end, shared_info):
    id = f'sum-{mp.current_process().pid}'
    result = 0
    start_time = time.time()
    for i in range(start, end):
        result += i
    end_time = time.time()
    shared_info[id] = {'time': (end_time - start_time), 'result': result}

if __name__ == '__main__':
    start_time = time.time()

    procs = []
    with Manager() as manager:
        shared_info = manager.dict()

        p1 = Process(target=sum, args=(1, 50000000, shared_info,))
        p2 = Process(target=sum, args=(50000000, 100000000, shared_info,))

        procs.append(p1)
        procs.append(p2)

        p1.start()
        p2.start()

        p1.join()
        p2.join()

        print(shared_info)
        
        result = 0
        for key, value in shared_info.items():
            result += shared_info[key]['result']

        print(result)

    print("걸린 시간 : ", time.time()-start_time)

<결과창>
{'sum-30500': {'time': 2.9461288452148438, 'result': 1249999975000000}, 
'sum-39700': {'time': 3.4378139972686768, 'result': 3749999975000000}}
4999999950000000
걸린 시간 :  3.0267534255981445
```
중첩 딕셔너리를 이용하여 각 프로세스의 pid를 기준으로 프로세스 실행 시간과 연산 결과를 저장하였습니다.

---

이번 포스팅에서는 멀티프로세스의 데이터를 공유하는 방법을 알아보았습니다.<br> 하나의 프로그램에 속하는 프로세스들 사이의 변수를 공유할 수 없다는 멀티프로세스의 단점을 보완할 수 있는 유용한 방법인 것 같습니다.

이번 포스팅은 여기까지입니다. 감사합니다.
