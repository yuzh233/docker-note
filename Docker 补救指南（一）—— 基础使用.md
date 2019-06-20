---
title: Docker 补救指南（一）—— 基础使用
date: 2019-05-11
tags:
- docker
categories:
- 技术
thumbnail: http://img.yuzh.xyz/docker-note/wallhaven-263173.jpg
toc: true
---

# 配置阿里云镜像加速

阿里云主页搜索 [容器镜像服务](https://help.aliyun.com/product/60716.html?spm=a2c4g.11186623.6.540.10823864peBl6O)，进入[「镜像仓库管理控制台」](https://cr.console.aliyun.com/cn-shanghai/instances/mirrors)，根据说明文档操作即可。

# 学会使用帮助命令

查看版本：

```sh
docker version
```

查看docker安装信息：

```sh
docker info
```

查看命令说明：

```sh
docker --help
```
<!-- more -->
使用方式：`docker COMMAND --help` 如：`docker logs --help`

```sh
Usage:	docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for
                       42 minutes)
      --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m
                       for 42 minutes)
```

# 镜像
## 查看本地镜像

**docker images / docker image ls**

参数：

- -a 列出本地所有的镜像（包含所有中间层镜像）

- -q 只显示镜像ID

- --digests 显示镜像的摘要信息（会多一个GUGEST展示字段，相当于一个镜像的备注信息，有些镜像没有。）

- --no-trunc 显示完整的镜像信息

## 从 Docker Hub 查询镜像

**docker search 镜像名**

如：`docker search tomcat`

```sh
NAME                                       DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
tomcat                                     Apache Tomcat is an open source implementati…   2345                [OK]
tomee                                      Apache TomEE is an all-Apache Java EE certif…   64                  [OK]
dordoka/tomcat                             Ubuntu 14.04, Oracle JDK 8 and Tomcat 8 base…   53                                      [OK]
davidcaste/alpine-tomcat                   Apache Tomcat 7/8 using Oracle Java 7/8 with…   34                                      [OK]
bitnami/tomcat                             Bitnami Tomcat Docker Image                     28                                      [OK]
cloudesire/tomcat                          Tomcat server, 6/7/8                            14                                      [OK]
meirwa/spring-boot-tomcat-mysql-app        a sample spring-boot app using tomcat and My…   12                                      [OK]
tutum/tomcat                               Base docker image to run a Tomcat applicatio…   11
aallam/tomcat-mysql                        Debian, Oracle JDK, Tomcat & MySQL              11                                      [OK]
jeanblanchard/tomcat                       Minimal Docker image with Apache Tomcat         8
arm32v7/tomcat                             Apache Tomcat is an open source implementati…   6
rightctrl/tomcat                           CentOS , Oracle Java, tomcat application ssl…   4                                       [OK]
maluuba/tomcat7-java8                      Tomcat7 with java8.                             3
amd64/tomcat                               Apache Tomcat is an open source implementati…   2
arm64v8/tomcat                             Apache Tomcat is an open source implementati…   2
fabric8/tomcat-8                           Fabric8 Tomcat 8 Image                          2                                       [OK]
camptocamp/tomcat-logback                  Docker image for tomcat with logback integra…   1                                       [OK]
99taxis/tomcat7                            Tomcat7                                         1                                       [OK]
s390x/tomcat                               Apache Tomcat is an open source implementati…   0
picoded/tomcat7                            tomcat7 with jre8 and MANAGER_USER / MANAGER…   0                                       [OK]
oobsri/tomcat8                             Testing CI Jobs with different names.           0
cfje/tomcat-resource                       Tomcat Concourse Resource                       0
1and1internet/debian-9-java-8-tomcat-8.5   Our tomcat 8.5 image                            0                                       [OK]
jelastic/tomcat                            An image of the Tomcat Java application serv…   0
swisstopo/service-print-tomcat             backend tomcat for service-print the true, …   0
```

参数：

- -s [数字]： 列出 star 不小于指定数值的镜像

    ```sh
    # 查询星星数不小于 30 个的 tomcat 镜像
    docker search -s 30 tomcat
    # -s 参数已废弃，可用 --filter 替代
    docker search --filter=stars=30 tomcat
    ```

- --no-trunc： 显示完整的镜像描述信息

- --automated：只列出 automated build 类型的镜像

## 拉取镜像

**docker pull 镜像名:版本号**

不加版本号默认 `:latest`


## 删除本地镜像

**docker rmi 镜像名字或ID**

参数：`-f` 删除中间层镜像

- 删除单个
	```sh
	docker rmi -f 镜像ID
	```

- 删除多个
	```sh
	docker rmi -f 镜像1:TAG 镜像2:TAG
	```

- 删除全部

  	```sh
	# 命令的组合使用，将 $(docker image ls -qa) 的结果作为 前一个命令的参数。 `docker image ls -qa` 的结果就是一串镜像ID数组
    docker rmi -f $(docker image ls -qa)
	```

**有容器正在使用此镜像，不能删除怎么办？**

docker rmi -f XXX

## 删除镜像的高级用法（过滤）

场景：工作和自己学习时产生了大量镜像，为了方便学习。需要将工作中的镜像删除，先看一下列表：

![](http://img.yuzh.xyz/docker-note/20190512162813.png)

我需要把工作中的镜像删除，一个个复制镜像ID删除不免有些麻烦。这个时候就需要抽取共性，合理运用命令了。

1）先过滤掉学习镜像，只展示工作镜像。

```sh
# 仔细观察，发现工作镜像上传的镜像服务地址都是上海的区域，可以以此作为条件过滤一次。
docker image ls | grep 'shanghai'
```
![](http://img.yuzh.xyz/docker-note/20190512163509.png)

2）过滤了不需要删除的镜像，接下来就是提取其中的 image id
作为删除镜像的参数了

```sh
# awk '{print $n}'
处理文本、表格数据的强大linux命令
```

通过这个命令，将列表的镜像ID提取：**docker image ls | grep 'shanghai' | awk '{print $3}'**

3）删除

**docker rmi -f $(docker image ls | grep 'shanghai' | awk '{print $3}')**

![](http://img.yuzh.xyz/docker-note/20190512164001.png)

## 保存镜像

**docker save -o 文件名.tar 镜像ID**

```
Usage:	docker save [OPTIONS] IMAGE [IMAGE...]

Save one or more images to a tar archive (streamed to STDOUT by default)

Options:
  -o, --output string   Write to a file, instead of STDOUT
```

「演示」

将 `izhiliu/centos:v2` 这个镜像保存并重新加载

```sh
# mkdir image && cd images
# docker save -o centos2.tar cf18ff34c415
~/Learning/docker-note/images(master*) » ll                                                                                                    harry@192
total 964608
-rw-------  1 harry  staff   458M May 12 16:47 centos2.tar
```

删除这个镜像：

```sh
docker rmi cf18ff34c415
```

![刚刚删除的镜像没有了](http://img.yuzh.xyz/docker-note/20190512165218.png)


## 加载镜像

**docker load -i 文件.tar**

```
Usage:	docker load [OPTIONS]

Load an image from a tar archive or STDIN

Options:
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output
```

「加载刚刚删除的镜像」

```sh
# docker load -i centos2.tar
6e7dbd1f6260: Loading layer [==================================================>]   55.3kB/55.3kB
Loaded image ID: sha256:cf18ff34c415b8ca05494a3d6faa276a0b1229cffb5563cffbb8f0af533b6f47
```


# 容器

*以 CentOS 容器为例*

## 新建并启动容器

**docker run [options] image [command] [ags...]**

options:

- --name="容器名"：为容器指定一个名称
- -d：后台运行容器，并返回容器ID，也即：**启动守护式容器**
- **-i**：以交互式模式运行容器，通常与 `-t` 同时使用
- **-t**：为容器重新分配一个伪输入终端，通常与 `-i` 同时使用
- -P：随机端口分配
- -p：指定端口映射，有一下四种格式：
	- ip:hostPort:containerPort
	- ip::containerPort
	- hostPort:containerPort
	- containerPort

**简单启动：**

```sh
docker run -it centos
```

**带容器名启动：**

```sh
docker run -it --name="my-centos" centos
```

## 查看当前所有正在运行的容器

**docker ps [options]**

options:

- -a：列出当前所有正在运行的容器 + 历史上运行过的
- -l：显示最近创建的容器
- -n：显示最近 n 个创建的容器
- -q：静默模式，只显示容器编号（不加其他参数默认显示当前运行的所有容器的ID）
- --no-trunc：不截断输出

结合 `-a`/`-q` 参数，可以实现删除所有容器：

```sh
# 删除最近的一个容器
docker rm $(docker ps -lq)
# 强制删除最近的一个容器（若最近的一个容器正在运行）
docker rm -f $(docker ps -lq)
# 删除历史所有的容器，包含正在运行的
docker rm -f $(docker ps -aq)
```

## 退出容器

一）exit：退出并停止

使用 `docker run -it --name="my-centos" centos` 的方式启动容器会以一个交互式终端形式进入到容器内部，当执行 `exit` 时容器就自动停止了。

```sh
# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
037deff7ded9        centos              "/bin/bash"         14 seconds ago      **Exited (0) 5 seconds ago**                       my-centos
```


二）Control + P + Q：退出不停止

重新运行容器，并在重启内执行快捷键之后：

```sh
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
f654ef93ea3f        centos              "/bin/bash"         About a minute ago   **Up About a minute**                       practical_sinoussi
```

**当不停止退出了，如何重新进去呢？** 查看下面的「进入正在运行的容器」章节

## 启动容器

**docker start 容器ID/容器名**

## 重启容器

**docker restart 容器ID/容器名**

## 停止容器

**docker stop 容器ID/容器名**

## 强制停止容器

**docker kill 容器ID/容器名**

## 删除已停止的容器

**docker rm 容器ID**

一次性删除多个容器？前面已经使用了一种方式：`docker rm -f $(docker ps -aq)`

还有一种方式：
```sh
# 利用 linux 的管道符可变参数 xargs，将 | 前面的命令结果作为参数作为 | 后面命令的入参
docker ps -aq | xargs docker rm -f
```

# 容器重要知识点
## 启动守护式容器

**docker run -d 容器名/容器ID**

以守护式方式启动容器后查看容器进程：

```sh
# docker run -d centos
4de40688015eaec0a015f5d408117d938459828e2379dec4268ae88a298a7704
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

发现并没有显示正在运行的容器进程，为啥？再看一下容器历史运行记录试试。。。

```sh
# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
4de40688015e        centos              "/bin/bash"         4 minutes ago       Exited (0) 4 minutes ago                       pedantic_benz
```

发现刚刚以 **守护式方式** 启动的容器，启动之后就立马停止了！

> Docker 容器后台运行，就必须要有一个前台进程。\
> 什么意思呢？如果容器运行的命令不是那些 **一直挂起** 的命令（如：top、tail），容器是会自动退出的。\
> 这是由于 **docker的机制问题**，例如使用 nginx，启动 nginx 服务的命令是：`service nginx start`，这是一个后台命令，在 linux 中会一直在后台运行着。但是在 docker 中，nginx容器为后台模式运行模式，导致 docker 的 nginx 容器前台没有运行的应用。所以容器启动后会立即自动停止，因为这个容器觉得自己没有事情可做！😅\
> 最佳解决方案是：将要运行的程序以前台的方式运行！

那么怎么让容器以**守护式容器启动**，但是避免前台没事可做导致容器自动关闭呢？简单的方式有：**让后台运行的容器前台一直打印日志**，这样就避免了前台没事可做导致容器退出😂。

```sh
# docker run -d centos /bin/bash -c "while true; do echo hello harry!; sleep 1; done"
2b8e4a0014ab7cb435ecc19f171daf027c1291fa24b77ae52132ba9c225962d8
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
2b8e4a0014ab        centos              "/bin/bash -c 'while…"   3 seconds ago       Up 3 seconds                            angry_aryabhata
```

可以看到此时容器正在以后台运行，但是没有关闭。解释一下 `/bin/bash`，

```
如果 -c 选项存在，命令就从字符串中读取。如果字符串后有参数，他们将会被分配到参数的位置上，从$0开始。
```

## 查看容器日志

**docker logs -f -t --tail 容器ID**

optins：

- -t：加入时间戳
- -f：跟随最新日志打印
- --tail 数字：显示最后多少条

查看刚刚以守护容器运行的进程：

```sh
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
04b01b87b990        centos              "/bin/bash -c 'while…"   5 seconds ago       Up 4 seconds                            modest_chaplygin
# docker logs -t -f --tail 10 04b01b87b990 从倒数第 10 条日志开始查看，不停的追加，并且显示日志打印的时间。
2019-04-14T10:56:11.635841663Z hello harry!
2019-04-14T10:56:12.638348360Z hello harry!
2019-04-14T10:56:13.645537151Z hello harry!
2019-04-14T10:56:14.651628446Z hello harry!
2019-04-14T10:56:15.654658791Z hello harry!
2019-04-14T10:56:16.658972279Z hello harry!
2019-04-14T10:56:17.664596508Z hello harry!
2019-04-14T10:56:18.668527236Z hello harry!
2019-04-14T10:56:19.671418530Z hello harry!
2019-04-14T10:56:20.676871204Z hello harry!
2019-04-14T10:56:21.682418501Z hello harry!
2019-04-14T10:56:22.683835449Z hello harry!
2019-04-14T10:56:23.688968112Z hello harry!
```

## 查看容器内运行的进程

**docker top 容器ID**

查看刚刚守护式运行的容器的内部进程：

```sh
# docker top 04b01b87b990
PID                 USER                TIME                COMMAND
8568                root                0:00                /bin/bash -c while true; do echo hello harry!; sleep 1; done
8921                root                0:00                sleep 1
```

## 查看容器内部细节

**docker inspect 容器ID**

## 进入正在运行的容器并以命令行交互

前面学到过 **不关闭容器并退出** 的快捷键是：`contrl + p + q`，那么退出之后怎么重新进入呢？有以下两种方式。

### docker attach 容器ID

「演示」

```sh
# docker rm -f $(docker ps -aq) 先把所有容器杀死
# docker run -it --name='my-centos' centos 以交互式终端启动容器
[root@b7324d8b56b9 /]# pwd
/
[root@b7324d8b56b9 /]# ll
total 56
-rw-r--r--   1 root root 12082 Mar  5 17:36 anaconda-post.log
lrwxrwxrwx   1 root root     7 Mar  5 17:34 bin -> usr/bin
drwxr-xr-x   5 root root   360 Apr 14 11:24 dev
drwxr-xr-x   1 root root  4096 Apr 14 11:24 etc
drwxr-xr-x   2 root root  4096 Apr 11  2018 home
lrwxrwxrwx   1 root root     7 Mar  5 17:34 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Mar  5 17:34 lib64 -> usr/lib64
drwxr-xr-x   2 root root  4096 Apr 11  2018 media
......

# 使用快捷键 control + p + q 不关闭容器并退出
[root@b7324d8b56b9 /]# %

# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
b7324d8b56b9        centos              "/bin/bash"         About a minute ago   Up About a minute                       my-centos

# 可以看到容器依然在运行，执行命令 docker attach b7324d8b56b9 重新进入。
[root@b7324d8b56b9 /]# pwd
/
[root@b7324d8b56b9 /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

### docker exec -it 容器ID bashShell

「演示」

进入容器内部执行命令：
```sh
# 先不关闭退出容器 control + p + q
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b7324d8b56b9        centos              "/bin/bash"         7 minutes ago       Up 7 minutes                            my-centos

# docker exec -it b7324d8b56b9 /bin/bash 一样进入了容器
[root@60711933f7dd /]# pwd
/
[root@60711933f7dd /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 执行 exit 命令退出容器，PS：这里容器会关闭吗？
[root@60711933f7dd /]# exit
exit
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
60711933f7dd        centos              "/bin/bash"         2 minutes ago       Up 2 minutes                            my-centos
# 可以看到，以 exec 方式进入的容器执行 exit 并不会关闭容器。
```

不进入容器，在宿主机执行容器内部命令：
```sh
# docker exec -it 60711933f7dd ls -l /usr
total 68
dr-xr-xr-x  2 root root 16384 Mar  5 17:36 bin
drwxr-xr-x  2 root root  4096 Apr 11  2018 etc
drwxr-xr-x  2 root root  4096 Apr 11  2018 games
drwxr-xr-x  3 root root  4096 Mar  5 17:35 include
dr-xr-xr-x 19 root root  4096 Mar  5 17:36 lib
dr-xr-xr-x 24 root root 16384 Mar  5 17:36 lib64
drwxr-xr-x 11 root root  4096 Mar  5 17:36 libexec
drwxr-xr-x 12 root root  4096 Mar  5 17:34 local
dr-xr-xr-x  2 root root  4096 Mar  5 17:36 sbin
drwxr-xr-x 52 root root  4096 Mar  5 17:36 share
drwxr-xr-x  4 root root  4096 Mar  5 17:34 src
lrwxrwxrwx  1 root root    10 Mar  5 17:34 tmp -> ../var/tmp
harry@192  ~/Learning/docker-note
# 这里以 linux 中的 `ls -l /usr` 命令代替上面的 `/bin/bash` 命令查看了容器 centos 中的 /usr 下的所有文件，并且将返回的结果直接展示到了宿主机上（即：本机MacOS）
```

两者的区别：

- attach：直接进入容器启动命令的终端，不会启动新的进程。
- exec：是在容器中打开新的终端，并且可以启动新的进程。

## 从容器内拷贝文件到主机上

**docker cp 容器ID:容器内路径 宿主机路径**

容器关闭其运行过程中产生的数据文件都会丢失，所以需要把容器中的数据文件拷贝到宿主机上。

```sh
# docker attach 60711933f7dd 进入容器
[root@60711933f7dd /]# ll
total 56
-rw-r--r--   1 root root 12082 Mar  5 17:36 anaconda-post.log
# 看到有一个日志文件，将其复制到宿主机，先宿主机当前文件：
harry@192  ~/Learning/docker-note  ll
total 40
-rw-r--r--@ 1 harry  staff    17K  4 14 19:56 Docker笔记.md
# 可以看到当前宿主句只有一个文件。将容器中的 anaconda-post.log 复制到当前文件夹 ~/Learning/docker-note
# control + p + q 需要先退出容器
# docker cp 60711933f7dd:/anaconda-post.log ./ 复制到当前目录下
# ll 查看当前宿主机文件
total 64
-rw-r--r--@ 1 harry  staff    17K  4 14 20:01 Docker笔记.md
-rw-r--r--  1 harry  staff    12K  3  6 01:36 anaconda-post.log
```

# 镜像原理
## 是什么
> 镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件。它包含运行某个软件所需的所有内容，包括代码、库、环境、环境变量和配置文件。

### UnionFS（联合文件系统）

> unionFS 是一种分层、轻量级并且高性能的文件系统，它支持 **对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一虚拟文件系统下。\
> 联合文件系统是 Docker 镜像的基础。镜像可以通过分层来继承，基于基础镜像（没有父镜像），可以制作各种具体的镜像。

特性：一次同时加载多个文件系统，但从外面看来，只能看到一个文件系统。联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

### Docker 镜像加载原理
### 为什么 Docker 镜像要采用分层的结构
共享资源。如：有多个镜像都是从相同的 base 镜像构建而来，那么宿主机只需要在磁盘上保存一份 base 镜像。同时内存中也祝需要加载一份 base 镜像，就可以为所有的容器服务了。而且镜像的每一层都可以共享。

## 特点
Docker 镜像都是只读的。当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称为「容器层」，容器层之下的都叫做「镜像层」。

## 镜像的 commit 操作
*docker commit 提交容器副本使之成为一个新的镜像*

**docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的镜像名[:标签名]**

「演示」

**1. 从 DockerHub 上下载 tomcat 镜像到本地并成功运行**

```sh
# docker pull tomcat
Using default tag: latest
latest: Pulling from library/tomcat
e79bb959ec00: Pull complete
d4b7902036fe: Pull complete
1b2a72d4e030: Pull complete
de423484a946: Pull complete
ceaac3b844f7: Pull complete
88f01b722a52: Pull complete
c23be56a9ac1: Pull complete
d852ffd6d31f: Pull complete
11775a3d792d: Pull complete
13fdd17462ac: Pull complete
2092995a1e54: Pull complete
Digest: sha256:409501d73062ab508930eab827fcb19d7d3f7e9bbe63bc6d587114c6af4bee12
Status: Downloaded newer image for tomcat:latest
# 以交互式终端并「映射容器端口到宿主机指定端口」运行 tomcat
# docker run -it -p 8888:8080 --name='my-tomcat' tomcat
# 日志太多不展示了，执行「退出不关闭容器 control + p + q」。
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                    NAMES
c492f4bbce3e        tomcat              "catalina.sh run"   About a minute ago   Up About a minute   0.0.0.0:8888->8080/tcp   my-tomcat
# 可以看到容器中的 8080 端口已经映射到了宿主机上的 8888 端口，访问 http://localhost:8888 可以正常访问。
```

PS：运行 tomcat 不以交互式终端 `-it` 也可以，因为其有前台进程运行（日志打印），所以容器不会退出。

```sh
# docker rm -f $(docker ps -aq) 先删除原有容器
# docker run -d -p 8888:8080 --name='my-tomcat' tomcat
96dad0f6272538e8a1c3359a42d0a6493814f1972c8adf86087dbd322d1033e6
# docker logs -f -t --tail 10 96dad0f62725 查看日志：逐条追加，显示日期，从倒数第 10 条开始。
......
org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/tomcat/webapps/manager] has finished in [18] ms
2019-04-14T16:44:25.733314600Z 14-Apr-2019 16:44:25.732 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
2019-04-14T16:44:25.749280600Z 14-Apr-2019 16:44:25.748 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["ajp-nio-8009"]
2019-04-14T16:44:25.754519300Z 14-Apr-2019 16:44:25.754 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in 889 ms
```

使用 `-P` 参数将会随机分配宿主机端口映射给容器。

**2. 故意删除上一步镜像生产 tomcat 容器的文档**

先访问 http://localhost:8888 查看 document 页面：

![tomcat主页](http://img.yuzh.xyz/docker-note/1555261778530.jpg)

点击 Documentation，可以正常看到页面:

![document页面](http://img.yuzh.xyz/docker-note/1555261916030.jpg)

```sh
# 重新进入上面启动的 tomcat 容器
# docker exec -it 96dad0f62725 /bin/bash
root@96dad0f62725:/usr/local/tomcat# pwd
/usr/local/tomcat
# 删除当前 tomcat 应用的 document 文档页面
root@96dad0f62725:/usr/local/tomcat# rm -rf webapps/docs/
root@96dad0f62725:/usr/local/tomcat#
```

刷新文档页面再次查看：

![文档页面已不存在](http://img.yuzh.xyz/docker-note/1555262087054.jpg)

PS：这只是修改了当前容器的 tomcat，并没有 commit 作为一个新容器。所以当当前容器被删除之后修改的内容也随着被删除了，如需要保存修改后的内容，可以 commit 这个容器作为一个新的镜像。

**3. 也即当前的 tomcat 运行实例是一个没有文档内容的容器。以它为模版 commit 一个没有 doc 的 tommcat 新镜像 my-tomcat**

```sh
# docker exec -it 1b498158b4ee7fc71d80d9da9ba30d890fa5cddcffb7d1edbffbbfe21b4d2317 /bin/bash
root@1b498158b4ee:/usr/local/tomcat# pwd
/usr/local/tomcat
root@1b498158b4ee:/usr/local/tomcat# rm -rf webapps/docs/
root@1b498158b4ee:/usr/local/tomcat# exit
exit # 这里只是退出新开的终端进程，实际上没有退出 tomcat 的进程。
# 执行 docker commit -m="del tomcat documentation" -a="harry" 1b498158b izhiliu/tomcat2:v2
# 镜像名规范：组织名/镜像名:TAG
sha256:cf18ff34c415b8ca05494a3d6faa276a0b1229cffb5563cffbb8f0af533b6f47
# 可以看到执行成功之后返回了新的镜像的唯一标识，再查看一下镜像：docker image ls | grep tomcat
izhiliu/tomcat2     v2                  cf18ff34c415        About a minute ago   465MB
tomcat              latest              5a069ba3df4d        41 hours ago         465MB
# 刚刚新建的镜像已经存在了，启动一下看看：docker run -d -p 8080:8080 izhiliu/tomcat2:v2
```

可以看到新的容器已经没有 documentation 页面了。

**4. 启动我们 commit 后的新景象并和原来的对比**

- 官方的 tomcat:latest 有 Documentation 页面
- 自定义的 izhiliu/tomcat:v2 没有 Documentation 页面

# 容器数据卷

> Docker 的理念
> - 将应用与运行的环境打包形成容器运行，运行可以伴随着容器，但是我们对数据的要求是持久化的。
> - 容器之间希望有可能共享数据

Docker 容器运行时产生的数据，如果不通过 `docker commit` 生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了。

之前学到过一个命令「从容器拷贝数据到宿主机」：`docker cp 容器ID:容器路径 宿主机路径`

但是这只能实现将数据手动保存，并不能实现容器之间的数据共享。解决这种需求，可以使用 **docker 的数据卷**

## 作用

一）容器持久化

二）容器间继承 + 数据共享

## 概念

> 卷就是目录或文件，存在于一个或多个容器中，由 Docker 挂载到容器，但不属于联合文件系统，因此可以能够绕过 UnionFS 提供的一些用于持久存储或共享数据的特性。\
> 卷设计的目的就是数据的持久化，完全独立于容器的生命周期，因此 Docker 不会在删除容器时删除其挂载的数据卷。

特点：

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续大没有容器使用它为止

## 使用一：命令方式挂载数据卷
**docker run -it -v /宿主机文件绝对路径:/容器内路径 镜像名**

「演示：查看数据卷挂载是否成功」

```sh
# 以数据卷挂载方式重新启动容器
# -----------------------------------------------------------------
# docker run -it -v ~/Learning/docker-note/host-data-volumes:/container-data-volumes centos
# PS：一定要使用绝对路径，宿主机目录和容器目录若不存在会自动创建

# 查看宿主机文件
# -----------------------------------------------------------------
# pwd
/Users/harry/Learning/docker-note
# ll
total 80
-rw-r--r--@ 1 harry  staff    26K  4 15 21:12 README.md
-rw-r--r--  1 harry  staff    12K  3  6 01:36 anaconda-post.log
drwxr-xr-x  2 harry  staff    64B  4 15 21:11 host-data-volumes # 被新创建的文件夹
drwxr-xr-x  5 harry  staff   160B  4 15 01:14 img

# 查看容器文件
# -----------------------------------------------------------------
# pwd
/
# ll
total 56
-rw-r--r--   1 root root 12082 Mar  5 17:36 anaconda-post.log
lrwxrwxrwx   1 root root     7 Mar  5 17:34 bin -> usr/bin
drwxr-xr-x   2 root root    64 Apr 15 13:11 container-data-volumes # 被创建的文件夹
drwxr-xr-x   5 root root   360 Apr 15 13:11 dev
drwxr-xr-x   1 root root  4096 Apr 15 13:11 etc
......
```

```
使用 「docker inspect 容器ID」 查看容器是否挂载成功
```

![可以看到，数据卷已经成功绑定了。](http://img.yuzh.xyz/docker-note/1555334292716.jpg)


「演示：容器与宿主机之间共享数据」

```sh
# 查看宿主机数据卷绑定文件路径下的所有文件
# pwd
/Users/harry/Learning/docker-note/host-data-volumes
# ll
total 0

# -----------------------------------------------------
# 查看容器中数据卷绑定文件路径下的所有文件
# pwd
/container-data-volumes
# ll
total 0

# -----------------------------------------------------
# 分别在宿主机和容器数据卷绑定的目录中创建不同的文件，观察文件是否同步过去了
# 在宿主机创建文件 touch host.txt && ll
-rw-r--r--  1 harry  staff     0B  4 15 21:25 host.txt

# 在容器中创建文件 touch contianer.txt && ll
-rw-r--r-- 1 root root 0 Apr 15 13:26 contianer.txt # 容器创建的
-rw-r--r-- 1 root root 0 Apr 15 13:25 host.txt # 前面宿主机创建的
# 可以看到，文件创建成功了并且刚刚在宿主机创建的文件已经同步到了容器中。

# 再观察宿主机有没有同步容器的文件过来 ll
-rw-r--r--  1 harry  staff     0B  4 15 21:26 contianer.txt
-rw-r--r--  1 harry  staff     0B  4 15 21:25 host.txt
```

「演示：容器停止后，主机修改后的数据是否会同步到容器中」

```sh
# 停止容器 / 或者删除容器重新建立数据卷（宿主机的目录不变）
# exit

# 宿主机修改文件： echo 'host update' >> host.txt && cat host.txt
host update

# 启动容器，查看文件内容
# docker start 30a574c0dbe1
# docker attach 30a574c0dbe1
# cd container-data-volumes/ && cat host.txt
host update
# 可以看到内容修改成功同步过来，重新创建的容器文件也是一样的会被同步过来。
```

「演示：使用带权限的挂载命令」

**docker run -it -v /宿主机文件绝对路径:/容器内路径:ro 镜像名**

- `:ro` read only

```sh
# 删除刚刚创建的容器
# docker rm -f $(docker ps -aq)
# 带权限的挂载数据卷
# docker run -it -v ~/Learning/docker-note/host-data-volumes/:/container-data-volumes:ro centos
# 依旧查看宿主机文件是否同步到位
# cd container-data-volumes/ && ll
-rw-r--r-- 1 root root  0 Apr 15 13:26 contianer.txt
-rw-r--r-- 1 root root 12 Apr 15 13:32 host.txt

# 重点来了，容器修改共享文件夹（数据卷）的数据
# touch modify.txt
touch: cannot touch 'modify.txt': Read-only file system
```

`:vo` 限制了容器对数据卷的写权限，使得容器只能够读数据。查看 `docker inspect 75b1409b8a9e`:

![](http://img.yuzh.xyz/docker-note/1555336040258.jpg)

## 使用二： DockerFile 挂载数据卷

DockerFile 的概念先不讨论，重点在于怎么使用 DockerFile 实现宿主机与容器之间的目录挂载。

「步骤」

**1）创建 DockerFile 所属目录**

```sh
mkdir dockerfile-volumes && ll
total 88
-rw-r--r--@ 1 harry  staff    29K  4 17 23:46 README.md
-rw-r--r--  1 harry  staff    12K  3  6 01:36 anaconda-post.log
drwxr-xr-x  2 harry  staff    64B  4 17 23:48 dockerfile-volumes # 存放 DockerFile 的文件夹
drwxr-xr-x  4 harry  staff   128B  4 17 23:22 host-data-volumes
drwxr-xr-x  7 harry  staff   224B  4 15 21:47 img
```

**2） 编写 Dockerfile 文件，在 DockerFile 中通过 `VOLUME` 指令给镜像添加「一个或多个」数据卷。**

```
PS: 以命令方式只能挂载一个数据卷
```

cd dockerfile-volumes && touch Dockerfile

添加以下内容
```sh
# Volume Dockerfile 方式挂载数据卷测试
FROM centos
# 挂载命令
VOLUME ["/data-volumes-container1","/data-volumes-container2"]
CMD echo "image is building..."
CMD /bin/bash
```

PS：VOLUME命令指定了分别在容器上挂载两个目录：`/data-volumes-container1` / `/data-volumes-container2`，但是没有指定宿主机目录。这是因为一个镜像会被不同的宿主机运行，但是宿主机路径不是每个执行镜像的宿主机都是相同的。**通过 DockerFile 挂载的数据卷宿主机目录会被默认分配。**


**3）执行构建命令 docker build -f /Dockerfile名（路径） -t 镜像名:TAG .**

- `-f`：指定 Dockerfile 名（路径），如果就在当前目录执行可以不加。
- `-t`：指定镜像名

```sh
# docker build -f ~/Learning/docker-note/dockerfile-volumes/Dockerfile -t yuzh/centos:v1 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 9f38484d220f
Step 2/4 : VOLUME ['/data-volumes-container1','data-volumes-container2'] # 挂载命令
 ---> Running in 0ad66b7ad4f9
Removing intermediate container 0ad66b7ad4f9
 ---> 1fad13e698d5
Step 3/4 : CMD echo "image is building..."
 ---> Running in 2a90e6187e24
Removing intermediate container 2a90e6187e24
 ---> fc3713190c97
Step 4/4 : CMD /bin/bash
 ---> Running in 2a361cb58947
Removing intermediate container 2a361cb58947
 ---> 31f766043abf
Successfully built 31f766043abf
Successfully tagged yuzh/centos:v1

# docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
yuzh/centos          v1                  31f766043abf        33 seconds ago      202MB
izhiliu/tomcat2      v2                  cf18ff34c415        2 days ago          465MB

# docker run -it yuzh/centos:v1 运行构建的镜像查看数据卷是否存在
# ll
drwxr-xr-x   2 root root  4096 Apr 17 16:24 data-volumes-container1
drwxr-xr-x   2 root root  4096 Apr 17 16:24 data-volumes-container2
```

**4）`docker inspect 容器ID` 查看宿主机位置，验证数据共享**

![](http://img.yuzh.xyz/docker-note/WX20190418-003627@2x.png)

## 使用三：使用数据卷容器实现数据共享

实现容器间数据共享的第三种方式，使用命令：

**docker run -it --volumes-from 父容器ID 镜像ID**

「演示」

沿用之前通过 Dockerfile 创建的镜像

**一）继承与被继承容器之间的数据共享：**

```sh
# 启动第一个容器，作为父容器
# docker run -it --name container01 yuzh/centos:v1
# ll 自然而然的可以看到有两个数据卷文件
drwxr-xr-x   2 root root  4096 Apr 18 16:19 data-volumes-container1
drwxr-xr-x   2 root root  4096 Apr 18 16:19 data-volumes-container2
# 进去 data-volumes-container1，创建一个文件
# touch container01.txt && ll
-rw-r--r-- 1 root root 0 Apr 18 16:21 container01.txt

# 不停止退出，创建第二个容器，继承第一个容器作为数据卷容器
# docker run -it --name container02 --volumes-from container01 yuzh/centos:v1
# cd data-volumes-container1 && ll
-rw-r--r-- 1 root root 0 Apr 18 16:21 container01.txt
# 可以看到容器1创建的文件同步过来了，在容器2创建一个文件，观察容器1是否同步：
# touch container01.txt

# docker attach container01
# cd data-volumes-container1 && ll
-rw-r--r-- 1 root root 0 Apr 18 16:27 container01.txt
-rw-r--r-- 1 root root 0 Apr 18 16:27 container02.txt
```

**二）相同父容器的不同子容器之间的数据共享**

```sh
# 创建第三个容器，继承容器1，操作容器3，观察容器1、2是否都有数据
# docker run -it --name container03 --volumes-from container01 yuzh/centos:v1
# cd data-volumes-container1 && ll
-rw-r--r-- 1 root root 0 Apr 18 16:27 container01.txt
-rw-r--r-- 1 root root 0 Apr 18 16:27 container02.txt
# 自然的看到了容器1、2之间共享的数据，在容器3创建文件：
# touch container03.txt

# 容器1肯定是有此文件的，因为容器3是挂载在容器1之上的。
# docker attach container01
# cd data-volumes-container1 && ll
-rw-r--r-- 1 root root 0 Apr 18 16:27 container01.txt
-rw-r--r-- 1 root root 0 Apr 18 16:27 container02.txt
-rw-r--r-- 1 root root 0 Apr 18 16:34 container03.txt

# 问题来了：容器2会有 container03.txt 吗？
# docker attach container02
# cd data-volumes-container1 && ll
-rw-r--r-- 1 root root 0 Apr 18 16:27 container01.txt
-rw-r--r-- 1 root root 0 Apr 18 16:27 container02.txt
-rw-r--r-- 1 root root 0 Apr 18 16:34 container03.txt
# 也是有的
```

**三）删除父容器，两个子容器之间的数据还会共享吗？**

```sh
# docker ps
0bce4751cdf2        yuzh/centos:v1      "/bin/sh -c /bin/bash"   7 minutes ago       Up 7 minutes                            container03
98d3db76ca2c        yuzh/centos:v1      "/bin/sh -c /bin/bash"   16 minutes ago      Up 16 minutes                           container02
fb12544f689b        yuzh/centos:v1      "/bin/sh -c /bin/bash"   21 minutes ago      Up 21 minutes                           container0
# docker rm -f 0bce4751cdf2
0bce4751cdf2

# 进入容器2，修改文件。
# docker attach container02
# touch container02-update.txt && ll
-rw-r--r-- 1 root root 0 Apr 18 16:27 container01.txt
-rw-r--r-- 1 root root 0 Apr 18 16:42 container02-update.txt
-rw-r--r-- 1 root root 0 Apr 18 16:27 container02.txt
-rw-r--r-- 1 root root 0 Apr 18 16:34 container03.txt

# ok，进入容器3，观察是否有 container02-update.txt
# [control + p + q]
# docker exec -it fb12544f689b /bin/bash
# cd data-volumes-container1 && ll
-rw-r--r-- 1 root root 0 Apr 18 16:27 container01.txt
-rw-r--r-- 1 root root 0 Apr 18 16:42 container02-update.txt # 容器2创建的
-rw-r--r-- 1 root root 0 Apr 18 16:27 container02.txt
-rw-r--r-- 1 root root 0 Apr 18 16:34 container03.txt

# 嗯，有点东西～
# 容器3修改文件进去容器2查看，也是一样的会被同步，不重复了。。。
```

**结论：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止。**

# DockerFile

> DockerFile 是用来构建 Docker 镜像的构建文件，是由一系列参数和命令构成的脚本。

**DockerFile 规则**

- 每条保留字指令都必须为大写字母且后面要至少包含一个参数
- 指令按照从上往下，顺序执行。
- 每条指令都会创建一个新的镜像层，并对镜像进行提交。

**DockerFile 执行流程**

1. docker 从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似于 docker commit 的操作提交一个新镜像层
4. docker 再基于刚提交的镜像运行一个新容器
5. 执行 dockerfile 中的下一条指令直到所有指令都执行完成

> 从应用软件的角度来看，DockerFile、Docker 镜像、Docker 容器分别代表软件的三个不同阶段
> - DockerFile 是软件的原材料
> - Docker 镜像时软件的交付品
> - Docker 容器则可以认为是软件的运行态
>
> DockerFile 面向开发，Docker 镜像成为交付标准，Docker 容器设计部署与运维。三者缺一不可，合力充当 Docker 体系的基石。

## DockerFile 指令

**FROM**

基础镜像，当前新镜像是基于哪个镜像的。

**LABLE**

用于为镜像添加原数据

LABLE key1=value1 key2=value2 ...

**MAINTAINER**

镜像维护者的姓名和邮箱地址

已过时，建议使用 LABLE 形式：`LABLE maintainer="harry"`

**RUN**

容器构建时需要执行的命令

**EXPOSE**

当前容器对外暴露的端口号，随机端口映射将会自动映射此端口。

**WORKDIR**

指定在容器创建后，终端登陆进来后默认的工作目录，一个落脚点。

**ENV**

在构建时设置环境变量

```
ENV MY_PATH /usr/mytest
这个环境变量可以在后续的任何 RUN 指令中使用，如同在命令前面指定了环境变量前缀一样。

也可以在其他指令中直接使用这些环境变量，如：
WORKDIR ${MY_PATH}
```

**ADD**

将宿主机下的文件拷贝进镜像且 ADD 命令会自动处理 URL 和解压 tar 压缩包

**COPY**

类似于 ADD，拷贝文件和目录到镜像中。

将从「构建上下文」目录中的文件/目录，复制到新的一层镜像的目录中

```sh
COPY source target       # shell 格式
COPY ['source','target'] # exec 格式
```

**VOLUME**

容器数据卷，用于数据保存和持久化。

**CMD**

指定一个容器「启动」时要运行的命令

```sh
CMD <命令>                            # shell
CMD ['可执行文件','参数1','参数2'...]   # exec
```

DockerFile 中可以有多个 CMD 命令，但只有最后一个生效。CMD 会被 `docker run` 之后的参数替换。

**ENTERPOINT**

指定一个容器启动时要运行的命令，ENTRYPOINT 的目的和 CMD 一样，都是指定容器启动命令和参数。

ENTRYPOINT 与 CMD 的区别在于：**若存在多个 CMD 指令，docker build 时只会以最后一条 CMD 为准，而 ENTRYPOINT 则是追加。**

**ONBUILD**

如果当前镜像是一个父镜像，并且 DockerFile 中存在 ONBUILD 指令，那么子镜像 build 后会「触发」父镜像的 ONBUILD 指令，相当于父镜像执行一些子镜像构建之后的收尾操作。


## EXPOSE 与 -p 的区别

EXPOSE 暴露端口，更容易理解的说法是「声明端口」。声明容器运行时提供的服务端口，注意：这只是一个声明，容器并不会因为此声明就会开启这个端口的服务。使用此命令有两个好处：

- 帮助镜像的使用者理解这个镜像的守护端口，以方便映射配置
- 容器运行时的随机端口 -p 分配会自动映射 EXPOSE 的端口。

## 案例

### Base镜像（scratch）

Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的。

### 自定义镜像 my-centos

目的：练习基础指令

需求：进入 centos 镜像中，会发现：

- 工作目录是 `/` 根目录
- 不支持 `vim` 命令
- 不支持 `ifconfig` 命令

这是因为 centos 镜像是一个最基础的镜像，共用本机系统内核，去除了不是必须的一些软件及命令或功能。我们的需求是自己构建一个 centos 镜像，使其默认进入我们指定的工作目录，并支持以上两个命令。

```sh
# pwd
/Users/harry/Learning/docker-note/docker-file
# touch DockerFile && ll
-rw-r--r--  1 harry  staff     0B  4 24 00:33 DockerFile
```

**「编写 DockerFile」**

```sh
# vim DockerFile
FROM centos  # 指定基础镜像
MAINTAINER harry<yuzh233@gmail.com> # 设置镜像构建者信息

ENV MY_PATH /usr/local # 设置一个环境变量
WORKDIR $MY_PATH # 指定工作目录，使用环境变量

RUN yum -y install vim # 执行命令：安装 vim
RUN yum -y install net-tools # 执行命令：安装 net-tools

EXPOSE 8088 # 暴露端口

CMD echo 'my workdir is $MY_PATH' # 容器启动执行：打印字符串
CMD /bin/bash # 容器启动真正执行的命令
```

**「执行构建」**

**docker build -f 路径 -t 镜像名:TAG .**

```sh
# docker build -f DockerFile -t izhiliu/centos:v1 .
# DockerFile 就在当前路径下，不用指定目录

Sending build context to Docker daemon  2.048kB
# 可以看到，有几行 DockerFile 就构建几层镜像。
Step 1/9 : FROM centos
 ---> 9f38484d220f
Step 2/9 : MAINTAINER harry<yuzh233@gmail.com>
 ---> Running in 4966da40f5e8
Removing intermediate container 4966da40f5e8
 ---> 32eb4f4c77ba
Step 3/9 : ENV MY_PATH /usr/local
 ---> Running in 8b4d372d1404
Removing intermediate container 8b4d372d1404
 ---> b412c79d5a02
Step 4/9 : WORKDIR $MY_PATH
 ---> Running in 42d2d2df5804
Removing intermediate container 42d2d2df5804
 ---> 082bb5098dbf
Step 5/9 : RUN yum -y install vim
# ......开始安装 vim v......
Complete!
Removing intermediate container 3536627f1160
 ---> 58f9bbd23b2e
Step 6/9 : RUN yum -y install net-tools
 ---> Running in 3a6d39924e48
#  ...... 开始安装 net-tools ......
Complete!
Removing intermediate container 3a6d39924e48
 ---> 2065a365b531
Step 7/9 : EXPOSE 8088
 ---> Running in b836df8bcfb6
Removing intermediate container b836df8bcfb6
 ---> 190f68eeb2d8
Step 8/9 : CMD echo 'my workdir is $MY_PATH'
 ---> Running in cadea312695e
Removing intermediate container cadea312695e
 ---> 8d6c248e24c9
Step 9/9 : CMD /bin/bash
 ---> Running in e5ff383a4407
Removing intermediate container e5ff383a4407
 ---> a8a7bed31bd7
# 出现下面两行代表构建成功
Successfully built a8a7bed31bd7
Successfully tagged izhiliu/centos:v1
```

**「验证新镜像」**

```sh
# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
izhiliu/centos       v1                  87ebc03d1d90        32 seconds ago      425MB
# >>>>>>>>>>>>>>>>>>>> 镜像构建成功！

# docker run -it 87ebc03d1d90
[root@1bd54814f9c5 local]# pwd
/usr/local
# >>>>>>>>>>>>>>>>>>>> 根目录指定成功！

# touch file
# vim file -> 有效
# >>>>>>>>>>>>>>>>>>>> vim 安装成功！

# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 12  bytes 968 (968.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
# >>>>>>>>>>>>>>>>>>>> net-tools 安装成功！

# docker history izhiliu/centos:v1
# 执行这条命令可以看到某个镜像的构建过程，可以看到镜像从头到尾构建，显示根据构建时间倒序展示。
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
87ebc03d1d90        14 minutes ago      /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B
ddc2e1121ff8        14 minutes ago      /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B
a1010b933008        14 minutes ago      /bin/sh -c #(nop)  EXPOSE 8088                  0B
670892d19e9f        14 minutes ago      /bin/sh -c yum -y install net-tools             84.1MB
fb03334297ef        14 minutes ago      /bin/sh -c yum -y install vim                   139MB
daceee7c426b        15 minutes ago      /bin/sh -c #(nop) WORKDIR /usr/local            0B
f39af63b22d3        15 minutes ago      /bin/sh -c #(nop)  ENV MY_PATH=/usr/local       0B
ef7f5536cec3        15 minutes ago      /bin/sh -c #(nop)  MAINTAINER harry<yuzh233@…   0B

# docker inspect izhiliu/centos:v1 查看容器信息
......
"Author": "harry<yuzh233@gmail.com>",
......
```

### CMD/ENTERPOINT 镜像案例

**一）DockerFile 中可以有不多个 `CMD` 指令，但只有最后一个生效。并且最后一个 CMD 指令会被 `docker run` 之后的参数替换（如果有）**

「演示」

```sh
# 查看 tomcat 官方 DockerFile，最后一行：
CMD ["catalina.sh", "run"]
# tomcat 容器就是通过这行命令启动起来的，我们试着在启动时指定其他命令：
docker run -it --name cat -p 7070:8080 tomcat ls -l
# 我们用 ls -l 命令替换掉了 ["catalina.sh", "run"]
total 152
-rw-r--r-- 1 root root  19539 Apr 10 14:33 BUILDING.txt
-rw-r--r-- 1 root root   6090 Apr 10 14:33 CONTRIBUTING.md
-rw-r--r-- 1 root root  57092 Apr 10 14:33 LICENSE
-rw-r--r-- 1 root root   1726 Apr 10 14:33 NOTICE
-rw-r--r-- 1 root root   3255 Apr 10 14:33 README.md
-rw-r--r-- 1 root root   7139 Apr 10 14:33 RELEASE-NOTES
-rw-r--r-- 1 root root  16262 Apr 10 14:33 RUNNING.txt
drwxr-xr-x 2 root root   4096 Apr 13 00:10 bin
drwxr-sr-x 2 root root   4096 Apr 10 14:33 conf
drwxr-sr-x 2 root staff  4096 Apr 13 00:10 include
drwxr-xr-x 2 root root   4096 Apr 13 00:09 lib
drwxrwxrwx 2 root root   4096 Apr 10 14:31 logs
drwxr-sr-x 3 root staff  4096 Apr 13 00:10 native-jni-lib
drwxr-xr-x 2 root root   4096 Apr 13 00:09 temp
drwxr-xr-x 7 root root   4096 Apr 10 14:31 webapps
drwxrwxrwx 2 root root   4096 Apr 10 14:31 work
# 可以看到最后一行命令执行成功了，查看容器进程：docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
202be91cd7ec        tomcat              "ls -l"             About a minute ago   Exited (0) About a minute ago                       cat
# tomcat 没有启动起来，原因是我们用 ls -a 命令代替了启动 tomcat 的命令
```

**二）docker run 之后的参数会被当作参数传递给 ENTRYPOINT，形成新的命令组合。**

定制一个查看当前 ip 信息的容器，DockerFile如下：

```sh
# 安装 curl 工具，发起请求到查ip的网站查询返回信息
# curl 命令可以用来执行下载、发送各种 HTTP 请求，指定 HTTP 头部等操作。
FROM centos
RUN yum install -y curl # 执行安装
CMD ["curl","-s","http://ip.cn"]
# 执行构建
docker build -f DockerFile2 -t show-ip .
```

启动容器，会看到 http://ip.cn 的相应信息展示到了控制台，`curl -s xxx` 命令执行成功。但是当我们需要在容器启动时变更启动中的参数时，CMD就不起作用了。

如我们希望在容器运行时给 `curl -s http://ip.cn` 加一个 `-i` 参数用来显示完整的相应头信息，此时会报错说找不到命令，原因是将 -i 当作一个命令去替换掉了 curl -s http://ip.cn 命令，这显然不是我们想看到的。

这种情况可以使用 ENTRYPOINT，只需在 DockerFile 中替换 CMD 指令为 ENTRYPOINT 即可。`ENTRYPOINT ["curl","-s","xxxx"]`，此时我们在容器启动时指定的参数会以参数的形式追加到 curl -s http://ip.cn 后面而不是替换原来命令了。

### 自定义镜像 Tomcat

自定义一个 tomcat 的镜像，使其启动就是运行了一个 tomcat，实际上就是以 centos 为基础镜像安装了 jdk 和 tomcat 的环境，并设置为启动而已。

「步骤」

1. 创建 DockerFile 所在目录，准备镜像所需要的 jdk 和 tomcat 安装包。

```sh
------------------------------------------------------------
~/Learning/docker-note/docker-file/my-tomcat(master*) » pwd && ll                                         harry@192
/Users/harry/Learning/docker-note/docker-file/my-tomcat
total 375368
-rw-r--r--  1 harry  staff     0B  5  2 21:35 Dockerfile
-rwxr-xr-x@ 1 harry  staff   8.5M  6  5  2018 apache-tomcat-7.0.75.tar.gz
-rwxr-xr-x@ 1 harry  staff   175M  6  5  2018 jdk-8u121-linux-x64.tar.gz
------------------------------------------------------------
```

2. 编写 DockerFile

```sh
~/Learning/docker-note/docker-file(master*) » cd my-tomcat                                                harry@192
FROM centos
MAINTAINER harry<yuzh233@gmail.com>

ADD apache-tomcat-7.0.75.tar.gz /usr/local
ADD jdk-8u121-linux-x64.tar.gz /usr/local

RUN yum -y install vim
ENV WORK_PATH /usr/local
WORKDIR $WORK_PATH

# 设置 java 环境目录，此文件夹名可以在本地将 jdk-8u121-linux-x64.tar.gz 得到
ENV JAVA_HOME /usr/local/jdk1.8.0_121
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.75
ENV CATALINA_BASE /usr/local/apache-tomcat-7.0.75

ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

# ENTRYPOINT ["/usr/local/apache-tomcat-7.0.75/bin/startup.sh"]
# CMD ["/usr/local/apache-tomcat-7.0.75/bin/startup.sh","run"]

# tail 命令：查看文件最后指定行，-f 参数不停的读取。
CMD /usr/local/apache-tomcat-7.0.75/bin/startup.sh \
    && tail -f /usr/local/c/logs/catalina.out
```

PS：容器启动命令加上 `tail` 是使其输出日志以免以 -d 后台运行时由于前台进程无事可做自动停止容器。

3. 构建镜像

**docker build -f Dockerfile -t yuzh/lovely-cat .**

```sh
Sending build context to Docker daemon  192.2MB
Step 1/14 : FROM centos
 ---> 9f38484d220f
Step 2/14 : MAINTAINER harry<yuzh233@gmail.com>
 ---> Using cache
 ---> ef7f5536cec3
Step 3/14 : ADD apache-tomcat-7.0.75.tar.gz /usr/local
 ---> ce38ea3cc5bf
Step 4/14 : ADD jdk-8u121-linux-x64.tar.gz /usr/local
 ---> 9bb44d59445f
Step 5/14 : RUN yum -y install vim
 ---> Running in 4e44d91a2a47
# ......
Complete!
Removing intermediate container 4e44d91a2a47
 ---> 5dfdd1d562ee
Step 6/14 : ENV WORK_PATH /usr/local
 ---> Running in 11087250850a
Removing intermediate container 11087250850a
 ---> 81440053e1c9
Step 7/14 : WORKDIR $WORK_PATH
 ---> Running in dbc004146319
Removing intermediate container dbc004146319
 ---> 240cc0a13f09
Step 8/14 : ENV JAVA_HOME /usr/local/jdk1.8.0_121
 ---> Running in 9c1f3a418a1f
Removing intermediate container 9c1f3a418a1f
 ---> 1a27a3df60ab
Step 9/14 : ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
 ---> Running in 0f8cc5d9ea9c
Removing intermediate container 0f8cc5d9ea9c
 ---> 35126d06a7ca
Step 10/14 : ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.75
 ---> Running in 0e015a7542f0
Removing intermediate container 0e015a7542f0
 ---> 22af3e7209e9
Step 11/14 : ENV CATALINA_BASE /usr/local/apache-tomcat-7.0.75
 ---> Running in 0dc6317ab454
Removing intermediate container 0dc6317ab454
 ---> cfd95a85a2d3
Step 12/14 : ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
 ---> Running in b1b9a6a3e859
Removing intermediate container b1b9a6a3e859
 ---> f03c22aef15d
Step 13/14 : EXPOSE 8080
 ---> Running in 61a077e13bde
Removing intermediate container 61a077e13bde
 ---> afc175ba3243
Step 14/14 : ENTRYPOINT ["/usr/local/apache-tomcat-7.0.75/bin/startup.sh"]
 ---> Running in 753701ca8f2b
Removing intermediate container 753701ca8f2b
 ---> 9a7ad3e1deb1
Successfully built 9a7ad3e1deb1
Successfully tagged yuzh/lovely-cat:latest
------------------------------------------------------------
```

4. 运行新建的容器

```sh
~/Learning/docker-note/docker-file/my-tomcat(master*) » docker image ls                                                                harry@192
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
yuzh/lovely-cat      latest              4d2b3d59f957        2 minutes ago       731MB
------------------------------------------------------------
~/Learning/docker-note/docker-file/my-tomcat(master*) » docker run -d -p 8080:8080 4d2b3d59f957                                        harry@192
------------------------------------------------------------
~/Learning/docker-note/docker-file/my-tomcat(master*) » docker ps                                                                      harry@192
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
e3b5a9979497        4d2b3d59f957        "/bin/sh -c '/usr/lo…"   5 minutes ago       Up 5 minutes        0.0.0.0:8080->8080/tcp   sleepy_hopper
------------------------------------------------------------
```

5. 验证

```sh
~/Learning/docker-note/docker-file/my-tomcat(master*) » docker exec -it e3b5a9979497 /bin/bash                                         harry@192
# 验证：工作目录
# [root@e3b5a9979497 local] # pwd
/usr/local
# 验证：vim
# [root@e3b5a9979497 local]# vim apache-tomcat-7.0.75/logs/catalina.out
May 02, 2019 2:21:19 PM org.apache.catalina.startup.VersionLoggerListener log
INFO: Server version:        Apache Tomcat/7.0.75
May 02, 2019 2:21:19 PM org.apache.catalina.startup.VersionLoggerListener log
INFO: Server built:          Jan 18 2017 20:54:42 UTC
May 02, 2019 2:21:19 PM org.apache.catalina.startup.VersionLoggerListener log
INFO: Server number:         7.0.75.0
May 02, 2019 2:21:19 PM org.apache.catalina.startup.VersionLoggerListener log
INFO: OS Name:               Linux
May 02, 2019 2:21:19 PM org.apache.catalina.startup.VersionLoggerListener log
INFO: OS Version:            4.9.125-linuxkit
May 02, 2019 2:21:19 PM org.apache.catalina.startup.VersionLoggerListener log
......
# 验证 jdk
------------------------------------------------------------
~/Learning/docker-note/docker-file/my-tomcat(master*) » docker exec -it e3b5a9979497 java -version                                     harry@192
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
------------------------------------------------------------
```

![](http://img.yuzh.xyz/docker-note/1556807500784.jpg)

6. 发布 Web 应用

挂载数据卷，重新启动容器。

这样做的好处是不用进入容器查看日志，直接查看主机与容器共享的文件。发布 web 应用只需要在本地将 war 包放到共享文件夹，重启容器即可。

```sh
# docker run -it -p 8888:8080 \
# -v ~/Learning/docker-note/docker-file/my-tomcat/webapp:/usr/local/apache-tomcat-7.0.75/webapps \
# -v ~/Learning/docker-note/docker-file/my-tomcat/logs:/usr/local/apache-tomcat-7.0.75/logs \
# 4d2b3d59f957
```

新建 web 项目，将 web 包复制到容器的  webapp 目录下。容器 tomcat 即可。

![](http://img.yuzh.xyz/docker-note/1556809298656.jpg)

## 安装 MySql

[Mysql DockerHub 官方文档](https://hub.docker.com/_/mysql?tab=description)

「总体步骤」

1. docker search --filter=stars=30 mysql
2. docker pull mysql:latest
3. **docker run  -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag**
4. 进入到了 mysql 容器，通过命令行连接到 mysql 客户端：mysql -u root -p
5. ...... 「输入密码」 ......
6. 数据库操作（show databases; use mysql; show tables; select * from user; ......）

**修改默认配置文件：**

Mysql 默认的配置文件在：`/etc/mysql/my.cnf`，我们可以在容器启动时以数据卷的方式挂载自己的配置文件到容器上：

```sh
# 假如 /my/custom  是我们的自定义配置文件
docker run  -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

这将会以 `/etc/mysql/my.cnf` 和 `/my/custom` 组合配置的方式新启动一个容器实例，后者优先级大于前者。

**不修改配置文件，使用命令参数配置 Mysql**

通过命令标志的形式传递给 mysqld，可以使我们更加灵活的配置 mysql 而不需要配置文件。如：修改所有表的编码方式只需在容器启动时添加以下参数

```sh
docker run -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

**存储数据**

有两种方式存储应用程序在 Docker 容器运行中产生的数据。

1. 让 Docker 使用自己的数据卷管理方式存储将数据库文件写入到主机，这种方式最方便快捷，但是缺点是数据文件不方便找到和管理。
2. 自己创建文件夹，使其挂载到 mysql 的数据文件。

```sh
docker run -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

**备份数据**

```sh
docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' &gt; /some/path/on/your/host/all-databases.sql
```

**Docker Compose 启动 Mysql**

最简单的例子：

```sh
# Use root/example as user/password credentials
version: '3.1'
services:
  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: admin
    ports:
      - 3336:3306

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```

**如何在主机客户端连接容器中的 Mysql 服务端**

Compose 中的应用属于自己的网络，外界不能访问。首先需要暴露端口，注意：尽量不要使用  3306，因为主机大多数安装了Mysql，此端口会被占用。

其次，连接命令类似于如下：

```sh
mysql -h localhost -P 3336 --protocol=tcp -u root
```

需要额外指定参数：`--protocol=tcp`，因为我们的 Mysql 在 Docker 里面，socket 将不可用，所以需要通过 TCP 连接它，在 MySQL 命令中设置“-protocol”将改变这一点。

_引用：https://stackoverflow.com/questions/33001750/connect-to-mysql-in-a-docker-container-from-the-host_

## 安装 Redis

[Redis DockerHub 官方文档](https://hub.docker.com/_/redis)

[redis 命令手册](http://doc.redisfans.com/index.html)

1. docker pull redis

2. 启动 redis 服务端：**docker run -d -p 6379:6379 -v ~/Learning/docker-note/redis/data:/data redis redis-server --appendonly yes**
暴露端口，挂载本地文件到 redis 的持久化文件。避免容器删除 key 丢失。

上面只是启动了服务端，容器运行后需要我们进入容器手动启动 redis 客户端进行测试。

```sh
如何找到 redis 命令地址？可以通过查看 DockerHub 上 Redis 的 DockerFile文件得知在：`/usr/local/bin/`。

cd /usr/local/bin/ && ./redis-cli
# exec redis commend ......
```

「测试持久化」

```sh
# 在当前容器设置了三个 key
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"

# 查看宿主机持久化文件：
------------------------------------------------------------
~/Learning/docker-note/redis/data(master*) » pwd                                                                                                                 harry@192
/Users/harry/Learning/docker-note/redis/data
------------------------------------------------------------
~/Learning/docker-note/redis/data(master*) » cat appendonly.aof                                                                                                  harry@192
*2
$6
SELECT
$1
0
*3
......

```

*证明是挂载成功的，然后删除容器，重新启动一个新容器，看数据是否同步过去了。*

![](http://img.yuzh.xyz/docker-note/1556880581267.jpg)

宿主机连接 docker 中的 redis：

![](http://img.yuzh.xyz/docker-note/1556880819448.jpg)

**自定义配置文件**

1. 定制 DockerFile 的形式

    ```sh
    FROM redis
    COPY redis.conf /usr/local/etc/redis/redis.conf
    CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
    ```
2. 命令行参数指定，灵活，推荐。

    ```sh
    docker run -v /myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf redis redis-server /usr/local/etc/redis/redis.conf
    ```

## 练习：Spring Boot 单体应用容器化

使用上面的 `test-dockerfile`  项目，构建一个镜像，通过容器运行并访问。

添加一个最简单的 Dockerfile

```sh
FROM tomcat:latest
ENV WORK_PATH /usr/local/tomcat
ADD /target/test-dockerfile-0.0.1-SNAPSHOT.jar ${WORK_PATH}/webapps/test-dockerfile-0.0.1-SNAPSHOT.jar
EXPOSE 8081
# PS：ENTRYPOINT ["","${WORK_PATH}"] 这种方式是 exec 的形式执行，不会解析环境变量。除非使用 shell 的方式。
ENTRYPOINT java -jar ${WORK_PATH}/webapps/test-dockerfile-0.0.1-SNAPSHOT.jar
```

在项目根路径下执行：`docker build -f Dockerfile -t my-webapp:latest .`

运行：`docker run -it -p 8081:8080 my-webapp:latest`

访问：http://localhost:8081/api/hello-docker

![](http://img.yuzh.xyz/docker-note/1556892499795.jpg)

*PS：IDEA 2019 版集成了 Docker 工具，不过我还是使用命令吧😂。*

![](http://img.yuzh.xyz/docker-note/1556892760857.jpg)

# 仓库

学会使用[「容器镜像服务」](https://help.aliyun.com/product/60716.html?spm=a2c4g.11186623.6.540.10823864peBl6O)，官方有详细文档说明，这里只记录主要步骤及问题。

**1）创建镜像仓库，指定代码源**

![创建阿里云 code 项目](http://img.yuzh.xyz/docker-note/1556888180808.jpg)

![创建镜像仓库](http://img.yuzh.xyz/docker-note/1556888080869.jpg)

**2) 登陆私有镜像仓库**

```sh
docker login --username=yuzh233 registry.cn-hangzhou.aliyuncs.com
```

*PS：修改密码入口：容器服务首页 -> 访问凭证*

**3） 上传镜像到私有仓库**

```sh
docker login --username=yuzh233 registry.cn-hangzhou.aliyuncs.com

# docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/yuzh/test-dockerfile:[镜像版本号]
# ImageId 就是本地构建好需要上传到远程仓库的镜像ID
docker tag fccdeb7abef7  registry.cn-hangzhou.aliyuncs.com/yuzh/test-dockerfile:latest

docker push registry.cn-hangzhou.aliyuncs.com/yuzh/test-dockerfile:[镜像版本号]
```

**4） 从私有仓库拉取镜像**

```sh
docker pull registry.cn-hangzhou.aliyuncs.com/yuzh/test-dockerfile:[镜像版本号]
```

**5）登出远程仓库**

docker logout [仓库地址]

```sh
e.g: docker logout registry.cn-shanghai.aliyuncs.com
```
