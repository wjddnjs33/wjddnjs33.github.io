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

![](https://github.com/wjddnjs33/image/blob/main/async-1.png?raw=true)

Like picture as above, print the `'undefined'`. Why?? :) This is because execute next code(`return result`) before operation of a  `$.get()` method finishes

> Second example code for Async Processing 

```javascript
console.log('Hello');
setTimeout(function(){
  console.log("Bye");
}, 5000);
console.log('Hello pocas');
```
The code above executes the logic after a specific time used `setTimeout()` method. So how would the above code print it out?

```
First, output
Hello
Bye
Hello pocas

Second, output
Hello
Hello pocas
Bye
```
What do you thinks? First? Second?

![](https://github.com/wjddnjs33/image/blob/main/async/async-2.png?raw=true)

Second!! The `setTimeout()` method is also async, execute the next code before the operation finishes. So, let's fix problem using the callback function in the First example.

> The sollution for first case

```javascript
function a(callback){
  $.get('http://burp.kr/product.html', function(res){
    callback(res);
  });
}
a(function(result){
  console.log(result);
});
```
I wrote sollution code as above. Above code is send to callback function a return value of `$.get()` method.

![](https://github.com/wjddnjs33/image/blob/main/async/async-3.png?raw=true)

So, it doens't print "undefined" and return value of `$.get()` method. Now, we know async and callback. So we will learn the `Promise`

---
I said about Promise from above. One more time, I will say. `Promise` is object used async processing. So we will learn about `Promise` from now on.

> Promise Code - First Example

```javascript
function as(){
  return new Promise(function(resolve, reject) {
    $.get('http://burp.kr/product.html', function(res){
      if(res){
        resolve(res);
      }
      reject(new Error("Request is failed"));
    });
  });
}

as().then(function(result){
  console.log(result);
}).catch(function(err){
  console.error(err);
});
```
The code above code is a basic example using `Promise`. first send to request using `$.get()` and when the operation is finished of `$.get()`, I call the `resolve()` and pass argument a return value of `$.get()`

And when the `esolve()` method is called, the value is passed using the `then()` method.

> hmm.. So what are the resolve() and reject() methods?

When Promise works, there are three states. states is pending, fulfilled, rejected! peding is async processing logic has not complete, fulfilled is async processing logic is completed and returns the result value, rejectd is async processing has failed or an error has occurred.

```javascript
new Promise(function(resolve, reject){
  // ~~~
});
```
As above, The moment the `Promoise()` it, it will be in Pending state. And you can define a callback. callback have two method. first is `resolve()` and second is `reject()`. `resolve()` is called when it is done request well and `reject()` is called when ti is return error.

```javascript
new Promise(function(resolve, reject) {
  resolve();
});
```
Here, When well execute `resolve()` method of callback it, It will be in fulfilled state.

```javascript
function a() {
  return new Promise(function(resolve, reject) {
    $.get('http://burp.kr/product.html', function(res){
      resolve(res);
    });
  });
}

a().then(function(result) {
  console.log(result); // 100
});
```
And as above, it becomes fulfilled state and receive data using `then()`.

```javascript
new Promise(function(resolve, reject) {
  reject();
});
```
And as above, When execute `reject()` method of callback it, It will be in rejected state :)

```javascript
function a(){
  return new Promise(function(resolve, reject){
    $.get('htt://burp.kr/product.html', function(res){
      reject(new error(res));
    }
  }
}

a().then.catch(function(err){
  console.log(err);
});
```
As above, Can print error using the `reject()`

> Promise Example

```javascript
function a(){
  return new Promise(function(resolve, reject){
    $.get('http://burp.kr/product.html', function(res){
      if(res){
        resolve(res);
      }
      reject(new error("Error"));
    });
  });
}

a().then(function(result){
  console.log(result);
}).catch(function(err){
  console.log(err);
});
```
![](https://github.com/wjddnjs33/image/blob/main/async/async-4.png?raw=true)

As above, You can see printed well by recevied the value.

Now let's see how to disopose error in Promise!

Have a two dispose of error method. first method is using second argument in `then()`. second method is using `catch()`. But not recommend method the dispose of error use second argument in `then()` because, `then()` method is dont well catch a error.

> Dispose of error using second argument in `then()`

```javascript
function a(){
  return new Promise(function(resolve, reject){
    $.get('http://burp.kr/product.html', function(res){
      if(res){
        resolve(res);
      }
      reject(new error("Error"));
    });
  });
}

a().then(function(result){
  console.log(result);
  throw new Error("Error in then()");
}, function(err){
  console.log("then error : ", err);
});
```
![](https://github.com/wjddnjs33/image/blob/main/async/async-5.png?raw=true)

First, an error occurred in the then() method, but when I caught the error using the second argument of `then()` method, I can see that it was not caught well :( So, Not recommand the dispose of error use second argument in `then()` method.

```javascript
function a(){
  return new Promise(function(resolve, reject){
    $.get('http://burp.kr/product.html', function(res){
      if(res){
        resolve(res);
      }
      reject(new error("Error"));
    });
  });
}

a().then(function(result){
  console.log(result);
}).catch(function(err){
  console.log(err);
});
```
![](https://github.com/wjddnjs33/image/blob/main/async/async-6.png?raw=true)

And I catch the error using the `catch()` method, you can see that it is caught well. :) Now, we will learn about async/await.

---
I am go to the bed because tired now :(
I will write about async/await tomorrow haha..

---
