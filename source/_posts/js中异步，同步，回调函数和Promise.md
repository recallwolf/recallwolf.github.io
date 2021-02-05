---
title: JS中异步，同步，回调函数和Promise
date: 2018-12-01 12:05:16
tags: JavaScript
categories: JavaScript
description: Event Loop, 异步
---
### 1.线程，同步，异步和回调函数

#### 线程
>JS单线程，是指在JS引擎中负责解释和执行JavaScript代码的线程只有一个,即主线程。  
实际上还存在其他的线程。例如：处理AJAX请求的线程、处理DOM事件的线程、定时器线程、读写文件的线程(例如在Node.js中)等等。这些线程可能存在于JS引擎之内，也可能存在于JS引擎之外。

**Javascript 有一个 main thread 主线程和 call-stack 调用栈(执行栈)，所有的任务都会被放到调用栈等待主线程执行。**

**并且一个 JavaScript 运行时包含了一个待处理的消息队列。每一个消息都有一个为了处理这个消息相关联的函数。**

#### Event Loop

1.函数调用形成了一个栈帧
    
    function foo(b) {
      var a = 10;
      return a + b + 11;
    }

    function bar(x) {
      var y = 3;
      return foo(x * y);
    }

    console.log(bar(7));  

>当调用bar时，创建了第一个帧 ，帧中包含了bar的参数和局部变量。当bar调用foo时，第二个帧就被创建，并被压到第一个帧之上，帧中包含了foo的参数和局部变量。当foo返回时，最上层的帧就被弹出栈（剩下bar函数的调用帧 ）。当bar返回的时候，栈就空了。

2.消息队列  

>在事件循环期间的某个时刻，运行时总是从最先进入队列的一个消息开始处理队列中的消息。正因如此，这个消息就会被移出队列，并将其作为输入参数调用与之关联的函数。为了使用这个函数，调用一个函数总是会为其创造一个新的栈帧，一如既往。

**当异步线程执行完并且有了结果，会往消息队列推送该结果。**  
**当主线程执行完调用栈里的任务，即调用栈为空时，会读取消息队列,并将其作为输入参数调用与之关联的函数。即添加到可执行栈中。**

以上机制就叫做事件循环机制，取一个消息并执行的过程叫做一次循环。

{% asset_img 1.png %}


剩下的概念就很好理解了：  
同步：如果在函数返回结果的时候，调用者能够拿到预期的结果，即同步。

异步：如果在函数返回的时候，调用者还不能购得到预期结果，而是将来通过一定的手段得到（例如回调函数），这就是异步。

回调函数：如上文说道，每一个消息都有一个为了处理这个消息相关联的函数。该函数即为回调函数。

>主线程在执行完当前循环中的所有代码后，就会到消息队列取出这条消息，并执行它。到此为止，就完成了工作线程对主线程的通知，回调函数也就得到了执行。

{% asset_img 2.png %}

    console.log("a");
    setTimeout(function() {
      console.log("b");
    }, 1000);
    setTimeout(function() {
      console.log("c");
    }, 0);
    console.log("d");

以上代码执行结果是a d c b

### 2.Promise与回调地狱

#### 回调地狱  

    listen( "click", function handler(evt){
      setTimeout( function request(){
        ajax( "http://some.url.1", function response(text){
          if (text == "hello") {
            handler();
          }
          else if (text == "world") {
            request();
          }
        } );
      }, 500) ;
    } );

当多个回调函数嵌套就构成了回调地狱。除了缩进产生的横向扩展导致代码的阅读难度，此外还有执行顺序上的问题。  
上面例子的执行顺序 **监听click事件 => setTimeout => ajax => if判断**，是直观逻辑上的顺序，但是在真实的异步JS程序中，经常会有很多噪音把事情搞乱。

比如

    doA( function(){
      doB();

      doC( function(){
        doD();
      } )

      doE();
    } );

    doF();

这段代码执行顺序： **doA => doF => doB => doC => doE => doE => doD**
这很不符合直观上的逻辑

当这样的回调嵌套出现很多的时候，代码可读性大大降低。

#### Promise

Promise就是为了解决回调地狱而产生的，Promise 本质上是一个绑定了回调的对象，而不是将回调传进函数内部。

Promise 对象有以下两个特点： 
>（1）对象的状态不受外界影响。Promise 对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和 Rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

>（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise 对象的状态改变，只有两种可能：从 Pending 变为 Resolved 和从 Pending 变为 Rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。就算改变已经发生了，你再对 Promise 对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

Promise基本用法

    const promise = new Promise(function(resolve, reject) {
      // ... some code

      if (/* 异步操作成功 */){
        resolve(value);
      } else {
        reject(error);
      }
    });
    

举个例子

    setTimeout(function() {
        console.log("log1");
        setTimeout(function() {
            console.log("log2");
            setTimeout(function() {
                console.log("log3");
            }, 3000);
        }, 2000); 
    }, 1000);

经过Promise封装后

    function firtTimeout() {
      return new Promise(function(resolve, reject) {
        setTimeout(function() {
          resolve("log1");
        }, 1000);
      })
    }

    function secondTimeout(result) {
      console.log(result);
      return new Promise(function(resolve, reject) {
        setTimeout(function() {
          ("log2");
        }, 2000);
      })
    }

    function ThirdTimeout(result) {
      console.log(result);
      return new Promise(function(resolve, reject) {
        setTimeout(function() {
          resolve("log3");
        }, 3000);
      })
    }

    firtTimeout()
    .then(secondTimeout)
    .then(ThirdTimeout)
    .then(function(result) {
      console.log(result)
    })
    


>参考资料 ：   
[阮一峰《ECMAScript6入门》](http://es6.ruanyifeng.com/#docs/promise)  
[MDN web docs 使用Promises
](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises)   
[MDN web docs 并发模型与事件循环
](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)  
[getify/You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/async%20%26%20performance/README.md)
