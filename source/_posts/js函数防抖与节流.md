---
title: JS函数防抖与节流
date: 2019-01-30 22:23:48
tags: JavaScript
categories: JavaScript
description: 关于函数防抖与节流的一些认识
---

### 为什么要进行函数节流与防抖
浏览器进行一些有关滚动或者输入框校验之类的事件，如果一直无限制的调用回调函数，会导致浏览器性能的下降，直接降低用户的用户体验。比如：

    function callback() {
      console.log(window.scrollY)
    }

    window.addEventListener('scroll', callback)
    

  {% asset_img scroll.png %}

  可以看到随便滚动一下就调用了那么多次回调函数。

### 函数防抖(debounce)
函数防抖指的是如果一件事情持续触发，则在该事件停止触发一段时间后才执行回调函数。

用代码说明：
    
    function debounce(delay) {
      var timeout = undefined
      return function() {
        if(timeout) {
          clearTimeout(timeout)
        }
        timeout = setTimeout(() => {
          console.log(window.scrollY)
        }, delay)
      }
    }

    window.addEventListener('scroll', debounce(1000))

如果`scroll`事件一直触发，则一直清除定时器并设置新的1s定时器。事件一旦不触发，定时器则正常执行。

### 函数节流(throttle)
函数节流指的是如果一件事持续触发，则每隔一个时间段调用一次回调函数。

函数节流能实现的方式有很多

    function throttle(delay) {
      var timer = undefined
      return function() {
        if (!timer) {
          timer = setTimeout(() => {
            console.log(window.scrollY)
            timer = undefined
          }, delay)
        }
      }
    }

    window.addEventListener('scroll', throttle(1000))

滚动停止时，由于定时器的delay延迟，可能还会执行一次函数。

时间控制也可以使用时间戳的形式

    function throttle(delay) {
      var start = Date.now()
      return function() {
        var end = Date.now()
        if (end - start >= delay) {
          console.log(window.scrollY)
          start = Date.now()
        }
      }
    }

    window.addEventListener('scroll', throttle(1000))

利用函数闭包，通过时间戳的形式，一般情况下第一次会立即执行(但是如果闭包函数执行和滚动时间相差小于delay则第一次不会执行)。滚动停止之前与最后一次执行回调这一段时间，也不会在触发回调事件。

### 总结
可以看出的是函数防抖是维护了一个计时器。只要判断事件是否持续运行。

而函数节流是判断一个时间段是否达到。可以通过维护时间戳和计时器来实现。