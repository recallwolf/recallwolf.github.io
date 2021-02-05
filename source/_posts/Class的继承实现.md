---
title: Class的继承实现
date: 2018-12-11 23:40:18
tags: [ES6, JavaScript] 
categories: JavaScript
description: Class与构造函数
---

ES6新增的“类”与构造函数息息相关，同样的“类”的继承与原型链也存在着紧密的联系。可以参考之前写过一篇关于[原型链](/2018/04/10/js的原型链/)的博客。

#### Class
`class`是ES6语法可以把他看做构造函数的语法糖

    class Point {
      constructor(x, y) {
        this.x = x;
        this.y = y;
      }

      toString() {
        return '(' + this.x + ', ' + this.y + ')';
      }
    }

ES5的构造函数改写

    function Point(x, y) {
      this.x = x;
      this.y = y;
    }

    Point.prototype.toString = function () {
      return '(' + this.x + ', ' + this.y + ')';
    };

    var p = new Point(1, 2);


可以看出构造函数的`prototype`属性，在 ES6 的“类”上面继续存在。类的所有方法都定义在类的`prototype`属性上面。

    class Point {
      constructor() {
        // ...
      }

      toString() {
        // ...
      }

      toValue() {
        // ...
      }
    }

    // 等同于

    Point.prototype = {
      constructor() {},
      toString() {},
      toValue() {},
    };

在类的实例上面调用方法，其实就是调用原型上的方法。

    class B {}
    let b = new B();

    b.constructor === B.prototype.constructor // true

上面代码中，b是B类的实例，它的constructor方法就是B类原型的`constructor`方法。

同样的实例对象的`__proto__`仍然是指向类的`prototype`

    class B {}
    let b = new B();

    b.__proto__ === B.prototype //true

#### Class的继承
Class的本质是构造函数所以继承也是通过原型链的形式，写法上Class可以通过`extends`关键字实现继承

      class A {

      }

      class B extends A {

      }

上面代码定义了一个B类，该类通过extends关键字，继承了A类的所有属性和方法。

由于构造函数同时有`prototype`属性和`__proto__`属性，所以“类”也会有两个属性。

对于“类”来说：  

（1）子类的`__proto__`属性，表示构造函数的继承，总是指向父类。

（2）子类`prototype`属性的`__proto__`属性，表示方法的继承，总是指向父类的`prototype`属性。

    class A {

    }

    class B extends A {

    }

    B.__proto__ === A  //true
    B.prototype.__proto__ === A.prototype  //true

以前说过构造函数的`__proto__`是指向`Fucntion.prototype`，`prototype.__proto__`是指向`Object.prototype`的，对于类A是这样的，对于类B要继承A则要重新指向。

可以通过`Object.setPrototypeOf`函数来重指定

    Object.setPrototypeOf = function (obj, proto) {
      obj.__proto__ = proto;
      return obj;
    }

所以`extends`本质上也是语法糖

    class A {
    }

    class B {
    }

    // B 的实例继承 A 的实例
    Object.setPrototypeOf(B.prototype, A.prototype);

    // B 继承 A 的静态属性
    Object.setPrototypeOf(B, A);

    const b = new B();

同样用ES5构造函数来模拟继承

    function A() {
    }

    function B() {
    }

    // B 的实例继承 A 的实例
    Object.setPrototypeOf(B.prototype, A.prototype);

    // B 继承 A 的静态属性
    Object.setPrototypeOf(B, A);

    const b = new B();

这样继承关系就变成了

    b.__proto__ === B.prototype  //true
    b.constructor === B.prototype.constructor //true

    B.prototype.__proto__ === A.prototype  //true
    B.__proto__ === A  //true

    A.prototype.__proto__ === Object.prototype  //true
    A.__proto__ === Function.prototype  //true


要注意的一点是用ES5构造函数来模拟的时候，所有方法都定义在构造函数的prototype属性上面。