---
title: Docker 补救指南（三）—— Docker Swarm
date: 2019-05-18
tags:
- docker
categories:
- docker
thumbnail: http://img.yuzh.xyz/docker-note/wallhaven-263173.jpg
toc: true
---

# 是什么
Docker Swarm 是一个创建和管理 Docker 集群的工具，该工具主要用于 Docker 的集群管理和容器编排。其特点有下：

- 方便创建和管理集群
- 可拓展
- 可实现期望的状态调节
- 集群中多主机网络自动拓展管理
- 提供服务发现功能
- 可实现负载均衡
- 安全性强
- 支持延迟更新和服务回滚

> Docker Swarm 是 Docker 官方在 Docker1.12 后自带支持的新型容器集群管理工具。在此之前比较成熟的容器集群管理工具有 Google 的 Kubernetes 和 Apache 的分布式管理框架 Mesos 等。


**[核心概念：节点、服务、任务的描述](https://yeasy.gitbooks.io/docker_practice/content/swarm_mode/overview.html)**

# 搭建 Swarm 集群

**环境准备**

- 三台装有 Docker 的主机，版本要求是 1.12 以上。

- 集群管理节点的 IP 必须固定，保证其他工作节点能够访问该管理节点。

- 集群节点之间必须使用相同的协议并保证以下端口可用：

  - 用于集群管理通信的 TCP 接口 **2377**
  - 用于节点间通信的 TCP/UDP 端口 **7946**
  - 用于覆盖网络流量的 UDP 端口 **4789**

由于主机有限，这里使用本地 Mac 主机和云端 ECS 主机，出口 IP 分别为：

```
本地（worker）：113.246.94.216
远程（manager）：120.78.69.82
```

**常用命令：**
```
Usage:  docker swarm COMMAND

Manage Swarm

Commands:
  ca          Display and rotate the root CA
  init        Initialize a swarm
  join        Join a swarm as a node and/or manager
  join-token  Manage join tokens
  leave       Leave the swarm
  unlock      Unlock swarm
  unlock-key  Manage the unlock key
  update      Update the swarm

Run 'docker swarm COMMAND --help' for more information on a command.
```

## 创建 Docker Swarm 集群

在 manager 创建一个集群：**docker swarm init --advertise-addr 120.78.69.82**

![](http://img.yuzh.xyz/docker-note/20190518111019.png)

集群创建成功，当前节点默认成为了管理节点。查看集群中的所有节点：**docker node ls**

![当前只有一个管理节点](http://img.yuzh.xyz/docker-note/20190518111234.png)

接下来，让工作节点加入 Swarm 集群，使用：**docker swarm join**
```
[root@izwz90qcppq4b8dajq7j3uz ~]# docker swarm join --help

Usage:  docker swarm join [OPTIONS] HOST:PORT

Join a swarm as a node and/or manager

Options:
      --advertise-addr string   Advertised address (format: <ip|interface>[:port])
      --availability string     Availability of the node ("active"|"pause"|"drain") (default "active")
      --data-path-addr string   Address or interface to use for data path traffic (format: <ip|interface>)
      --listen-addr node-addr   Listen address (format: <ip|interface>[:port]) (default 0.0.0.0:2377)
      --token string            Token for entry into the swarm
```

_有很多种方式加入集群，这里使用创建集群成功之后的提示命令就可以了。如果忘记了加入集群的指令，可以使用 **docker swarm join-tokens [worker / manager] 查看加入集群的 token 信息：**_

![](http://img.yuzh.xyz/docker-note/20190518112303.png)

工作节点加入集群，在工作节点终端执行：

![](http://img.yuzh.xyz/docker-note/20190518112434.png)

此时在工作节点上可以看到该集群中有两个节点，注意：工作节点不能执行 docker swarm 的其他命令，只能执行 **docker swarm leave** 离开此集群。

![](http://img.yuzh.xyz/docker-note/20190518112710.png)

## 向 Docker Swarm 部署服务

使用 **docker service** 来管理 Swarm 集群中的服务，此命令只能在管理节点执行。

```
[root@izwz90qcppq4b8dajq7j3uz ~]# docker service --help

Usage:  docker service COMMAND

Manage services

Commands:
  create      Create a new service
  inspect     Display detailed information on one or more services
  logs        Fetch the logs of a service or task
  ls          List services
  ps          List the tasks of one or more services
  rm          Remove one or more services
  rollback    Revert changes to a service's configuration
  scale       Scale one or multiple replicated services
  update      Update a service

Run 'docker service COMMAND --help' for more information on a command.
```

在 Docker Swarm 集群中运行一个 nginx 的服务：**docker service create --replicas 1 -p 8086:80 --name swarm-nginx nginx**

- `--replicas 1` 指定该服务的副本数量
- `--name` 与创建容器一样
- `-p` 与创建容器一样

```
PS: Docker 服务管理的命令与容器管理类似，只不过一个是 docker service，一个是 docker contianer。但有个别服务的特例命令如：--replicas
```

接下来可以查看集群中的服务列表：**docker service ls**

![](http://img.yuzh.xyz/docker-note/20190518114355.png)

查看服务在集群上的分配和运行情况：**docker service ps swarm-nginx**

![](http://img.yuzh.xyz/docker-note/20190518115036.png)

> 由前面知道：一个服务是多个任务的集合，任务是 Swarm 中的最小调度单元，一个任务可以看作是单个容器。
> 我们运行一个服务之后，Swarm 分配任务到不同的副本执行，在不同副本中的体现可以查看容器运行情况。

任务被分配到了工作节点，通过查看容器进程可以看到：

![](http://img.yuzh.xyz/docker-note/20190518115639.png)

在集群中部署的服务，如果只运行一个副本，就无法体现集群的优势。一旦该机器或副本奔溃，该服务就无法访问，所以通常一个服务会创建多个副本。

添加副本的命令是：**docker service scale 服务名=数量**

![](http://img.yuzh.xyz/docker-note/20190518120449.png)

添加副本之后再查看服务运行情况：

![](http://img.yuzh.xyz/docker-note/20190518120559.png)

```
PS：Docker Swarm 部署服务下的容器，默认网络方式都是使用 overlay 驱动的 ingress 网络，一般情况下我们会自定义一个基于 overlay 覆盖网络驱动的网络。使用方式与创建容器一样，都是指定 `--network 网络名`
```

最终，两个节点都运行着我们的 nginx 服务，打开浏览器分别访问：

![](http://img.yuzh.xyz/docker-note/20190518165502.png)

故意 down 掉一个容器（本地容器），查看是否动态分配了节点？

![](http://img.yuzh.xyz/docker-note/20190518220055.png)

down 掉一个容器之后，过不了多久又开启了一个容器。Docker Swarm 始终保持 swarm-nginx 服务有两个节点运行。

**删除服务：docker service rm 服务名**

## 总结

集群管理命令：

- **docker swarm init --advertise-addr [IP]**
- **docker node ls**
- **docker swarm join**
- **docker swarm join-tokens [worker / manager]**
- **docker swarm leave**

集群服务命令：

- **docker service create -p [port:port] --replicas n [镜像名:TAG/ID]**
- **docker service ls [服务名/ID]**
- **docker service inspect [服务名/ID]**
- **docker service ps [服务名/ID]**
- **docker service scale=[n] [服务名/ID]**
- **docker service rm [服务名/ID]**
