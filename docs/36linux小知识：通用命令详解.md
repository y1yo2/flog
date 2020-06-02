### 查看占用端口的PID，并通过PID查找启动脚本和位置

使用 `netstat` , `ps` , `pwdx` 



- 查找使用8080端口的 socket

```shell
netstat -anlp|grep 8080
```

-a：显示所有socket(套接字)，包括正在监听的（LISTEN

-n ：resolve hardware names，不进行DNS轮询，显示IP(可以加速操作)

-l ：display listening server sockets，仅显示监听套接字

-p：display PID/Program name for sockets，显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。



- 查找 pid 123 进程的详细信息

```shell
ps -ef|grep 123
```

-e：所有进程都显示

-f：更完整的输出



- 查找 pid 123进程的对应目录

```shell
pwdx 123
```



### 查看文件夹大小

```shell
ll			# 显示文件和文件夹的字节大小
ls -lh		# 以KB、MB等为单位
du -h		# 显示所有文件和文件夹（包含隐藏文件、隐藏文件夹）
du -sh		# 查询当前目录总大小，s代表统计汇总的意思，即只输出一个总和大小。
```













