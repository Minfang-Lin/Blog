---
title: Spring学习笔记（三）
date: 2016-10-22 20:00
tags: [Spring,JavaWeb]
---

## 实现Controller

``` java
@Controller
@RequestMapping(vaule = "/hello")
public class HelloController {
	@RequestMapping(vaule = "/spring")
	public void spring(HttpServletResponse response) throw IOException {
		response.getWriter().write("Hello, Spring Web!");
	}
}
```

## 定义Controller

``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
 	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
	http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="com.netease.course" />

	<mvc:annotation-driven />

</bean>
```

<!-- more -->

## @RequestMapping

* name: 名称
* value & path： 路径，如"`/hello`"
* method: 请求方法，如"GET"
* params: 请求参数
* headers： 请求头，如Accept头
* consumes： 请求的媒体类型，"Content-Type"
* product： 响应的媒体类型，"Accept"

RestAPI

路径参数匹配：
``` java
@RequestMapping(path = "/users/{userId}")
public String webMethod(@PathVariable String userId) {
	// ...
}
```

``` java
@RequestMapping(path = "/users/{userId:[a-z]+}")
public String webMethod(@PathVariable String userId) {
	// ...
}
```

## Controller函数参数

* HttpServletRequest/HttpServletResponse, HttpSession(Servlet API)
* Reader/Writer
* @PathVariable
* @RequestParam
* @RequestHeader
* HttpEntity
* @RequestBody
* Map/Model/ModelMap

## 函数返回值

代表返回给view的名称

* void: 请求处理函数不需要返回view
* String： view名称，可以用@ResponseBody说明返回的不是view名称，而是响应内容
* HttpEntity
* View
* Map/Model/ModelAndView

## 函数实现

``` java
@RequestMapping(value = "/spring/{user}")
public void spring(
	@PathVariable("user") String user,
	@RequestParam("msg") String msg,
	@RequestHeader("host") String host,
	HttpServletRequest request,
	Writer writer) throws IOException {	
	writer.write("URI:" + request.getRequestURI());
	writer.write("Hello, " + user + ": " + msg + ", host=" + host);
}
```

#### 表单

``` java
@RequestMapping("/spring/login")
public void login(
	@RequestParam("name") String name,
	@RequestParam("password") String password
	Writer writer)
		throws IOException {
	// ...
}
```

``` java
@RequestMapping("/spring/login")
public void login(
	@ModelAttribute User user,
	Writer writer)
		throws IOException {
	// ...
}
```

#### 上传文件

先定义一个文件上传相关的bean，并添加`commons-fileupload`和`commons-io`依赖。
``` xml
<bean id="multipartResover"
	class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="maxUploadSize" value="100000"/>
</bean>
```

``` java
@RequestMapping(path = "/form", method = RequestMethod.POST)
public String handleFormUpload(@RequestParam("file") MultipartFile file) {
	// ...
}
```

#### HttpEntity

``` java
@RequestMapping("/something")
public ResponseEntity<String> handler(HttpEntity<byte[]> requestEntity) {
	String requestHeader = requestEntity.getHeaders().getFirst("MyRequestHeader");
	byte[] requestBody = requestEntity.getBody();
	
	// ...
	
	HttpHeaders responseHeaders = new HttpHeaders();
	responseHeaders.set("MyResponseHeader", "MyValue");
	return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
}
```

#### @RequestBody & ResponseBody

``` java
@RequestMapping(/spring)
@ReponseBody
public String spring(@RequestBody String body) throws IOException {
	return "Hello " + body;
}
```

## View解析

#### InternalResourceViewResolver

``` xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/jsp/"/>
	<property name="suffix" value=".jsp"/>
</bean>
```

对于resultView，路径为/WEB-INF/jsp/resultView.jsp。

#### FreeMarkerViewResolver

``` xml
<bean id="freemarkerConfig"
	class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
	<property name="templateLoaderPath" value="/WEB-INF/freemarker" />
</bean>

<bean id="viewResolver"
	class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
	<property name="cache" value="true" />
	<property name="prefix" value="" />
	<property name="suffix" value=".ftl" />
</bean>
```

对于resultView，路径为/WEB-INF/freemarker/resultView.ftl。

#### ContentNegotiatingViewResolver

ViewResolver的组合，通过扩展名user.json、user.xml、user.pdf或者Accept头(media types) application/json、application/xml获取返回类型。

``` xml
<bean
	class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
	<property name="viewResolvers">
		<list>
			<bean id="viewResolver"
				class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
				<property name="cache" value="true" />
				<property name="prefix" value="" />
				<property name="suffix" value=".ftl" />
				<property name="contentType" value="text/html; charset=utf-8" />
			</bean>
		</list>
	</property>
	<property name="defaultViews">
		<list>
			<bean
				class="org.springframework.web.servlet.view.json.MappingJackson2JsonView" />
		</list>
	</property>
</bean>
```