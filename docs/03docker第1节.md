# Docker第一节

1. linux安装docker
2. Client与Docker daemon交互
3. 使用Dockerfile

## 1.linux安装docker

```bash
sudo wget -qO- https://get.docker.com | sh
```

命令解释：

**sudo**：以root(管理员)身份运行该命令

**wget**：命令行下载工具。跟踪HTML页面上的链接依次下载来创建远程服务器的本地版本，完全重建原始站点的目录结构。这又常被称作”递归下载”。在带宽很窄的情况下和不稳定网络中有很强的适应性。

**-q**：安静(quiet)模式，减少输出

**-O-**：将wget的输出，输出给标准输出，而不是文件

 **| sh**：管道，将输出交给sh命令



```bash
sudo usermod -aG docker your-username
```

命令解释：

If you would like to use Docker as a non-root user, you should now consider adding your user to the "docker" group with something like this.

修改该用户的组，将该用户加入到docker组中。

在docker组中的用户，不用sudo就可执行docker命令。


#### 结构解析：

|          | Client       | DOCKER_HOST(宿主机)                               | Registry(共享docker镜像的服务) |
| ------------ | ------------ | :--------------------------------- | ------------------------------ |
| 作用 | 与daemon交互 | Docker daemon(运行该核心程序，下载镜像，运行容器) | 拖取/提交 images               |
| 交互 | docker build |                                ||
|  | docker pull | ||
|  | docker run | ||


## 2.Client与Docker daemon交互

```shell
docker run -p 8080:80 -d nginx
```

命令解释：

**-p**:   端口映射，把image本身的80端口映射到本地的host的8080端口 

**-d**：运行该程序直接返回。即把这个container作为守护进程执行。

尝试将一个html文件复制到container里面去

```bash
docker cp index.html 573cb0617c0f://usr/share/nginx/html
```

但是

```bash
docker stop 573cb0617c0f 

docker run -p 8080:800 -d nginx
```

停止容器后，镜像重新生成容器，html文件并没有改变。

container的改动，不会影响image。如果要同步改动到image，需要commit container

```bash
docker commit -m 'message' 573cb0617c0f nginx-change
```


#### 命令小结

|      命令         |      用途     |
| ------------- | --------- |
| docker pull   | 获取image |
| docker build  | 创建image |
| docker images | 列出image |
| docker run | 运行container |
| docker ps | 列出container |
| docker rm | 删除container |
| docker rmi | 删除image |
| docker cp | 在host和container之间拷贝文件 |
| docker commit | 提交container的改动，保存为新的image |


## 3.使用Dockerfile

通过编写简单文件创建docker image

例子1：

```dockerfile
FROM alpine:latest		# Set base image

MAINTAINER fan			# 共享时表明Maintainer/Author

RUN echo 'hi dockerfile'# 运行命令

CMD echo 'hello docker'	# 运行命令并创建一个新的容器
```

alpine是专门针对docker开发的，极小的linux环境

```shell
docker build -t hello_docker .
```

**-t**：设置镜像标签

**.**：路径名，将该路径里所有内容都送给docker engine，生成image

问题：运行container，只显示'hello docker'，不显示'hi dockerfile'



例子2：

```dockerfile
FROM ubuntu

MAINTAINER fan

RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list

RUN apt-get update

RUN apt-get install -y nginx					#安装nginx -y,询问y/n时默认选择y

COPY index.html /var/www/html					#拷贝本地文件到容器中去

ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]

EXPOSE 80										#为构建的镜像设置监听端口，使容器在运行时监听。

```

命令解释：

**apt-get update**

UBUNTU系统中，存在一个源列表，源列表里是一些网址信息，每个网址指向的软件，就是我们可以安装的软件，就是源。

```shell
sudo gedit /etc/apt/sources.list	#编辑源列表
sudo gedit /etc/apt/sources.list.d

sudo apt-get update					#从源读取软件最新的索引

sudo apt-get upgrade				#根据索引进行软件更新

```



```dockerfile
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
```

ENTRYPOINT 用于给容器配置一个可执行程序。也就是说，每次使用镜像创建容器时，通过 ENTRYPOINT 指定的程序都会被设置为默认程序。ENTRYPOINT 用于给容器配置一个可执行程序。也就是说，每次使用镜像创建容器时，通过 ENTRYPOINT 指定的程序都会被设置为默认程序。

ENTRYPOINT 与 CMD 非常类似，不同的是通过`docker run`执行的命令不会覆盖 ENTRYPOINT，而`docker run`命令中指定的任何参数，都会被当做参数再次传递给 ENTRYPOINT。Dockerfile 中只允许有一个 ENTRYPOINT 命令，多指定时会覆盖前面的设置，而只执行最后的 ENTRYPOINT 指令。

所以

```shell
docker run -d  hello_docker /usr/sbin/nginx -g "daemon off;"
```

run后面的参数会传递给ENTRYPOINT，等同于上面的语句



再解释一下 

```shell
/usr/sbin/nginx -g "daemon off;"
```

 指令：

因为docker run命令，如果想以守护进程模式运行，要加-d标签，而且镜像中的指令必须以前台模式运行(阻塞)

因为nginx默认以守护进程模式启动，所以使用 `-g "daemon off;"` 调整为前台模式运行。



配置中科大ustc镜像源

```shell
sudo sed -i s/us.archive.ubuntu.com/mirrors.ustc.edu.cn/g /etc/apt/sources.list
sudo sed -i s/security.ubuntu.com/mirrors.ustc.edu.cn/g /etc/apt/sources.list
```



配置国内的网易镜像源

```shell
sudo sed -i s/us.archive.ubuntu.com/mirrors.163.com/g /etc/apt/sources.list
sudo sed -i s/security.ubuntu.com/mirrors.163.com/g /etc/apt/sources.list
```



#### Dockerfile语法小结

|      命令         |      用途     |
| ------------- | --------- |
| FROM   | base image |
| RUN        | 执行命令 |
| ADD | 添加文件(包括远程文件) |
| COPY | 拷贝文件 |
| CMD | 执行命令                   |
| EXPOSE | 暴露端口 |
| WORKDIR | 指定我们运行这些命令的路径 |
| MAINTAINER | 维护者 |
| ENV | 为容器内部设置环境变量 |
| ENTRYPOINT | 容器入口 |
| USER | 指定执行该命令的用户(容器内部一般不适用root) |
| VOLUME | 指定容器挂载的卷(mount point) |





