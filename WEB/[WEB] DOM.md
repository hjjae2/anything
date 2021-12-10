---
layout: post
title: "WEB :: DOM"
author: "leehyunjae"
tags: ["html"]
---

> DOM 이란?

### **DOM(Document Object Model, 문서 객체 모델) 이란?**

1. <u>XML이나 HTML과 같은 문서에 접근(조작)하기 위한 인터페이스</u>
   - 문서 내의 모든 요소(예를 들어, 태그)를 정의하고 (그 요소들에)접근하는 방법을 제공
2. W3C의 표준 객체 모델

<br>

(위의 말을 조금 쉽게 풀어쓰면) 우리는 DOM 덕분에 아래와 같은 작업을 할 수 있다.
- (JS) 새로운 HTML 요소 추가/수정/삭제
- (JS) 새로운 HTML Event 추가
- (JS) CSS 변경 등

<br>

DOM은 아래와 같이 노드 트리 형태로 나타난다고 한다.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/DOM-model.svg/2560px-DOM-model.svg.png" width="50%" height="50%">

> 출처: [위키백과: 문서 객체 모델](https://ko.wikipedia.org/wiki/문서_객체_모델)

<br>

단순히 텍스트로 작성된 HTML 소스(코드)를 Element로 변환하여 구조화된 형태로 만들고, 다양하게 사용될 수 있도록(예를 들어, 위에서 말한 API 인터페이스를 제공하는 것 등) 하는 것이다.

<br>

### **DOM 의 종류**

DOM 의 종류는 아래와 같은 3 가지 모델이 있음

- Core DOM : 모든 문서(XML, HTML, ...) 타입을 위한 DOM 모델
- HTML DOM : HTML 문서를 위한 DOM 모델
- XML DOM : XML 문서를 위한 DOM 모델

<br>

### **주의할 것**

**DOM과 문서(HTML, XML)는 다르다.**<br>
   - (HTML, XML과 같은)문서에 의해 DOM이 생성되지만, 문서와 DOM이 100% 동일한 것은 아니다. 

예를 들어, 아래와 같은 경우에서 <u>DOM과 문서는 다를 수 있다.</u>

<br>

**1.작성된 HTML 문서가 유효하지 않을 때**

```html
<html>
    HI!
</html>
```

위의 코드는 HTML 필수 사항인 `<head>, <body>` 가 빠져있는데, DOM 에서는 교정된다.

```
html
 |
 |- head
 |- body
     |
     |- HI!
```

<br>

**2.자바스크립트에 의해 DOM이 추가/수정/삭제될 때**

JS에 의해 DOM에 동적으로 객체가 추가/수정/삭제될 수 있다. 이렇게 동적으로 변경된 DOM은 원본 문서(HTML)과 다르다.

<br>

**3.DOM != 브라우저에 보여지는 것**

<u>최종적으로 브라우저에 보여지는 것은 DOM이 아니라 `Render Tree`라고 한다.</u> 

`Render Tree` 
   - DOM + CSSOM으로 만들어진 것
   - 최종적으로 화면에 보여지는 것만으로 구성되어 있는 것


예를 들면 아래와 같다.

```html
<html>
    <head>
    </head>
    <body>
        <h1>HI!</h1>
        <div style="display:none;">HI!</div>
    </body>
</html>
```

```
// DOM
html
 |
 |- head
 |- body
     |
     |- h1
        |
        |- HI!
     |- div
        |
        |- HI!



// Render Tree
html
 |
 |- head
 |- body
     |
     |- h1
        |
        |- HI!
```

<br><br>

### **CRP(Critical Rendering Path)**

> (DOM에 대해 간략히 살펴보았으니) 웹페이지가 어떻게 빌드되는지에 대해서 살펴본다.

`Critical Rendering Path` : 브라우저가 HTML 문서를 읽고, 스타일을 입히고 화면에 표시되기까지의 과정

CRP 는 아래의 6개의 단계로 볼 수 있다고 한다.

1. DOM Tree 생성
2. CSSOM Tree 생성
   - `render blocking resource` : Render Tree 는 CSS 전체를 파싱하기 전에는 생성될 수 없다.
3. Running Javascript
   - `parser blocking resource` : HTML 문서를 파싱하는 것이 JS에 의해 blocking(stop)될 수 있다.
4. Render Tree 생성
   - 최종적으로 페이지에 렌더링 되는 내용(tree)이다.
5. Generating the Layout
   - viewport의 size 를 결정한다. (-> viewport size에 종속적인 css context 제공한다.)
   - default viewport width : 980px
   - Layout computes the exact position and size of each object.
   - 상대적인 사이즈(50%, 25%) 등은 pixel 값(절대값)으로 계산된다.
6. Painting
   - content -> 화면에 보여질 pixles 로 표현한다.
   - DOM, Style(즉, Render Tree)에 따라 painting 의 시간이 결정된다.
   - It takes in the final render tree and renders the pixels to the screen.


> 1. Send Request - GET request sent for index.html
> 2. Parse HTML and Send Request - Begin parsing of HTML and DOM construction. Send GET request for style.css and main.js
> 3. Parse Stylesheet - CSSOM created for style.css
> 4. Evaluate Script - Evaluate main.js
> 5. Layout - Generate Layout based on meta viewport tag in HTML
> 6. Paint - Paint pixels on document
> <br>
> <br>[출처: Understanding the Critical Rendering Path](https://bitsofco.de/understanding-the-critical-rendering-path/)

위 과정을 간소화하면, 다음의 두 단계로 볼 수 있다.<br>
1. 브라우저는 문서를 읽고/파싱한다. 어떤 내용을 페이지에 렌더링할 지 결정한다.
   - DOM, CSSOM, Render Tree 를 생성한다.
2. 브라우저는 렌더링 작업을 수행한다.

<br>

### 참고

- [DOM의 개념](http://tcpschool.com/javascript/js_dom_concept)
- [DOM은 정확히 무엇일까?](https://wit.nts-corp.com/2019/02/14/5522)
- [Render-tree Construction, Layout, and Paint](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction)
- [Understanding the Critical Rendering Path](https://bitsofco.de/understanding-the-critical-rendering-path/)