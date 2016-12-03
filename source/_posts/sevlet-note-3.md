---
title: Servlet学习笔记（三）
date: 2016-10-02 10:00
tags: [Servlet,JavaWeb]
---

## 请求转发

forward：将当前的request和response对象交给指定的web组件处理。一次请求，一次响应。浏览器的地址不会变。

RequestDispatcher

``` java
// 通过HttpServletRequest获取转发对象，可以使用绝对路径或者相对路径
RequestDispatcher rd = request.getRequestDispatcher("/forwardExample");

// 传入对应Servlet名称
rd = this.getServletContext().getNamedDispatcher("ServletName");

// 通过getServletContext获取转发对象，只能填绝对路径
rd = this.getServletContext().getRequestDispatcher("/forwardExample");

rd.forward(request, response);
```

<!-- more -->


## 请求重定向

sendRedirect：通过response对象发送给浏览器一个新url地址，让其重新请求。两次请求，两次响应。浏览器的地址会改变。

``` java
// 使用相对路径
response.sendRedirect("redirectExample");

// 使用相对路径
response.sendRedirect(request.getContextPath()+"/redirectExample");
```

## 过滤器

过滤请求与响应，自定义过滤规则，用于对用户请求进行预处理，和对请求响应进行后处理的web应用组件。

过滤器应用场景：用户认证、编解码处理、数据压缩处理。

过滤器的生命周期：init -> doFilter -> destory

``` java
public class TestFilter implements Filter {

	@Override
	public void destroy() {
		System.out.println("filter destory method");
	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
			throws IOException, ServletException {
		System.out.println("filter doFilter method");
		HttpServletRequest req = (HttpServletRequest) arg0;
		HttpSession session = req.getSession();
		if(session.getAttribute("userName")==null){
			HttpServletResponse res = (HttpServletResponse) arg1;
			res.sendRedirect("../index.html");
		} else{
			// 传递到下一个过滤器或者请求资源的Servlet
			arg2.doFilter(arg0, arg1);
		}
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		System.out.println("filter init method");
	}

}
```

``` xml
<filter>
	<filter-name>TestFilter</filter-name>
	<filter-class>com.netease.hello.TestFilter</filter-class>
	<init-param>
		<param-name>111</param-name>
		<param-value>222</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>TestFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

## 监听器

监听事件发生，在事件发生前后能够做出相应处理的web应用组件。

#### 监听器的分类

* 监听应用程序环境（ServletContext）
  * ServletContextListener
  * ServletContextAttributeListener
* 监听用户请求对象（ServletRequest）
  * ServletRequestListener
  * ServletRequestAttributeListener
* 监听用户会话对象（HTTPSession）
  * HTTPSessionListener
  * HTTPSessionAttributeListener
  * HTTPSessionActivationListener
  * HTTPSessionBindingListener

监听器的应用场景：应用统计、任务触发...

创建顺序：监听器 -> 过滤器 -> Servlet

``` xml
<listener>
	<listener-class>com.netease.hello.TestListener</listener-class>
</listener>
```

## 并发处理

* 变量线程安全
  * 参数变量本地化
  * 使用同步快synchronized
* 属性的线程安全
  * ServletContext线程不安全
  * HttpSession理论上线程安全
  * ServletRequest线程安全
* 避免在Servlet中创建线程
* 多个Servlet访问外部对象加锁 

