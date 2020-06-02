# MySQL重置密码和定时备份

##### 创建特殊字符名称的文件和文件夹

创建文件夹 `0<folder<12` ，用双引号 `""` 可以创建名字有特殊字符的文件和文件夹

```shell
mkdir "0<folder<12"
mv "24>folder>12" "12<folder<24"
```

删除、移动这类名字有特殊字符的，也是使用双引号 `""`



## Crontab 安装使用

### 安装 cron

```shell
yum install vixie-cron
yum install crontabs
```

`vixie-cron` 软件包是 `cron` 的主程序。

`crontabs` 软件包是用来安装、卸装、或列举用来驱动 `cron` 守护进程的表格的程序

遇到 No package *** available 问题，解决办法见下章



### 使用 cron

```shell
service crond start #启动服务 

service crond stop #关闭服务 

service crond restart #重启服务 

service crond reload #重新载入配置

service crond status #查看状态
```



#### 设置 `crond` 为开机自启

```shell
chkconfig crond on
systemctl enable crond

# 查看 crond 开机启动状态的两种方法
chkconfig --list |grep crond
systemctl list-unit-files |grep crond #centos7自启项管理不用chkconfig, 改为systemctl
```



#### 定时任务

查看 `crond` 的任务

```shell
crontab -l
```



添加定时任务

```shell
crontab -e
# vi /etc/crontab

tail -f /var/log/cron #查看日志
```



定时任务例子：

基本格式:  

`* * * * *   command` 

`分 时 日 月 周 命令`

```shell
30 21 * * * /usr/local/etc/rc.d/lighttpd restart #每晚的21:30重启apache

45 4 1,10,22 * * /usr/local/etc/rc.d/lighttpd restart #每月1、10、22日的4 : 45重启apache

10 1 * * 6,0 /usr/local/etc/rc.d/lighttpd restart #每周六、周日的1 : 10重启apache

0,30 18-23 * * * /usr/local/etc/rc.d/lighttpd restart #每天18 : 00至23 : 00之间每隔30分钟重启apache

0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart #每星期六的11 : 00 pm重启apache。

* */1 * * * /usr/local/etc/rc.d/lighttpd restart #每一小时重启apache

* 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart #晚上11点到早上7点之间，每隔一小时重启apache

0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart #每月的4号与每周一到周三的11点重启apache
0 4 1 jan * /usr/local/etc/rc.d/lighttpd restart #每年一月一号的4点重启apache
```



参考文章：

Crontab安装以及以些常见问题：https://cloud.tencent.com/developer/article/1121284

LINUX-Crontab使用总结：https://www.jianshu.com/p/908b805e098e



## MySQL备份

##### 批量操作文件

移动所有备份文件（.sql.gz结尾）到某个文件夹

```shell
mv ./*.sql.gz /var/mysqlbackup/
```



### 全量备份

```shell
mysqldump -uroot -p123 -h127.0.0.1 --max-allowed-packet=256M --single-transaction --events --triggers --routines --flush-logs --master-data=2 mysql|gzip > home/mysql.sql.gz
```

mysqldump 是 mysql 自带的备份工具。命令解析：

mysql 用户名root密码123主机127.0.0.1。

参数：

--max-allowed-packet=256M：使client端到server端传递[大数据](http://lib.csdn.net/base/hadoop)时，系统能够分配更多的扩展内存来处理。 

--single-transaction： 该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。

--events： 备份时，事件表会被备份。

--triggers：备份时，触发器会被备份。

--routines：备份时，存储过程和存储函数也会被备份。

--flush-logs：使用一个新的日志文件来记录接下来的日志参数

--lock-all-tables：锁定所有数据库

--master-data=2：
**此值为0**：表示在使用mysqldump进行备份时，不记录对应二进制日志文件位置，将此值显式的设置为0与不使用此选项的效果相同。

**此值为1**：表示在使用mysqldump进行备份时，记录对应二进制日志文件位置，此值为默认值，也就是说，使用--master-data与使用--master-data=1的效果相同，如果将此选项的值设置为1，则会在备份文件中生成对应的"CHANGE MASTER TO"语句，此语句中标明了备份开始时二进制日志的前缀名以及其所处的position(位置)，生成此语句的目的是，在主从复制结构中的"从服务器"中通过备份sql还原数据以后，告诉"从库"，从"主库"的二进制日志文件中的哪个位置开始"同步"。如果我们没有使用主从复制结构，同时又想要在备份时记录二进制日志文件的position，则可以将此选项的值设置为2。

**此值为2**：表示在使用mysqldump进行备份时，记录对应二进制日值文件的位置，如果将此选项的值设置为2，则会在备份文件中生成对应的"CHANGE MASTER TO"语句，此语句中标明了备份开始时二进制日志的前缀名以及其所处的position(位置)，但是"CHANGE MASTER TO"语句将会被注释，与此值为1时不同，此选项值为1时，"CHANGE MASTER TO"语句不会被注释，此选项值为2时，"CHANGE MASTER TO"语句会被注释，所以，如果只是单纯的为了记录备份时的二进制日志文件位置，那么将此选项值设置为2即可。



参考文章：

http://www.zsythink.net/archives/1450/



`|gzip > home/mysql.sql.gz` 

`|` 是管道，将 `mysqldump` 备份的文件给 `gzip` 压缩。

`>` 将压缩后的数据存为文件 mysql.sql.gz



#### 恢复全量备份

```shell
mysql -h localhost -uroot -p123 < bakdup.sql
```

或者进入服务器的 MySQL中执行

```bash
mysql> source /path/bakdup.sql
```

参考文章：

https://www.jianshu.com/p/1f594782a7ba



### 增量备份和将MySQL备份上传到私有云

#### 增量备份：

##### 1. 开启 log_bin 

```shell
mysql -uroot -p***
show variables like `%log_bin%`;
```

查看是否开启 `log_bin` ，修改 MySQL 配置项：/etc/mysql/mysql.conf.d/mysqld.cnf 

```php
[mysqld]
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
datadir     = /var/lib/mysql
#log-error  = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address   = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

#binlog setting，开启增量备份的关键
log-bin=/var/lib/mysql/mysql-bin
server-id=123454
```

重启 MySQL



##### 2. 查看 log_bin

```shell
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000024 |      428 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```



##### 3. 备份

使用新的日志文件

```shell
mysqladmin -uroot -p* flush-logs
```

日志文件从 mysql-bin.000024 变为 mysql-bin.000025

```shell
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000025 |      123 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```



##### 4. 恢复增量备份

```shell
mysqlbinlog /var/lib/mysql/mysql-bin.000024 | mysql -uroot -p*** mysql;
```



参考文章：

https://www.jianshu.com/p/e7393fa2a100

https://www.jianshu.com/p/95ff0144bd66



## 重置服务器的MySQL密码

1. 关掉 MySQL 后，以安全模式启动 MySQl

   ```shell
   sudo mysqld_safe --skip-grant-tables --skip-networking &  
   ```

2. 无密码登录后，重设密码

   ```shell
   mysql -uroot
   mysql> use mysql;  
   mysql> update user set password=PASSWORD("newpassword") where User='root';  
   mysql> flush privileges;
   mysql> quit
   ```

3. 重启 MySQL

   ```shell
   sudo service mysql restart
   ```



参考文章：

https://www.jianshu.com/p/c8f92531bfb7

https://www.jianshu.com/p/679c5be98fcd

























