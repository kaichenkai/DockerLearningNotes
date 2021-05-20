## DockerFile

### 初识DockerFile

> dockerfile 就是用来构建docker镜像的构建文件
>
> 通过这个脚本可以生成镜像, 镜像一层一层的, 脚本是一个一个的命令, 每个命令都是镜像的一层

### 构建步骤

1. 编写一个 dockerfile 文件
2. docker build 构建成为一个镜像
3. docker run 运行镜像
4. docker push 发布镜像 (DockerHub, 阿里云镜像仓库)

```shell
# 创建一个dockerfile文件, 名字可以随机, 建议 dockerfile
# 文件中的指令要大写

# demo
FROM centos

VOLUME ["volume01", "volume02"]  # 匿名挂载

CMD echo "------end------"

CMD /bin/bash

# 构建命令, 注意最后有个点，默认使用 “上下文目录（Context）下的名为Dockerfile 的文件作为 Dockerfile”，
docker build -f dockerfile路径 -t 镜像名称:TAG .

# 测试
[chenkai@centos7 dockerfile_dir]$ docker build -f ./Dockerfile -t test/centos:2.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 300e315adb2f
Step 2/4 : VOLUME ["volume01", "volume02"]
 ---> Running in ca8ba1aeb2bc
Removing intermediate container ca8ba1aeb2bc
 ---> b3f52bdac5b8
Step 3/4 : CMD echo "------end------"
 ---> Running in bf8986da03f0
Removing intermediate container bf8986da03f0
 ---> 8adaec6b8657
Step 4/4 : CMD /bin/bash
 ---> Running in 5b48aa95a3ab
Removing intermediate container 5b48aa95a3ab
 ---> a9b22603d085
Successfully built a9b22603d085
Successfully tagged test/centos:2.0

# 如果dockerfile文件的命名是 dockerfile 或 Dockerfile, 则可以简写成(否则需要指定文件):
[chenkai@centos7 dockerfile_dir]$ docker build -t test/centos:2.0 .
```

<br>

### 构建过程

> 很多官方镜像都是基础包, 很多功能没有, 通常会自己搭建自己的镜像
>
> 基础指令要求: 

1. 每个保留关键字都需要大写
2. 命令从上到下顺序依次执行
3. \# 表示注释 (和python一样)
4. 每一个指令都会创建提交一个新的镜像层, 并提交

**dockerfile是面向开发的, 我们以后要发布项目, 做镜像, 就需要编写dockerfile文件, 这个文件十分简单**

**docker镜像逐渐成为企业交付的标准, 必须要掌握!**

<br>

### Docker指令说明

```shell
FROM				# 基础镜像, 一切从这里开始
MAINTAINER			# 镜像是谁写的, 姓名+邮箱
RUN					# 镜像构建的时候需要运行的命令
ADD					# 步骤, tomcat镜像, 这个tomcat压缩包,添加内容
WORKDIR				# 镜像的工作目录
VOLUME				# 挂载的目录
EXPOSE				# 保留端口配置
CMD					# 指定这个容器启动的时候要运行的命令, 只有最后一个会生效, 可被替代
ENTRYPOINT			# 指定这个容器启动的时候要运行的命令, 可以追加命令
ONBUILD				# 当构建一个被继承 DockerFile,这个时候就会运行 ONBUILD 的指令, 触发指令
COPY				# 类似ADD, 将我们文件拷贝到镜像中
ENV					# 构建的时候设置环境变量
```

![Docker指令说明](/Users/kai/Documents/blog/Docker/assess/1629331-20201216233445858-1291138099.png)

<br>

### 构建自己的centos

docker hup 中 99% 的镜像都是从 scratch 这个基础镜像过来的, 然后配置需要的软件和配置来进行构建

![构建自己的centos](/Users/kai/Documents/blog/Docker/assess/1629331-20201216233621175-2033352804.png)


##### 编写 dockerfile, 从centos基础模板开始

```shell
# 编写dockerfile文件内容
[chenkai@centos7 mycentos]$ cat dockerfile 
FROM centos
MAINTAINER chenkai<940799064@qq.com>

ENV MYPATH /usr/local
# 工作目录
WORKDIR $MYPATH

# 安装 vim 
RUN yum -y install vim
# ifconfig 命令， 查看ip地址
RUN yum -y install net-tools

# 默认暴露80端口
EXPOSE 80

CMD echo $MYPATH
CMD echo "------end------"
# 默认进入命令行
CMD /bin/bash

# 构建镜像
[chenkai@centos7 mycentos]$ docker build -t mycentos:1.0 .

# 查看镜像
[chenkai@centos7 mycentos]$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
mycentos        1.0       1df495968aec   7 seconds ago   291MB

# 运行容器(默认路径是工作目录 /usr/local)
[chenkai@centos7 mycentos]$ docker run -it mycentos:1.0 /bin/bash
[root@c99c5d83c8e8 local]# 

# 自己构建的centos镜像可以使用 vim, ifconfig 命令

# 使用 docker history 镜像名称/镜像ID  查看镜像构建过程详情, 使用这个命令可以研究一下官方的镜像是怎么做成的
[chenkai@centos7 mycentos]$ docker history mycentos:1.0 
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
25e31305256a   6 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B        
147b72f6517a   6 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
6e3a1e31a623   6 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
ab5431abce53   6 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
7b8d2ddb6b7f   6 minutes ago   /bin/sh -c yum -y install net-tools             23.4MB    
797cc9cec56e   6 minutes ago   /bin/sh -c yum -y install vim                   58.1MB    
52c06240259a   7 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B        
56c71b39e15c   7 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B        
6c41213f80e1   7 minutes ago   /bin/sh -c #(nop)  MAINTAINER chenkai<940799…   0B        
300e315adb2f   8 days ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      8 days ago      /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      8 days ago      /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   209MB 
```

 <br>

### RUN 和 ENTRYPOINT 的区别

```shell
CMD			# 指定这个容器启动的时候要运行的命令, 只有最后一个会生效, 可被代替
ENRTYPOINT 	# 指定这个容器启动的时候要运行的命令, 可以追加命令
```

##### 测试 CMD

```shell
# 编写dockerfile
[chenkai@centos7 testcmd]$ cat dockerfile 
FROM centos
CMD ["ls", "-a"]

# 生成镜像
[chenkai@centos7 testcmd]$ docker build -t testcmd:1.0 .

# 运行容器, 会执行 ls -a 命令
[chenkai@centos7 testcmd]$ docker run -it testcmd:1.0
.   .dockerenv	dev  home  lib64       media  opt   root  sbin	sys  usr
..  bin		etc  lib   lost+found  mnt    proc  run   srv	tmp  var

# 想在 ls -a 追加一个 -l 参数时, 报错了, 原因是因为 -l 替换了原来的 ls -a ,并没有追加
[chenkai@centos7 testcmd]$ docker run -it testcmd:1.0 l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:370: starting container process caused: exec: "l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled 
```

##### 测试 ENRTYPOINT

```shell
# 编写dockerfile
[chenkai@centos7 testentrypoint]$ cat dockerfile 
FROM centos
ENTRYPOINT ["ls", "-a"]

# 生成镜像
[chenkai@centos7 testentrypoint]$ docker build -t testentrypoint:1.0 .

# 运行容器
[chenkai@centos7 testentrypoint]$ docker run -it testentrypoint:1.0
.   .dockerenv	dev  home  lib64       media  opt   root  sbin	sys  usr
..  bin		etc  lib   lost+found  mnt    proc  run   srv	tmp  var

# 想在 ls -a 追加一个 -l 参数, 可以追加命令
[chenkai@centos7 testentrypoint]$ docker run -it testentrypoint:1.0 -l
total 0
drwxr-xr-x.   1 root root   6 Dec 16 14:30 .
drwxr-xr-x.   1 root root   6 Dec 16 14:30 ..
-rwxr-xr-x.   1 root root   0 Dec 16 14:30 .dockerenv
lrwxrwxrwx.   1 root root   7 Nov  3 15:22 bin -> usr/bin
drwxr-xr-x.   5 root root 360 Dec 16 14:30 dev
drwxr-xr-x.   1 root root  66 Dec 16 14:30 etc
drwxr-xr-x.   2 root root   6 Nov  3 15:22 home
lrwxrwxrwx.   1 root root   7 Nov  3 15:22 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3 15:22 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4 17:37 lost+found
drwxr-xr-x.   2 root root   6 Nov  3 15:22 media
drwxr-xr-x.   2 root root   6 Nov  3 15:22 mnt
drwxr-xr-x.   2 root root   6 Nov  3 15:22 opt
dr-xr-xr-x. 259 root root   0 Dec 16 14:30 proc
dr-xr-x---.   2 root root 162 Dec  4 17:37 root
drwxr-xr-x.  11 root root 163 Dec  4 17:37 run
lrwxrwxrwx.   1 root root   8 Nov  3 15:22 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 Nov  3 15:22 srv
dr-xr-xr-x.  13 root root   0 Dec 15 15:03 sys
drwxrwxrwt.   7 root root 145 Dec  4 17:37 tmp
drwxr-xr-x.  12 root root 144 Dec  4 17:37 usr
drwxr-xr-x.  20 root root 262 Dec  4 17:37 var
```

<br>

### 构建自己的tomcat

##### 步骤

1. 准备镜像文件, tomcat压缩包, jdk的压缩包 (提前准备好了 tomcat 和 jdk 的linux压缩包)

   ```shell
   [chenkai@centos7 mytomcat]$ ll -h
   total 194M
   -rw-rw-r--. 1 chenkai chenkai  11M Dec 16 10:11 apache-tomcat-9.0.41.tar.gz
   -rw-rw-r--. 1 chenkai chenkai 183M Dec 16 10:11 jdk-8u201-linux-x64.tar.gz
   -rw-rw-r--. 1 chenkai chenkai    0 Dec 16 09:40 readme.md
   ```

2. 编写dockerfile 文件, 官方命名: Dockerfile 或 dockerfile, build 时会自动查找, 就不需要 -f 指定dockerfile文件了

   ```shell
   [chenkai@centos7 mytomcat]$ cat dockerfile 
   FROM centos
   MAINTAINER chenkai<940799064@qq.com>
   
   COPY readme.md /usr/local/readme.md
   # 添加压缩包到容器中， 会自动解压
   ADD jdk-8u201-linux-x64.tar.gz /usr/local
   ADD apache-tomcat-9.0.41.tar.gz /usr/local
   
   RUN yum -y install vim
   
   ENV MYPATH /usr/local
   ENV JAVA_HOME /usr/local/jdk-8u201
   # 多个类路径通过 ： 隔开
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   
   # 配置tomcat目录
   ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.41
   ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.41
   ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
   
   WORKDIR $MYPATH
   
   EXPOSE 8080
   
   CMD /usr/local/apache-tomcat-9.0.41/bin/startup.sh && tail -f /usr/local/apache-tomcat-9.0.41/bin/logs/catalina.out
   ```

3. 构建镜像, 查看镜像

   ```shell
   # 构建
   [chenkai@centos7 mytomcat]$ docker build -t mytomcat:1.0 .
   
   # 查看
   [chenkai@centos7 mytomcat]$ docker images
   REPOSITORY       TAG       IMAGE ID       CREATED             SIZE
   mytomcat         1.0       c1c6691a0c79   56 seconds ago      680MB
   ```

4. 运行容器

   ```shell
   # 映射了两个目录, 分别是 webapps 和 logs
   [chenkai@centos7 mytomcat]$ docker run -d -p 9090:8080 --name mytomcat -v /home/chenkai/build/tomcat/test:/usr/local/apache-tomcat-9.0.41/webapps/test -v /home/chenkai/build/tomcat/tomcatlogs:/usr/local/apache-tomcat-9.0.41/logs mytomcat:1.0
   3a37ea169a8db97e576e60eee046f8e1311c01f2156436c0815c7456264dbfe8
   ```

5. 访问测试

   ```shell
   
   ```

   





<br>



### 发布镜像到DockerHub



<br>



### 发布镜像到阿里云仓库



<br>

<br>

完 ~