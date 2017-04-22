---
title: 数据库学习笔记（三）- 事务
date: 2016-10-03 18:00
tags: [JDBC,JavaWeb]
---

事务（Transaction）是并发控制的基本单位，指作为单个逻辑工作单元执行的一系列操作，而这些逻辑单元需要满足ACID（原子性atomicity、一致性consistency、隔离性isolation、持久性durability）的特性。

JDBC的Connection提供三个方法实现事务

1. `.setAutoCommit()`设置为false，开启事务，默认为true。
2. `.commit()`提交事务
3. `.rollback()`回滚事务

<!-- more -->

``` java
public class TransactionTest {

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

	public void dbPoolTest(String buyer, String ProductName) {
		Connection conn = null;
		PreparedStatement ptmt1 = null;
		PreparedStatement ptmt2 = null;
		Savepoint sp = null;
		try {
			conn = ds.getConnection();
			conn.setAutoCommit(false);
			ptmt1 = conn.prepareStatement("UPDATE Inventory SET Inventory=Inventory-1 where ProductName = ?");
			ptmt1.setString(1, ProductName);
			ptmt1.execute();
			sp = conn.setSavepoint(); //保存检查点
			ptmt2 = conn.prepareStatement("INSERT INTO OrderList (buyer, ProductName) VALUES (?, ?)");
			ptmt2.setString(1, buyer);
			ptmt2.setString(2, ProductName);
			ptmt2.execute();
			conn.commit();
		} catch (SQLException e) {
			if (conn != null) {
				try {
					conn.rollback();
				} catch (SQLException e1) {
					e1.printStackTrace();
				}
			}

		} finally {
			try {
				if (conn != null) {
					conn.close();
				}
				if (ptmt1 != null) {
					ptmt1.close();
				}
				if (ptmt2 != null) {
					ptmt2.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args) {
		dbpoolInit();
		new TransactionTest().dbPoolTest("XiaoMing", "bag");
	}

}
```

脏读：一个事物读取了另一个事物未提交的更新。
不可重复读：同一事物中两次读取相同的记录，结果不一样。
幻读：两次读取的结果包含的行记录数不一样。

事务隔离级别：
1. 读未提交（read uncommitted）：允许出现脏读。
2. 读提交（read committed)：不允许出现脏读，可以有不可重复读。
3. 重复读（repeatable read）：不允许出现不可重复读，但可以有幻读。
4. 串行化（serializable）：不允许出现幻读。

MySQL的默认事物隔离级别为重复读。事务隔离级别越高，数据库性能越差。可以通过Connection对象的`.getTransactionIsolation`和`.setTransactionIsolation`设置隔离级别。

死锁是指两个或两个以上的事务在执行过程中，因争夺锁资源而造成的一种相互等待的现象。

死锁产生的必要条件
1. 互斥：并发执行事务为了进行必要的隔离保证执行正确，在事务结束前，需要对事务的数据库记录持锁，保证多个事务对相同数据库记录串行修改。对于大型并发系统**无法避免**。
2. 请求与保持：一个事务需要多个锁资源，已经持有一个资源锁，等待另外一个资源锁。死锁仅发生在请求两个或者两个以上的锁对象。由于应用需要，**难以消除**。
3. 不剥夺：已经获得锁资源的事务，在未执行前，不能被强制剥夺，只能使用完时，由事务自己释放。一般用于已经出现死锁时，通过破坏该条件达到解除死锁的目的。数据库系统通常通过一定的死锁检测机制发现死锁，强制回滚代价相对较小的事务，达到解除死锁的目的。
4. 环路等待：发生死锁时，必然存在一个事务——锁的环形链。按照同一顺序获取锁，可以破坏该条件。通过分析死锁事务之间的锁竞争关系，调整SQL的顺序，达到消除死锁的目的。

MySQL中主要由两种锁，排它锁（X)、共享锁（S)

|已有锁\预加锁|X|S|
|:--:|:--:|:--:|
|X|冲突|冲突|
|S|冲突|兼容|

加锁方式：
1. 外部加锁：由应用程序添加，锁依赖关系较容易分析，主要有两种形式
  * 共享锁（S）：`select * from table lock in share mode`
  * 排它锁（X）：`select * from table for update`
2. 内部加锁：为了实现ACID特性，由数据库系统内部自动添加。加锁规则繁琐，与SQL执行计划、事务隔离级别、表索引结构有关。

