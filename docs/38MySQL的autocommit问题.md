# MySQL的 autocommit 问题



## 使用命令行登陆 mysql

```shell
# linux可以直接使用 mysql 登陆
mysql -u root -p

# windows需要到MySQL安装目录的bin目录，再使用 mysql 登陆
cd C:\Program Files\MySQL\MySQL Server\bin
mysql -u root -p
```



## mysql 重启

```shell
# linux
mysqld start
mysqld stop
mysqld restart

# windows(假设已安装mysql服务)
net stop mysql
net start mysql
# 或者在mysql的bin目
mysqladmin shutdown -u root -p

# 或者 控制面板->管理工具->服务 进行关闭和启动
```



## 打开 windows的 DOS

1. win + r 输入 cmd 打开
2. 创建 bat文件，输入 `start` ，运行 bat文件打开



## autocommit 模式

默认情况下，MySQL 启用自动提交模式（autocommit=ON）。

只要执行 DML 操作（INSERT, UPDATE, DELETE）、DQL/DCL/DDL ？，MySQL 会立即隐式提交事务（Implicit Commit）。

参考文章：https://www.cnblogs.com/kerrycode/p/8649101.html?page=32



## 查看mysql的autocommit模式

`autocommit` 分**会话系统变量**与**全局系统变量** 

```mysql
mysql> show session variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.00 sec)

mysql> show global variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```



## 修改autocommit模式

```mysql
mysql> set session autocommit=0;

mysql> show session variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> show global variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.00 sec)

mysql> set global autocommit=0;
 
mysql> show global variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.01 sec)
```



注意：上述通过 SQL 修改会话系统变量和全局系统变量，只对当前实例有效。如果 MySQL 服务重启，这些设置会丢失。如果要永久生效，必须在配置文件中修改系统变量。

```ini
[mysqld]
autocommit=0
```



## autocommit 与显性事务的关系

对于显性事务 `start transaction` 或 `begin` ，在自动提交模式关闭的情况下，开启一个事务上下文。

首先数据库会隐式提交之前的**还未被提交的操作**，同时开启一个新事务。

例子如下：

```mysql
# 会话1
set autocommit=0;
delete from MyDB.test where name='kerry';
```



在会话2中查看，此时可以查询到会话ID为1的事务信息， 如下所示

```mysql
# 会话2
mysql> SELECT a.trx_state, 
    ->        b.event_name, 
    ->        a.trx_started, 
    ->        b.timer_wait / 1000000000000 timer_wait, 
    ->        a.trx_mysql_thread_id        blocking_trx_id, 
    ->        b.sql_text 
    -> FROM   information_schema.innodb_trx a, 
    ->        performance_schema.events_statements_current b, 
    ->        performance_schema.threads c 
    -> WHERE  a.trx_mysql_thread_id = c.processlist_id 
    ->        AND b.thread_id = c.thread_id; 
+-----------+-----------+------------+------------+-----------------+-----------+
| trx_state | event_name| trx_started| timer_wait | blocking_trx_id | sql_text  |
+-------+--------------------+-------------------+------+-+----------------------------------------+
|RUNNING|statement/sql/delete|2018-03-23 14:55:00|0.0010|1|delete from MyDB.test where name='kerry'|
+-------+--------------------+-------------------+------+-+----------------------------------------+
1 row in set (0.00 sec)
```



如果在会话1当中开启显性事务，那么之前挂起的事务会自动提交，然后，你再去会话2当中查询，就发现之前的DELETE操作已经提交。

```mysql
# 会话1
start transaction;
```



```mysql
# 会话2
mysql> SELECT a.trx_state, 
    ->        b.event_name, 
    ->        a.trx_started, 
    ->        b.timer_wait / 1000000000000 timer_wait, 
    ->        a.trx_mysql_thread_id        blocking_trx_id, 
    ->        b.sql_text 
    -> FROM   information_schema.innodb_trx a, 
    ->        performance_schema.events_statements_current b, 
    ->        performance_schema.threads c 
    -> WHERE  a.trx_mysql_thread_id = c.processlist_id 
    ->        AND b.thread_id = c.thread_id; 
Empty set (0.00 sec)
```

使用 `start transaction` ，自动提交将保持禁用状态，直到你使用 `COMMIT` 或 `ROLLBACK` 结束事务，自动提交将恢复到之前的状态。（`start transaction` 前 `autocommit=1` ，结束事务后 `autocommit=1` 。）



结合 `autocommit` 的作用：

1. 关闭自动提交（ `autocommit=0` ），事务在用户对数据进行操作时自动开启，或用 `start transaction` 开启。开启后，到执行 `commit` 提交事务为一个完整的事务周期。若不执行 `commit` 则默认事务回滚。

2. 开启自动提交（ `autocommit=1` ）

   ①. 执行 `start transaction` 开启事务（隐式提交上一个已开启的事务），执行 `commit` 提交事务。若不执

   ​      行 `commit` 则默认事务回滚。

   ②. 未执行 `start transaction` 对数据库进行操作，系统默认对数据库的每一个操作作为一个单独的事务。

   ​      也就是每进行一次操作都会即时提交或回滚，每个操作都是一个完整的事务周期。



参考文章：https://www.cnblogs.com/Renyi-Fan/p/8547306.html























