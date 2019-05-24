---
title: Docker 补救指南（二）—— 网络管理
date: 2019-05-12
tags:
- docker
categories:
- docker
thumbnail: http://img.yuzh.xyz/docker-note/65.jpg
toc: true
---

# 网络管理

Docker 默认使用 **bridge**（单主机互联）和 **overlay**（可跨主机互联）两种**网络驱动**来进行容器的网络管理。如果需要，用户可以自定义网络驱动插件进行 Docker 的网络管理。以下内容针对 Docker 默认的网络管理和自定义网络管理进行演示学习。

<!-- more -->
## 查看指令

不管其他细节，先看有哪些常用命令：

```sh
Usage:	docker network COMMAND
Manage networks
Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks
Run 'docker network COMMAND --help' for more information on a command.
```

可以看到对于网络的操作大概都有这些操作：`连接、创建、取消连接、查看细节、查看列表、移除。`

## Docker 默认网络管理

Docker 安装时会自动创建三种网络。查看所有网络指令：**docker netwok ls**

![默认的三种网络](http://img.yuzh.xyz/docker-note/20190512220300.png)

可以看到三种网络分别是 bridge、host、none。bridge网络是默认的 bridge 驱动网络，也是容器创建时默认的网络管理方式，配置后可以与宿主机通信从而实现与互联网通信的功能。host、和none属于无线网络，容器添加到这两个网络不能与外界网路通信。

*「演示默认的网络管理方式」*

1）启动一个以默认网络管理的容器

docker run -it --name=network-test centos

2）使用网络指令查看网络详情

**docker network inspect bridge**

![bridge默认网络详情](http://img.yuzh.xyz/docker-note/20190512220850.png)

执行命令后可以看到 birdge 网络的详细信息，其中包括了刚刚以默认的 bridge 网络管理方式启动的 network-test 容器。

```
PS：这里的三种网络方式 bridge、host、none都是非集群环境下 Docker 提供的默认网络。而在 Docker Swarm 集群环境下，除了这三种方式外，Docker 还提供了 docker_gwbridge 和 ingress 两种默认网络。
```

## 自定义网络介绍

在 Docker 中，可以自定义 bridge 网络（桥接网络）、overlay 网络（覆盖网络），也可以创建 network plugin或者远程网络以实现容器网络的完全定制和控制，以下是简要介绍。

**1）Bridge Network**

为了容器的安全性，可以使用基于 bridge 的驱动创建新的 bridge 网络，这种基于 bridge 驱动的自定义网络可以较好的实现容器隔离。

需要注意的是，这种用户自定义的基于 bridge 驱动的网络对于单主机的小型网络管理环境是个不错的选择，但是对于大型的网络环境管理（如集群）就需要考虑使用自定义的 overlay 集群网络。

**2）Overlay network in swarm mode**

在 Docker Swram 集群环境下可以创建基于 Overlay 驱动的自定义网络。为了保证安全性，Swarm集群使自定义的 overlay 网络只适用于需要服务的集群中的节点，而不会对外部其他服务或者 Docker 主机开放。

*mark*

**3）Custome network plugins**

了解即可

## 自定义 bridge 网络并启动容器

创建网络：**docker network create --driver 驱动名 网络名**

- `--driver`：可省略为 -d，用于指定网络驱动类型。

创建一个基于 bridge 驱动的名为 isolated_nw 的网络：
```sh
docker network create --driver bridge isolated_nw
```

查看是否创建成功：**docker network ls**

![新建网络](http://img.yuzh.xyz/docker-note/20190512223500.png)

查看网络详情：**docker network inspect 网络ID**

自定义网络创建成功之后，使用该网络启动一个容器：**docker run -d --network=isolated_nw centos**

查看容器网络细节：docker inspect bb68248d7f847b7302

![可以看到容器已连接网络](http://img.yuzh.xyz/docker-note/20190512224730.png)

**给正在运行的容器新增网络：docker network connect 网络名 容器ID**
给刚刚以 isolated_nw 连接的容器再连接一个默认网络：

```sh
docker network connect birdge bb68248d7f847b7302
```

此时 centos 容器已连接到两个网络环境：

![](http://img.yuzh.xyz/docker-note/20190512225816.png)

**断开容器网络连接：docker disconnect 网络名 容器ID**

**删除自定义网络：docker network rm 网络名**

*PS：删除之前，需要断开所有与该网络连接的容器*

## 容器之间的网络通信

步骤：

1. 创建两个容器，均连接至默认网络
2. 创建第三个容器，连接新建网络：isolated_nw
3. 给第二个容器增加一个网络环境：isolated_nw
4. 测试容器1能否 ping 通容器2、3
5. 测试容器2能否 ping 通容器1、3
6. 测试容器3能否 ping 通容器1、2

```sh
# docker run -itd --name=container1 centos
# docker run -itd --name=container2 centos
# docker run -itd --name=container3 --network=isolated_nw centos
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
803165d19df0        centos              "/bin/bash"         3 seconds ago        Up 2 seconds                            container3
fc4210dba15c        centos              "/bin/bash"         About a minute ago   Up About a minute                       container1
d465908beb23        centos              "/bin/bash"         About a minute ago   Up About a minute                       container2
------------------------------------------------------------
# docker network connect isolated_nw container2
```

**分别查看三个容器详情：**

container1
![container1](http://img.yuzh.xyz/docker-note/20190512232848.png)

container2
![container2](http://img.yuzh.xyz/docker-note/20190512232935.png)

container3
![container3](http://img.yuzh.xyz/docker-note/20190512232956.png)

**分别进入三个容器查看IP：**

container1
![172.17.0.3](http://img.yuzh.xyz/docker-note/20190512233416.png)

container2
![172.17.0.2 和 172.18.0.3](http://img.yuzh.xyz/docker-note/20190512233457.png)

container3
![172.18.0.2](http://img.yuzh.xyz/docker-note/20190512233544.png)

可以看到，容器2由于连接了两个网络内部有两个网卡，ip分别对应为 172.17.0.2 和 172.18.0.3

**测试连通性**

container1 ping container2、container3:
![](http://img.yuzh.xyz/docker-note/20190512234638.png)

container2 ping container1、container3:
![](http://img.yuzh.xyz/docker-note/20190512235157.png)

container3 ping container1、container2:
![](http://img.yuzh.xyz/docker-note.20190512235503.png)

通过上面的测试得出结论：

1. 不同的容器之间想要互相通信必须在同一个网络环境下。
2. 使用默认的 bridge 网络的容器可以互相之间使用容器IP通信，但是无法通过容器名称互相通信。
3. 使用自定义网络管理的容器可以同时使用容器IP和容器名称进行互联。

> TIPS：
> 使用命令行参数 --link 通过容器名称进行通信
> 如：**docker run -it --name=container4 --link container1:c1 centos**
> 以上命令意思为与容器1通信并给容器1分配一个别名c1
>
> 不建议使用 --link 参数，在未来可能会废弃此参数。但是对于默认网络连接方式的容器之间只能通过 --link 指定容器名连接到其他容器。虽然默认网络方式下可以通过容器IP连接，但是容器IP是可变的。
> **最好的方式是自定义网络连接，既可以通过容器IP、容器名互联，又保证了安全性。**
