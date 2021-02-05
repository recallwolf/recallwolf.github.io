---
title: Nodejs和树莓派小车
date: 2018-12-03 14:31:46
tags: [RaspberryPi, Nodejs]
categories: RaspberryPi
comments: false
description:  nodejs驱动树莓派
---

简略的说一下树莓派小车的制作过程

所需材料：  
>  1.树莓派3b+  
  2.l298n电机驱动模块  
  3.3.3v直流电机  
  4.dht11温湿度传感器  
  5.sg90舵机

#### 基本接线图 
{% asset_img sketch.jpg %} 


#### 小车代码
一开始采用的是rpio库但是pwm的问题一直没解决
提了个issue https://github.com/jperkin/node-rpio/issues/80

之后采用了johnny-five库， raspi-io库。
由于JavaScript控制微秒级不精准所以采用了第三方库node-dht-sensor调用c++来控制传感器，之后通过websocket传递温湿度

摄像头部分 用ffmpeg流传输到websocket转发到http上

所需环境： 

>  1.os: raspbian  
  2.nodejs  
  3.ffmpeg

具体api文档：  
>[Johnny-Five](http://johnny-five.io/api/)  
[raspi-io](https://github.com/nebrius/raspi-io)  
[node-dht-sensor](https://github.com/momenso/node-dht-sensor)  
[JSMpeg](https://github.com/phoboslab/jsmpeg)

    var express = require('express');
    var app = express();
    var path = require('path');
    var http = require('http').Server(app);
    var five = require('johnny-five');
    var Raspi = require('raspi-io');
    var sensor = require('node-dht-sensor');
    var ws = require('socket.io')(http);

    app.use(express.static(path.join(__dirname, 'public')));

    var board = new five.Board({
      io: new Raspi()
    });

    function dht11() {
      return sensor.read(11, 26)
    }

    board.on('ready', () => {

      var io23 = new five.Pin({pin:23,mode:4}); //pwm
      var io0 = new five.Pin({pin:0,mode:1}); //board_11 left back
      var io1 = new five.Pin({pin:1,mode:1}); //board_12 left push
      var io2 = new five.Pin({pin:2,mode:1}); //board_13 right push
      var io3 = new five.Pin({pin:3,mode:1}); //board_15 right back

      var angle = 90;
      var servo = new five.Servo({pin:23,startAt:angle});

      function set_low(){
        io0.low();
        io1.low();
        io2.low();
        io3.low();
      };

      set_low(); //set default

      ws.on('connection',function(socket) {
        setInterval(() => {
          ws.emit('sensor', dht11());
        }, 15000);
      });

      app.get('/', function(req, res){
        res.render('index');
      });

      app.post('/w', function(req, res){
        res.send('w');
        io2.high();
        io1.high();
      });

      app.post('/ws', function(req, res){
        console.log('done');
        res.send('ws');
        io2.high();
        io1.high();
        setTimeout(() => {
          set_low();
        }, 800);
      });

      app.post('/l', function(req, res){
        res.send('l');
        io0.high();
        io2.high();
      });

      app.post('/ls', function(req, res){
        res.send('ls');
        io0.high();
        io2.high();
        setTimeout(() => {
          set_low();
        }, 600);
      });

      app.post('/r', function(req, res){
        res.send('r');
        io1.high();
        io3.high();
      });
      
      app.post('/rs', function(req, res){
        res.send('rs');
        io1.high();
        io3.high();
        setTimeout(() => {
          set_low();
        }, 600);
      });

      app.post('/b', function(req, res){ 
        res.send('b');
        io0.high();
        io3.high();
      });
        
      app.post('/bs', function(req, res){ 
        res.send('bs');
        io0.high();
        io3.high();
        setTimeout(() => {
          set_low();
        }, 800);
      });
      
      app.post('/stop', function(req, res){
        res.send('stop');
        set_low();
      });

      app.post('/cl', function(req, res){ 
        res.send('cl');
        if (angle >= 0 && angle <= 180) {
          angle = angle - 45;
          servo.to(angle);
        }
      });

      app.post('/cr', function(req, res){ 
        res.send('cr');
        if (angle >= 0 && angle <= 180) {
          angle = angle + 45;
          servo.to(angle);
        }
      });

      app.post('/creset', function(req, res){ 
        res.send('creset');
        angle = 90;
        servo.to(angle);
      });
    });

    http.listen(3000, function(){
      console.log('listening on *:3000');
    });

可以把这些功能封装成一个对象 可能可读性更高

同时要启动ffmpeg和ws服务器

前端显示：
{% asset_img index.png %} 


#### 语音控制代码
{% asset_img pi0.jpg %} 
之前用了respeaker的2mic的麦克风阵列 原来的的树莓派3b，没有足够的io接口，所以用raspberrypi zero来做语音终端向3b发送控制指令。

所需环境
>  1.os: raspbian  
  2.nodejs  
  3.arecord

文档：  
>[respeaker文档](http://wiki.seeedstudio.com/cn/ReSpeaker_2_Mics_Pi_HAT/#2)

大致流程：   
**安装好respeaker驱动(respeaker文档) => arecord录音 => 调用百度语音识别api，上传百度服务器 => 拿到识别结果 向3b上的http服务器发送消息**

    var fs = require ('fs');
    var exec = require('child_process').exec; 
    var axios = require('axios');

    function get_token() {
      var url = 'https://aip.baidubce.com/oauth/2.0/token?'
      return axios.get(url, {
        params: {
          grant_type: 'client_credentials',
          client_id: 'Your ID',
          client_secret: 'Your Secret'
        }
      }).then((response) => {
        return Promise.resolve(response.data.access_token)
      })
      .catch((error) => {
        console.log(error);
      });
    }

    function get_text(token, data) {
      var data = fs.readFileSync('demo.pcm');
      return axios({
        url: `http://vop.baidu.com/server_api?cuid=1E-30-7E-09-21-FD&token=${token}`,
        method: 'post',
        data: data,
        headers: {
          'Content-Type': 'audio/pcm;rate=16000'
        }
      }).then(function(response) {
        console.log(response.data)
        return Promise.resolve(response.data.result[0]);
      }).catch(function (error) {
        console.log(error);
      });
    }

    function forward() {
      axios.post('http://192.168.1.xx:3000/ws', {});
    }

    function back() {
      axios.post('http://192.168.1.xx:3000/bs', {});
    }

    function left() {
      axios.post('http://192.168.1.xx:3000/ls', {});
    }

    function right() {
      axios.post('http://192.168.1.xx:3000/rs', {});
    }

    function record() {
      var cmdStr = 'arecord -d 3 -c 1 -f cd -r 8000 -Dhw:1 /home/voice/demo.pcm';
      console.log('----------------begin recording----------------');
      return new Promise((resolve,reject) => {
        exec(cmdStr, function(err, stdout, stderr) {
          if (err) {
            reject(err);
          }
          else {
            resolve();
            console.log('----------------end recording------------------');
          }
        });
      });
    }

    setInterval(() => {
      record()
      .then(get_token)
      .then(get_text)
      .then(function(text) {
        console.log(text);
        if(text === '前进，') {
          forward();
        }
        else if(text === '后退，') {
          back();
        }
        else if(text === '左转，') {
          left();
        }
        else if(text === '右转，') {
          right();
        }
      });
    }, 4500);

终端如图：
{% asset_img terminal.png %}  

{% asset_img car.jpg %} 