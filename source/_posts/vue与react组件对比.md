---
title:  vue与react组件对比
date: 2018-08-13 00:08:36
tags: [vue, react]
categories: JavaScript
comments: false
description: vue与react数据绑定,DOM渲染,生命周期之间的对比
---

## 1.数据渲染
### Vue
Vue采用了传统的 html 模板 通过`{% raw %}{{}}{% endraw %}`或者v-text语法来进行数据的渲染  
对于数组数据 Vue 中使用类似 Smarty 语法的 `v-for`指令来渲染数组  
可以使用computed来对需要渲染的数据进行处理

### React
React通过`{}`将数据嵌套在JSX表达式中  
对于数组数据 React利用`map`将数据片段映射成一段 JSX，然后在列表区域中直接引用`map`后的 JSX 变量来渲染数组数据  
React处理渲染的数据 需要 render 函数中定义一个新变量，将转换后的值赋给这个变量后 render 


__渲染数组 以数组 a 为例__  
Vue

    <p v-for="num in a">{{num}}</p>

React

    render() {
        {a.map((num) => 
            <p>{num}</p>
        )}
    }  

  
## 2.数据更新
### Vue  
由于Vue和React实现原理不同所以数据更新的方式也不同  
Vue Hack 了对象的 setter getter 
赋值操作时可以setter直接获知状态树中的修改位置

    this.age = 18

__Vue 中数据状态的更新是即时的 DOM状态是异步更新的__  
有些时候需要在数据变动 DOM重渲染完做一些操作需要用`Vue.nextTick(callback)` DOM 更新完成后就会调用

### React  
React 需要显式 setState 来获知变更的内容触发DOM的重渲染 

    this.setState({ age: 18 })

__React 中 DOM 状态和数据状态都是异步更新的__

## 3.数据的双向绑定
### Vue
Vue提供了`v-model`语法对 input 数据进行双向绑定 本质上是v-bind绑定事件和v-on监听事件组成的语法糖

### React
React语法 进行数据绑定 onChange事件进行监听数据变化

## 4.事件传递
### Vue
> Vue 通过v-on和$emit来实现的。在父组件模板中声明子组件时，通过 `v-on:childEventName="parentHandler"` 语法来指定对子组件特定名称事件的 Handler method

### React
> React Handler 同样是在父组件中声明，但 Handler 需要以 props 的形式传入子组件，在子组件中触发事件时，以 `this.props.parentHandler` 形式调用父组件传入的 props 状态

## 5.生命周期
### Vue

{% asset_img vue.png %}

>Vue destroyed 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。
不会清除已有DOM。

### React

{% asset_img react.png %}  

图片来源   (https://huanghui8030.github.io/react/component.html)
>React 卸载后不会清除组件的实例 需要在componentWillUnmount生命钩子手动清除  

例如ajax的数据还没获取到就dom就已经不存在，会产生报错
>Warning: setState(...): Can only update a mounted or mounting component. This usually means you called setState() on an unmounted component. This is a no-op. Please check the code for the Timer component.

所以需要在 componentWillUnmount() 手动清除

    componentWillUnmount(){
        this.setState=()=>{};
    }

