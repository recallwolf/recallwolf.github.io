---
title: vue结合express后台开发
date: 2018-05-04 17:22:29
categories: vue
tags: vue
comments: false
description: vue与express协同开发
---

最近打算写一个有即时通讯功能的app练手，用到之前使用过的`socket.io`库，所以需要`vue`与`express`同步协调api（一个人开发就不打算前后端分离了，方便即时调控）

### 项目准备
1.创建webpack模板(需要安装vue-cli)
```
vue init webpack demo
```

2.安装依赖
```
cd demo
npm install
```

3.修改文件结构  
将`src`文件夹修改为`client`  
将`webpack.base.conf.js`内的`src`地址修改为`client`

4.创建服务端(需要安装express-generator)
```
express server
cd server
npm install
```
5.修改文件结构  
将 ***app.js*** 中
`app.use(express.static(path.join(__dirname, 'public')));`
修改为
`app.use(express.static(path.resolve(__dirname, '../dist')));`

### 打包部署
```
cd demo
npm run build
cd server
npm start
```

访问`http://localhost:3000`
