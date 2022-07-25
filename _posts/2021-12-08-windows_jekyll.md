---
title: "로컬에서 github 블로그 실행하기"
last_modified_at: 2021-12-08
author: Do-soo, KIM

---

보통 블로그에 올릴 포스트를 github 저장소에 push 한 뒤, github에 접속해서 확인하는데 이럴 경우 혹시 의도대로 표시되지 않는 내용이 있거나 수정사항이 있다면
수정 후에 다시 push 해야 하는 번거로움이 있습니다. 

하지만 github 저장소에 push하기 전에 로컬에서 미리 확인할 수 있다면 편리하겠죠?^^

여기서는 Windows 10 환경을 기준으로 설명합니다.



여러분들이 보고 계시는 이 블로그 역시 github 블로그 입니다. github 블로그는 jekyll 이라는 static site generator를 이용해 만들어 졌기 때문에 블로그를 실행하기 위해서는 jekyll이 설치되어야 합니다.

(static site generator에 대해서 간단히 설명하자면 markdown 형식의 텍스트 문서를 웹 사이트로 변환해 주는 엔진이라고 할 수 있습니다.)

jekyll은 ruby라는 스크립트로 만들어졌기 때문에, 먼저 ruby를 설치합니다.

---

#### 1. ruby 설치

[다운로드 사이트](https://rubyinstaller.org/downloads/){: target="_blank"}에 접속해서 설치 파일을 다운로드 합니다.

![image](https://user-images.githubusercontent.com/92565548/145144208-c187c6ea-0ea2-46b1-a39d-edef31481c98.png)

다운로드한 설치 파일을 실행하여 설치를 시작합니다.

Add Ruby executables to your PATH 항목에 체크한 뒤, Install 버튼을 클릭합니다.

체크하지 않으면 나중에 따로 환경변수를 설정해야 하는 번거로움이 있으니, 잊지 말고 체크하시길....

![image](https://user-images.githubusercontent.com/92565548/145144379-f2d3f32e-07b3-4998-995a-2f7be4391c43.png)


MSYS2 development toolchain 항목에 체크가 되어 있는지 확인하고 Next 버튼을 클릭합니다.

MSYS2는 cmd 창에서 ruby 명령어를 사용할 수 있게 해주는 것이기 때문에, 나중에 따로 설치하지 않으려면 여기서 꼭 체크하고 넘어가야 합니다.

![image](https://user-images.githubusercontent.com/92565548/145144598-9afe156e-f8c1-44f9-a812-333bd7980094.png)

설치가 진행되고 마지막 설치 화면에서 Run 'ridk install' to setup MSYS2 ~ ...... 항목이 체크되어 있는지 확인합니다. 체크하지 않은 채 설치를 종료하면 cmd 창을 실행하고 risk install이라는 명령어를 실행해야 하므로 매우 귀찮습니다. 귀찮은 게 싫으신 분들은 반드시 체크하고 넘어가시길^^

![image](https://user-images.githubusercontent.com/92565548/145145172-845e6408-9e87-4a07-bbd6-1dbc8b519e28.png)


앞에서 Finish 버튼을 클릭했다면, 다음과 같은 화면이 보일 겁니다.

![image](https://user-images.githubusercontent.com/92565548/145145385-1ab8f62c-5e3c-444a-95a1-ed5badbbccfb.png)

순차적으로 1, 2, 3을 입력하여 각각 설치를 진행합니다.

먼저 1을 입력하고 설치를 진행해 보겠습니다. 설치가 완료되면 다음과 같은 화면이 보입니다.

![image](https://user-images.githubusercontent.com/92565548/145145599-442f3764-f82b-4596-a41b-003988dce182.png)

대충 해석해 보면, MSYS2를 처음 실행했고, 필수사항을 적용하려면 쉘을 재시작 하라는 의미입니다.
시키는 대로 하면 되죠^^

enter 키를 눌러서 종료한 다음, cmd 창을 열어서 risk install 명령어를 실행합니다.

![image](https://user-images.githubusercontent.com/92565548/145145925-1ef9606c-7a47-409e-89af-7ad5bad3ee10.png)

그러면, 아까 봤던 화면에 또 보이는군요.

![image](https://user-images.githubusercontent.com/92565548/145146089-16c4a476-5ca0-43b1-8cd3-f8a315ba5325.png)

아까는 1번을 설치했으니 이번에는 2번을 입력하고 설치합니다. 2번이 끝나면 마지막으로 3번을 입력하고 설치를 계속 진행하면 됩니다.

![image](https://user-images.githubusercontent.com/92565548/145146141-51529d81-b342-4802-8bda-2c375e592ef3.png)

![image](https://user-images.githubusercontent.com/92565548/145146202-5a234bae-3848-4fad-8445-ed51f251dc3c.png)


3번까지 설치가 끝났으면, 종료하고 설치가 제대로 되었는지 확인해 봅니다.

ruby -v 명령어를 실행하면 설치된 버전이 출력되는 것을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/92565548/145146476-8b82f06e-8227-4ae3-bf49-99eb0e8a0b6f.png)

---

#### 2. jekyll 설치

먼 길을 돌고돌아 이제 드디어 주인공 jekyll을 설치해 볼 차례입니다.

나름 이번 포스팅에서 주인공임에도 불구하고 설치는 정말 허무하리만큼 간단합니다.

cmd창을 열어 gem install jekyll bundler 명령어를 실행합니다.

![image](https://user-images.githubusercontent.com/92565548/145146766-231d20b9-543f-4791-b448-90c2292a939f.png)

믿을 수 없으시겠지만, 이렇게 해서 jekyll 설치는 끝난 겁니다. 정말 허무하죠?

---

#### 3. 블로그 실행

jekyll 설치도 끝났으니, 이제 마지막으로 블로그를 실행해 봐야겠죠?

cmd 창을 열어 github 저장소로부터 clone한 폴더로 이동하여 jekyll serve 명령어를 실행합니다.

![image](https://user-images.githubusercontent.com/92565548/145147138-4bff0160-0bb8-4f62-a075-eb8cad2ab647.png)

이제 실행이 되었으니, 블로그에 접속해 보도록 하겠습니다.

http://127.0.0.1:4000로 접속하면 아래와 같이 블로그를 볼 수 있습니다.

![image](https://user-images.githubusercontent.com/92565548/145147324-f9dbd791-bfcc-453e-ab71-8faf2937a7fc.png)

잘 보이시나요?
긴 글 읽으시느라 수고하셨습니다.^^

오늘 포스팅은 여기서 마치겠습니다.












