---
title: "word_counting"
last_modified_at: 2023-04-27
author: 김혜원
---

본 포스팅은 word counting에 대한 내용입니다.

---
&nbsp;

> ## Word Counting?

&nbsp;

Word Counting은 어떤 문장이 있을 때 공백을 기준으로 구분된 단어의 수를 세는 것을 말합니다[[1]](https://en.wikipedia.org/wiki/Word_count)[[2]](https://www.languagehumanities.org/what-is-word-count.htm). NLP(Natural Language Processing, 자연어처리) 분야에서는 구분된 단어를 이용하여 다른 언어로 번역하는데에 가장 많이 사용하고 있습니다. 이외에도 법적 절차나 저널리즘 등 특정 단어 수 내에서 글을 작성하는 경우 단어 수 세기를 사용합니다.
기호의 유무, 숫자의 유무, 한/영 복합어 등은 세지않는 경우도 있기 때문에 단어 카운터마다 계산되는 갯수 차이가 있습니다. 


&nbsp;

>## 예제 코드

&nbsp;


하기 예제는 문장에서 한글과 영어 단어를 구분하여 단어의 개수를 알려주는 코드입니다[[3]](https://codingspooning.tistory.com/entry/Python-%EC%A0%95%EA%B7%9C%ED%91%9C%ED%98%84%EC%8B%9D%EC%9C%BC%EB%A1%9C-%ED%95%9C%EA%B8%80-%EC%B6%94%EC%B6%9C%ED%95%98%EA%B8%B0-%EB%AC%B8%EC%9E%90%EC%97%B4-text-%EC%A0%84%EC%B2%98%EB%A6%AC#ref). 



    import re
  
    def count_korean_and_english_words(text):
      # 한글 단어 추출을 위한 정규식
      korean_regex = re.compile('[가-힣]+')

      # 입력 문자열에서 단어 추출
      words = text.split()
      print(words)

      # 한글 단어와 영어 단어 개수 초기화
      korean_count = 0
      english_count = 0

      # 각 단어가 한글인지 영어인지 구분하여 개수 세기
      for word in words:
          if korean_regex.match(word):
              korean_count += 1
          else:
              english_count += 1

      # 한글 단어 개수와 영어 단어 개수 반환
      return korean_count, english_count

    text = '한글과 English가 섞여 있는 sentence입니다. 어떻게 word counting을 할 수 있을까요?'

    korean_count, english_count = count_korean_and_english_words(text)
    print(korean_count)
    print(english_count)




-------
> 참고문헌

[1] [word count 정의1](https://en.wikipedia.org/wiki/Word_count)

[2] [word count 정의2](https://www.languagehumanities.org/what-is-word-count.htm)

[3] [텍스트 전처리 예제](https://codingspooning.tistory.com/entry/Python-%EC%A0%95%EA%B7%9C%ED%91%9C%ED%98%84%EC%8B%9D%EC%9C%BC%EB%A1%9C-%ED%95%9C%EA%B8%80-%EC%B6%94%EC%B6%9C%ED%95%98%EA%B8%B0-%EB%AC%B8%EC%9E%90%EC%97%B4-text-%EC%A0%84%EC%B2%98%EB%A6%AC#ref)