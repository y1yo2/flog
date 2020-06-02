# Docker 与 iptables



## Docker 容器与 iptables 的关系

默认情况下，Docker daemon（守护程序）在启动 container 时，向 iptables 添加转发规则。

docker 利用这个规则向外暴露 container 端口。

参考资料：https://www.jianshu.com/p/69d3ab177655



docker daemon启动过程会初始化一系列的iptables规则以及修改部分内核参数：

分别在 filter 和 nat 建立名为 DOCKER 的chain，在 forward 转发链增加一些 ACCEPT 规则，在 nat 增加 postrouting 和 prerouting 和 output 的规则。

- filter 表的 forward 主要是对容器和宿主机之间数据包的放心。

- 解析 nat 表的规则 `-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE` 

  处理 docker 的 SNAT地址伪装 用的，将容器网络网段发送到外部的数据包（`! -o docker0`）伪装成宿主机的 ip。SNAT 就是将数据包原来的容器ip 换成宿主机ip。

  不经 snat 转换的容器数据包被其他节点的防火墙给拦截（限制非vm网段的ip）。

-  `-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER` 

  作用是，把目标地址类型属于主机系统的本地网络地址的数据包，在进入 NAT表 PREROUTING链时，jump到一个名为 DOCKER 的链。

参考资料：https://www.jianshu.com/p/5e941739196d

 

## 网桥管理工具 brctl

安装 `bridge-utils` ，linux系统网桥管理工具 `brctl` 安装及使用

```shell
yum install bridge-utils -y
# 查看所有网桥
sudo brctl show
# 删除网桥
sudo brctl delbr br0
# 创建br0网桥
sudo brctl addbr br0

# 将eth0端口加入br0网桥
brctl addif br eth0
# 删除br0网桥的eth0端口
brctl delif br0 eth0
```

参考资料：https://blog.csdn.net/skh2015java/article/details/82466718

​					https://www.jianshu.com/p/665382d70ab1



## 问题与解决办法

### 1. 安装 `iptables` 后，docker run容器报错

```shell
docker: Error response from daemon: driver failed programming external connectivity on endpoint quizzical_thompson (c2b238f6b003b1f789c989db0d789b4bf3284ff61152ba40dacd0e01bd984653):  (iptables failed: iptables --wait -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 172.17.0.3 --dport 24224 -j ACCEPT: iptables: No chain/target/match by that name.
```

上面的 `docker run` 报错是因为 filter 和 nat 中没有 DOCKER chain 导致的。

解决方法：

1. kill掉 docker所有进程

   ```shell
   pkill docker
   ```

2. 清空 nat 表的所有链

   ```shell
   iptables -t nat -F
   ```

3. 停止docker默认网桥 docker0

   ```shell
   ifconfig docker0 down
   ```

4. 删除 docker0网球

   ```shell
   brctl delbr docker0
   ```

5. 重启 docker服务并重启容器

   ```shell
   systemctl restart docker
   docker run -d demo
   ```






### 2. 端口暴露给外部

docker默认直接操作iptables，导致外网可以直接访问主机的容器绑定的端口。

解决办法：

#### 关闭 docker 直接操作 iptables

- 修改ufw默认的配置

```shell
vim /etc/default/ufw

#把DEFAULT_FORWARD_POLICY修改为下面
DEFAULT_FORWARD_POLICY="ACCEPT"
```

- 修改docker的默认配置
  取消注释DOCKER_OPTS这行，在参数后添加添加-iptables=false

```shell
vim /etc/default/docker

#修改文件 添加 --iptables=false
DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4 --iptables=false"
```

- 修改/etc/ufw/before.rules

```shell
vim /etc/ufw/before.rules

在`*filter`前面添加下面内容
###########我是分割线############
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -o docker0 -s 172.17.0.0/16 -j MASQUERADE
COMMIT
###########我是分割线############
```

- 新增文件/etc/docker/daemon.json

```shell
vim /etc/docker/daemon.json

###########我是分割线############
{
"iptables": false
}
###########我是分割线############
```

- 重启

使用这种方法带来的问题是需要全部手动配置docker的iptable



#### 使用内网 IP

**手动指定 docker 映射时的本地 ip**

使用 `-p 127.0.0.1:8080:8080` 来代替 `-p 8080:8080` 



**docker 使用固定ip并使用nginx反向代理**

例如：

指定docker容器为固定ip

```shell
docker network create --subnet=172.18.0.0/16 mynet
docker network ls

docker run -d --restart=always --net mynet --ip 172.18.0.2 --name rsshub diygod/rsshub
```



使用nginx来反向代理docker容器的内网ip

```nginx
# 在网站的配置文件中添加一个upstream
upstream rsshub {
    server 172.18.0.2:1200;
}

server {
    ……
    
    location /rss/ {
		proxy_redirect off;
		proxy_pass http://rsshub/;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $http_host;
	}
   
    ……
}
```

参考资料：https://wajd.ml/docker-an-quan-pei-zhi-zhi-iptables/

参考资料：https://juejin.im/post/5c1616186fb9a049a42ef21d



### 3. 我的例子：

使用方法1设置好iptables和docker之后，

直接指定docker容器为固定ip

```shell
docker run -d -p 127.0.0.1:8080:80 -p 127.0.0.1:8081:80 --name nginx_container nginx_myimg
```



使用nginx来反向代理docker容器的内网ip

```nginx
# 在网站的配置文件中添加一个upstream
upstream nginxcontainer{
	    server 127.0.0.1:8080 weight=1;
		server 127.0.0.1:8081 backup;
	}
	server {
        listen 8082;
		server_name nginx_contain;
		
		# 对 "/" 启用反向代理
		location / {
		    proxy_pass http://nginxcontainer; #请注意此处端口号不能与虚拟主机监听的端口号一样（也就是server监听的端口）
			proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
             
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			
			#以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;

            #允许客户端请求的最大单文件字节数
            client_max_body_size 10m;

            #缓冲区代理缓冲用户端请求的最大字节数，
            #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
            #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
            client_body_buffer_size 128k;

            #表示使nginx阻止HTTP应答代码为400或者更高的应答。
            proxy_intercept_errors on;

            #后端服务器连接的超时时间_发起握手等候响应超时时间
            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout 90;
			
			#后端服务器数据回传时间(代理发送超时)
            #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
            proxy_send_timeout 90;

            #连接成功后，后端服务器响应时间(代理接收超时)
            #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
            proxy_read_timeout 90;

            #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            #设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
            proxy_buffer_size 4k;

            #proxy_buffers缓冲区，网页平均在32k以下的设置
            #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
            proxy_buffers 4 32k;

            #高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k;

            #设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_file_write_size 64k;
		}
	}

```

参考资料：https://juejin.im/post/5c1616186fb9a049a42ef21d



重启 nginx

```shell
# 检查指定的nginx配置文件，是否正确
nginx -t -c /usr/nginx/conf/nginx.conf  

# 检查默认的nginx配置文件
/usr/nginx/sbin/nginx -t

# 重启
/usr/nginx/sbin/nginx -s reload
```

参考资料：https://www.jianshu.com/p/3b045336b4b1



## 网桥是什么？

参考资料：https://cloud.tencent.com/developer/article/1087504



docker 与网桥的关系

参考资料：https://www.cnblogs.com/yy-cxd/p/6553624.html



Docker 如何自定义默认网桥 docker0

参考资料：https://www.jianshu.com/p/ae3e6132a3bd

​	







