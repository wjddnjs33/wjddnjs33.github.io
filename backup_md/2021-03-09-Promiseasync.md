---
layout: post
date: 2021-03-09 00:00:10
title: "Promise & Async processing of javascript"
categories: Development
tags: [Javascript]
author:
  - Jeongwon Jo

---
I studied about Promise and Async process today. Promise is object used for async processing and it is mainly used to display the data received from the server on screen.

First, We need to know `async processing` before studying `Promise`. `Async processing` is a feature of javascript that executed the next code without stopping before operation of a particular code finishes.

> First example code for Async Processing 

```javascript
function a(){
  var result;
  $.get('http://burp.kr/product.html', function(response){
    result = response;
  });
  return result;
}
console.log(a());
```
You see code as above and what thinks output value?

![](https://github.com/wjddnjs33/image/blob/main/async/async-1.png?raw=true)

Like picture as above, print the `'undefined'`. Why?? :) This is because execute next code(`return result`) before operation of a  `$.get()` method finishes

Writing
---
