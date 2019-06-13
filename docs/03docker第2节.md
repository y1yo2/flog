# Docker第二节

1. 镜像分层
2. Volume
3. Registry
4. Compose

## 1.镜像分层

**Dockerfile定义的镜像是分层的。**

每一行都单独保存，每一行都产生一个新层。

镜像里面的层都是只读（RO），只有运行镜像生成的容器层是可读写（RW）的。

相同的镜像层可以共享，减少了空间的使用。

## 2.Volume

提供独立容器外的持久化存储

**使用空间卷**：

```bash
docker run -d --name hello-nginx -v /usr/share/nginx/html nginx
```

**查看容器详情**：

```bash
docker inspect hello-nginx
```

**进入容器内部**：

```shell
docker exec -it hello-nginx /bin/bash
```

命令解释

**-i**  :  interactive(交互的) 

​	Keep STDIN open even if not attached

​	保持标准输入打开，即使没有被连接

**-t**  :  Allocate a pseudo-TTY

​	分配伪TTY

**-it** :  分配一个伪tty，并保持stdin打开，即使没有被连接。

​	这将允许您像传统的VM一样使用容器，只要bash提示符正在运行。 

**/bin/bash**：表示使用的shell是/bin/bash。

​			shell：就是操作系统和人之间交互的平台。例如windows的桌面环境也是一个shell。

​			/bin：该目录中基本是可执行的命令。

​			bash：大多数Linux发行版默认的shell。最常用的一种shell。



**使用空间卷：（指定本地目录）**

```bash
docker run -d --name hello-nginx -v $PWD/html:/usr/share/nginx/html nginx
```

**$PWD**:	shell的一个宏定义变量，始终指向当前目录。$PWD/html指向当前目录下的html目录。



**创建容器但不运行**

```bash
docker create -v $PWD/data:/var/mydata --name hello-nginx nginx
```

**将一个容器作为volume挂载到其他容器上**

```shell
docker run -d  --volumes-from hello-nginx ubuntu
```


## 3.Registry

docker的镜像仓库

注册，存储镜像



交互命令:

docker search whalesay

docker pull whalesay

docker push mywhalesay

docker tag docker/whalesay test/whalesay



- 注意：push镜像前需要登录自己的docker hub账号

  docker login

- 注意：登陆成功后，push还是报错，

  denied: requested access to the resource is denied

  因为镜像名需要是 (自己的dockerID)/whalesay(镜像名)

## 4.Compose

### - docker-compose的安装

docker-compose 多容器app。独立于docker的程序。

Mac/Windows：自带

Linux：

```bash
curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s)-$(uname -m) > /usr/local/bin/docker-compose
```

命令结束：

**curl**：下载网页内容

 **-L**    Follow redirects：跟踪跳转

**$(uname -s)**：将uname -s这个命令的输出加在这个地方

`>`：将下载的数据重定向到/usr/local/bin/下的一个文件

usr 指 Unix System Resource

**/usr/bin**下面的都是系统预装的可执行程序，会随着系统升级而改变。

**/usr/local/bin**目录是给用户放置自己的可执行程序的地方，推荐放在这里，不会被系统升级而覆盖同名文件。

完整命令：将这个地址用curl download，用管道的方式写入到文件里面 



```bash
ls -l /usr/local/bin/docker-compose
chmod a+x /usr/local/bin/docker-compose
```

ls -l 可以查看文件或目录的权限。
chmod 修改文件或目录的权限。

### - docker-compose的配置文件

##### 1. 创建目录，nginx、ghost、data
##### 2. 准备ghost镜像所需resources

​	编写Dockerfile

```dockerfile
FROM ghost
MAINTAIN yiyoe
COPY ./config.production.json /var/lib/ghost/content/config.production.json
EXPOSE 2368
#CMD ["npm", "start", "--production"]
```

​	从镜像中心找到ghost的config.production.json文件，将配置文件copy上服务器。

​	注意：ghost项目存在多版本问题

##### 3. 准备nginx镜像所需resources

​	编写Dockerfile

```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

​	不需要启动命令，因为基础镜像中包含启动命令

​	创建nginx.conf

```nginx
worker_processes 4;					//四个worker进程
events {worker_connections 1024;}	//
http{								//配置http服务器
        server{
        	listen 80;				//监听80端口
            location / {			//路径
            	proxy_pass http://ghost-app:2368;	//反向代理的语法，链接到后面的地址
        }
    }
}

```

##### 4. 创建compose文件

docker-compose.yaml

```yaml
version: '2'					#使用第2版的语法

networks:						#声明网络。如果不声明，compose会主动为你创建网络
	ghost:

services:
	ghost-app:					#ghost-app服务
		build: ghost			#请到ghost目录构建
		networks:
			-ghost				#它的网络是ghost
		depends_on:
			-db
		ports:
			-"2368:2368"
			
nginx:
	build: nginx
	networks:
		-ghost
	depends_on:
		-ghost-app
	ports:
		-"80:80"
db:								#因为ghost-app依赖的服务是db
	image: "mysql:5.7.15"		#从网络下载这个镜像
	networks:
		-ghost
	environment:				#声明环境变量，设置mysql参数
		MYSQL_ROOT_PASSWORD: mysqlroot
		MYSQL_USER: ghost
		MYSQL_PASSWORD: ghost
	volumes:					#将data挂载到mysql里面去
		- $PWD/data:/var/lib/mysql
	ports:
		-"3306:3306"
	
```



##### 5. 使用compose启动

```bash
docker-compose up -d				#建议先用非守护线程模式启动，查看日志排除启动日常

docker-compose stop

docker-compose rm					#删除停止的服务容器

docker-compose build				#构建镜像
```



