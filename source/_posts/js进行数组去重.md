---
title: JS进行数组去重
date: 2019-01-09 20:55:48
tags: [JavaScript,转载]
categories: JavaScript
comments: false
description: 实现数组去重的几种方式
---
注：本文[转载](https://zhuanlan.zhihu.com/p/24753549)，  作者：TooBug


## 判断相等
### NaN
初看NaN时，很容易把它当成和null、undefined一样的独立数据类型。但其实，它是数字类型。

    // number
    console.log(typeof NaN);

根据规范，比较运算中只要有一个值为NaN，则比较结果为false，所以会有下面这些看起来略蛋疼的结论：

    // 全都是false
    0 < NaN;
    0 > NaN;
    0 == NaN;
    0 === NaN;

以最后一个表达式`0 === NaN`为例，在规范中有明确规定（<http://www.ecma-international.org/ecma-262/6.0/#sec-strict-equality-comparison>）：

>1. If Type(x) is Number, then  
a. If x is NaN, return false.
b. If y is NaN, return false.  
c. If x is the same Number value as y, return true.  
d. If x is +0 and y is −0, return true.  
e. If x is −0 and y is +0, return true.
f. Return false.  

这意味着任何涉及到NaN的情况都不能简单地使用比较运算来判定是否相等。比较科学的方法只能是使用`isNaN()`：

    var a = NaN;
    var b = NaN;

    // true
    console.log(isNaN(a) && isNaN(b));

### 原始值和包装对象
如果你研究过`'a'.trim()`这样的代码的话，不知道是否产生过这样的疑问：'a'明明是一个原始值（字符串），它为什么可以直接调用.trim()方法呢？当然，很可能你已经知道答案：因为JS在执行这样的代码的时候会对原始值做一次包装，让'a'变成一个字符串对象，然后执行这个对象的方法，执行完之后再把这个包装对象脱掉。可以用下面的代码来理解：

    // 'a'.trim();
    var tmp = new String('a');
    tmp.trim();

这段代码只是辅助我们理解的。但包装对象这个概念在JS中却是真实存在的。

    var a = new String('a');
    var b = 'b';

a即是一个包装对象，它和b一样，代表一个字符串。它们都可以使用字符串的各种方法（比如`trim()`），也可以参与字符串运算（+号连接等）。

但他们有一个关键的区别：类型不同！

    typeof a; // object
    typeof b; // string

在做字符串比较的时候，类型的不同会导致结果有一些出乎意料：

    var a1 = 'a';
    var a2 = new String('a');
    var a3 = new String('a');
    a1 == a2; // true
    a1 == a3; // true
    a2 == a3; // false
    a1 === a2; // false
    a1 === a3; // false
    a2 === a3; // false

同样是表示字符串a的变量，在使用严格比较时竟然不是相等的，在直觉上这是一件比较难接受的事情，在各种开发场景下，也非常容易忽略这些细节。


### 对象和对象

在涉及比较的时候，还会碰到对象。具体而言，大致可以分为三种情况：纯对象、实例对象、其它类型的对象。

#### 纯对象

纯对象（plain object）具体指什么并不是非常明确，为减少不必要的争议，下文中使用纯对象指代由字面量生成的、成员中不含函数和日期、正则表达式等类型的对象。
如果直接拿两个对象进行比较，不管是==还是===，毫无疑问都是不相等的。但是在实际使用时，这样的规则是否一定满足我们的需求？举个例子，我们的应用中有两个配置项：

    // 原来有两个属性
    // var prop1 = 1;
    // var prop2 = 2;
    // 重构代码时两个属性被放到同一个对象中
    var config = {
        prop1: 1,
        prop2: 2
    };

假设在某些场景下，我们需要比较两次运行的配置项是否相同。在重构前，我们分别比较两次运行的`prop1`和`prop2`即可。而在重构后，我们可能需要比较`config`对象所代表的配置项是否一致。在这样的场景下，直接用`==`或者`===`来比较对象，得到的并不是我们期望的结果。

在这样的场景下，我们可能需要自定义一些方法来处理对象的比较。常见的可能是通过`JSON.stringify()`对对象进行序列化之后再比较字符串，当然这个过程并非完全可靠，只是一个思路。

如果你觉得这个场景是无中生有的话，可以再回想一下断言库，同样是基于对象成员，判断结果是否和预期相符。
实例对象

实例对象主要指通过构造函数（类）生成的对象。这样的对象和纯对象一样，直接比较都是不等的，但也会碰到需要判断是否是同一对象的情况。一般而言，因为这种对象有比较复杂的内部结构（甚至有一部分数据在原型上），无法直接从外部比较是否相等。比较靠谱的判断方法是由构造函数（类）来提供静态方法或者实例方法来判断是否相等。



#### 其它对象

其它对象主要指数组、日期、正则表达式等这类在Object基础上派生出来的对象。这类对象各有各的特殊性，一般需要根据场景来构造判断方法，决定两个对象是否相等。

比如，日期对象，可能需要通过`Date.prototype.getTime()`方法获取时间戳来判断是否表示同一时刻。正则表达式可能需要通过`toString()`方法获取到原始字面量来判断是否是相同的正则表达式。

#### ==和===
在一些文章中，看到某一些数组去重的方法，在判断元素是否相等时，使用的是==比较运算符。众所周知，这个运算符在比较前会先查看元素类型，当类型不一致时会做隐式类型转换。这其实是一种非常不严谨的做法。因为无法区分在做隐匿类型转换后值一样的元素，例如0、''、false、null、undefined等。

同时，还有可能出现一些只能黑人问号的结果，例如：

    [] == ![]; //true


## 去重实现
### 双重遍历

    function unique(arr) {
      var ret = [];
      var len = arr.length;
      var isRepeat;
      for(var i=0; i<len; i++) {
        isRepeat = false;
        for(var j=i+1; j<len; j++) {
          if(arr[i] === arr[j]){
            isRepeat = true;
            break;
          }
        }
        if(!isRepeat){
          ret.push(arr[i]);
        }
      }
      return ret;
    }

或者

    function unique(arr) {
      var ret = [];
      var len = arr.length;
      for(var i=0; i<len; i++){
        for(var j=i+1; j<len; j++){
          if(arr[i] === arr[j]){
            j = ++i;
          }
        }
        ret.push(arr[i]);
      }
      return ret;
    }

大致的思路是第a元素与后面n-a个元素进行比较    
第一种方式 如果不相等，退出二层循环推入数组直到最后没有相等的元素   
第二种方式 如果相等则在跳过第一层一次循环（相当于第一层循环直接进下一次），如果第二层循环都不相等则推入数组

注意的是由于`NaN === NaN`返回的是false，所以不能使用该表达是判断NaN，否则两种方式仍会有NaN重复
可以使用`isNaN()`添加一层判断

### Array.prototype.indexOf()
  `Array.prototype.filter` 为数组中的每个元素调用一次 `callback` 函数，并利用所有使得 `callback` 返回 `true` 或 等价于 `true` 的值 的元素创建一个新数组

  `Array.prototype.indexOf` 使用`strict equality `(无论是 ===, 还是 `triple-equals`操作符都基于同样的方法)进行判断 `searchElement`与数组中包含的元素之间的关系。

    function unique(arr) {
      return arr.filter(function(item, index){
        // indexOf返回第一个索引值，
        // 如果当前索引不是第一个索引，说明是重复值
        return arr.indexOf(item) === index;
      });
    }

或者

    function unique(arr) {
      var ret = [];
      arr.forEach(function(item){
        if(ret.indexOf(item) === -1){
          ret.push(item);
        }
      });
      return ret;
    }

第一种是通过判断索引是否是第一个索引来判断重复与否 通过`filter`函数过滤掉不等于第一个索引的数值

第二种是通过遍历arr数组的值找出ret数组是否存在相同的值，再判断是否推入ret数组

通过[规范]（<http://www.ecma-international.org/ecma-262/6.0/#sec-array.prototype.indexof>），indexOf()使用的是严格比较，也就是===。所以第一种方式NaN都不存在，第二种方式仍存在。

>再次强调：按照前文所述，===不能处理NaN的相等性判断。

### Array.prototype.includes()
`Array.prototype.includes()`方法用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 true，否则返回false。

    function unique(arr) {
      var ret = [];
      arr.forEach(function(item){
          if(!ret.includes(item)){
              ret.push(item);
          }
      });
      return ret;
    }


### 使用对象key来去重

由于对象的key不可重复性

    function unique(arr) {
      var ret = [];
      var len = arr.length;
      var tmp = {};
      for(var i=0; i<len; i++){
        if(!tmp[arr[i]]){
            tmp[arr[i]] = 1;
            ret.push(arr[i]);
        }
      }
      return ret;
    }

但由于对象key只能为字符串，因此这种去重方法有许多局限性：
1. 无法区分隐式类型转换成字符串后一样的值，比如1和'1'

2. 无法处理复杂数据类型，比如对象（因为对象作为key会变成[object Object]）

3. 特殊数据，比如`__proto__`会挂掉，因为tmp对象的`__proto__`属性无法被重写

对于第一点和第三点，有人提出可以为对象的key增加一个类型，或者将类型放到对象的value中来解决：

    function unique(arr) {
        var ret = [];
        var len = arr.length;
        var tmp = {};
        var tmpKey;
        for(var i=0; i<len; i++){
            tmpKey = typeof arr[i] + arr[i];
            if(!tmp[tmpKey]){
                tmp[tmpKey] = 1;
                ret.push(arr[i]);
            }
        }
        return ret;
    }

而第二个问题，如果像上文所说，在允许对对象进行自定义的比较规则，也可以将对象序列化之后作为key来使用。这里为简单起见，使用JSON.stringify()进行序列化。

    function unique(arr) {
      var ret = [];
      var len = arr.length;
      var tmp = {};
      var tmpKey;
      for(var i=0; i<len; i++){
          tmpKey = typeof arr[i] + JSON.stringify(arr[i]);
          if(!tmp[tmpKey]){
              tmp[tmpKey] = 1;
              ret.push(arr[i]);
          }
      }
      return ret;
    }

但是这样会导致正则表达式消失序列化之后为{}

### Map Key

可以看到，使用对象key来处理数组去重的问题，其实是一件比较麻烦的事情，处理不好很容易导致结果不正确。而这些问题的根本原因就是因为key在使用时有限制。

那么，能不能有一种key使用没有限制的对象呢？答案是——真的有！那就是ES2015中的Map。

>Map是一种新的数据类型，可以把它想象成key类型没有限制的对象。此外，它的存取使用单独的get()、set()接口。

    var tmp = new Map();
    tmp.set(1, 1);
    tmp.get(1); // 1
    tmp.set('2', 2);
    tmp.get('2'); // 2
    tmp.set(true, 3);
    tmp.get(true); // 3
    tmp.set(undefined, 4);
    tmp.get(undefined); // 4
    tmp.set(NaN, 5);
    tmp.get(NaN); // 5
    var arr = [], obj = {};
    tmp.set(arr, 6);
    tmp.get(arr); // 6
    tmp.set(obj, 7);
    tmp.get(obj); // 7

由于Map使用单独的接口来存取数据，所以不用担心key会和内置属性重名（如上文提到的`__proto__`）。使用Map改写一下我们的去重方法：

    function unique(arr) {
      var ret = [];
      var len = arr.length;
      var tmp = new Map();
      for(var i=0; i<len; i++){
        if(!tmp.get(arr[i])){
          tmp.set(arr[i], 1);
          ret.push(arr[i]);
        }
      }
      return ret;
    }


### Set

既然都用到了ES2015，数组这件事情不能再简单一点么？当然可以。

除了Map以外，ES2015还引入了一种叫作Set的数据类型。顾名思义，Set就是集合的意思，它不允许重复元素出现，这一点和数学中对集合的定义还是比较像的。

    var s = new Set();
    s.add(1);
    s.add('1');
    s.add(null);
    s.add(undefined);
    s.add(NaN);
    s.add(true);
    s.add([]);
    s.add({});

如果你重复添加同一个元素的话，Set中只会存在一个。包括NaN也是这样。于是我们想到，这么好的特性，要是能和数组互相转换，不就可以去重了吗？

    function unique(arr){
        var set = new Set(arr);
        return Array.from(set);
    }

我们讨论了这么久的事情，居然两行代码搞定了，简直不可思议。

然而，不要只顾着高兴了。有一句话是这么说的“不要因为走得太远而忘了为什么出发”。我们为什么要为数组去重呢？因为我们想得到不重复的元素列表。而既然已经有Set了，我们为什么还要舍近求远，使用数组呢？是不是在需要去重的情况下，直接使用Set就解决问题了？这个问题值得思考。

## 小结

    var arr = [1,1,'1','1',0,0,'0','0',undefined,undefined,null,null,NaN,NaN,{},{},[],[],/a/,/a/]
    console.log(unique(arr));

>测试中没有定义对象的比较方法，因此默认情况下，对象不去重是正确的结果，去重是不正确的结果。

{% asset_img unique.png %}