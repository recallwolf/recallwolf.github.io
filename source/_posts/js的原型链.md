---
title: JS的原型链
date: 2018-04-10 17:31:44
updated: 2018-12-11 22:53:00
tags: JavaScript
categories: JavaScript
comments: false
description:  __proto__,  prototype,  constructor
---
JavaScript的继承机制比较特殊 使用prototype链来实现继承

#### 1. `__proto__`,  `prototype`,  `constructor`

JavaScript有几种方式来生成实例对象

##### *第一种方式生成实例对象*

    var person = {}
    person.name = "recallwolf"
    person.age = 18

`console.log(person)` ，我们看到如下图

{% asset_img 选区1.png %}

除了自身创建的name和age，我们发现还有一个   `__proto__` 对象。

`__proto__` 对象是一个指针，指向上一级对象的prototype，这里指向的Object.prototype
>关于 `Object.prototype` 参考[MDN web docs](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)

##### *第二种方式生成实例对象*

    function Person(name, age){
        this.name = name
        this.age = age
    }
    var f = new Person("recallwolf", 18)

`Person()`就是构造函数，`f`是生成的实例对象
和第一种方式**不一样的地方**在于，在我们声明构造函数的时候，会设置一个 `prototype` 属性，这个属性包含 `prototype` 对象。此外仍存在一个   `__proto__` 对象，该对象一般指向`Function.prototype`。

1.如下图所示，`prototype` 包含一个 `constructor` 和 `__proto__`

{% asset_img 选区2.png %}

`contructor` 可以看到是指向构造函数本身的指针
`__proto__` 是一个指向上一级对象prototype的指针，这里也是指向的Object.prototype

2.如下图所示，`__proto__`指向`Function.prototype`

{% asset_img 选区3.png %}

**相同的是**该创建的实例对象和第一种创建实例对象都会有一个 `__proto__` 对象，都指向上一级 `prototype` ，该实例对象指向的是 `Person.prototype`。

{% asset_img 选区4.png %}

**不同的是**该创建的实例对象存在`constructor`并且与构造函数`prototype`中的`constructor`相等。

{% asset_img 选区5.png %}

构造函数、原型和实例的关系如下图所示
>图作者：[manxisuo](https://link.jianshu.com/?t=https://segmentfault.com/u/manxisuo)

{% asset_img 图1.png %}

> `prototype` 对象有什么用可以参考阮一峰老师的博文[Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html)

#### 2.原型链
我们查看Person对象是否有name属性时,JS引擎做了：
1.查看对象本身有没有name属性，没有则下一步
2.查看 `person.__proto__` 对象有没有 name 属性，如果没有，那么浏览器会继续查看 `name.__proto__.__proto__` ，直到找到name属性或者 `__proto__` 为null

这个 `__proto__` 组成的链子就是原型链

如下图所示
>图作者：[manxisuo](https://link.jianshu.com/?t=https://segmentfault.com/u/manxisuo)

{% asset_img 图2.png %}

> `p` 指 `prototype` 属性，`[p]` 即 `__proto__` ，`[p]` 形成的链（虚线部分）就是原型链

值得注意的是：

1.`Object.prototype` 是顶级对象，所有对象都继承自它
2.`Object.prototype.__proto__ === null`
