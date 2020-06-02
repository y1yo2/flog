# 安装 PLSQL

步骤：

1. 安装 Oracle数据库或 Oracle客户端
2. 安装 PL/SQL Developer



### 1. 安装 Oracle客户端

下载 instantclient-basic、instantclient-sqlplus

https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html

解压到相同文件夹下，得到如：D:\instantclient

在此路径下建立文件夹 `NETWORK/ADMIN` ，在 `ADMIN` 文件下建立 `tnsnames.ora` 文件，文件内容即为连接的数据库的TNS信息。例如：

```
MY-ORCL=
(DESCRIPTION=
    (ADDRESS_LIST=
        (ADDRESS=(PROTOCOL=TCP)(HOST=192.168.00.00)(PORT=1521))
     )
    (CONNECT_DATA=
        (SERVICE_NAME=orcl)
     )
)
```

MY-ORCL：tns的别名 

HOST：服务器ip 

PORT：端口

SERVICE_NAME：数据库实例名



### 2. 安装 PL\SQL

https://www.jb51.net/softs/664139.html#downintro2

安装后需要破解（シ）

打开后不用登陆，点取消进入主界面，找到 配置——首选项——连接，填写：

Oracle主目录名：D:\instantclient 

OCI库：D:\instantclient\oci.dll



关闭重新打开 PL\SQL ，可以选择 Oracle Client 设置好的数据库连接，填写用户名和密码即可登录使用。









