---
title: 数据库学习笔记（一）- JDBC
date: 2016-10-03 10:00
tags: [JDBC,JavaWeb]
---

## JDBC API

#### Driver & DriverManager

Driver是一个接口，定义了各个驱动程序都必须要实现的功能，是驱动程序的抽象。通过操作Driver接口，即可以实现对各个驱动程序的操作。DriverManager是Driver的管理类，用户通过`Class.forname(DriverName)`的方式，就可以向DriverManager注册一个驱动程序，然后通过DriverManager的`getConnection`方法就可以调用该驱动程序，建立到后端数据库的物理连接。

``` java
// 1. 加载数据库驱动
Class.forname(JDBC_DRIVER);
// 2. 获取数据库连接
conn = DriverManager.getConnection(DB_URL, USER, PASS);
```

<!-- more -->

`DB_URL`是后端数据库的唯一标识符，应用程序通过该标识符，即可唯一确定后端的某个数据库实例。它由三个部分组成
```
jdbc:mysql://10.164.172.20:3306/cloud_study
  |    |          |          |       |
  |    |         主机       端口     数据库     
 协议 子协议            子名称

jdbc:mysql://<ip>:<port>/database
jdbc:oracle:thin:@<ip>:<port>:database
jdbc:microsoft:sqlserver://<ip>:<port>;DatabaseName=database
```

#### Connection

Connection对象代表Java应用程序对后端数据库的一条物理连接，基于这条连接，可以执行一些sql语句。

常用方法：

``` java
Statement stmt = conn.createStatement();
```

#### Statement

是一个sql的容器，在这个sql容器中可以进行Select、Delete、Update操作，容器中可以承载我们放进去的sql语句，通过Statement对象的executeQuery方法，可以进行数据库查询，得到数据库查询结果的一个集合，这个集合以ResultSet对象表示。Statement对象也可以进行更新、删除语句，这时需要用execute和executeUpdate语句，它返回的是int值的对象，代表这条更新或者删除语句影响了多少条数据库记录。

常用方法：

``` java
ResultSet rs = stmt.executeQuery("select userName from user);
```

#### ResultSet

代表sql查询结果，是一个由行和列组成的二元表，ResultSet对象内部有一个指针，用于指向当前的行记录，默认指向第一行记录，有以下一些方法。

* .next()
* .previous()
* .absolute()
* .beforeFirst() 第一条记录的前面，需要通过.next()调到第一条记录
* .afterLast() 最后一行的下一条记录

获取列记录，可以通过列名（ColumnName）或者列号（从0开始排序）

* .getString(ColumnName/Index)
* .getInt(ColumnName/Index)
* .getObject(ColumnName/Index)

#### 构建步骤
1. 装载驱动程序
2. 建立数据库连接
3. 执行sql语句
4. 获取执行结果
5. 清理环境

``` java
static final String DRIVER_NAME = "com.mysql.jdbc.Driver";
// 设置UTF-8编码 新版JDBC中DE_URL需要设置useSSL参数，可以写上userSSL=false
static final String DB_URL = "jdbc:mysql://localhost:3306/jdbctest?characterEncoding=utf8&useSSL=false";
static final String DB_USER_NAME = "root";
static final String DB_PASSWORD = "123456";

public static void getData() throws ClassNotFoundException {
	Connection conn = null;
	Statement stmt = null;
	ResultSet rs = null;

	// 1. 获取驱动程序
	Class.forName(DRIVER_NAME);
	try {
		// 2.建立数据库连接
		conn = DriverManager.getConnection(DB_URL, DB_USER_NAME, DB_PASSWORD);
		// 3.执行SQL语句
		stmt = conn.createStatement();
		rs = stmt.executeQuery("select ProductName, Inventory from Product");
		// 4. 获取执行结果
		while (rs.next()) {
			System.out.println(rs.getString("ProductName") + ": " + rs.getString("Inventory"));
		}
	} catch (SQLException e) {
		// 异常处理
		e.printStackTrace();
	} finally {
		// 5. 清理环境
		try {
			if (conn != null) {
				conn.close();
			}
			if (stmt != null) {
				stmt.close();
			}
			if (rs != null) {
				rs.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}

```

## 游标

对于读取很多条记录内存放不下导致溢出，可以用游标的方式。游标提供一种客户端读取部分服务端结果集的机制，通过在DB_URL中写上`useCursorFetch=true`开启游标。

```
jdbc:mysql://<ip>:<port>/<database>?useCursorFetch=true
```

使用游标

``` java
PreparedStatement ptmt = null;
String sql = "select * from user where sex = ?";
ptmt = conn.prepareStatement(sql);
ptmt.setFetchSize(10);
ptmt.setString(1, "男");
rs = ptmt.executeQuery();
```

## 流方式

大对象读取，由于对象太大了，即使只读取一条记录也可能导致内存溢出。每次读取一个区间。

``` java
while (rs.next()) {
	// 读取流对象
	InputStream in = rs.getBinaryStream("blog");
	// 将对象流写入文件
	File f = new File(File_URL);
	OutputStream out = null;
	out = new FileOutputStream(f);
	int temp = 0;
	while ((temp = in.read()) != -1) { //边读边写
		out.write(temp);
	}
	in.close();
	out.close();
}
```

## 批处理

用于插入海量数据。涉及到Statement的`addBatch()`、`executeBatch()`、`clearBatch()`。

``` java
for (String user : users) {
	stmt.addBatch("insert into user(userName) values(" + user + ")");
}
stmt.executeBatch();
stmt.clearBatch();
```
