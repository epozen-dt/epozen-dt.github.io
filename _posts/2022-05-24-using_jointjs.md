---
title: "자바스크립트: Joint JS 사용하기"
last_modified_at: 2022-05-24
author: Do-soo, KIM

---

이전 포스팅에서 프로세스마이닝 기법 중 하나로 알파 알고리즘에 대해서 보셨을 겁니다. 

알파 알고리즘을 통해 Petri net이라는 다이어그램을 도출했었죠?

기억이 안 나신다면, 아래의 이전 포스팅 내용을 확인해 보세요. 

[[프로세스 마이닝 - 알파 알고리즘 적용]](https://epozen-dt.github.io/2022/04/28/ProcessMining_AlphaAlgorithm.html) (클릭하시면 해당 포스팅으로 이동합니다.)

이번에 진행하는 프로젝트 중 Perti Net을 웹 화면에 그려야 해서 관련 오픈 소스 라이브러리를 조사해 보던 중 괜찮은 라이브러리가 있어서 이번 포스팅에서 소개할까 합니다.

바로 Joint JS라는 라이브러리인데, 대화형 SVG 기반 다이어그램 및 시각적 응용 프로그램을 구축하기 위한 유연한 Javascript 라이브러리입니다.

정적 다이어그램은 물론이고 완전한 대화형 다이어그램 도구 및 응용 프로그램 빌더를 만드는 데 사용할 수 있습니다.

위에서 언급한 SVG(Scalable Vector Graphics)는 2차원 그래픽을 표현하기 위해 만들어진 XML파일 형식의 마크업 언어입니다. 

Vector는 기본적인 도형, 패스 등 일러스트에서 작업하는 모든 것이 될 수 있는데, 일반적으로 웹 상 이미지 파일형식은 JPEG를 많이 사용합니다. 

SVG에 대해서는 이 정도로만 설명하기로 하고 본론으로 돌아와 Joint JS에 대해서 살펴 보겠습니다.

이전 포스팅에서 도출했던 Petri Net은 다음과 같습니다.

![image](https://user-images.githubusercontent.com/92565548/169958923-6997ba1c-f1c1-41b9-be0a-350614ce1add.png)

이 Petri Net을 Joint JS를 사용하여 웹 화면에서 구현하는 과정을 이제부터 차근차근 해 보도록 하겠습니다. 


### 1. 페이지 생성

먼저 웹 페이지를 하나 생성하고 Joint JS를 사용하기 위한 js 파일을 등록해 줍니다.

(js 파일은 이미 빌드가 되어 있는 상태라고 가정하고 진행하겠습니다.)

```html
<div id="paper"></div>

<script src="https://cdn.jsdelivr.net/npm/fake-smile@1.0.1/smil.user.min.js"></script>
<script src="../../node_modules/jquery/dist/jquery.js"></script>
<script src="../../node_modules/lodash/lodash.js"></script>
<script src="../../node_modules/backbone/backbone.js"></script>
<script src="../../build/joint.js"></script>
<script>
    var graph = new joint.dia.Graph();
    new joint.dia.Paper({
        el: document.getElementById('paper'),
        width: 800,
        height: 450,
        gridSize: 10,
        defaultAnchor: { name: 'perpendicular' },
        defaultConnectionPoint: { name: 'boundary' },
        model: graph
    });
</script>
```


### 2. Transition 생성


위 Petri Net은, 7개의 Transition과 10개의 Place가 필요하군요.

그럼 먼저, Transition을 생성해 보겠습니다.

```html
var transition_a = new joint.shapes.pn.Transition({
    position: { x: 150, y: 100 },
    attrs: {
        '.label': {
            'text': 'a',
            'fill': '#fe854f'
        },
        '.root': {
            'fill': '#9586fd',
            'stroke': '#9586fd'
        }
    }
});
```

이제 나머지 6개도 생성해 볼 건데, 위와 같이 생성하지 않아도 됩니다.

어차피 라벨과 위치 정보만 다르고 모든 속성은 동일하기 때문에 위에서 생성했던 Transition 객체를 복제한 다음 라벨과 위치 정보만 추가해 주면 됩니다.

```html
var transition_b = transition_a.clone()
    .attr('.label/text', 'b')    
    .position(150, 300);

var transition_c = transition_a.clone()
    .attr('.label/text', 'c')
    .position(400, 50);

var transition_d = transition_a.clone()
    .attr('.label/text', 'd')
    .position(400, 150);

var transition_e = transition_a.clone()
    .attr('.label/text', 'e')
    .position(400, 250);

var transition_f = transition_a.clone()
    .attr('.label/text', 'f')
    .position(400, 350);

var transition_g = transition_a.clone()
    .attr('.label/text', 'g')
    .position(650, 200);

graph.addCell([transition_a, transition_b, transition_c, transition_d, transition_e, transition_f, transition_g]);  
```

### 3. Place 생성

Transition은 다 생성했으니 이제 Place도 생성해 보겠습니다.

Transition과 마찬가지로 1개를 생성하고 나머지 9개는 복제하는 방법으로 생성하면 됩니다.

```html
var place_start = new joint.shapes.pn.Place({
    position: { x: 30, y: 200 },
    attrs: {
        '.label': {
            'text': 'start',
            'fill': '#7c68fc'
        },
        '.root': {
            'stroke': '#9586fd',
            'stroke-width': 3
        },
    }
});

var place_1 = place_start.clone()
    .attr('.label/text', '')
    .position(250, 50);

var place_2 = place_start.clone()
    .attr('.label/text', '')    
    .position(250, 150);
    
var place_3 = place_start.clone()
    .attr('.label/text', '')
    .position(250, 250);

var place_4 = place_start.clone()
    .attr('.label/text', '')
    .position(250, 350);

var place_5 = place_start.clone()
    .attr('.label/text', '')
    .position(500, 50);

var place_6 = place_start.clone()
    .attr('.label/text', '')
    .position(500, 150);

var place_7 = place_start.clone()
    .attr('.label/text', '')
    .position(500, 250);

var place_8 = place_start.clone()
    .attr('.label/text', '')
    .position(500, 350);

var place_end = place_start.clone()
    .attr('.label/text', 'end')
    .position(750, 200);

graph.addCell([place_start, place_1, place_2, place_3, place_4, place_5, place_6, place_7, place_8, place_end]);
```

### 4. Link 생성

이제 생성된 객체끼리 연결만 하면 되겠네요.

Link 객체를 이용해서 화살표를 생성하면 됩니다.

Link 객체는 연결의 source와 target 파라미터가 각각 다르므로, 함수로 만들어서 각각 호출하여 사용해 보려고 합니다.

```html
function link(a, b) {
    return new joint.shapes.standard.Link({
        source: { id: a.id, selector: '.root' },
        target: { id: b.id, selector: '.root' },
        attrs: {
            line: {
                stroke: '#ffffff',
                sourceMarker: {
                },
                targetMarker: {
                    'type': 'path',
                    'stroke': '#ffffff',
                    'fill': '#ffffff',
                }
            }
        }
    });
}
```

이렇게 함수를 만들었구요. 

호출만 하면 되겠죠?

```html
graph.addCell([
    link(place_start, transition_a),
    link(place_start, transition_b),
    link(transition_a, place_1),
    link(transition_a, place_2),
    link(transition_b, place_3),
    link(transition_b, place_4),
    link(place_1, transition_c),
    link(place_2, transition_e),
    link(place_3, transition_d),
    link(place_4, transition_f),
    link(transition_c, place_5),
    link(transition_c, place_6),
    link(transition_d, place_6),
    link(transition_d, place_7),
    link(transition_e, place_7),
    link(transition_e, place_8),
    link(transition_f, place_5),
    link(transition_f, place_8),
    link(place_5, transition_g),
    link(place_6, transition_g),
    link(place_7, transition_g),
    link(place_8, transition_g),
    link(transition_g, place_end),
]);
```

자! 만들어진 Petri Net을 확인해 볼까요?

![image](https://user-images.githubusercontent.com/92565548/169959121-a9ee7c28-82cc-412b-9e5b-75cc314e0f1f.png)

짜잔!! 비슷하게 만들어 졌나요? 

그럼 이번 포스팅은 여기서 마치겠습니다.

### References

```
https://svgontheweb.com/ko/
https://resources.jointjs.com/demos/pn
```