---
title: "웹에서 dropzone을 이용한 파일 업로드 기능 구현하기"
last_modified_at: 2022-10-21
author: Do-soo, KIM
---

<br>
UI 개발 프로젝트를 하는데, Nifty라는 UI 프레임워크를 도입하게 되어서 기능별로 테스트를 하는 중입니다.

그런데 파일 업로드 기능이 일반적인 자바 스크립트와는 다르게 구현되는 걸 발견하여 이에 대해서 포스팅하려고 합니다.

보통 일반적으로 파일 업로드 개발 시, `<input type = 'file'>` 태그를 사용하여 form 데이터로 전송하는 방식으로 많이 사용합니다.

하지만, Nifty에서는 파일업로드 화면을 dropzone이라는 라이브러리를 사용하여 구현하도록 되어 있어서 많이 다릅니다.

먼저, Dropzone이라는 라이브러리에 대해서 설명할까 합니다.

Dropzone은 우리가 웹에서 파일 업로드 할 때 많이 보던 파일 드래그 앤 드롭 방식의 기능을 추가하는 데 도움이 되는 간단한 JavaScript 라이브러리입니다. 

프론트 페이지에서 사용하는 대표적인 파일 업로드 라이브러리입니다.

기본적으로 드래그 앤 드롭 기능을 지원하며, 라이브러리 기본 스타일 또한 동적인 애니메이션과 고급스러운 파일 첨부 상호작용이 매우 훌륭합니다. 

특히 이미지 파일을 업로드 할 때, 기본적으로 이미지 썸네일 미리보기 뿐만 아니라 멀티 업로드 기능 역시 지원 합니다.

또한 Amazon S3 멀티파트 업로드도 지원한다고 한다.

Nifty에서는 위와 같은 Dropzone을 적용한 자바스크립트 템플릿을 제공하여 손쉽게 파일 업로드 기능을 구현하도록 해 줍니다.

그럼 샘플 코드를 살펴보겠습니다.

이 포스팅에서는 드래그 앤 드롭 방식이 아니라, 일반적인 파일 선택 버튼을 클릭하여 파일을 업로드하는 화면으로 테스트 하였습니다.

우선 UI 프레임워크를 통해 다음과 같은 파일 업로드 화면을 구성하였습니다.

![image](https://user-images.githubusercontent.com/92565548/197094190-834ac813-ed4f-4ccb-9799-ece4ae95d4c8.png)

"Upload" 버튼을 클릭하여 업로드할 파일을 선택한 화면입니다.


그럼 선택한 파일을 업로드 해 볼 텐데, 제공하는 자바스크립트 코드 템플릿을 약간만 수정하면 됩니다.

이미 제공되는 아래와 같은 템플릿에서 각 항목의 값만 적절하게 수정해 주면 완료됩니다.
(수정이 필요한 일부 코드만 적었습니다.)

각 항목에 대한 자세한 설명은 따로 찾아보시면 되겠습니다.

저는 간단하게 paramName, maxFilesize, url 항목만 변경하고 테스트 해 봤는데 아주 잘 됩니다.

url 항목은 업로드 요청한 파일을 수신하여 처리할 백엔드 처리 url 입니다.

백엔드 처리 코드는 뒤에 따로 나옵니다.


```
Dropzone.options.demoDropzone = {
    paramName: "file", 
    maxFilesize: 100, 
    autoProcessQueue: false,
};


const customStyleDz = new Dropzone(document.body, {
    url: "/upload", // Set the url
    thumbnailWidth: 50,
    thumbnailHeight: 50,
    parallelUploads: 5,
    previewTemplate: previewTemplate,
    autoQueue: false, 
    previewsContainer: "#_dm-dzPreviews", 
    clickable: "._dm-fileInputButton" 
});
```

아래와 같은 버튼 클릭 이벤트에 의해서 업로드를 실행하도록 코드를 작성하였습니다.

실행하는 코드가 달랑 1줄입니다. 너무 간단하죠?

```
uplodaBtn.addEventListener( "click", () => {
    customStyleDz.enqueueFiles(customStyleDz.getFilesWithStatus(Dropzone.ADDED));
}); 
```

이제 앞에서 언급한 백엔드 처리 코드를 작성하면 됩니다.

Java 언어를 사용하였습니다.

```java
@RequestMapping(value = "/upload", method = RequestMethod.POST)
@ResponseBody
public void upload(MultipartHttpServletRequest request, 
		@RequestParam HashMap<String, Object> parameter) throws Exception{
	Map<String, MultipartFile> fileMap = request.getFileMap();

	for (MultipartFile multipartFile : fileMap.values()) {
	    System.out.println(multipartFile.getOriginalFilename());
	}
}
```

백엔드에서 전송받은 파일의 이름을 출력하는 코드 입니다.

이를 적절하게 사용하면 원하는 처리를 하실 수 있습니다.

처음 접하게 되면, 기존 방식과 달라서 까다로워 보일 수 있지만 간단하게 수정만 하여 구현할 수 있는 템플릿 형태로 코드를 제공하기 때문에 한번 해 보시면 쉽게 구현할 수 있을 거라고 생각합니다. 

그럼 이번 포스팅은 여기서 마치겠습니다.

### References

https://docs.dropzone.dev/<br>
https://kasumil.tistory.com/222<br>
https://inpa.tistory.com/entry/Dropzone-📚-이미지-파일-업로드-드래그-앤-드롭-라이브러리-사용법
