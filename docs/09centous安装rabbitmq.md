## 	《阿里巴巴Java开发手册v1.4.0（详尽版）》

在安装任何软件包之前，建议您使用以下命令更新软件包和存储库。

```
yum -y update
```

RabbitMQ是用Erlang语言编写的，在本教程中我们将安装最新版本的Erlang到服务器中。 Erlang在默认的YUM存储库中不可用，因此您将需要安装EPEL存储库。 运行以下命令相同。

```shell
yum -y install epel-release

yum -y update
```

现在使用以下命令安装Erlang。

```
yum -y install erlang socat
```

您现在可以使用以下命令检查Erlang版本。

erl -version

RabbitMQ为预编译并可以直接安装的企业[Linux系统](https://www.linuxprobe.com/)提供RPM软件包。 唯一需要的依赖是将Erlang安装到系统中。 我们已经安装了Erlang，我们可以进一步下载RabbitMQ。 通过运行下载Erlang RPM软件包。

```
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm
```

通过运行导入GPG密钥：

```
rpm –import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
```

运行RPM安装RPM包：

```
rpm -Uvh rabbitmq-server-3.6.10-1.el7.noarch.rpm
```

RabbitMQ现已安装在您的系统上。



service rabbitmq-server start

service rabbitmq-server stop

service rabbitmq-server restart

配置为守护进程随系统自动启动，root权限下执行:

```
systemctl enable rabbitmq-server
```

```
chkconfig rabbitmq-server on
```





启动RabbitMQ Web管理控制台，方法是运行：

```
rabbitmq-plugins enable rabbitmq_management
```

通过运行以下命令，将RabbitMQ文件的所有权提供给RabbitMQ用户：

```
chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/
```

现在，您将需要为RabbitMQ Web管理控制台创建管理用户。 运行以下命令相同。

```
rabbitmqctl add_user admin StrongPassword
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin “.*” “.*” “.*”
```

将管理员更改为管理员用户的首选用户名。 确保将StrongPassword更改为非常强大的密码。

要访问RabbitMQ的管理面板，请使用您最喜爱的Web浏览器并打开以下URL。

```
http://Your_Server_IP:15672/
```







(1) 新增一个用户

```
rabbitmqctl  add_user  Username  Password
```

(2) 删除一个用户

```
rabbitmqctl  delete_user  Username
```

(3) 修改用户的密码

```
rabbitmqctl  change_password  Username  Newpassword
```

(4) 查看当前用户列表

```
rabbitmqctl  list_users
```





配置与端口问题：

mq默认端口：5672

管理端默认端口：15672



需要配置文件：

/etc/rabbitmq/rabbitmq.config

```
[
{rabbit, [{tcp_listeners, [5672]}, {loopback_users, ["admin"]}]}
].
```





```
//把包名相关的包都列出来  
rpm -qa | grep 包名
//你想卸载的软件，后面是包名称，最后的版本号是不用打的
rpm -e 文件名

-------------卸载
/sbin/service rabbitmq-server stop
yum list | grep rabbitmq
yum -y remove rabbitmq-server.noarch
 
yum list | grep erlang
yum -y remove erlang-*
yum remove erlang.x86_64 
rm -rf /usr/lib64/erlang
rm -rf /var/lib/rabbitmq
```



安装完整erlang，https://www.erlang-solutions.com/resources/download.html

```
wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
rpm --import https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
```



Add the following lines to some file in "/etc/yum.repos.d/":

```
[erlang-solutions]
name=CentOS $releasever - $basearch - Erlang Solutions
baseurl=https://packages.erlang-solutions.com/rpm/centos/$releasever/$basearch
gpgcheck=1
gpgkey=https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
enabled=1
```



```
yum -y install epel-release
yum install erlang
```



```bash
[bintray-rabbitmq-server]
name=bintray-rabbitmq-rpm
baseurl=https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.7.x/el/7/
gpgcheck=0
repo_gpgcheck=0
enabled=1
```



```
wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.12/rabbitmq-server-3.7.12-1.el7.noarch.rpm
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
yum install rabbitmq-server-3.7.12-1.el7.noarch.rpm

chkconfig rabbitmq-server on

rabbitmq-plugins enable rabbitmq_mqtt
rabbitmq-plugins enable rabbitmq_stomp
rabbitmq-plugins enable rabbitmq_web_mqtt
rabbitmq-plugins enable rabbitmq_web_stomp

# 由于RabbitMQ默认的账号用户名和密码都是guest。为了安全起见, 先删掉默认用户
 rabbitmqctl delete_user guest
 # 添加新用户
 rabbitmqctl add_user username password
 # 设置用户tag
 rabbitmqctl set_user_tags username administrator
 # 赋予用户默认vhost的全部操作权限
 rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
 # 查看用户的权限
 rabbitmqctl list_user_permissions username
```



```
rabbitmqctl reset #参考 http://blog.csdn.net/wochunyang/article/details/52524977
```



后续：

布置成功后，无法连接和使用，有可能是一下问题。

本机ip和名称并不是127.0.0.1和localhost。

将本机ip和名称写入HOST文件。

查看本机用户名：root权限登陆，root@本机用户名。

查看本机ip：ifconfig 或者  ip addr show  





重写hosts echo 9.2.2.4 rabbit-api-001 >> /etc/hosts 

测试  ping rabbit-api-001 