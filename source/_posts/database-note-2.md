---
title: 数据库学习笔记（二）- 数据库连接池
date: 2016-10-03 16:00
tags: [JDBC,JavaWeb]
---

JDBC客户端与服务器端的交互：

```
MySQL客户端          MySQL服务端
     |                   |
     |    请求建立连接   	 |
     |------------------>|
     |                   |
     |   发送随机密码种子	 |
     |<------------------|
     |                   |
     |    发送加密密码	 |
     |------------------>|
     |                   |
     |    连接建立成功  	 |
     |<------------------|
```

一个getConnection方法需要4次客户端与服务器端的网络传输，由于跨机器网络传输有较大网络开销，使用这种方式建立连接开销很大。

<!-- more -->

以连接池管理对数据库的连接，每个需要访问数据库的线程，每次从连接池中租借数据库连接，使用完毕后归还给连接池，这样就可以实现连接的重复使用，避免每次访问数据库都需要创建对数据库的连接。即将“创建”改为“租借”。

DBCP连接池包括3个jar包，分别是`commons-dbcp.jar`，`commons-pool.jar`，`commons-logging.jar`。DBCP底层是通过JDBC实现的，所以需要把一些必要的数据告诉DBCP。

``` java
public class productTest2 {

	public static BasicDataSource ds = null;

	static final String DRIVER_NAME = "com.mysql.jdbc.Driver";
	static final String DB_URL = "jdbc:mysql://localhost:3306/jdbctest?characterEncoding=utf8&useSSL=false";
	static final String DB_USER_NAME = "root";
	static final String DB_PASSWORD = "123456";

	public static void dbpoolInit() {
		ds = new BasicDataSource();
		ds.setUrl(DB_URL);
		ds.setDriverClassName(DRIVER_NAME);
		ds.setUsername(DB_USER_NAME);
		ds.setPassword(DB_PASSWORD);
	}

	public void dbPoolTest() {
		Connection conn = null;
		Statement stmt = null;
		ResultSet rs = null;
		try {
			conn = ds.getConnection();
			stmt = conn.createStatement();
			rs = stmt.executeQuery("select * from Product");
			while (rs.next()) {
				System.out.println(rs.getString("ProductName") + ": " + rs.getString("Inventory"));
			}
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				if (conn != null) {
					// 将数据库连接归还给连接池
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

	public static void main(String[] args) {
		dbpoolInit();
		new productTest2().dbPoolTest();
	}

}
```

BasicDataSource高级配置

* `.setInitialSize()`：通过InitialSize设置连接池创建时预建的连接数。一般设置为预期业务平均访问量比较合适。
* `.setMaxTotal()`：当连接池没有空闲连接，又有线程需要访问数据库时，会创建新的数据库连接，但如果已经达到MaxTotal设置的最大连接数时，连接池就不会新建一个数据库连接，而是强制让该线程进入等待队列等待，直到有其他线程归还连接时，再进行分配。它起到了限流保护数据库的作用。
* `.setMaxWaitMillis()`：设置最大等待时间，当超过等待时间时，客户端将得到SQLexception异常。
* `.setMaxIdle()`：当空闲连接数大于MaxIdle时，会销毁数据库连接。
* `.setMinIdle()`：当空闲连接数小于MinIdle时，连接池会创建数据库连接。为了避免频繁销毁创建，建议将MaxIdle和MinIdle设置为相同的值。

数据库服务器端为了释放空闲等待的资源，默认会关闭超过一定阈值的数据库连接，MySQL会默认关闭超过8小时的空闲连接。服务器端将连接关闭后，客户端的连接池可能不知道服务器端已经将连接关闭，当应用程序的线程向连接池租借连接时，连接池可能将失效的连接租借给应用程序，线程在使用该连接时，会抛出SQLexception异常。为了避免上述情况，尽量保证连接池中的连接都是有效的，可以定期对连接池中的连接的空闲时间检查，在服务器端关闭连接之前，保证将连接销毁，重新补充新的连接，保证应用程序从连接池中租借的连接都是有效的。`.setTestWhileIdle(True)`可以开启该功能。`.setMinEvictableIdleTimeMills()`表示销毁连接的最小空闲时间，只有当连接的空闲时间超过该值时，会被连接池自动销毁。`.setTimeBetweenEvictionRunsMillis()`表示检查运行时间的间隔，建议该数据设置小于服务器端自动关闭连接的时间（对于MySQL是8个小时），这样才能有效的检测空闲时间超过该值的空闲连接，主动关闭它，并补充新的连接。
