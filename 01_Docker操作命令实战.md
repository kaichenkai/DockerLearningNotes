## Docker操作命令实战

### 基本命令

```shell
docker version			# 显示docker的版本信息
docker info 			# 显示docker的具体信息, 包括镜像和容器的数量
docker 命令 --help		   # 帮助命令
```

帮助文档地址:  https://docs.docker.com/engine/reference/commandline/docker/ 

<br>

### 镜像命令

##### docker images			# 查看本地主机上的镜像

```shell
[chenkai@centos7 ~]$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
redis        5.0       aa27923130e6   3 weeks ago   98.4MB

# 解释
REPOSITORY			# 镜像的仓库源
TAG					# 标签
IMAGE ID			# 镜像ID
CREATED				# 创建时间
SIZE				# 大小

# 可选项
Options:
  -a, --all             Show all images (default hides intermediate images)					# 列出所有镜像
      --digests         Show digests  # 格式化
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs  # 只显示镜像的ID

# 常用: 显示所有镜像的ID
docker images -aq
```

##### docker search		# 搜索镜像

```shell
[chenkai@centos7 ~]$ docker search mysql

# 可选项, 通过条件过滤搜索想要的
Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
--filter=STARS=3000		# 搜索点赞数量大于3000的镜像源(stars忽略大小写)
[chenkai@centos7 ~]$ docker search mysql --filter=stars=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   10247     [OK]       
mariadb   MariaDB is a community-developed fork of MyS…   3785      [OK]
```

##### docker pull		# 下载镜像

```shell
[chenkai@centos7 ~]$ docker pull mysql		
Using default tag: latest				# 默认下载最新版本

# 通过 : (冒号) 指定版本
[chenkai@centos7 ~]$ docker pull mysql:5.7
5.7: Pulling from library/mysql

# docker会使用分层下载, 这是docker image的核心 (联合文件系统)
```

##### docker rmi		# 删除镜像

```shell
[chenkai@centos7 ~]$ docker rmi ae0658fdbad5

# 删除所有镜像(-f[force] 表示强制)
[chenkai@centos7 ~]$ docker rmi -f  $(docker images -aq)
# 使用 xargs 命令实现删除所有镜像
[chenkai@centos7 ~]$ docker images -aq | xargs docker rmi -f
```

<br>

### 容器命令

有了镜像之后才可以运行容器, 我们下载一个 centos 镜像测试一下

```shell
docker pull centos
```

##### 新建容器并启动

```shell
docker run [可选参数] image
# 参数说明
--name="container_name"		# 容器名字,例如tomcat1, tomcat2, 用来区分容器
-i							# 保持容器运行
-d 							# 后台守护方式运行
-it							# 使用交互方式运行, 进入容器查看内容
-p							# 指定容器的端口,例如 8080:8080 前面是宿主机器端口
-P							# 大P, 随机指定端口

# 测试, 启动并进入容器
[chenkai@centos7 ~]$ docker run -it centos /bin/bash
[root@fc8b9409f4d8 /]# 
```

##### 退出容器

```shell
[root@538f6e2e1f8a /]# exit
exit		# 直接容器停止, 并退出
[chenkai@centos7 ~]$ 
	
ctrl + p + q		# 退出之后, 容器仍在运行
[chenkai@centos7 ~]$ docker run -it centos /bin/bash
[root@62c27b2cc4b3 /]# 
[root@62c27b2cc4b3 /]# [chenkai@centos7 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
62c27b2cc4b3   centos    "/bin/bash"   13 seconds ago   Up 11 seconds             magical_galileo
```

##### docker ps		# 查看容器

```shell
# docker ps [可选参数]
-a			# 所有容器
-n			# 指定数量的容器信息
-q			# 只显示容器ID

# 查看正在运行中的容器信息
[chenkai@centos7 ~]$ docker ps

# 查看所有容器信息(包括已经停止的容器)
[chenkai@centos7 ~]$ docker ps -a

# 根据创建时间输出最近的 n 个容器信息(包括已经停止的容器)
[chenkai@centos7 ~]$ docker ps -n=3
```

##### docker rm		# 删除容器

```shell
docker rm 容器ID		# 不能删除正在运行中的容器, 强制删除需要加 -f 参数

# 删除所有容器(-f[force] 表示强制)
[chenkai@centos7 ~]$ docker rm -f  $(docker ps -aq)
# 使用 xargs 命令实现删除所有容器
[chenkai@centos7 ~]$ docker ps -aq | xargs docker rm -f
```

##### 启动和停止容器的操作

```shell
docker start 容器ID
docker restart 容器ID
docker stop 容器ID
docker kill 容器ID		# 强制停止容器
```

##### 进入当前正在运行的容器

```shell
# 我们通常容器都是后台方式运行的, 需要进入容器, 修改一些配置

# 方式1 (进入容器后, 开启一个新的终端, 可以在里面进行操作)
docker exec -it 容器ID /bin/bash

# 方式2 (进入容器正在执行的终端, 不会启动新的进程)
docker attach 容器ID
```

<br>

### 常用其他命令

##### 后台启动容器

```shell
# 命令: docker run -d 镜像名
[chenkai@centos7 ~]$ docker run -d centos /bin/bash

# 问题: docker ps, 发现 centos 停止了
# 常见的坑, docker 容器使用后台运行, 就必须要有一个前台进程, docker 发现没有应用, 就会自动停止
# nginx 容器启动后, 发现自己没有启动服务, 就会立刻停止, 就是没有程序了...
```

##### 查看日志

```shell
# 查看最后 10 条日志, -f 是 follow, -t 是timestamps 时间戳, 也可以写成 -tf
docker logs -f -t --tail 10 容器ID

# 动态显示所有日志
docker logs -f -t 容器ID

# demo
[chenkai@centos7 ~]$ docker run -d centos /bin/bash -c "while true; do echo kaichenkai; sleep 1; done"
0c36e7a02175d97c371b754295a93a34ebc0dc94cb20355a7bbc4ac0b2b7485a
[chenkai@centos7 ~]$ 
[chenkai@centos7 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
0c36e7a02175   centos    "/bin/bash -c 'while…"   5 seconds ago   Up 4 seconds             trusting_aryabhata
[chenkai@centos7 ~]$ docker logs -f -t --tail 10 0c36
2020-12-13T13:32:52.898576651Z kaichenkai
2020-12-13T13:32:53.900576323Z kaichenkai
2020-12-13T13:32:54.902584598Z kaichenkai
2020-12-13T13:32:55.905237048Z kaichenkai
2020-12-13T13:32:56.907263964Z kaichenkai
2020-12-13T13:32:57.909457535Z kaichenkai
2020-12-13T13:32:58.911857015Z kaichenkai
2020-12-13T13:32:59.913973956Z kaichenkai
```

##### 查看容器中的进程信息

```shell
# 命令: docker top 容器ID

# demo
[chenkai@centos7 Desktop]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
0c36e7a02175   centos    "/bin/bash -c 'while…"   20 minutes ago   Up 20 minutes             trusting_aryabhata
[chenkai@centos7 Desktop]$ docker top 0c36
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                5910                5889                0                   08:32               ?                   00:00:00            /bin/bash -c while true; do echo kaichenkai; sleep 1; done
root                9461                5910                0                   08:53               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
```

##### 从容器内拷贝文件到宿主机器上

```shell
docker cp 容器id:容器内路径 宿主机器路径

# demo (注意空格)
[chenkai@centos7 ~]$ docker cp fd17070accec:/test/test.py ~/
[chenkai@centos7 ~]$ ll
total 4
drwxr-xr-x. 5 chenkai chenkai 184 Dec 13 08:54 Desktop
drwxr-xr-x. 2 chenkai chenkai   6 Sep  8  2019 Documents
drwxr-xr-x. 2 chenkai chenkai  30 Dec 10 02:06 Downloads
drwxr-xr-x. 2 chenkai chenkai   6 Sep  8  2019 Music
drwxr-xr-x. 3 chenkai chenkai  44 Sep 11  2019 Pictures
drwxr-xr-x. 2 chenkai chenkai   6 Sep  8  2019 Public
drwxr-xr-x. 2 chenkai chenkai   6 Sep  8  2019 Templates
-rw-r--r--. 1 chenkai chenkai  13 Dec 13 09:30 test.py
drwxr-xr-x. 2 chenkai chenkai   6 Sep  8  2019 Videos

# 拷贝是一个手动过程, 需要使用挂载卷的方式来实现
```

##### 从宿主机器拷贝文件到容器中

~~~shell
docker cp 宿主机器路径 容器id：容器内路径
~~~

##### 查看容器状态

```shell
# 查看容器状态 docker stats 容器ID/容器名称
docker stats elasticsearch 或者 docker stats a23331750011

CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT    MEM %     NET I/O       BLOCK I/O    PIDS
a23331750011   elasticsearch   0.25%     349.2MiB / 7.62GiB   4.48%     3.79kB / 0B   0B / 649kB   40
```

<br>

### Docker 部署nginx, tomcat 等服务

> docker 部署 nginx

```shell
# 1. 搜索镜像 search
# 2. 下载镜像 pull
[chenkai@centos7 ~]$ docker pull nginx 
Using default tag: latest
latest: Pulling from library/nginx
6ec7b7d162b2: Already exists 
bbce32568f49: Pull complete 
5928664fb2b3: Pull complete 
a85e904c7548: Pull complete 
ac39958ca6b1: Pull complete 
Digest: sha256:31de7d2fd0e751685e57339d2b4a4aa175aea922e592d36a7078d72db0a45639
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
[chenkai@centos7 ~]$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
redis        5.0       5a3c8e192943   3 days ago   98.4MB
nginx        latest    7baf28ea91eb   3 days ago   133MB
centos       latest    300e315adb2f   7 days ago   209MB

# 3. 运行容器 run
# -d 后台运行守护容器, --name 给容器命名(多个容器分别命名), -p 端口映射 宿主端口:容器内部端口
[chenkai@centos7 ~]$ docker run --name nginx_master -d -p 8080:80 nginx
1238ddbea007cb1b486c9bc7a738fbdac027caa6194977675bcd1205cc06b483

# 4. 端口测试! (需要做端口映射才能访问!)
[chenkai@centos7 ~]$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# 进入容器, 使用 whereis nginx 查找 nginx 配置文件
[chenkai@centos7 ~]$ docker exec -it nginx_master /bin/bash
root@1238ddbea007:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
```

<br>

> docker部署tomcat

```shell
# 官方的使用
docker run it --rm -p 8080:8080 tomcat:9.0
# 我们之前都是后台启动, 停止了容器之后, 容器还是可以查看到, --rm 参数一般用来测试, 表示用完即删除容器

# 使用下载镜像再启动的方式
docker pull tomcat:9.0

# 启动容器
[chenkai@centos7 ~]$ docker run --name tomcat_master -d -p 8080:8080 tomcat:9.0

# 访问测试(404页面)
[chenkai@centos7 ~]$ curl localhost:8080
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/9.0.41</h3></body></html>

# 进入容器
[chenkai@centos7 ~]$ docker exec -it tomcat_master /bin/bash
root@03e462c2c1c6:/usr/local/tomcat# ll
bash: ll: command not found
root@03e462c2c1c6:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@03e462c2c1c6:/usr/local/tomcat# cd webapps
root@03e462c2c1c6:/usr/local/tomcat/webapps# ls
root@03e462c2c1c6:/usr/local/tomcat/webapps#
# 发现问题:1. linux 命令少了, 2.没有webapps应用, 阿里云镜像的原因, 默认是最小的镜像, 所有不必要的都剔除掉, 保证最小可运行的环境

# 将 webapps.dist 的欢迎页面内容cp到webapps
# root@03e462c2c1c6:/usr/local/tomcat# cp -r  webapps.dist/* webapps
# 再次测试访问 curl localhost:8080, 出现欢迎页面, (不会出现404了)
```

思考问题: 每次要部署项目, 如果每次都要进入容器是不是特别麻烦 ? 要是在容器外面提供一个映射目录 webapps, 我们在外面放置好目录, 就可以自动同步到容器内部了.

<br>

> docker 部署 ES + Kibana

```shell
# es 暴露的端口很多
# es 十分的耗内存
# es 的数据一般需要放置到安全目录 ~ 挂载卷
# 官方指导文档地址: https://hub.docker.com/_/elasticsearch

# 官方运行容器命令 
$ docker network create somenetwork
# --net somenetwork ? 网络配置 ?
$ docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag
# tag: 版本号, 使用 6.6.0

# 运行容器, ES 默认以 1g 内存启动, 如果阿里云服务器内存很小,则会很卡
[chenkai@centos7 ~]$ docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:6.6.0
Unable to find image 'elasticsearch:6.6.0' locally
6.6.0: Pulling from library/elasticsearch
a02a4930cb5d: Pull complete 
7decffa70b3b: Pull complete 
498315b62345: Pull complete 
c84790c95eb9: Pull complete 
6a3b0a2308b2: Pull complete 
33b21a7b5e00: Pull complete 
f620bc30e187: Pull complete 
Digest: sha256:ed1f27b9a16dc29d19fc607a1e6e281d1ddf83d81734427d895f5b91c23a6ee5
Status: Downloaded newer image for elasticsearch:6.6.0
c6b9c32be5a860b514fe7ba5f5d559e090e546af753054d1c48980d28c4c444e

# 访问测试
[chenkai@centos7 ~]$ curl localhost:9200
{
  "name" : "7-em8I4",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "haoHQGn9SsGeoCEASSO7NA",
  "version" : {
    "number" : "6.6.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "a9861f4",
    "build_date" : "2019-01-24T11:27:09.439740Z",
    "build_snapshot" : false,
    "lucene_version" : "7.6.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}

# 运行ES容器时, 对内存进行限制(最小64m, 最大512m), 使用 -e 参数
-e ES_JAVA_OPTS="-Xms64m -Xmx512m"
# 指定内存运行容器
[chenkai@centos7 ~]$ docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m"  elasticsearch:6.6.0
a23331750011eac317d6a263bac8e14cdc05cfafe1f03a6884b0ef0ae652a8d5

# 查看容器状态 docker stats 容器ID/容器名称
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT    MEM %     NET I/O       BLOCK I/O    PIDS
a23331750011   elasticsearch   0.25%     349.2MiB / 7.62GiB   4.48%     3.79kB / 0B   0B / 649kB   40
```

这里还缺少 kibana6.6.0的安装部署内容

<br>

### 如何提交一个自己的镜像

```shell
# 命令与 git 原理类似
docker commit -m="提交的描述信息" -a="作者" 容器ID 目标镜像名:TAG

# 测试, 以上面tomcat的容器为例, 把 webapps.dist 的欢迎页面内容cp到webapps, 然后提交新的镜像
[chenkai@centos7 ~]$ docker commit -m="有欢迎页面的tomcat容器" -a="chenkai" 03e462c2c1c6 my_tomcat:2.0
sha256:f4fca90d1212033367cab555ab66d04b06e7bed850e7b0faada384123609f9b8

# 查看自己提交的镜像
[chenkai@centos7 ~]$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
my_tomcat       2.0       f4fca90d1212   38 seconds ago
```

将操作过的容器通过 commit 提交为一个镜像, 以后就可以直接使用修改过的镜像了

> 将容器提交为镜像, 就好比在 VM 中生成一个当前的快照

<br>
<br>
<br>



完 ~ 