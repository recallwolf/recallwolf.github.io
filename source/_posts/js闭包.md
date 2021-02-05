---
title: JS闭包
date: 2018-12-07 22:26:48
tags: JavaScript
categories: JavaScript
comments: false
description: 有关JS闭包
---

#### 作用域

词法作用域中使用的域，是变量在代码中声明的位置所决定的。嵌套的函数可以访问在其外部声明的变量。

    var a = 100;
    function init() {
      var b = 101;
      console.log("init a:" + a);
      function display() {
        console.log("display a:" + a);
        console.log("display b:" + b);
      }  
      display();
    }
    init();

运行结果：

    init a:100
    display a:100
    display b:101

#### 闭包
闭包能够读取其他函数内部变量。允许将函数与其所操作的某些数据（环境）关联起来。

举个列子

    function add(a) {
      return function(b) {
        a = a + b
        console.log(a)
      }
    }
    var add3 = add(100);
    var add10 = add(110);
    add3(3);  //103
    add10(10);  //120

add3和add10都是闭包。它们共享相同的函数定义，但是保存了不同的词法环境。在add3的环境中，a为3。而在add10中，a则为10。

闭包可以使变量的值始终保持在内存中。

    function init() {
      var a = 1;
      return function() {
        a += 1;
        console.log(a);
      }
    }

    var func = init();
    func();  //2
    func();  //3
    func();  //4
    

a并没有在func调用完成后销毁，而是一直保存在内存中。init是匿名函数的父级，匿名函数被赋值给全局变量，会一直存在于内存中。所以init函数也会存在于内存之中。

大量的使用闭包会占用大量的内存，可能会导致内存泄漏，所以在退出函数之前删除局部变量。

    function init() {
      var a = 1;
      return {
        add: function() {
          a += 1;
          console.log(a);
        },
        clear: function() {
          a = undefined;
        }
      } 
    }
    var func = init();
    func.add();  //2
    func.clear(); 
    func.add(); //NaN
    

要避免大量的使用闭包，在模拟私有方法的时候可以考虑用let关键词。