## Docker容器卷

[toc]



### 什么是容器数据卷?

如果数据都在容器中, 那么我们删除容器, 数据就会丢失 ~  需求: 数据可以持久化

Mysql, 容器删了, 需求: Mysql 的数据可以存储在本地(宿主机器)

可以有一个数据共享的技术, Docker 容器中产生的数据, 同步到本地, 这就是卷技术, 目录的挂载, 将linux的目录, 挂载到容器上面(目录中的数据是双向绑定的)

**容器的持久化和同步操作, 容器之间也是可以数据共享的**

<br>

### 使用数据卷

```shell
# 直接使用命令来挂载卷 -v 需要使用绝对路径
docker run -it -v 宿主机器目录:容器内目录

# 测试, 如果目录不存在会自动创建
[chenkai@centos7 ~]$ docker run -it -v /home/ceshi:/home centos /bin/bash
# 1. 在宿主机器/home/ceshi 目录下创建文件, 进入docker容器后 /home 目录下也会同步到文件
# 2. 在docker容器中 /home 目录下创建文件, 修改文件, 宿主机器 /home/ceshi 目录下也会同步文件
# 3. 先停止容器, 然后在宿主机器上修改文件内容, 启动容器, 查看容器中的数据内容是否依旧是同步的?   是!
# 实现了文件数据的双向绑定

# 启动起来后, 可以使用 docker inspect 容器ID 来检查挂载目录
[chenkai@centos7 ceshi]$ docker inspect 5c0ec5f1a23d
...
...
        Mounts": [
            {
                "Type": "bind",
                "Source": "/home/ceshi",
                "Destination": "/home",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ]
...
...
```

<br>

### Mysql数据持久化

```shell
# 获取镜像
[chenkai@centos7 ~]$ docker pull mysql:5.7

# 运行容器, 需要做数据挂载,  # 安装启动mysql, 需要配置密码的, 这是要注意的点
# 官方测试: docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
[chenkai@centos7 ~]$ docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
5bcdba2653c3efc81605cdcc998be071e23bc5f4900d5efaefbcb0d3e4408f51

# 启动成功之后,进入容器内部, 登录mysql
root@5bcdba2653c3:/# mysql -uroot -p123456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.32 MySQL Community Server (GPL)
Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 

# 查看宿主机器/home/mysql/data 目录数据, 和 /home
[chenkai@centos7 data]$ pwd
/home/mysql/data
[chenkai@centos7 data]$ ll
total 188484
-rw-r-----. 1 polkitd input       56 Dec 15 11:12 auto.cnf
-rw-------. 1 polkitd input     1676 Dec 15 11:12 ca-key.pem
-rw-r--r--. 1 polkitd input     1112 Dec 15 11:12 ca.pem
-rw-r--r--. 1 polkitd input     1112 Dec 15 11:12 client-cert.pem
-rw-------. 1 polkitd input     1680 Dec 15 11:12 client-key.pem
-rw-r-----. 1 polkitd input     1353 Dec 15 11:12 ib_buffer_pool
-rw-r-----. 1 polkitd input 79691776 Dec 15 11:12 ibdata1
-rw-r-----. 1 polkitd input 50331648 Dec 15 11:12 ib_logfile0
-rw-r-----. 1 polkitd input 50331648 Dec 15 11:12 ib_logfile1
-rw-r-----. 1 polkitd input 12582912 Dec 15 11:12 ibtmp1
drwxr-x---. 2 polkitd input     4096 Dec 15 11:12 mysql
drwxr-x---. 2 polkitd input     8192 Dec 15 11:12 performance_schema
-rw-------. 1 polkitd input     1680 Dec 15 11:12 private_key.pem
-rw-r--r--. 1 polkitd input      452 Dec 15 11:12 public_key.pem
-rw-r--r--. 1 polkitd input     1112 Dec 15 11:12 server-cert.pem
-rw-------. 1 polkitd input     1676 Dec 15 11:12 server-key.pem
drwxr-x---. 2 polkitd input     8192 Dec 15 11:12 sys


root@5bcdba2653c3:/var/lib/mysql# pwd
/var/lib/mysql
root@5bcdba2653c3:/var/lib/mysql# ls -l
total 188484
-rw-r-----. 1 mysql mysql       56 Dec 15 16:12 auto.cnf
-rw-------. 1 mysql mysql     1676 Dec 15 16:12 ca-key.pem
-rw-r--r--. 1 mysql mysql     1112 Dec 15 16:12 ca.pem
-rw-r--r--. 1 mysql mysql     1112 Dec 15 16:12 client-cert.pem
-rw-------. 1 mysql mysql     1680 Dec 15 16:12 client-key.pem
-rw-r-----. 1 mysql mysql     1353 Dec 15 16:12 ib_buffer_pool
-rw-r-----. 1 mysql mysql 50331648 Dec 15 16:12 ib_logfile0
-rw-r-----. 1 mysql mysql 50331648 Dec 15 16:12 ib_logfile1
-rw-r-----. 1 mysql mysql 79691776 Dec 15 16:12 ibdata1
-rw-r-----. 1 mysql mysql 12582912 Dec 15 16:12 ibtmp1
drwxr-x---. 2 mysql mysql     4096 Dec 15 16:12 mysql
drwxr-x---. 2 mysql mysql     8192 Dec 15 16:12 performance_schema
-rw-------. 1 mysql mysql     1680 Dec 15 16:12 private_key.pem
-rw-r--r--. 1 mysql mysql      452 Dec 15 16:12 public_key.pem
-rw-r--r--. 1 mysql mysql     1112 Dec 15 16:12 server-cert.pem
-rw-------. 1 mysql mysql     1676 Dec 15 16:12 server-key.pem
drwxr-x---. 2 mysql mysql     8192 Dec 15 16:12 sys

# 新建一个数据库, 查看两边数据变化
mysql> create database test charset=utf8;
Query OK, 1 row affected (0.00 sec)
# 发现两边目录下, 同时多了 test 目录
drwxr-x---. 2 polkitd input       20 Dec 15 11:20 test
drwxr-x---. 2 mysql mysql       20 Dec 15 16:20 test
```

假如我们将容器删除, 发现我们挂载到本地的数据卷内容依旧没有丢失, 这就实现了容器数据的持久化

<br>

### 匿名和具名挂载

```shell
# 匿名挂载, -v 只写了一个路径, 它指的是容器内路径, 缺少容器外的路径
-v 容器内路径
[chenkai@centos7 data]$ docker run -d -P --name nginx_1 -v /etc/nginx nginx

# 查看所有的卷
[chenkai@centos7 data]$ docker volume ls
DRIVER    VOLUME NAME
local     57ede296c18c38e8b3d16aafd4de8ef68bcaed1436a366728e1d023f4f329444
local     62a163bef603a873c01dcd16f7f56a25f4166becc64833263b0a2727abf39dc7
local     68dfad7250b4f2a39e174eb142f5e8459a9cf930a49b09cb5ee376cf645b13e3
local     01803775ba6daf8c82971fa85fc827fb32a6218e4f2ee79bd5d8c8e6644fb7ae
local     d30ce148bd7cf674e601326d4898e07ea94d74d541925fad83187e3fd711afde
local     d46ba9f2e6681e5d9c7e428764a5f995e7cc1693d9254543a7c63747374572b0

# 具名挂载, -v 卷名:容器内路径
[chenkai@centos7 data]$ docker run -d -P --name nginx_2 -v juming:/etc/nginx nginx

# 检查一下 juming 这个卷
# 具名挂载,没有指定绝对目录的情况下, 默认都是 /var/lib/docker/volumes/xxx/_data 目录下(匿名挂载， 也是在这个目录下)
[chenkai@centos7 data]$ docker volume inspect juming
[
    {
        "CreatedAt": "2020-12-16T02:59:33-05:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming/_data",
        "Name": "juming",
        "Options": null,
        "Scope": "local"
    }
]
# 查看 /var/lib/docker 目录下的内容
[chenkai@centos7 docker]$ pwd
/var/lib/docker
[chenkai@centos7 docker]$ sudo ls
buildkit    image    overlay2  runtimes  tmp	volumes
containers  network  plugins   swarm	 trust
```

##### 如何确定是匿名挂载还是具名挂载, 还是指定路径挂载?

```shell
-v /容器内路径			# 匿名
-v 卷名:/容器内路径	   # 具名
-v /宿主机路径:/容器内路径  # 指定路径挂载 
```

<br>

### 拓展: 目录的读写权限

```shell
# 通过 :ro :rw 改变读写权限
ro  readonly  # 只读
rw  readwrite  # 可读可写

# 一旦设置了容器卷权限, 容器对我们挂载出来的内容就有限定了
docker run -d -P --name nginx_1 -v juming:/etc/nginx:ro nginx
docker run -d -P --name nginx_2 -v juming:/etc/nginx:rw nginx

# ro 只要看到ro就说明这个路径只能通过宿主机器来操作, 容器内部是无法操作的 
```



<br><br><br>

完 ~