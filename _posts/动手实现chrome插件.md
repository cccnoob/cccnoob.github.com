---
title: 简单实现chrome插件
date: 2017-12-18 21:00:10
categories:
- chrome
tags:chrome
---

>  [图灵社区《Chrome扩展及应用开发（首发版）》](http://www.ituring.com.cn/book/1421)
>  这本书里面讲的很详细，推荐去看看。

## 一、项目结构
![](http://p01zuhw8m.bkt.clouddn.com/2017/12/18a1.png)

***

### Manifest文件
Chrome扩展都包含一个Manifest文件——manifest.json，这个文件可以告诉Chrome关于这个扩展的相关信息，它是整个扩展的入口，也是Chrome扩展必不可少的部分。
    
```
{
    "manifest_version": 2,
    "name": "隐藏图片",
    "version": "1.0",
    "description": "可暂时显示隐藏图片",
    "icons": {
        "16": "icon.png",
        "48": "icon.png",
        "128": "icon.png"
    },
    "browser_action": {
        "default_icon": {
            "19": "icon.png",
            "38": "icon.png"
        },
        "default_title": "time",
        "default_popup": "popup.html"
    },
    "content_scripts": [
        {
            "matches": ["http://*/*","https://*/*"],
            "js": ["js/content_script.js"],
            "run_at": "document_end",
            "all_frames": true 
        }
    ]
}
```
***

### content_script.js 文件
通过Chrome扩展我们可以对用户当前浏览的页面进行操作，实际上就是对用户当前浏览页面的DOM进行操作。通过Manifest中的content_scripts属性可以指定将哪些脚本何时注入到哪些页面中，当用户访问这些页面后，相应脚本即可自动运行，从而对页面DOM进行操作。


```
var imgs = document.getElementsByTagName('img');
chrome.runtime.onConnect.addListener(function (port) {
    if(port.name == "hidimg"){
        port.onMessage.addListener(function (msg) {
            if(msg.jia){
                for(var i = 0;i<imgs.length;i++){
                   imgs[i].style.visibility = 'hidden';
                }
                port.postMessage({jia: false});
            }else{
                for(var i = 0;i<imgs.length;i++){
                   imgs[i].style.visibility = 'visible';
                }
                port.postMessage({jia: true});
            }
        });
    }
});
```
content_scripts中的脚本只是共享页面的DOM1，而并不共享页面内嵌JavaScript的命名空间。也就是说，如果当前页面中的JavaScript有一个全局变量a，content_scripts中注入的脚本也可以有一个全局变量a，两者不会相互干扰。当然你也无法通过content_scripts访问到页面本身内嵌JavaScript的变量和函数。

***

### popup.html 文件
>Popup页面是当用户点击扩展图标时，展示在图标下面的页面。
>像这样

![](http://p01zuhw8m.bkt.clouddn.com/a2.png)

```
<html>
<head>
    <style>
        * { margin: 0; padding: 0; }
        body { width: 200px; height: 120px; }
        div { line-height: 70px; font-size: 38px; text-align: center; }
        #hide_img{display: block;margin:10px auto; width:76px;height:30px;line-height: 30px;color:#fff;background:#3B5998;border-radius: 5px;text-align: center;cursor: pointer;user-select: none;}
        #hide_img:hover{background:#CBD9F5;color:#3B5998;}
    </style>
</head>
<body>
<div id="clock_div"></div>
<span id="hide_img">隐藏图片</span>
<script src="js/popup.js"></script>
</body>
</html>
```
***

### popup.js 文件
>Popup页面 的脚本文件 可以调用 background.js 文件 也可以和 content_script.js 进行通信。

```
function my_clock(el){
    var today=new Date();
    var h=today.getHours();
    var m=today.getMinutes();
    var s=today.getSeconds();
    m=m>=10?m:('0'+m);
    s=s>=10?s:('0'+s);
    el.innerHTML = h+":"+m+":"+s;
    setTimeout(function(){my_clock(el)}, 1000);
}

var clock_div = document.getElementById('clock_div');
my_clock(clock_div);

var hideImg = document.getElementById('hide_img');
var isShow = true;

chrome.tabs.query(
{active: true, currentWindow: true},
function (tabs) {
    var port = chrome.tabs.connect(//建立通道
        tabs[0].id,
        {name: "hidimg"}//通道名称
    );
    hideImg.onclick = function(){
        port.postMessage({jia: isShow});
    }
    port.onMessage.addListener(function (msg) {
        if(msg.jia){
            hideImg.innerHTML = "隐藏图片";
            isShow = msg.jia;
        }else{
            hideImg.innerHTML = "显示图片";
            isShow = msg.jia;
        }
    });
});
```

***

### icon.png 图片
>插件的 各种图片，具体的看介绍，我这里所有的都用的一张。。。

Browser Actions可以在 Manifest 中设定一个默认的图标，比如：
```
"browser_action": {
    "default_icon": {
        "19": "images/icon19.png",
        "38": "images/icon38.png"
    }
}
```

一般情况下，Chrome会选择使用19像素的图片显示在工具栏中，但如果用户正在使用视网膜屏幕的计算机，则会选择38像素的图片显示。两种尺寸的图片并不是必须都指定的，如果只指定一种尺寸的图片，在另外一种环境下，Chrome会试图拉伸图片去适应，这样可能会导致图标看上去很难看。

***


## 二、运行调试


### 进入 chrome://extensions/

![](http://p01zuhw8m.bkt.clouddn.com/a3.png)
![](http://p01zuhw8m.bkt.clouddn.com/a6.png)
![](http://p01zuhw8m.bkt.clouddn.com/a5.png)


### 调试

![](http://p01zuhw8m.bkt.clouddn.com/a2.png)

右键popup页面 可以打开控制台。

***

## 三、打包

### 选择打包扩展程序

![](http://p01zuhw8m.bkt.clouddn.com/2017/12/18a8.png)

### 复制文件夹
这里选择 文件夹之前 要把文件 所在的文件夹复制一份
比如

![](http://p01zuhw8m.bkt.clouddn.com/2017/12/18a13.png)

如果选择 之前的文件夹 
![](http://p01zuhw8m.bkt.clouddn.com/2017/12/18a9.png)

 就会出现
![](http://p01zuhw8m.bkt.clouddn.com/2017/12/18a10.png)

所以 把程序根目录选择为刚 复制好的 文件夹
![](http://p01zuhw8m.bkt.clouddn.com/2017/12/18a12.png)

直接点击 打包扩展程序 就成功了，
crx 后缀的文件 就是一个chrome插件了。
![](http://p01zuhw8m.bkt.clouddn.com/2017/12/18a14.png)

***
