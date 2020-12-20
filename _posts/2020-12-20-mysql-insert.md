---
layout:       post
title:        "mysql insert 语句的用法"
subtitle:     "插入数据真的有那么简单吗？"
date:         2020-12-20 16:00:00
author:       "Alvin"
header-img:   "img/yinhebuxiban.webp"
header-mask:  0.3
catalog:      true
tags:
	- 技术
    - mysql
    - insert
	- insert on duplicate key update
	- insert delayed statement





---

## 前言

> 工作中遇到一个很简单又很隐蔽的bug，这个时候才发现自己对数据库的增删改查仅仅停留在会有的层面，对原理一无所知。bug的场景是，在执行alert语句时，如果有dml（insert on duplicate key update ）执行会报唯一键出现重复值的异常。



## mysql[官网问题描述](https://bugs.mysql.com/bug.php?id=76895)

> ```mysql
> When running an online ALTER TABLE operation, the thread that runs the ALTER TABLE operation will apply an “online log” of DML operations that were run concurrently on the same table from other connection threads. When the DML operations are applied, it is possible to encounter a duplicate key entry error (ERROR 1062 (23000): Duplicate entry), even if the duplicate entry is only temporary and would be reverted by a later entry in the “online log”. This is similar to the idea of a foreign key constraint check in InnoDB in which constraints must hold during a transaction.
> ```

**原因不详-_-,如果有哪位大佬知道也请教教我**（912806786@qq.com)

## INSERT 用法

### INSERT...SELECT

> When selecting from and inserting into the same table, MySQL creates an internal temporary table to hold the rows from the [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html) and then inserts those rows into the target table. However, you cannot use `INSERT INTO t ... SELECT ... FROM t` when `t` is a `TEMPORARY` table, because `TEMPORARY` tables cannot be referred to twice in the same statement. For the same reason, you cannot use `INSERT INTO t ... TABLE t` when `t` is a temporary table.

当查询的表和插入的表是同一个时表名时，mysql会创建一个临时表来存储查询的数据，然后将临时表的数据插入到目标表。

如果查询的表时临时表，不能执行INSERT...SELECT语句，因为临时表不能被引用两次

### LOW_PRIORITY

> If you use the `LOW_PRIORITY` modifier, execution of the [`INSERT`](https://dev.mysql.com/doc/refman/5.6/en/insert.html) is delayed until no other clients are reading from the table. This includes other clients that began reading while existing clients are reading, and while the `INSERT LOW_PRIORITY` statement is waiting. It is possible, therefore, for a client that issues an `INSERT LOW_PRIORITY` statement to wait for a very long time (or even forever) in a read-heavy environment. (This is in contrast to [`INSERT DELAYED`](https://dev.mysql.com/doc/refman/5.6/en/insert-delayed.html), which lets the client continue at once.)

当有low_priority修饰符时，数据需要等待该表没有任何操作（增删改查）时进行插入数据

**`LOW_PRIORITY`通常不应与`MyISAM`表一起使用，因为这样做会禁用并发插入**

### DELAYED

> If you use the `DELAYED` modifier, the server puts the row or rows to be inserted into a buffer, and the client issuing the [`INSERT DELAYED`](https://dev.mysql.com/doc/refman/5.6/en/insert-delayed.html) statement can then continue immediately. If the table is in use, the server holds the rows. When the table is free, the server begins inserting rows, checking periodically to see whether there are any new read requests for the table. If there are, the delayed row queue is suspended until the table becomes free again.

INSERT 语句有该修饰符时，mysql会将数据先插入缓冲区

**从MySQL 5.6.6开始，[`INSERT DELAYED`](https://dev.mysql.com/doc/refman/5.6/en/insert-delayed.html)不推荐使用；希望在将来的版本中将其删除。使用[`INSERT`](https://dev.mysql.com/doc/refman/5.6/en/insert.html) （不带`DELAYED`）代替。**

### HIGH_PRIORITY

> `HIGH_PRIORITY` affects only storage engines that use only table-level locking (such as `MyISAM`, `MEMORY`, and `MERGE`).

### IGNORE

> If you use the `IGNORE` modifier, errors that occur while executing the [`INSERT`](https://dev.mysql.com/doc/refman/5.6/en/insert.html) statement are ignored

批量数据插入时，如果其中一条数据失败，不影响其他的数据，可以使用该语句

### ON DUPLICATE KEY UPDATE

> If you specify an `ON DUPLICATE KEY UPDATE` clause and a row to be inserted would cause a duplicate value in a `UNIQUE` index or `PRIMARY KEY`, an [`UPDATE`](https://dev.mysql.com/doc/refman/5.6/en/update.html) of the old row occurs.

存在即更新，不存在即插入也可以进行批量操作

> With `ON DUPLICATE KEY UPDATE`, the affected-rows value per row is 1 if the row is inserted as a new row, 2 if an existing row is updated, and 0 if an existing row is set to its current values. If you specify the `CLIENT_FOUND_ROWS` flag to the [`mysql_real_connect()`](https://dev.mysql.com/doc/c-api/5.6/en/mysql-real-connect.html) C API function when connecting to [**mysqld**](https://dev.mysql.com/doc/refman/5.6/en/mysqld.html), the affected-rows value is 1 (not 0) if an existing row is set to its current values.

在上述mysql官方文档中说，如果不存在影响条数为1，**如果存在，数据有变化时，影响条数为2**，如果存在数据无变化时，影响条数为0。其中最奇怪的就是数据有变化时，为什么影响条数为2？？？？原理等我搞清楚了再补充

#### 底层实现逻辑（待补充）

#### 注意

- 如果表中存在多个唯一索引，并且插入的数据中包含多个唯一索引时，该语句只能更新第一条，所以用的时候要弄清楚表的唯一索引。



