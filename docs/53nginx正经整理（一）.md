# Nginx 整理（一）

具体概念：

Nginx（和 Apache一样）是一种 WEB服务器。基于 REST架构风格，以统一资源描述符URI 或 统一资源定位符URL 作为沟通依据，通过 HTTP协议提供服务。



| Apache               | Nginx                                        |
| -------------------- | -------------------------------------------- |
| 重量级、不支持高并发 | 轻量级、高并发；跨平台兼容性差、模块稳定性差 |



### 代理

被代理角色、目标、代理服务器



##### 正向代理

被代理客户端，通过代理服务器（nginx），访问目标服务器。

客户端清楚访问的地址，服务器不知道客户端，隐藏正式客户端的信息。



##### 反向代理

分发请求给分布式集群，隐藏服务器的信息。



### 实际操作

安装

```bash
yum install nginx -y
sudo apt-get install nginx -y
```



启动：  `nginx`



重新加载配置文件

```bash
nginx -s reload
```



配置文件：

/etc/nginx/nginx.conf





关于 REST风格：（可以深入了解）

怎样用通俗的语言解释REST，以及RESTful？ - houhaichao830的回答 - 知乎 https://www.zhihu.com/question/28557115/answer/79275672



关于 URI 和 URL：

URI 是可以确定资源的一个标识。

URL是URI的一种，描述一个资源的位置。这个位置是这个资源的一个标识。



HTTP 协议中 URI 和 URL 有什么区别？ - daixinye的回答 - 知乎 https://www.zhihu.com/question/21950864/answer/154309494





nginx入门文章推荐：

http://nginx.org/en/docs/beginners_guide.html

















