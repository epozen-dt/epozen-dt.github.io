---
title: "Docker Remote API 내부 접속"
last_modified_at: 2023-07-31
author: 최선빈
---

본 포스팅은 Docker RemoteAPI 컨테이너 내부 접속에 대한 내용입니다.

---
&nbsp;

> ## Docker Exec(Docekr Remote API)

&nbsp;

- exec_run()은 Docker Remote API Python SDK에서 컨테이너 내부에 명령어를 전달하는 메소드
- 'Docker exec ~' 명령어와 같음
- return 값 -> exit_code와 명령어 수행 결과를 담은 객체(Generator, Bytes, Tuple, Socket)
- 서버에서 exec 명령어('docker exec -t test-vol-l ls) 실행 시 클라이언트 코드는 아래와 같음

      _, result = test_container.exec_run(tty=True, cmd='ls')
      print(result.decode('utf-8'))


- 출력 결과

        bin dev hello.py lib media opt root sbin sys tmp var
        boot etc home lib64 mnt proc run srv test usr

---
&nbsp;

> ## 주요 파라미터

&nbsp;

- CMD
  - 명령어 입력 (input type: str or list)

- stdout, stderr, stdin
  - 표준 입출력, 에러 (input type: bool), 기본값(stdout: True, stderr: True, stdin: False)

- user
  - 명령어를 수행하는 user (input type: str), 기본값 (user='root')

- stream
  - exec_rin() 수행결과 stream (input type bool), 기본값 (stream=False)
  - 'stream=True'로 설정 시, generator(bytes) 반환
  - 'stream=Fasle'로 설정 시, bytes 문자열 반환

- socket
  - 읽기/쓰기가 가능한 SSLSoket 객체 반환 (input type: bool), 기본값 (socket= False)
  - SSLSocket 객체의 send(), recv()를 통해 데이터 읽기/쓰기 가능

- environmnet
  - 환경 변수 입력 (input type: dict or list)
  - ex. environment=['PASSWORD=XXX', ...], environment={'PASSWORD':'XXX',...}

- workdir
  - 명령어(cmd)를 실행할 경로 설정 (input type: str)
  - ex. workdir='/home/test/...'

- tty
  - 수행 결과를 pseudo-TTY로 출력 (input type: bool), 기본값(tty=False)

---

> ## 실행 결과

&nbsp;

- 'ls -l'(bytes 출력)

        _, result = test_container.exec_run(cmd='ls -l',)
        prunt(result.decode('utf-8'))

- 콘솔 출력 결과

        total 4
        drwxr-xr-x.    1  root  root  179  dec  21  2021  bin
        drwxr-xr-x.    2  root  root    6  dec  21  2021  boot
        drwxr-xr-x.    5  root  root  340  Aug  26  14:04  dev
        drwxr-xr-x.    1  root  root   66  Aug  26  14:03  etc
        -rw-rw-r--.    1  root  root  121  Aug  22  15:37  hello.py
        drwxr-xr-x.    2  root  root    6  dec  21  2021  home
        drwxr-xr-x.    1  root  root   41  dec  21  2021  lib

* hello.py는 임의로 컨테이너에 삽입한 파일

- 'python hello.py' (generator 출력)

        _, result = test_container.exec_run(cmd='python hello.py', tty=True, stream=True)
        
        *helllo.py
        while True:
            try:
                print(next(result).decode('utf-8'))
            except StopIteration:
                break

- 콘솔 출력 결과
  - hello.py는 2초 간격으로 날짜가 포함된 문자열을 출력하는 프로그램 (프로그램 종료 시까지)


      2022-08-26 18:17:26.666365 hello world

      2022-08-26 18:17:28.666365 hello world

      2022-08-26 18:17:30.666365 hello world

- 'cd /test'

        _, result = test_container.exec_run(cmd='ce /test')
        
        OCI runtime exec failed: exec failed:
        : exec: "cd": executable file not found in $PATH: unknown

- '/bin/bash -c 'cd /test; ls; cp test.txt test-2.txt; ls;' (bash를 사용한 filesystem 제어)

        _, result = test_container.exec_run(cmd="/bin/bash -c 'cd /test; ls; cp test.txt test-2.txt; ls;'", tty=True, stream=True)


- 콘솔 출력 결과

        test.txt

        test-2.txt  test.txt

- 컨테이너 내부 접속 후 확인

        root@c5059143748e:/test$ ls
        test.txt
        
        root@c5059143748e:/test$ ls
        test-2.txt   test.txt

- socket=True로 설정 (SSLSocket 객체 활용)
  - /bin/bash로 시작하는 socket 객체 생성 후 send(), recv()로 통신 시도

        _, socket = test_con.exec_run(cmd'/bin/bash', socket=True, user='root', tty=True, stdin=True)

        while True:
          input1 = input(socket recv().decode('utf-8', 'ignore')) + '\n'
          socket.send(input1.encode('utf-8'))
        socket.close()



        root@c5059143748e:/test$ ls
        ls
        ls
        bin  dev  hello.py  lib  media  opt  root  sbin  sys  tmp  var
        boot  etc  home  lib64  mnt  proc  run  srv  test  usr
        ls -l
        ls -l
        root@c5059143748e:/test$ ls -l
        ls

  
----
    &nbsp;

> ##  정리

&nbsp;

- exec_run()
  - 단발성 수행
  - 내부 접속 시 흔히 사용하느 bash 기능 사용에 지장
  - 사용하는 명령어가 한정적인 경우 적합 (bash가 필요 없는 경우)

- exec_run(socket=True)
  - 소켓 연결 종료 시까지 연속적 수행
  - bash로 시작하는 소켓 생성 시, 내부 접속 기능 구현 가능성 있음



