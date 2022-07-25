---
title: "chart.js를 사용하여 웹에서 차트 그리기"
last_modified_at: 2022-02-23
author: Do-soo, KIM

---

이번 포스팅은 chart.js에 관한 내용입니다. 웹 개발 프로젝트 진행 중 차트를 그려야 할 상황이 생겨 여러 라이브러리를 알아보던 중 개발자가 사용하기 쉬운 라이브러리가 있어 간단히 소개하고 예제 소스 코드를 통해 사용하는 방법을 소개하고자 합니다.

### 1. chart.js란?

먼저 chart.js에 대해서 공식사이트에 소개된 내용을 토대로 요약해 보면 간단하고 유연한 자바스크립트 차트 라이브러리라고 할 수 있습니다.

복잡한 기능의 차트 구현에는 한계가 있을지 모르겠지만 일반적인 차트 구현에는 크게 무리가 없을 것이라고 판단하여 이 라이브러리를 선택하게 되었습니다.

chart.js를 이용해 차트를 그리려면 html5에서 새로 나온 canvas라는 태그를 사용하게 됩니다. canvas는 캔버스 스크립팅 API나 WebGL API와 함께 그래픽과 애니매이션을 그릴 수 있는 HTML의 한 요소입니다.


### 2. chart.js 사용 방법

그럼 chart.js를 어떻게 사용하는지 예제 소스 코드를 보면서 알아보겠습니다.

chart.js를 사용하려면 2가지 방법이 있는데 첫번째는 Github로부터 다운로드 한 뒤 html 파일에서 include해 주는 방법이고, 두 번째는 CDN을 사용하는 방법입니다.

이 포스팅에서는 CDN을 사용하는 방법을 이용해 보겠습니다.

CDN(Content Delivery Network)은 서버와 사용자 사의 물리적 거리를 줄여 콘텐츠 로딩에 소요되는 시간을 최소화해 줍니다. 

먼저 캐시 서버를 설치한 후, 원본 서버와 멀리 있는 사용자가 웹사이트에 접속할 때 캐시 서버가 컨텐츠를 전달하게 되는 것입니다. 

CDN에 대해서는 이 정도까지만 간략하게 소개하도록 하겠습니다.

다음과 같이 작성하여 chart.js을 사용할 것임을 선언합니다.

```
<!-- chart.js를 사용하기 위한 cdn 설정 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.4.0/Chart.min.js"></script>
```

이제 차트를 그리기 위해 canvas를 선언합니다.

```
<div style="width: 95%; height: 900px;">
    <canvas id="myChart"></canvas>
</div>
```

div 태그를 먼저 작성하고 그 안에 canvas 태그를 작성합니다.

이제 차트를 그리기 위한 코드를 작성합니다. 
body 태그 내에 script 태그를 작성하여 그 안에서 코드를 작성해야 합니다.

```
<script type="text/javascript">
	//차트 그리기 코드 작성 부분
</script>
```

위에서 'myChart'라고 선언한 canvas로부터 getContext 속성을 통해 2d로 설정합니다.

```
var ctx = document.getElementById('myChart').getContext('2d');
```

그런 다음, chart() 함수를 통해 차트의 데이터를 생성하고 차트를 그리면 됩니다.

아래 소스 코드는 일반적인 선형 그래프를 샘플로 그려본 겁니다.

```
var myChart = new Chart(ctx, {
    type: 'line', //차트 형태:  bar, line, pie
    data: { //차트 데이터
        labels: [ //x축 값
            'Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'
        ],
        datasets: [
            {
                label: 'sample chart', //차트 범례
                data: [12, 19, 3, 5, 2, 3],
                backgroundColor: 'rgba(54, 162, 235, 0.2)', 
                borderColor: 'rgba(255, 0, 0, 1)',
                borderWidth: 1,
                fill: false //차트 영역 내 범위 색상 여부
            }
        ]
    }
});
```

제대로 차트가 보이는지 확인해 볼까요?
웹브라우저를 띄워 접속해 봅니다.

![image](https://user-images.githubusercontent.com/92565548/155244422-f7c51502-834d-4510-b2fa-d7eabedd38f7.png)


그럼 이번 포스팅은 여기서 마치겠습니다.































