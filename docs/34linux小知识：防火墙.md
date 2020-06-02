# Linux 防火墙

https://yq.aliyun.com/zt/59078

Centos7和Centos6 防火墙的区别：


使用的工具不一样。Centos6 默认使用的是iptables，Centos7 默认使用的是filewalld。



filewalld 和 iptables 的区别：

1. firewalld的配置存储在各种XML文件中
   在/usr/lib/firewalld和/etc/firewalld中
   这允许极大的灵活性，因为文件可以被编辑、写入、备份、用作其他安装的模板等等。
   
2. iptables 是与最新的 3.5 版本 Linux 内核集成的 IP 信息包过滤系统。如果 Linux 系统连接到因特网或 LAN、服务器或连接 LAN 和因特网的代理服务器， 则该系统有利于在 Linux 系统上更好地控制 IP 信息包过滤和防火墙配置。

3. iptables 用于过滤数据包，属于网络层防火墙。
   firewall 能够允许哪些服务可用，哪些端口可用。

4. firewalld和iptables一样需要通过内核的netfilter来实现，他们的作用都是用于维护规则，而真正使用规则干活的是内核的netfilter，只不过firewalld和iptables的结构以及使用方法不一样。



## firewalld 使用

https://www.jianshu.com/p/dbf2f49fb9cc

firewalld的操作方法主要有：firewall-cmd 和 直接编辑xml文件



## iptables 使用

https://blog.51cto.com/xjsunjie/1902993

https://yq.aliyun.com/articles/738189?spm=a2c4e.11155472.0.0.7bd942c4VIJlIm

https://www.jianshu.com/p/586da7c8fd42

https://cloud.tencent.com/developer/article/1335131

https://www.jianshu.com/p/d4efe88c20d2



使用例子：

```shell
# 查看状态
firewall-cmd --state
systemctl status firewalld.service
# 查看过滤的列表信息
firewall-cmd --list-all

# 关闭 firewall
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl mask firewalld.service
# Created symlink from /etc/systemd/system/firewalld.service to /dev/null.

# 安装 iptables
yum install iptables-services -y
# 启动 iptables 并设置开机启动
systemctl start iptables
systemctl enable iptables
# 查看 iptables 状态
systemctl status iptables

# 查看 iptables 具体规则（按照命令/链类别）
iptables -S
iptables -L -n
# NAT表的显示
iptables -t nat -nL

# 将当前防火墙策略备份
iptables-save > /tmp/iptables-save.bak1220
# 恢复
# iptables -restore < /tmp/iptables-save.bak1220
# 清除所有的规则
# 清除预设表filter中所有规则链中的规则
iptables -F
# 清除预设表filter中使用者自定链中的规则
iptables -X
iptables -Z
# 清楚NAT表规则
iptables -F -t nat

# 设置链的默认策略。一般有两种方法：允许所有的包，然后再禁止有危险的包通过放火墙
# 禁止所有的包，然后根据需要的服务允许特定的包通过防火墙
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

# 开启SSH服务端口
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

# 开启WEB服务端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

# 开放网络接口，允许loopback，不然会导致DNS无法正常关闭等问题
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
iptables -A INPUT -i eth0 -j ACEPT
iptables -A OUTPUT -o eth1 -j ACCEPT
# 开启转发功能
iptables -A FORWARD -i eth1 -j ACCEPT
iptables -A FORWARD -o eth1 -j ACCEPT

# 设置icmp服务
iptables -A OUTPUT -p icmp -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT

# iptables 规则保存
service iptables save
# 重启 iptables 并设置开机启动
systemctl restart iptables
systemctl enable iptables
```



## systemctl参数：stop、disable、mask 的区别

http://www.kbase101.com/question/15548.html

http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html

- stop：停止服务
- enabled：服务对应的配置文件，建立启动链接
- disabled：取消建立的启动链接
- masked：该配置文件被禁止建立启动链接

当`mask`服务时，会从`/etc/systemd/system`创建一个符号链接到`/dev/null`



## tcpdump 使用

https://yq.aliyun.com/articles/687970

https://www.jianshu.com/p/8d23c35fccf2

https://www.jianshu.com/p/bcc24f8456a1

```shell
# 安装tcpdump
yum instabll tcpdump -y

# 截获经过网卡0从本机发出或发送到本机的数据
tcpdump -n -i eth0 host 192.168.0.***

# 查看本机ip
ip addr

# 抓取本地环路数据包
tcpdump -i lo udp #抓取UDP数据
tcpdump -i lo udp port 1234 #抓取端口1234的UDP数据
tcpdump -i lo port 1234 #抓取端口1234的数据

# 过滤端口：
# 抓取所有经过网卡1，目的端口为1234的网络数据
tcpdump -i eth1 dst port 1234

# 截获经过网卡0，发送到192.168.0.214服务器且端口为80的数据，并存储
tcpdump -n -i eth0 dst host 192.168.0.214 and port 80 -w /tmp/xxx.back
```



具体用法：

第一种是关于类型的关键字，主要包括host，net，port

第二种是确定传输方向的关键字，主要包括src , dst ,dst or src, dst and src ,这些关键字指明了传输的方向。

第三种是协议的关键字，主要包括fddi,ip,arp,rarp,tcp,udp等类型。

`host` 指**从本机发出**或**发送到本机**，`dst` 指**发送到本机**，`src `指**从本机发出**

`-i tap10` 指数据包通过`tap10`设备，一定要保留





## Docker 与 iptables（见下节）



















