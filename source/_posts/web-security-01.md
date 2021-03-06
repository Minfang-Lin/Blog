---
title: Web安全学习笔记（1）
date: 2017-03-25 18:55:06
tags: Web安全
---

## Web介绍

### Web发展史

#### 什么是Web？

Web的本意指网，这里指的是万维网（World Wide Web）。它是由许多互相链接的超文本组成的，通过互联网访问。Web是非常普遍的互联网应用，每天都有数以亿万计的Web资源，如图片、html页面、音频、视频等，从全世界的Web服务器快速的传递到我们的浏览器上。

#### Web1.0与Web2.0

* Web1.0以个人网站、门户站点为主，主要使用大量静态页面，提供信息给用户。只能阅读，不能添加或修改。常见的安全问题包括SQL注入、文件包含、命令执行、上传漏洞、挂马、暗链等。主要危害Web服务器。
* Web2.0如微博、博客，可以进行人与人之间的互动。常见的安全问题包括XSS、CSRF、数据劫持、URL跳转、钓鱼、框架漏洞、逻辑漏洞等。逐渐开始针对上网的Web用户。

Web安全问题数量迅速增长，种类迅速增多，从危害服务器逐渐转变为针对Web用户。

### Web流程

1. 用户打开浏览器
2. 在浏览器中输入URL
3. 浏览器向服务器发送请求
4. Web服务器收到请求后做相应处理（如果有数据相关操作，服务器先会与数据库进行交互）
5. 服务器处理完之后会返回处理结果
6. 浏览器收到处理结果后将页面展示给用户

### 浏览器

通过域名获取Web服务器的IP地址（DNS解析），通过解析到的IP地址访问Web服务器。

## Web通信

### URL协议

URL（Uniform Resource Locator，统一资源定位符），支持多种协议，如HTTP、FTP等。URL用于定位服务器资源。

```
schema://host[:port#]/path/.../[?query-string][#anchor]
  ①       ②    ③     ④            ⑤           ⑥
  
① schema：底层协议，例如http、https、ftp
② host：我们要访问的服务器域名或IP地址
③ port：服务器端口号，HTTP默认端口是80（可省略），其他端口要指明
④ path：访问资源的路径
⑤ ?query-string：发送给http服务器的数据
⑥ #anchor：锚点，一般表示页面的指定位置
```

### HTTP协议

超文本传输协议（Hyper Text Transfer Protocol），它是Web通信时使用的协议，是Web的基础，也是互联网上应用最广泛的一种协议。

## 常见Web安全事件

1. “钓鱼”
2. “篡改”网页
	* 使用搜索引擎查找被黑的网站
		1. 使用关键词：Hacked by
		2. 使用搜索引擎语法：
			* Intitle:keyword 标题中含有关键词的网页
			* Intext:keyword 正文中含有关键词的网页
			* Site:domain 在某个域名和子域名下的网页
3. “暗链”：它是隐藏在网站中的链接，不能被用户点击，主要用来提高植入暗链在搜索引擎中的排名
4. Webshell：通过建立“传送门”的机制，上传一个后门，这个后门是可执行环境（该环境可以使用不同的语言，如asp、c++等），让我们可以随时随地回到被入侵的服务器上，进行恶意操作。

## 常见Web漏洞

### 1.XSS

全称为Cross Site Script（跨站脚本），属于客户端漏洞，常见的危害有盗取用户信息、钓鱼、制造蠕虫等等。它本质上是一种针对前端语言的注入。黑客通过“HTML注入”篡改网页，插入恶意脚本（XSS脚本），当用户在浏览网页时，实现控制用户浏览器行为的一种攻击方式。

|XSS类型|存储型|反射型|DOM型|
|:--|:--|:--|:--|
|触发过程|1.黑客制造XSS脚本<br>2.正常用户访问携带<br>XSS脚本的页面|正常用户访问携带XSS<br>脚本的页面|正常用户访问携带XSS<br>脚本的页面|
|数据存储|数据库|URL|URL|
|谁来输出|后端WEB应用程序|后端WEB应用程序|前端JavaScript|
|输出位置|HTTP响应中|HTTP响应中|动态构造的DOM节点中|

### 2.CSRF

全称为Cross-site request forgery（跨站请求伪造），属于客户端漏洞，常见的危害有执行恶意操作（“被转账”，“被发垃圾评论”等），制造蠕虫。黑客利用用户已登录的身份，在用户毫不知情的情况下，以用户的名义完成非法操作。

### 3.点击劫持

通常利用iframe或其他标签如flash等实现，隐蔽性较高，它也被称为“UI-覆盖攻击”，即在一个网站背后嵌套一个网站，以达到骗取用户操作的目的。

### 4.URL跳转

借助未验证的URL跳转，将应用程序引导到不安全的第三方区域，从而导致安全问题。可以通过Header头、JavaScript、META标签实现跳转。

### 5.SQL注入

由于数据与代码未分离，数据当做了代码来执行。它可以获取数据库信息（包括管理员后台用户名、密码以及数据库中的敏感信息等）、获取服务器权限、植入WebShell、读取服务器敏感数据。

### 6.命令注入

Windows命令行中的命令：

* & 为命令拼接符，将依次执行拼接的命令。
* | 为管道符，将前面命令的输出作为后面命令的注入。

要实现命令注入，需要满足三个条件：1.调用可执行系统命令的函数（如PHP中的system、exec、shell_exec、eval等）；2.函数或函数的参数可控；3.拼接注入命令。

### 7.文件操作漏洞

上传漏洞可以上传WebShell、木马；下载漏洞可以下载系统任意文件、程序代码。常见的文件操作漏洞包括文件上传漏洞、任意文件下载漏洞、文件包含漏洞。

文件上传漏洞需要满足以下条件：1.可以上传可执行脚本；2.脚本拥有执行权限。

任意文件下载漏洞需要满足以下条件：1.未验证下载文件格式；2.未限制请求路径。