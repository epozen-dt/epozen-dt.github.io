---
title: "Unittest - Python 단위 테스트 프레임워크 사용법"
last_modified_at: 2022-6-30
author: 이한솔
---

이 포스팅에서는 Python 단위 테스트 프레임워크인 unittest 모듈 이용 방법에 대해 알아보겠습니다.

개발 단계에서 단위 테스트는 필수적입니다. 

Python에서는 내장 모듈인 unittest를 통해서 별도의 설치 없이 비교적 간단하게 단위 테스트를 작성할 수 있습니다.

간단한 예제를 통해 unittest 사용법을 알려드리겠습니다.

 
---

## **Test case**

테스트 케이스는 테스트의 개별 단위로 특정 입력 모음에 대해 특정 결과를 확인할 수 있습니다.

unittest는 베이스 클래스인 TestCase를 지원하고 이 클래스는 새로운 테스트 케이스를 만드는데 사용됩니다.

**이때, 각각의 테스트 함수의 이름은 test_로 시작해야합니다.**

test_로 시작하는 메소드들은 각각 실행해 결과를 확인 할 수 있습니다.

```python      
import unittest

class Test(unittest.Testcase):

    def test_equal(self):
        ...
```

---

### **Assert 메서드**
TestCase 클래스는 값을 검사하고 실패를 보고하기 위해 몇 개의 **assert 메서드**를 제공합니다.

몇가지 예제를 보여드리겠습니다.

```python   
import unittest


def cal_equal(a, b):
    return a+b


def cal_bool(a, b):
    if a == b:
        return True
    else:
        return False


class Test(unittest.TestCase):

    def test_equal(self):
        # assertEqual(a,b)가 검사하는 내용 a == b
        self.assertEqual(cal_equal(1, 2), 3)

    def test_bool(self):
        # assertTrue(x)가 검사하는 내용 bool(x) is True
        self.assertTrue(cal_bool(1, 1))

if __name__ == '__main__':
    unittest.main()
```
실행하면 테스트 시간과 결과가 출력됩니다.

![image](https://user-images.githubusercontent.com/96156882/176571414-564718c3-a0d3-437a-9145-01eae1883338.png)

이외에 값을 확인하는 다른 assert 메소드는 다음과 같습니다.

![image](https://user-images.githubusercontent.com/96156882/176571652-ca87abc7-0e6c-4f16-9a23-5f00cfacf00c.png) 

---

### **setUp**
만약 init과 같이 메소드를 호출 하기 이전에 호출되는 메소드가 필요하다면 **setUp()** 을 활용할 수 있습니다.
```python   
def cal_equal(a, b):
    return a+b

class Test(unittest.TestCase):

    def setUp(self) -> None:
        self.a = 1
        self.b = 2

    def test_equal(self):
        self.assertEqual(cal_equal(self.a, self.b), 3)
```


---

### **Assert 메소드 - 예외 발생**
또한 **예외, 경고, 로그 메시지의 발생**을 검사할 수 있습니다.

assertRaises()는 예외가 발생하는지 확인합니다. 

```python   
import unittest

def cal_bool(a, b):
    if a == b:
        return True
    else:
        raise Exception("result is not True")


class Test(unittest.TestCase):

    def test_exception(self):
        # exception이 발생하는 경우
        with self.assertRaises(Exception): cal_bool(1, 2)

    def test_exception2(self):
        # exception이 발생하지 않는 경우
        with self.assertRaises(Exception): cal_bool(1, 1)

if __name__ == '__main__':
    unittest.main()

```
이처럼 두 개 중 예외가 발생하지 않는 하나의 결과가 실패했다는 것을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/96156882/176573956-c195b622-9c44-477a-876f-182d2e155ba1.png)
![image](https://user-images.githubusercontent.com/96156882/176573977-24c18e59-c821-4c2c-bc39-8700d2f65a97.png) 

이밖에도 경고, 로그 메시지 발생을 검사하는 메소드는 다음과 같습니다.
![image](https://user-images.githubusercontent.com/96156882/176576987-1b240ae2-4a83-4808-8ef2-ad756f349fd3.png) 

---

이번 포스팅에서는 unittest를 이용해 파이썬에서 간단하게 단위테스트를 할 수 있는 방법에 대해 알아보았습니다. 소개해드린 메소드 외에도 다양한 기능이 있으니 참고자료의 링크를 확인해주시면 될 것 같습니다. 

---

### 참고자료
https://docs.python.org/ko/3/library/unittest.html

