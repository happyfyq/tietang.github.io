---
title: JDBC如何开启事务
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/P70430-092839.jpg'
date: 2017-04-10 18:22:47
categories:
	- JDBC
tags:
	- JDBC
	- 事务
keywords:
	- JDBC
	- 事务
description:
---


面试了很多人，每每问到“JDBC如何开启一个事务？”，大部分人的回答是：“通过openTransaction方法”，有的说是通过Connection，有的是说通过Statement，更有的说通过Connection拿到一个`Transaction`实例，再通过`openTransaction`方法来开启，那么同样关闭事务就有close方法“closeTransaction”。

想必这些人都是被“hibernate”害了还是“不注重java基础”，一上来就被框架误导呢？

先看看一个标准的JDBC例子伪代码：

```java
Connection conn = DriverManager.getConnection(...);
try{
  con.setAutoCommit(false);
  Statement stmt = con.createStatement();

   //1 or more queries or updates

   con.commit();
}catch(Exception e){
   con.rollback();
}finally{
   con.close();
}

```

所以，看到上面的例子，开启手动事务的关键是`con.setAutoCommit(false)`，JDBC事务默认是开启的，并且是自动提交：

1. 关闭自动提交：`java.sql.Connection.setAutoCommit(false)`
	- `setAutoCommit(true)`：每次操作都会被认为是一个事务并且自动提交
2. 手动提交事务：`con.commit()`;
3. 出现异常时回滚，不一定在catch语句中，只要在`con.commit()`前需要回滚时执行都可：`con.rollback()`;
4. 关闭连接：`con.close();`
5. 设置事务隔离级别: java.sql.Connection#setTransactionIsolation
	 


参考：https://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html