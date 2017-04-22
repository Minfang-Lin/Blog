---
title: JavaScript事件处理程序笔记
date: 2015-08-22 19:40
tags: JavaScript
---

**事件流**：描述的是从页面中接受事件的顺序。

**事件冒泡**：即事件最开始由最具体的元素（文档中嵌套层次最深的那个节点）接收，然后逐级向上传播至最不具体的那个节点（文档）
——IE浏览器的处理方式。

**事件捕获**：不太具体的节点应该更早接收到事件，而最具体的节点最后接收到事件。
——Netscape浏览器的处理方式。

## 使用事件处理程序
1.HTML事件处理程序

``` html
<!--直接嵌入JS代码-->
<input type="button" id="btn" onclick="alert('hello world')">
<!--调用函数-->
<input type="button" id="btn" onclick="showMsg()">
<script type="text/javascript">
	function showMsg(){
		alert("hello world");
	}
</script>
```

缺点：HTML和JS紧密耦合在一起，若需要修改程序则需要同时修改HTML和JS，故已经基本被摒弃。
<!--more-->
2.DOM 0级事件处理程序
``` html
<input type="button" id="btn">
<script type="text/javascript">
	var btn = document.getElementById("btn");
	btn.onclick = function(){
		alert("hello world");
	}
	btn.onclick = null; //删除事件
</script>
```
较传统HTML方式，把一个函数赋值给一个事件的处理程序属性。此方法使用较多，可跨浏览器。

3.DOM 2级处理程序
定义了两个方法，用于处理指定和删除事件处理程序的操作，分别是`addEventListener()`和`removeEventListener()`
接收三个参数：要处理的事件名，作为事件处理程序的函数，布尔值（true表示在捕获阶段处理，false表示在冒泡阶段处理，常用false）。
``` html
<input type="button" id="btn">
<script type="text/javascript">
	var btn = document.getElementById("btn");
	btn.addEventListener('click', showMsg, false);
	btn.removeEventListener('click', showMsg, false);  //不能用onclick=null删除事件
	function showMsg(){
		alert("hello world");
	}
</script>
```
DOM 0级和2级都可以添加多个事件处理程序。

4.IE事件处理程序
`attachEvent()`添加事件和`detachEvent()`删除事件。
接收两个参数：事件处理程序的名称和事件处理程序的函数，IE8及之前版本的IE浏览器只支持事件冒泡，所以不像DOM 2级有第三个参数。
``` html
<input type="button" id="btn">
<script type="text/javascript">
	var btn = document.getElementById("btn");
	btn.attachEvent('click', showMsg);
	btn.detachEvent('click', showMsg);
	function showMsg(){
		alert("hello world");
	}
</script>
```

5.跨浏览器的事件处理
``` html
<input type="button" id="btn">
<script type="text/javascript">
	var eventUtil={
		// 添加句柄
		addHandler:function(element,type,handler){
			if(element.addEventListener){
				element.addEventListener(type,handler,false);
			}else if(element.attachEvent){
				element.attachEvent('on'+type,handler);
			}else{
				element['on'+type]=handler;
			}
		},
		// 删除句柄
		removeHandler:function(element,type,handler){
			if(element.removeEventListener){
				element.removeEventListener(type,handler,false);
			}else if(element.detachEvent){
				element.detachEvent('on'+type,handler);
			}else{
				element['on'+type]=null;
			}
		},
	}
	var btn = document.getElementById("btn");
	eventUtil.addHandler(btn, 'click', showMsg);
	eventUtil.removeHandler(btn, 'click', showMsg);
	function showMsg(){
		alert("hello world");
	}
</script>
```
