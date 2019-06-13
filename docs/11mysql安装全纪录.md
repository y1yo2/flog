1:检查是否本地已经安装了mysql

```bash
rpm -qa | grep mysql
```

2.卸载以前的mysql

```bash
rpm -e 已经存在的MySQL全名
```

3.首先[CentOS](http://www.linuxidc.com/topicnews.aspx?tid=14)7 已经不支持mysql，因为收费了你懂得，所以内部集成了mariadb，而安装mysql的话会和mariadb的文件冲突，所以需要先卸载掉mariadb，以下为卸载mariadb，安装mysql的步骤。

3.1 列出所有被安装的rpm package 
rpm -qa | grep mariadb

3.2 卸载
rpm -e mariadb-libs-5.5.37-1.el7_0.x86_64
错误：依赖检测失败：
libmysqlclient.so.18()(64bit) 被 (已安裝) postfix-2:2.10.1-6.el7.x86_64 需要
libmysqlclient.so.18(libmysqlclient_18)(64bit) 被 (已安裝) postfix-2:2.10.1-6.el7.x86_64 需要

3.3 强制卸载，因为没有--nodeps
rpm -e --nodeps mariadb-libs-5.5.37-1.el7_0.x86_64

4.安装mysql依赖
yum install vim libaio net-tools



CentOS7的yum源中默认好像是没有mysql的。为了解决这个问题，我们要先下载mysql的repo源。

1. 下载mysql的repo源

```bash
wget https://dev.mysql.com/get/mysql80-community-release-el7-2.noarch.rpm
```

 若报-bash: wget: command not found，则表明没有安装wget，需要安装，安装命令如下：

```
yum -y install wget
```

安装完成即可以使用。



2. 安装 `mysql80-community-release-el7-2.noarch.rpm` 包

```bash
sudo rpm -ivh mysql80-community-release-el7-2.noarch.rpm


```

 安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo。

3.安装mysql

```bash
sudo yum install mysql-server
```

根据步骤安装就可以了，不过安装完成后，没有密码，需要重置密码。

[[[[[    新版本有问题，请跳过

4.重置密码

重置密码前，首先要登录

$ mysql -u root

登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ (2)，原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：

$ sudo chown -R openscanner:openscanner /var/lib/mysql

然后，重启服务：

$ service mysqld restart

接下来登录重置密码：

$ mysql -u root

mysql > use mysql;

mysql > update user set password=password(‘123456‘) where user=‘root‘;

mysql > exit;

]]]]]

设置开机启动

```bash
sudo systemctl enable mysqld
sudo systemctl daemon-reload
```

修改 root 本地登录密码
MySQL 安装完成之后，在 /var/log/mysqld.log 文件中给 root 生成了一个默认密码。通过下面的方式找到 root 默认密码，然后登录 mysql 进行修改：

```bash
sudo grep 'temporary password' /var/log/mysqld.log
```

修改 root 本地登录密码
MySQL 安装完成之后，在 /var/log/mysqld.log 文件中给 root 生成了一个默认密码。通过下面的方式找到 root 默认密码，然后登录 mysql 进行修改：

```bash
sudo grep 'temporary password' /var/log/mysqld.log
```

用默认的密码登录之后，修改本地登录密码

```bash
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourNewPassWord!';
```

注意：从 mysql5.7 以后默认安装了密码安全检查插件（validate_password），默认密码检查策略要求密码必须 包含：大小写字母、数字和特殊符号，并且长度不能少于8位。否则会提示 ERROR 1819 (HY000): Your password does not satisfy the current policy requirements 错误。
添加远程登录用户
默认只允许 root 帐户在本地登录，如果要在其它机器上连接 MySQL，必须修改 root 允许远程连接，或者添加一个允许远程连接的帐户，为了安全起见，我添加一个新的帐户：

```bash
create user jayden identified by 'jayden!spassword'; # 创建用户
grant all privileges on *.* to 'jayden'@'%'; # 分配权限
flush privileges; # 刷新权限
```



最近下载新的MySQL8.0 来使用的时候， 通过sqlyog、或者程序中连接数据库时，提示：Authentication plugin 'caching_sha2_password' cannot be loaded 的错误，经查看发现，8.0改变了 身份验证插件 ， 打开 my.ini (或者my.cofg) 可以看到变更了 5.7及其以前的方式: mysql_native_password

解决
Authentication plugin ‘caching_sha2_password’ cannot be loaded 的方法，可以往你的连接工具、或者程序应用显示指定身份验证方式，或者直接改为以前的版本方式：
你可以使用如下方式：

这里root的密码改为11111111，使用老版本的身份验证插件方式：

```bash
ALTER USER root@localhost IDENTIFIED WITH mysql_native_password BY ‘11111111’;


```


打开防火墙 3306 远程端口

```bash
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```



开放3306端口 {

$ sudo vim /etc/sysconfig/iptables

添加以下内容：

-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT

保存后重启防火墙：

$ sudo service iptables restart

}







mysql语法错误：this is incompatible with sql_mode=only_full_group_by

Mysql5.7版本之后对sql_mode做了修改，其中 ONLY_FULL_GROUP_BY 成为了默认模式之一。

执行以下命令，可查看sql_mode默认模式：

select @@GLOBAL.sql_mode;
-- 或者
select @@sql_mode;
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

像oracle数据库采用的就是开启ONLY_FULL_GROUP_BY 模式。

简单理解ONLY_FULL_GROUP_BY模式，用于限制group by语法，要求select的字段要和group by的字段一致，保证select字段的唯一性。否则会造成搜索引擎不知道该返回哪一条。

 

比如有一张temp表：   

group_id	name	age
1	AA	34
2	BB	43
2	CC	12
3	DD	45
执行SQL：select group_id,name from temp group by group_id

返回结果中group_id为2的数据肯定会变成只有一条，但对应的name字段的值不唯一性，所以会抛出异常信息：

Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'temp.name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

解决方案：

方案一：

使用mysql函数 any_value(field) 用于包含非分组字段的出现

比如：select group_id,any_value(name) from temp group by group_id

这样就不用关闭ONLY_FULL_GROUP_BY 模式

方案二：

修改mysql配置文件my.ini (linux系统修改my.cnf)，把 ONLY_FULL_GROUP_BY 从sql_mode中去掉

sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

如果没有以上配置就添加到[mysqld]中，注意：需要重启mysql服务生效。

