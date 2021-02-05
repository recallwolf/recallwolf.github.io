---
title: 常用css技巧
date: 2018-09-18 17:01:27
categories: CSS
tags: CSS
description: css的一些经典技巧
---

### css创建纯三角  

    #demo {
        width: 0;
        height: 0;
        border-width: 20px;
        border-style: solid;
        border-color: transparent transparent red transparent;
    }

结果： {% asset_img triangle.png %}
### css水平居中  

   #### 1.设置宽度 添加margin: 0 auto属性  

    #demo {
        width: 50px;
        height: 50px;
        margin: 0 auto;
        background-color: red;
    }

   #### 2.浮动元素居中

    #demo {
        width: 60px;
        height: 50px;
        margin: -25px 0 0 -30px;
        position: relative;
        background-color: red;
        left: 50%;
        right: 50%;
    }

   #### 3.绝对定位居中

    #demo {
        position: absoulte;
        width: 50px;
        height: 50px;
        background-color: red;
        margin: 0 auto;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0; 
    }

   #### 4.flex弹性盒子

    .container {
        display: flex;
        width: 70px;
        height: 70px;
        border-style: solid;
        justify-content: center; /*水平居中*/
    }
    .demo {
        width: 30px;
        height: 30px;
        background-color: red;
    }


### css垂直居中  

   #### 1.单行文本 line-height设置和height值相等
   #### 2.已知高度块级子元素 绝对定位+负边距

    .container {
		width: 60px;
        position: relative;
        height: 100px;
        border-style: solid;
    }
    .demo {
        width: 60px;
        height: 50px;
        position: absolute;
        top: 50%;
        margin-top: -25px;
        background-color: red;
    }

   #### 3.未知高度的块级子元素居中 模拟表格布局
   (IE6,7不兼容 父级overflow: hidden无效)

        .container {
            display: table;
            border-style: solid;
            width: 50px;
            height: 50px;
        }
        .demo {
            display: table-cell;
            vertical-align: middle;
        }
        .cell {
        	height: 20px;
            background-color: red;
        }

   #### 4.flex弹性盒子

    .container {
        display: flex;
        width: 70px;
        height: 70px;
        border-style: solid;
        justify-content: center; /*水平居中*/
        align-items: center; /*垂直居中*/
    }
    .demo {
        width: 30px;
        height: 30px;
        background-color: red;
    }

### 圣杯布局
三列布局 中间主题内容前置，宽度自适应   
两边内容定宽
利用相对定位，浮动，负边距布局

css部分

    <style>
        .container {
            padding-left: 190px;
            padding-right: 190px;
        }
        .main {
            float: left;
            min-height: 130px;
            width: 100%;
            background-color: yellow;
        }
        .left {
            float: left;
            width: 190px;
            min-height: 130px;
            position: relative;
            left: -190px;
            margin-left: -100%;
            background-color: red;
        }
        .right {
            float: left;
            width: 190px;
            min-height: 130px;
            margin-left: -190px;
            position: relative;;
            right: -190px;
            background-color: blue;
        }
	</style>

dom结构

    <div class="container">
        <div class="main"></div>
        <div class="left"></div>
        <div class="right"></div>
	</div>

结果：{% asset_img grail.png %}

### 双飞翼布局
对圣杯布局的改进，消除相对定位布局  
主体元素上设置左右边距，预留两翼位置。左右两栏使用浮动和负边距归位，消除相对定位。

css部分

    <style>
        .container {
            width: 100%;
            float: left;
        }
        .main {
            margin-left: 190px;
            margin-right: 190px;
            min-height: 130px;
            background-color: yellow;
        }
        .left {
            float: left;
            width: 190px;
            min-height: 130px;
            margin-left: -100%;
            min-height: 130px;
            background-color: red;
        }
        .right {
            float: left;
            width: 190px;
            min-height: 130px;
            margin-left: -190px;
            min-height: 130px;
            background-color: blue;
        }
    </style>

dom结构

    <div class="container">
		<div class="main"></div>
	</div>
	<div class="left"></div>
	<div class="right"></div>

结果：{% asset_img wings.png %}