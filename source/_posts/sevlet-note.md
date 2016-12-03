---
title: Servlet学习笔记（一）
date: 2016-09-27 10:00
tags: [Servlet,JavaWeb]
---

## 概述

Servlet == Server + Applet

运行在服务端的Java程序，Servlet不同于普通的Java类，它没有main方法，它不能独立运行在JVM上，它需要在特殊的容器中装载运行，由容器管理其生成和销毁。Servlet几乎能处理所有的Http请求，并能够给客户端提供相应的Http响应。故可以将Servlet的功能总结如下：

> 一个Servlet就是一个Java类，并提供基于请求-响应模式的Web服务。

## Servlet与Servlet容器

Servlet容器装载和管理Servlet（包括Servlet创建、执行、销毁等等），它是一个服务端程序，这个程序负责把请求转发给Servlet，交由Servlet处理，Servlet处理完毕后，将结果返回给客户端。

<!--more-->

## Servlet处理流程

在浏览器中输入一个URL，Servlet容器根据URL地址，通过配置文件（web.xml）找到对应的Servlet，同时将请求转发给Servlet的service方法。如果通过Get方法发送请求，service方法就将请求转发给doGet方法。

## Servlet生命周期

1. 初始化：Servlet初始化对应init方法，默认的情况下，只有当web客户端第一次请求某Servlet的时候，Servlet容器才会创建这个Servlet对象的实例，这时Servlet容器会回调Servlet的init方法。如果在Servlet配置文件中配置了load-on-startup元素，那Servlet也可能是在容器启动时被相应加载。
2. 请求处理：对应service方法，Servlet会根据请求类型将不同的请求交由不同的Servlet处理。最常用的是doGet和doPost方法。
3. 销毁阶段：对应destroy方法，是Servlet在销毁前由Servlet容器回调。会做一些资源的回收和清理工作。

测试代码：
``` java
//HelloWorld.java
package com.netease.hello;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloWorld extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest requst, HttpServletResponse response) 
			throws ServletException, IOException {
		System.out.println("doGet method");
		PrintWriter pw = response.getWriter();
		pw.print("hello world");
		pw.close();
	}

	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) 
			throws ServletException, IOException {
		System.out.println("service method");
		super.service(request, response);
	}

	@Override
	public void destroy() {
		System.out.println("destory method");
	}

	@Override
	public void init() throws ServletException {
		System.out.println("init method");	
	}
	
}

```

``` xml
<!-- web.xml -->
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
	<display-name>Archetype Created Web Application</display-name>
	<servlet>
		<servlet-name>HelloWorld</servlet-name>
		<servlet-class>com.netease.hello.HelloWorld</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>HelloWorld</servlet-name>
		<url-pattern>/hello</url-pattern>
	</servlet-mapping>
</web-app>
```

## Get与Post

|.|Get|Post|
|:--:|:--:|:--:|
|传输方式|HTTP header，URL可见|HTTP body，URL不可见|
|传输长度|受限于URL长度，一般在2k~8k|一般没有限制|
|设计目的|获取数据|发送数据|
|安全性|低|高|

测试代码：
``` java
//GetPostTest.java
package com.netease.hello;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class GetPostTest extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse response) 
			throws ServletException, IOException {
		request.setCharacterEncoding("UTF-8");
		response.setContentType("text/html;charset=UTF-8");
		PrintWriter out = response.getWriter();
		String name1 = request.getParameter("name1");
		String pwd1 = request.getParameter("pwd1");
		out.println("调用doGet方法");
		out.println("用户名: " + name1);
		out.print("密码： " + pwd1);
	}

	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse response) 
			throws ServletException, IOException {
		request.setCharacterEncoding("UTF-8");
		response.setContentType("text/html;charset=UTF-8");
		PrintWriter out = response.getWriter();
		String name2 = request.getParameter("name2");
		String pwd2 = request.getParameter("pwd2");
		out.println("调用doPost方法");
		out.println("用户名: " + name2);
		out.print("密码： " + pwd2);		
	}

}
```

``` html
<!-- index.jsp -->
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<html>
<head>
	<meta charset="UTF-8">
	<title>Get/Post方法提交</title>
</head>
<body>
	<form action="./GetPostServlet" method="get">
		用户名：<input type="text" name="name1" value=""><br>
		密码: <input type="password" name="pwd1" value=""><br>
		<input type="submit" value="使用Get提交">
	</form>
	<form action="./GetPostServlet" method="post">
		用户名：<input type="text" name="name2" value=""><br>
		密码: <input type="password" name="pwd2" value=""><br>
		<input type="submit" value="使用Post提交">
	</form>
</body>
</html>
```

``` xml
<!-- web.xml中配置部分 -->
	<servlet>
		<servlet-name>GetPostTest</servlet-name>
		<servlet-class>com.netease.hello.GetPostTest</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>GetPostTest</servlet-name>
		<url-pattern>/GetPostServlet</url-pattern>
	</servlet-mapping>
```


#### 解决doGet方法中文乱码的问题：

将tomcat安装目录下的conf/server.xml文件中
``` xml
<Connector connectionTimeout="20000" port="8080" 
  protocol="HTTP/1.1" redirectPort="8443"/>
```
改成
``` xml
<Connector connectionTimeout="20000" port="8080" 
  protocol="HTTP/1.1" redirectPort="8443" URIEncoding="utf-8"/>
```

## Servlet配置参数

#### ServletConfig对象

测试代码：
``` xml
	<!-- ServletConfig Test -->
	<servlet>
		<servlet-name>ServletConfigTest</servlet-name>
		<servlet-class>com.netease.hello.ServletConfigTest</servlet-class>
		<init-param>
			<param-name>data1</param-name>
			<param-value>value1</param-value>
		</init-param>
		<init-param>
			<param-name>data2</param-name>
			<param-value>value2</param-value>
		</init-param>
	</servlet>
	<servlet-mapping>
		<servlet-name>ServletConfigTest</servlet-name>
		<url-pattern>/ServletConfigTest</url-pattern>
	</servlet-mapping>
```

``` java
// ServletConfigTest.java
package com.netease.hello;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ServletConfigTest extends HttpServlet {

	@Override
	public void init() throws ServletException {
		ServletConfig config = this.getServletConfig();
		String v1 = config.getInitParameter("data1");
		System.out.println("v1: " + v1);
		String v2 = config.getInitParameter("data2");
		System.out.println("v2: " + v2);

	}

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
			throws ServletException, IOException {
		PrintWriter pw = resp.getWriter();
		pw.print("ServletConfig test");
		pw.close();
	}

}
```

Servlet初始化过程中，`<init-param>`参数将被封装到ServletConfig对象中，每个Servlet支持一个或者多个`<init-param>`，这个配置是以Servlet为单位，不是全局共享的。

#### ServletContext对象

Servlet容器在启动时会为每一个web应用创建一个ServletContext对象，这个ServletContext对象代表了当前的web应用。ServletContext对象在web应用中是全局唯一的，所以在Servlet容器中的任何Servlet访问的是同一个ServletContext对象。

``` java
ServletContext ctx = this.getServletContext();
String gv1 = ctx.getInitParameter("globalData1");
System.out.println("gv1: " + gv1);
```

``` xml
<!-- ServletContext Test -->
<context-param>
	<param-name>globalData1</param-name>
	<param-value>globalValue1</param-value>
</context-param>
<context-param>
	<param-name>globalData2</param-name>
	<param-value>globalValue2</param-value>
</context-param>
```

如果不同Servlet之间的共享信息不是事先知道的，通过setAttribute、getAttribute、removeAttribute进行配置信息的增删改查。每一对属性是key-value的键值对。
``` java
ServletContext ctx = this.getServletContext();
ctx.setAttribute("attrbute1", "111");
ctx.getAttribute("attribute1");
ctx.removeAttribute("attribute1");
```

## 读取外部资源配置信息

#### getResource
``` java
ServletContext ctx = this.getServletContext();	
try {
	// 此处路径是相对于web应用的
	URL url = ctx.getResource("/WEB-INF/classes/log4j.properties");
	InputStream in = url.openStream();
	String propertyValue = GeneralUtil.getPropery("log4j.rootLogger",
			in);
	System.out.println("property value: " + propertyValue);
} catch (MalformedURLException e) {
	e.printStackTrace();
} catch (IOException e) {
	e.printStackTrace();
}
```

``` java
public class GeneralUtil {
	public static String generateId() {
		return UUID.randomUUID().toString().replaceAll("-", "");
	}

	public static String getPropery(String key, InputStream in) {
		Properties props = new Properties();
		try {
			props.load(in);
		} catch (IOException e) {
			e.printStackTrace();
		}
		String value = props.getProperty(key);
		return value;
	}
}
```

#### getResourceAsStream

``` java
InputStream in2 = ctx
		.getResourceAsStream("/WEB-INF/classes/log4j.properties");
String p2 = GeneralUtil.getPropery("log4j.rootLogger", in2);
System.out.println("p2: " + p2);
```

#### getRealPath

``` java
String path = ctx.getRealPath("/WEB-INF/classes/log4j.properties");
System.out.println("real path: " + path);
File f = new File(path);
try {
	InputStream in3 = new FileInputStream(f);
	String p3 = GeneralUtil.getPropery("log4j.rootLogger", in3);
	System.out.println("p3: " + p3);
} catch (FileNotFoundException e) {
	e.printStackTrace();
}
```

## Web应用程序结构
``` bash
webapp
   |--css
   |--js
   |--images
   |--html
   |--META-INF
   |--WEB-INF
        |--classes
        |--lib
        web.xml
```

## 部署描述符

web.xml是用于配置部署描述符的文件，用来设置web应用程序的组件部署信息。Servlet容器需要支持部署描述符中的所有元素。

#### Servlet URL匹配

Servlet支持多个url-pattern对应同一个Servlet，url-pattern支持模糊匹配。

ServletMapping匹配原则

1. 精确路径匹配，完全匹配
2. 最长路径匹配，最长前缀匹配
3. 扩展名匹配
4. default servlet或者放弃

#### load-on-startup

用于改变Servlet默认初始化时间，当大于或等于0，表示Servlet启动时会加载这个Servlet。如果不设置或设置负数的时候，只有当第一次请求这个Servlet时，才会被加载。正数值越小，越优先启动。

#### 配置自定义错误页面

``` xml
<error-page>
	<error-code>404</error-code>
	<location>/404.html</location>
</error-page>
```

更高级的做法还可以添加excepti-type元素捕获一个Java异常类型。

#### 欢迎页面

用户在浏览器中输入的url并不包括某个servlet的映射路径，或者某个具体的静态页面的时候，会根据`<welcome-file-list>`元素显示默认指定的文件。

``` xml
<welcome-file-list>
	<welcome-file>index.html</welcome-file>
	<welcome-file>index.htm</welcome-file>
	<welcome-file>index.jsp</welcome-file>
	<welcome-file>default.html</welcome-file>
	<welcome-file>default.htm</welcome-file>
	<welcome-file>default.jsp</welcome-file>
</welcome-file-list>
```

按列表顺序查找。

#### MIME类型映射
MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。
`<mime-mapping>`定义扩展文件名映射类型。包括两个元素`<extension>`和`<mime-type>`。