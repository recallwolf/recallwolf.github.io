---
title: JS的this指向
date: 2018-04-08 12:50:44
tags: JavaScript
categories: JavaScript
comments: false
description: 关于JS this指向的总结
---
**一般情况下 this 不是在函数定义的时候确定指向！而是在函数执行的时候确定，this指向的是调用函数的那个对象**

### this指向的几种情况
#### 1.纯粹函数调用
    function people(){
        this.name = "recallwolf";
        var age = 18;
        
        console.log(this); //Window
        console.log(this.name); //recallwolf
        console.log(this.age); //undefined
    }
    people();

this最终指向的是调用它的对象，这里的函数people实际是被Window对象所调用
    
    people() === window.people() //true

#### 2.作为对象方法的调用
    var o = {
        name: "recallwolf",
        fn: function(){
            console.log(this.name);  //recallwolf
        }
    }
    o.fn();

fn是通过o.fn()执行的,这里的this就是指向调用fn的对象，即o

##### 值得注意的是   

    var o = {
        a: 10,
        b:{
            a: 12,
            fn: function(){
                console.log(this.a); //12
            }
        }
    }
    o.b.fn();

this的指向并不是o而是b
**如果一个函数中有this，这个函数被多个对象包含，尽管这个函数是被最外层的对象所调用，this指向的也只是它上一级的对象**

    var o = {
        a: 18,
        fn: function(){
            var test = function(){
                console.log(this.a); //undefined
            }
            test();
        }
    }
    o.fn();

**this不是在函数定义的时候确定指向，在函数里面定义的函数也是要看是谁调用的这个函数(方法)this才是谁；在这里并不是o调用fn里面的test函数，实际上是window**

**换种说法，如果一个函数中有this，但是它没有被上一级的对象所调用，那么this指向的就是window，这里需要说明的是在JS的严格版中this是undefined**

    var o = {
            a: 18,
            fn: function(){
                var self = this;
                var test = function(){
                    console.log(self.a); //18
                }
                test();
            }
        }
        o.fn();

要达到预期目标可以把o这个对象用self保存下来

#### 3.作为构造函数调用
    function Fn(){
        this.name = "recallwolf";
    }
    var a = new Fn();
    console.log(a.name); //recallwolf

需要注意的是new关键字可以改变this的指向，将这个this指向对象a
new关键字创建一个对象实例，这里用变量a创建了一个fn的实例（相当于复制了一份fn到对象a里面)
此时new有三个作用：
**1.在构造函数内部声明一个临时对象this
2.在构造函数fn中默认返回这个临时对象this，赋给a
3.将临时对象的_proto_指向fn的prototype**

    function Fn(){
        this.name = "recallwolf";
        console.log(this); //Window
    }
    var a = Fn();
    console.log(a.name); //Uncaught TypeError: Cannot read property 'name' of undefined

这是因为this指向的是window对象

##### 如果构造函数有返回值
**如果返回值是一个对象，那么this指向的就是那个返回的对象，如果返回值不是一个对象那么this还是指向函数的实例。**

    function Fn(){  
        this.name = "recallwolf";  
        return {};  
    }
    var a = new Fn();  
    console.log(a.name); //undefined


    function Fn(){  
        this.name = "recallwolf";
        return 1;
    }
    var a = new Fn();  
    console.log(a.name); //recallwolf

    function Fn(){  
        this.name = "recallwolf"; 
        return undefined;
    }
    var a = new Fn();  
    console.log(a.name); //recallwolf

虽然null也是对象，但是在这里this还是指向那个函数的实例,null比较特殊

    function Fn(){  
        his.name = "recallwolf";  
        return null;
    }
    var a = new Fn();  
    console.log(a.name); //recallwolf


#### 4.apply调用
apply()是函数对象的一个方法，它的作用是改变函数的调用对象，它的第一个参数就表示改变后的调用这个函数的对象。因此，this指的就是这第一个参数。
apply()的参数为空时，默认调用全局对象。因此，这时的运行结果为0，证明this指的是全局对象。

    var age = 18;
    function fn(){
        console.log(this.age);
    }
    var o = {};
    o.age =  20;
    o.m = fn;

    o.m.apply(); //18
    o.m.apply(o); //20

### es6箭头函数
**箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象**

    var o = {
        id: 42,
        fn: () => {
            console.log(this.id);
        }
    }
    o.fn(); //undefined

    var o = {
        id: 42,
        fn: function(){
            console.log(this.id);
        }
    }
    o.fn(); //42

普通函数this指向调用它的对象 即o
箭头函数this即对象o的this 这里是window

再举一个例子

    var o = {
        id: 12,
        fn: function(){
            var test = function(){
                console.log(this.id) //undefined
            }
            test();
        }
    }
    o.fn();

    var o = {
        id: 12,
        fn: function(){
            var test = () => {
                console.log(this.id); //12
            }
            test();
        }
    }
    o.fn();

箭头函数this即函数fn的this 这里为o