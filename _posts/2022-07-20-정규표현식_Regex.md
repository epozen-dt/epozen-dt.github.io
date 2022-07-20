---
layout: post
title: "정규표현식 - Regex 정리"
date: 2022-7-20
author: 권재우
---

# 정규표현식 - Regex 정리

이번 포스팅은 불규칙속에서 규칙을 찾는 방법인 정규표현식(Regex)에 대해서 포스팅 하겠습니다.


# Regex란?
Regex는 "Regular expression"의 약자로 "정규표현식" 또는 "정규식"으로 불립니다. 
정규표현식은 많은 텍스트 편집기와 프로그래밍 언어에서 문자열의 검색과 치환을 위해 사용합니다.   
자바, 파이썬, C언어 등 프로그래밍 언어는 정규표현식의 표준 라이브러리를 제공합니다.   
(python에서는 re모듈이 정규표현식 라이브러리)  

![정규표현식1](https://user-images.githubusercontent.com/102636902/179650033-1481539a-b21b-46c8-aba5-e899a79d8200.png)


정규표현식에서 사용되는 기호를 Meta문자라고 표현합니다.
- Meta문자 : 정규표현식 내 특별한 의미를 갖는 문자 기호  
ex) |, [], (), 앵커(`^$\) 등

정규식에 위 메타 문자를 사용하면 특별한 의미를 가지게 됩니다.  
그 Meta문자를 간단하게 정리하면 아래 표와 같습니다. 
(x,y가 문자를 나타냄)
|표현식|의미|
|----|---|
|^x|문자열의 시작을 표현하며 x 문자로 **시작**됨을 의미|
|x$|문자열의 종료를 표현하며 x 문자로 **종료**됨을 의미|
|.x|임의의 한 문자의 자리수를 표현하며 문자열이 x 로 끝난다는 것을 의미|
|x+|**반복**을 표현하며 x 문자가 한번 이상 반복됨을 의미|
|x?|**존재여부**를 표현하며 x 문자가 존재할 수도, 존재하지 않을 수도 있음을 의미|
|x*|반복여부를 표현하며 x 문자가 0번 또는 그 이상 반복됨을 의미|
|x\|y|or 를 표현하며 x 또는 y 문자가 존재함을 의미|
|(x)|그룹을 표현하며 x 를 그룹으로 처리함을 의미|
|(x)(y)|그룹들의 집합을 표현하며 앞에서 부터 순서대로 번호를 부여하여 관리하고 x, y 는 각 그룹의 데이터로 관리|
|(x)(?:y)|그룹들의 집합에 대한 예외를 표현하며 그룹 집합으로 관리되지 않음을 의미 |
|x{n}|반복을 표현하며 x 문자가 n번 반복됨을 의미|
|x{n,}|반복을 표현하며 x 문자가 n번 이상 반복됨을 의미|
|x{n,m}|반복을 표현하며 x 문자가 최소 n번 이상 최대 m 번 이하로 반복됨을 의미|

## 문자 클래스
정규표현식에서 [abc]라면 이 표현식의 의미는 "a, b, c 중 한 개의 문자와 매치"를 뜻합니다. 그 사이에 하이픈(-)를 사용하면 두 문자 사이의 범위를 의미합니다.   
예를들어 가장많이 사용하는 정규식으로
> [a-zA-Z] : 알파벳 모두(대소문자 전부)  
> [0-9] : 숫자  

위 정규식을 가장 많이 사용합니다.

이렇게 자주 사용하는 정규식은 별도의 다른 표기법으로 표현할 수도 있습니다.  
(메타클래스 안에서는 주의해야할 문자가 있는데 그건 바로 "^" 입니다.
문자클래스 안에서 "^"의 의미는 반대(not)라는 의미를 갖습니다.)

|표현식|다른표현식|의미|
|---|------|---|
|[0-9]|\d|숫자와 매치
|[^0-9]|\D|숫자가 아닌 것과 매치|
|[ \t\n\r\f\v]|\s|whitespace 문자와 매치|
|[^ \t\n\r\f\v]|\S|whitespace 문자가 아닌 것과 매치|
|[a-zA-Z0-9_]|\w|문자+숫자(alphanumeric)와 매치|
|[^a-zA-Z0-9_]|\W|문자+숫자(alphanumeric)가 아닌 문자와 매치|

## Flag
정규표현식을 사용할 때 Flag라는 것이 존재합니다. 
이 Flag는 정규식을 얼마나 자주 처리할지 정하는 기호입니다. 
|Flag|의미|
|---|------|
|g|"Global"의 표현하며 대상 문자열내에 모든 패턴들을 검색하는 것을 의미|
|i|"Ignore case"를 표현하며 대상 문자열에 대해서 대/소문자를 식별하지 않는 것을 의미|
|m|"Multi line"을 표현하며 대상 문자열이 다중 라인의 문자열인 경우에도 검색하는 것을 의미|


# 실전에서 자주사용되는 정규표현식

1. 한글사용  
/[가-힣]/

2. 전화 번호  
ex) '02-111-2222' > /\d{2,3}-\d{3,4}-\d{4}/g

3. 휴대폰 번호  
ex) '010-1234-5678' > /^\d{3}-\d{3,4}-\d{4}$/

4. 웹사이트 주소  
ex) 'https://zeuskwon.com' > /https?:\/\/[\w\-\.]+/g

5. 이메일 주소  
ex) 'zeuskwon@epozen.com' > /^&#91;0-9a-zA-Z]([-_\.]?[0-9a-zA-Z])*@&#91;0-9a-zA-Z](-_\.]?[0-9a-zA-Z])*\.[a-zA-Z]{2,3}$/

6. 문자가 특정 단어로 끝나는지 검사  
ex) index.html > /html$/

7. ​숫자로만 이루어져 있는지 검사  
ex) index.html > /html$/

8. 아이디 사용 검사  
ex) abcd123 > /^[A-Za-z0-9]{4,10}$/


# 정규표현식을 테스트해 볼 수 있는 사이드
[정규표현식테스트](https://regexr.com/)

![정규표현식2](https://user-images.githubusercontent.com/102636902/179666922-e2ff3220-18ed-4c18-889a-8bfb9a93ff34.png)

# [Python]  re 모듈 사용법
파이썬에서 정규표현식을 사용할 때 내장 모듈인 re를 사용하고 있습니다.   
간단하게 파이썬의 re모듈에서 제공하는 함수의 쓰임새를 예제와 함께 설명하겠습니다.

- match(패턴, 문자열, Flag)  

match()는 문자열의 처음부터 시작해서 작성한 패턴이 일치하는지 확인합니다.

``` python
import re

print(re.match('a', 'ab'))
print(re.match('a', 'ab'))
print(re.match('a', 'bba'))
print(re.match('a', 'ba'))

# <re.Match object; span=(0, 1), match='a'>
# <re.Match object; span=(0, 1), match='a'>
# None
# None
```
예시의 1,2번의 시작은 a로 시작하여 매칭이 된 것으로 확인되는데 3,4번은 시작이 b로 시작하므로 매칭이 안 된 것을 확인할 수 있습니다.

- fullmatch(패턴, 문자열, Flag)

fullmatch()는 문자열에 시작과 끝이 정확하게 패턴과 일치할 때 반환합니다.
match()와는 다르게 fullmatch()는 처음과 끝이 정확하게 일치해야합니다.

``` python
import re

print(re.fullmatch('a', 'a'))
print(re.fullmatch('a', 'aaa'))
print(re.fullmatch('a', 'ab'))
print(re.fullmatch('a', 'ba'))
print(re.fullmatch('a', 'baa'))


# <re.Match object; span=(0, 1), match='a'>
# None
# None
# None
# None
```

- search(패턴, 문자열, Flag)

search()는 match()와 유사하지만 패턴이 문자열의 처음부터 일치하지 않아도 괜찮습니다.

``` python
import re


print(re.search('a', 'ab'))
print(re.search('a', 'ab'))
print(re.search('a', 'bba'))
print(re.search('a', 'ba'))

# <re.Match object; span=(0, 1), match='a'>
# <re.Match object; span=(0, 1), match='a'>
# <re.Match object; span=(2, 3), match='a'>
# <re.Match object; span=(1, 2), match='a'>
```

패턴과 일치하는 문자열을 전부 찾아서 결과를 반환해줍니다.

- findall(패턴, 문자열, Flag)

findall()은 문자열 안에 패턴에 맞는 케이스를 전부 찾아서 리스트로 반환합니다

``` python
import re

print(re.findall('a', 'a'))
print(re.findall('a', 'aba'))
print(re.findall('a', 'baa'))
print(re.findall('aaa', 'aaaaa'))
print(re.findall('aaa', 'aaaaaa'))
print(re.findall('\d', '숫자123이 이렇게56 있다8'))
print(re.findall('\d+', '숫자123이 이렇게56 있다8'))


# ['a']
# ['a', 'a']
# ['a', 'a']
# ['aaa']
# ['aaa', 'aaa']
# ['1', '2', '3', '5', '6', '8']
# ['123', '56', '8']
```

- split(패턴, 문자열, 최대 split 수, Flag)

split은 문자열에서 패턴이 맞으면 이를 기점으로 리스트로 쪼개는 함수입니다.

``` python
import re

print(re.split('a', 'abaabca'))
print(re.split('a', 'abaabca', 2))


# ['', 'b', '', 'bc', '']
# ['', 'b', 'abca']
```

## 예제

정규표현식을 사용해서 전화번호만 추출해보겠습니다.

``` python
import re
 
text = "문의사항이 있으면 032-232-3245 으로 연락주시기 바랍니다."
 
regex = re.compile(r'(\d{3})-(\d{3}-\d{4})')
matchobj = regex.search(text)
areaCode = matchobj.group(1)
fullNum = matchobj.group()
print(fullNum) # 032-232-3245
print(areaCode) # 032
```
위 코드를 통해 전화번호 전체를 출력하고
.group 메서드를 사용해서 지역번호만 추출할 수 있었습니다.


python의 re라이브러리를 짧게 소개시켜 드렸습니다. 

이것으로 이번 포스팅을 마치겠습니다.


## 출처
- https://hamait.tistory.com/342 [HAMA 블로그:티스토리]
- https://inpa.tistory.com/entry/JS-%F0%9F%93%9A-%EC%A0%95%EA%B7%9C%EC%8B%9D-RegExp-%EB%88%84%EA%B5%AC%EB%82%98-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-%EC%89%BD%EA%B2%8C-%EC%A0%95%EB%A6%AC#%EC%A0%95%EA%B7%9C%EC%8B%9D_%EC%8B%A4%EC%A0%84_%EC%98%88%EC%A0%9C
- https://brownbears.tistory.com/506
