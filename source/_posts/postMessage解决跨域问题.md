---
layout: w
title: postMessage解决跨域问题
date: 2018-04-09 17:53:23
tags: postMessage
categories: 前端
---

## 需求

项目首页部署在一个域名之下，首页提供入口，集成其它不同域名的第三方应用，并在第三方应用中**“提交”**成功后再跳转至首页。

![](ky.png)
<!--more-->
## 方案

我首先想到的是：在首页点击跳转时，新开一个含iframe的页面链接第三方应用，在iframe同级添加 **”返回首页“** 按钮用来跳转到首页，这个方案虽解决了跨域页面之间的通信问题却无用户体验可言。本着以用户为中心的产品理念，寻找一个解决跨域通信的方案成了必然选择。
{% codeblock lang:objc %}	
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no, minimal-ui">
	<title></title>
</head>
<body>
	<button>返回首页</button>
	<iframe src='https://www.e.com'></iframe>
</body>
</html>	
{% endcodeblock %}

## postMessage 和 onmessage

HTML5 提供了在网页文档之间互相接收与发送信息的功能。使用这个功能，只要获取到网页所在窗口对象的实例，不仅同源（域 + 端口号）的 Web 网页之间可以互相通信，甚至可以实现跨域通信。
我们在网页所在的对象实例上执行postMessage方法传输数据。该方法使用两个参数：第一个参数为所发送的消息文本，但也可以是任何 JavaScript 对象（通过 JSON 转换对象为文本），第二个参数为接收消息的对象窗口的 URL 地址，可以在 URL 地址字符串中使用通配符'\*'指定全部的url。
要想接收从其他窗口发送来的信息，我们需要对窗口对象的 onmessage 事件进行监听。

###### 父页面
{% codeblock lang:objc %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title></title>
  <style>
    ol{
      margin: 0;
	  padding: 0;
	  list-style-position: inside;
	  line-height: 30px
	}
	iframe{
	  border: 1px ddd solid
	}
  </style>
</head>
<body>
  <iframe id="frame" width="500" height="200" src="http://10.1.61.34:8080/personalFile/luo/postMessage/child.html"></iframe>
  <br>
  <input id="input" type="text" value="父页面数据" />
  <button id="button">提交</button>
  <ol id="list"></ol>
  <script>
		
	var btn=document.getElementById('button'),
	  Input=document.getElementById('input'),
	  list=document.getElementById('list'),
	  iframe=document.getElementById('frame'),
	  originalWin=iframe.contentWindow;

	iframe.onload=function(){
	  originalWin.postMessage({
	    data:Input.value
	  },'http://10.1.61.34:8080');
	}

	btn.onclick=function(){
	  originalWin.postMessage({
	    data:Input.value
	  },'http://10.1.61.34:8080');
	}

	window.onmessage = function(e){  

	// if(e.origin !== 'http://localhost:8080') return;  //只接受制定域名传输数据

	  var oLi = document.createElement('li');
	  oLi.innerHTML = e.data.data;
	  list.appendChild(oLi);
	}

	</script>
</body>
</html>
{% endcodeblock %}

###### 子页面
{% codeblock lang:objc %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title></title>
  <style>
	ol{
	  margin: 0;
	  padding: 0;
	  list-style-position: inside;
	  line-height: 30px
	}
  </style>
</head>
<body>
  我是子页面
  <br>
  <br>
  <ol id="list"></ol>
  <input value="子页面数据" id="input" type="text" value="" />
  <button id="button">提交</button>
  <script>

	var list=document.getElementById('list'),
		btn=document.getElementById('button'),
		Input=document.getElementById('input');;
	window.onmessage = function(e){  

	  // if(e.origin !== 'http://localhost:8080') return; //只接受制定域名传输数据 
	  console.log(e.data.data);
	  var oLi = document.createElement('li');
	  oLi.innerHTML = e.data.data;

	  list.appendChild(oLi);

	  btn.onclick=function(){
		e.source.postMessage({
		  data:Input.value
		},e.origin);
	  }

	}

	</script>
</body>
</html>
{% endcodeblock %}

###### 效果图

![](2.png)

点击按钮父子级页面进行数据插入
