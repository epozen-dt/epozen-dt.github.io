---
title: "Oracle VirtualBox로 가상환경 구성하기"
last_modified_at: 2024-12-18
author: 이현지
---

<br>
지난 포스트에선 Oracle의 VirtualBox 소개 및 설치에 대한 내용을 다뤘습니다.<br>
VirtualBox에서는 다양한 운영 체제를 설치하여 호스트 머신에서 실행할 수 있습니다.<br>
이번에는 VirtualBox에 운영체제를 설치하여 가상환경을 구성하는 방법에 대해 알아보도록 하겠습니다.<br>
<br>

# VirtualBox로 가상환경 구성하기

VirtualBox를 실행하여 새로만들기를 클릭한 다음 가상머신의 이름, 가상머신이 저장될 폴더, 설치할 운영체제의 ISO 파일을 설정해줍니다.<br>(ISO 파일은 사전에 다운로드)
<br>

![가상머신 만들기](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FblPmyG%2FbtsKz3SMFqC%2FYn2uapb8EnGWEGqKurlxXK%2Fimg.png)

<br>
무인 게스트 OS 설치가 가능한 경우 아래 이미지와 같이 정보를 입력합니다. 
<br>

![무인 게스트 OS 설치](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEtmee%2FbtsKCfRvlTG%2Fwz0KNdNnzSKwGLHaPEy62K%2Fimg.png)

<br>
기본 메모리와 프로세서도 설정할 수 있습니다.
<br>

![기본메모리 프로세서 설정](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FR2SJN%2FbtsKz36hWs1%2F4GJzgoPDEv49L8QrYe8ap0%2Fimg.png)

<br>
하드디스크 크기를 지정합니다. 
<br>

![하드디스크 크기 지정](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbxuZoT%2FbtsKzUaxNI3%2FgC08Kdi8ChThIRKknOgW6k%2Fimg.png)

<br>
설정을 마치고 완료를 클릭하면 가상머신이 생성됩니다.
<br>

![가상머신 생성됨](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Ft2fO7%2FbtsKB5nUvqn%2FnUv4ZyOnyJLxizM0enLMc0%2Fimg.png)

<br>
이제 운영체제 설치를 위해 생성된 가상머신을 선택하여 설정 - 저장소 - 가상 광 디스크 선택/만들기를 클릭합니다. 
<br>

![가상광디스크만들기](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FQdaQP%2FbtsKBiBAKWo%2FvjbXUXFSsH1wfUj9V69wU1%2Fimg.png)

<br>
추가- 운영체제에서 ISO를 선택한 후 '열기'를 클릭한 다음 확인을 눌러줍니다. 
<br>

![ISO선택](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbHeI84%2FbtsLj3RHPji%2FJntDpsVfpnAC7jjIKX6kk1%2Fimg.png)
<br>

![확인](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdamUVd%2FbtsKAJl68vK%2FiBJj8b3mCnMLsUQwn74Iq0%2Fimg.png)
<br>

가상머신을 클릭한 다음 시작을 클릭하면 가상머신이 실행됩니다. 
<br>

![가상머신 실행](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbxgSpH%2FbtsKBiVVx1r%2FnntQjoBLgrABQANvpaN4b0%2Fimg.png)
<br>

운영체제 설치화면이 보이면 Enter를 누릅니다. 
<br>

![운영체제설치](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbNSht6%2FbtsKChWfJli%2FGkdNzshP03m0Nq5LkqYKbk%2Fimg.png)
<br>

운영체제 설치가 완료되면 언어를 설정하고 계속 진행을 클릭합니다. 
<br>

![언어설정](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbcgTyt%2FbtsKCek0ymv%2FSSowRV3Se0rxAYrKYkXRW0%2Fimg.png)
<br>

네트워크 및 호스트명을 클릭하여 이더넷 끔 -> 켬으로 변경하고 좌측 상단 완료를 누릅니다. 호스트명도 변경할 수 있습니다. 
<br>

![네트워크및호스트명](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtCd3W%2FbtsKBk0xAs6%2Fhnt7Mr7FA6ZoapWTprCB1K%2Fimg.png)
<br>

![이더넷끔](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwWEF7%2FbtsKzUB1hUp%2F82XWSt4VY6bYKj6THbrCe1%2Fimg.png)
<br>

Kdump 활성화 체크를 해제한 후 완료를 클릭합니다. 
<br>

![kdump](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQTXOJ%2FbtsKB4wgHuo%2FqTalSlNCKGQ4OWDi5pELeK%2Fimg.png)
<br>

설치대상 - 로컬 표준 디스크를 선택한 후 완료를 클릭합니다.
<br>

![로컬 표준디스크 선택](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoFg3x%2FbtsKz27QJ65%2FktdT9NaH5OrWLWgzkkJeHK%2Fimg.png)
<br>

우측 하단 설치 시작버튼을 클릭합니다. 
<br>
![설치 시작](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbm90XX%2FbtsKB7fvUlL%2FZrRjaXrYdHJkRfcISNnOzK%2Fimg.png)
<br>

하단에 설치 진행 바가 표시되며, ROOT암호를 설정하고 사용자를 생성합니다. 
<br>

![루트암호 설정 사용자 생성](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbc5bgm%2FbtsKCha6xmQ%2F5KJ1ugOqFoa0hFlRrcj61k%2Fimg.png)
<br>

설치가 완료되면 우측 하단의 재부팅을 클릭합니다.
<br>

![재부팅](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb26DL6%2FbtsKBMo8zFY%2FPwee53gR5KQgPjuvj0zIsK%2Fimg.png)
<br>

ROOT 계정 또는 일반 사용자 계정으로 로그인 합니다.
<br>
![로그인](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbXc8tI%2FbtsKABIZv64%2Flccvh5NHO9VwmI82wd0fo1%2Fimg.png)
<br>

<br>
이상으로 VirtualBox에 대한 설치 및 사용 방법에 대한 포스트를 마치겠습니다.

> Reference
- https://velog.io/@kiyoog02/%EB%A6%AC%EB%88%85%EC%8A%A4-CentOS-%EC%84%A4%EC%B9%98-in-VirtualBox