---
title: Spring学习笔记（二）
date: 2016-10-16 10:00
tags: [Spring,JavaWeb]
---

DAO（Data Access Object）代表数据访问对象，它一般会包含一些数据访问的接口，针对接口做出不同实现。

ORM（Object Relation Mapping）代表对象关系映射，将数据库种的一行记录转换为一个Java对象。

## DataSource配置

``` xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="db.properties"/>
```

``` xml
<bean id="headerProperties" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	<property name="location" value="classpath:db.properties" />
</bean>
```

<!-- more -->

## JdbcTemplate

``` sql
// 查
int rowCount = this.jdbcTemplate.queryForObject(
		"select count(*) from user", Integer.class);

int countOfNameJoe = this.jdbcTemplate.queryForObject(
		"select count(*) from user where first_name = ?", 
		Integer.class, "Joe"_);

String lastName = this.jdbcTemplate.queryForObject(
		"select last_name from user where id = ?",
		new Object[]{1212L}, String.class);

// 增
this.jdbcTemplate.update(
	"insert into user (first_name, last_name) values (?, ?)",
	"Meimei", "Han");

// 改
this.jdbcTemplate.update(
	"update user set last_name = ? where id = ?", "Li", 5276L);

// 删
this.jdbcTemplate.update(
	"delete from user where id = ?", Long.valueOf(userId));

// 创建表
this.jdbcTemplate.execute("create table user (id integer,
	first_name varchar(100), last_name varchar(100))");
```

