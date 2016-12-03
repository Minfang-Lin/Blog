---
title: 数据库学习笔记（四）- MyBatis
date: 2016-10-03 20:00
tags: [JDBC,JavaWeb]
---

面向对象的世界中数据是对象，关系型数据库中数据是以行、列、二元表的形式存储的。
ORM（Object/Relation Mapping）框架帮助我们持久化类与数据库表之间的映射关系，对持久化对象的操作自动转换成对数据库表的操作。关系型数据库的每一行映射为每一个对象，每一列映射为对象的每个属性。

MyBatis的工作流机制：

首先需要在应用程序启动时，加载一个xml文件，这个文件里定义了后端数据库的地址，同时也定义了Sql和Java方法之间的映射关系。应用程序调用MyBatis的API接口传入参数与SQL ID，MyBatis会自动匹配对应的Sql语句，生成完整的Sql语句，访问后端的数据库，转换执行的结果为Java对象，然后将执行结果返回给应用程序。

<!-- more -->

MyBatis环境搭建

1. jar包：`mybatis-3.2.3.jar`、`mysql-connector-java-5.1.12.jar`
2. sqlSessionFectory：包含MyBatis系統的核心配置，包括后端数据库连接实例的数据源，决定事务范围和控制方式的事务管理器。