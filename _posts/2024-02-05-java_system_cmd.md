---
title: "Java로 시스템 명령어 실행하기"
last_modified_at: 2024-02-05
author: 이현지
---


본 포스팅은 Java로 시스템 명령어를 실행하는 방법에 대한 글입니다. 

<br><br>

Java에서 시스템 명령어를 실행하려면 다음 두 가지 클래스 중 하나를 선택해서 사용하면 됩니다.

<br>

- java.lang.Runtime (JDK 1.4 이상)<br>
- java.lang.ProcessBuilder (JDK 1.5 이상)  

<br>

각 클래스를 통해 얻어지는 Process 객체는 JDK 에서 제공하는 외부 프로세스에 대한 접점으로서, 외부 명령어 실행을 가능하게 합니다. 

<br><br>

## Runtime 클래스를 통한 시스템 명령어 실행

```java
private static void runTime() throws IOException {

    // 운영체제 타입
    String osType = System.getProperty("os.name");

    // 명령어
    String cmd = null; 

    // 윈도우 운영체제 
    if (osType.toLowerCase().startsWith("windows")) { 

        cmd = "cmd.exe /c dir";
		
    // 그 외 운영체제 
    } else { 

        cmd = "sh -c ls";

    }

    // 명령어 실행
    Process process = Runtime.getRuntime().exec(cmd);
	// exec()의 매개변수로 String[]도 가능
    // new String[] {"cmd.exe", "/c", "java", "-version"}
    
    BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()));

    String line = null;

    while ((line = br.readLine()) != null) {

        // 명령어 결과 출력
        System.out.println(line);

    }

}
```

<br><br>
## ProcessBuilder를 통한 시스템 명령어 실행 

ProcessBuilder는 directory() 메소드를 통해 작업 디렉토리를 지정할 수 있습니다. <br>
```java
private static void processBuilder() throws IOException {

    String osType = System.getProperty("os.name");

    String cmd = null; 

    // 윈도우 운영체제
    if (osType.toLowerCase().startsWith("windows")) { 

        cmd = "cmd.exe /c dir";

    // 그 외 운영체제
    } else { 

        cmd = "sh -c ls";

    }

    ProcessBuilder processBuilder = new ProcessBuilder(cmd);
	// 명령어를 ArrayList<String> 형태로 전달하는 것도 가능
    // ArrayList<String> cmdArrList = new ArrayList<>();
    // cmdArrList.add("sh");
    // cmdArrList.add("-c");
    // cmdArrList.add("java");
    // cmdArrList.add("-version");
    
    // 명령어가 실행되는 작업 디렉토리를 사용자의 home 디렉토리로 설정
    processBuilder.directory(new File(System.getProperty("user.home")));

    // 명령어 실행
    Process process = processBuilder.start();

    // 명령어 결과 read
    BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()));

    String line = null;

    while ((line = br.readLine()) != null) {

        // 명령어 결과 출력
        System.out.println(line);

    }

}
```
<br><br>
## 파이프라인 사용 관련 이슈
파이프라인 사용 시, 해당 명령어를 하나의 문자열로 전달해야 명령어 실행 결과가 정상적으로 리턴됩니다. 
<br><br>

### docker ps | grep 'hello' 예시
```java
// Runtime 예시 -----------------------------------------------------------------

Runtime.getRuntime().exec(new String[] {"sh", "-c", "docker ps | grep 'hello'"});
// new String[] {"sh", "-c", "docker", "ps", "grep", "'hello'"} 전달 시 오류 발생


//ProcessBuilder 예시 -----------------------------------------------------------

ArrayList<String> cmdArrList = new ArrayList<>();
cmdArrList.add("sh");
cmdArrList.add("-c");
cmdArrList.add("docker ps | grep 'hello'");

ProcessBuilder processBuilder = new ProcessBuilder(cmdArrList);
```
<br>
이상으로 본 포스팅을 마치겠습니다.

<br><br>
 > Reference 
- https://www.baeldung.com/run-shell-command-in-java<br>
- https://ontoidea.tistory.com/86<br>
- https://d2.naver.com/helloworld/1113548<br>