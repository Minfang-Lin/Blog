---
title: Servlet学习笔记（二）
date: 2016-09-27 16:00
tags: [Servlet,JavaWeb]
---

Cookie将会话数据保存在浏览器客户端。
Session将会话数据保存在服务器端。

Cookie工作流程：

1. 客户端（浏览器）向服务端发送请求；
2. 服务端根据业务需求生成对应的Cookie对象，把需要保存的业务数据保存在Cookie对象中；
3. 把Cookie对象放到HTTP响应头中，发还到客户端（浏览器）；
4. 浏览器接收到请求响应，将Cookie保存；
5. 当客户端（浏览器）再次访问该网站时，就会把Cookie放到请求中，一起发到服务端；
6. 服务端从请求中取出Cookie，获取到对应的数据，做出相应的响应。

<!-- more -->

Cookie默认会话结束后失效，可以通过setMaxAge设置cookie有效期

Cookie有大小和数量的限制，一般来说每个站点最多保存20个cookie，每个cookie的大小限制一般在4k以内。由于HTTP请求中的cookie是明文传递，存在数据安全性问题。

示例代码：
``` java
protected void doGet(HttpServletRequest request, HttpServletResponse response)
		throws ServletException, IOException {
	request.setCharacterEncoding("UTF-8");
	response.setContentType("text/html;charset=UTF-8");
	String name = request.getParameter("name1");
	String pwd = request.getParameter("pwd1");

	Cookie userNameCookie = new Cookie("userName", name);
	Cookie pwdCookie = new Cookie("pwd", pwd);
	response.addCookie(userNameCookie);
	response.addCookie(pwdCookie);

	Cookie[] cookies = request.getCookies();
	if (cookies != null) {
		for (Cookie cookie : cookies) {
			if (cookie.getName().equals("userNmae")) {
				name = cookie.getValue();
			}
			if (cookie.getName().equals("pwd")) {
				pwd = cookie.getValue();
			}
		}
	}
}
```

Session工作流程：

1. 浏览器发出请求到服务器；
2. 服务器会根据需求生成Session对象，并且给这个Session对象一个唯一的ID，一个ID对应一个Session对象；
3. 服务器把需要记录的数据封装到这个Session对象里，然后把这个Session对象保存下来；
4. 服务器把这个Session对象的编号放到一个Cookie里，随着响应发送给浏览器；
5. 浏览器接收到这个Cookie就会保存下来；
6. 当下一次浏览器再次请求该服务器服务，就会发送该Cookie；
7. 服务器得到这个Cookie，取出它的内容，它的内容就是一个Session的ID；
8. 凭借这个Session编号找到对应的Session对象，然后利用该Session对象把保存的数据取出。

Session默认有效期为30分钟，可以通过`setMaxInactiveInterval`设置有效期（优先级高于通过部署描述符设置），也可以通过部署描述符配置有效期。通过`invalidate`使Session失效。

``` java
String pwd = request.getParameter("pwd1");

HttpSession session = request.getSession();
String username = (String) session.getAttribute("userName");
if (username != null) {
	System.out.println("second login");
}
session.setAttribute("userName", name);
session.setAttribute("pwd", pwd);
```

通过`setMaxInactiveInterval`设置有效期：

``` java
// 单位为秒
session.setMaxInactiveInterval(2*60);
```

通过部署描述符配置有效期：

``` xml
<!-- 单位为分钟 -->
<session-config>
	<session-timeout>2</session-timeout>
</session-config>
```

Cookie与Session的比较

|.|Cookie|Session|
|:--:|:--:|:--:|
|数据储存|客户端|服务器端|
|安全性|以明文方式存放在客户端，相对安全性较弱，但可以通过加密方式存放|存放在服务器端内存中，相对安全性较强|
|生命周期|累计时间，到点失效|间隔时间，从最后一次访问开始计时，与cookie不同的是可以直接调用API使得session失效|
|使用原则|有限制，每个站点最多20个cookie，每个最大4k|存放在服务器端，会占用服务器内存，不建议存放过多过大的对象|




