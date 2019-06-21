---
title: Docker 补救指南（二）—— Docker Compose
date: 2019-05-23
tags:
- docker
categories:
- 技术
thumbnail: http://img.yuzh.xyz/blog48.jpg
toc: true
---

# What It Is？

> reference：[Overview of Docker Compose](https://docs.docker.com/compose/overview/)

Compose 是一个定义和运行多个容器的 Docker 应用。在 Compose，你可以使用一个 YAML 文件配置你的应用的（多个）服务。然后，使用一条命令你可以创建和启动所有的服务从你的配置中。要学习更多的关于 Compose 的特性，请参看：[特性列表](https://docs.docker.com/compose/overview/#features)
<!-- more -->
Compose 工作在所有环境：生产、开发、测试，以及 CLI 工作流。你可以学习更所的用例在：[Comm Use Cases](https://docs.docker.com/compose/overview/#common-use-cases)

Compose 的使用基本上是以下三个设置的过程：

- 使用 Dockerfile 定义你的应用环境，它可以复制到任何地方。
- 在 docker-compose.yml 中定义组成你的应用的服务，他们可以在隔离的环境下共同运行。
- 运行 `docker-compose up` 启动 Compose，运行你的整个 app

一个 docker-compose.yml 文件看起来像这样：

	version: '3'
	services:
	  web:
	    build: .
	    ports:
	    - "5000:5000"
	    volumes:
	    - .:/code
	    - logvolume01:/var/log
	    links:
	    - redis
	  redis:
	    image: redis
	volumes:
	  logvolume01: {}

关于更多 compose 文件信息，请查看：[Compose file reference.](https://docs.docker.com/compose/compose-file/)

compose 有管理你应用整个生命周期的命令：

- 启动、停止、重新构建服务
- 查看所有运行中服务的状态
- 正在运行的服务的日志输出流
- 在服务上运行一次性命令。

**特性：**
有效的 Compose 功能有：

- 单个主机上多个隔离的环境
- 保存数据卷数据当容器被创建
- 仅重新创建已更改的容器
- 变量和环境更改

......

# How To Do It？

一个较为详细的微服务项目所需要的 docker-compose.yml 及主要一级关键字：

```yml
version: '3'
# 用来声明服务，在该节点之下配置多个服务。
service:

	# web 服务
    web:
        image: id/imagename:lable
		# 指定重启策略（on：默认值，若启动失败不做任何动作 / always：会一直重新启动 / on-failure：服务提示失败错误后会重新启动 / unless-stopped：只有服务在停止后才会重启）
        restart: on-failure
        container_name: my-web-container_name
        ports:
            - 8080:8080
    	# 指定网络，该网络引用下面的配置。
        networks:
            - example-net
        # 服务依赖，指定服务启动顺序。（3版本之后该属性被忽略）
        depends_on:
            - db
        # 服务 swarm 集群部署，非集群下无效。
        deploy:
			# 服务副本数量
            replicas: 2
			# 集群环境下的重启策略，属性值与上面的一样
            restart_policy:
                condition: on-failure

	# 数据库服务
    db:
        images: mysql:5.6
        restart: on-failure
        container_name: mysql-container
        ports:
            - 3306:3306
        volumes:
            - example-mysql: /var/lib/mysql
        networks:
            - example-net:
        # 环境变量
        environment:
            MYSQL_ROOT_PASSWORD: root
            MYSQL_DATABASE: mysql_database
        deploy:
            replicas: 1
            restart_policy:
                condition: on-failure
			# 该服务只有管理节点才能启动
            placement:
                constraints: [node.role == manager]

networks:
    example-net:

volumes:
    example-mysql:
```



关于 docker-compose 文件的所有细节，参考：https://docs.docker.com/compose/compose-file/

# 微服务下的 Docker 部署
## 准备微服务测试项目

四个微服务，分别是：

- `euraka-server`
- `gateway-zuul`
- `microservice-orderservice`
- `microservice-userservice`

## 编写 Dockerfile

分别为四个基础服务提供 Dockerfile 用以构建镜像上传到远程仓库，四个 Dockerfile 都是类似的：

euraka-server:
```sh
FROM java:8-jre
MAINTAINER harry<yuzh233@gmail.com>

ADD ./target/microservice-eureka-server-0.0.1-SNAPSHOT.jar /app/microservice-eureka-service.jar
CMD ["java", "-Xmx200m", "-jar", "/app/microservice-eureka-service.jar"]

EXPOSE 8761
```

gateway-zuul:
```sh
FROM java:8-jre
MAINTAINER harry<yuzh233@gmail.com>

ADD ./target/microservice-gateway-zuul-0.0.1-SNAPSHOT.jar /app/microservice-gateway-zuul.jar
CMD ["java", "-Xmx200m", "-jar", "/app/microservice-gateway-zuul.jar"]

EXPOSE 8050
```

microservice-orderservice:
```sh
FROM java:8-jre
MAINTAINER harry<yuzh233@gmail.com>

ADD ./target/microservice-orderservice-0.0.1-SNAPSHOT.jar /app/microservice-orderservice.jar
CMD ["java", "-Xmx200m", "-jar", "/app/microservice-orderservice.jar"]

EXPOSE 7900
```

microservice-userservice:
```sh
FROM java:8-jre

MAINTAINER harry<yuzh233@gmail.com>

ADD ./target/microservice-userservice-0.0.1-SNAPSHOT.jar  /app/microservice-userservice.jar

EXPOSE 8030

CMD ["java", "-Xmx200m", "-jar", "/app/microservice-userservice.jar"]
```

## 构建镜像

构建后面需要部署的基础镜像，有了镜像后可以通过执行 `docker run` 命令一一启动，但是对于微服务项目来说逐个启动效率不免有点太低，我们可以通过 docker-compose 快速启动。

关于镜像的构建与推送，可以使用 maven 的 dockerfile 插件 👉 [笔记]()

举例：
```sh
<plugin>
	<groupId>com.spotify</groupId>
	<artifactId>dockerfile-maven-plugin</artifactId>
	<version>1.4.3</version>
	<executions>
		<execution>
			<id>default</id>
			<!-- 如果不指定 phase 则会使用默认的 goals，即：package -> build, deploy -> push -->
			<phase>install</phase>
			<goals>
				<goal>build</goal>
				<goal>push</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<repository>registry.cn-hangzhou.aliyuncs.com/yuzh/${artifactId}</repository>
		<tag>${project.version}</tag>
		<useMavenSettingsForAuth>true</useMavenSettingsForAuth>
	</configuration>
</plugin>
```

_NOTE：需要先安装 JDK 和 MAVEN 环境。_

## 非集群环境下的部署

**一）登陆远程服务器，拉取镜像。**

如果是在镜像上传的主机上运行，则默认存在了。

**二）准备 docker-compose.yml**

```yml
version: "3"
services:
  # 第一个服务
  mysql:
    image: mysql
    restart: on-failure
    ports:
      - 3336:3306 # 容器外可以通过 3336 连接到容器内的服务端
    volumes:
      - microservice-mysql:/var/lib/mysql
    networks:
      - microservice-net
    environment:
      MYSQL_ROOT_PASSWORD: harry@admin
      MYSQL_DATABASE: microservice_mallmanagement
    deploy:
      replicas: 1 # 副本数量
      restart_policy: # 重启策略
        condition: on-failure # 失败时重启
      placement:
        constraints: [node.role == manager]

  # 第二个服务
  eureka-server:
    image: registry.cn-hangzhou.aliyuncs.com/yuzh/microservice-eureka-server:0.0.1-SNAPSHOT
    restart: on-failure
    ports:
      - 8761:8761
    networks:
      - microservice-net
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

  # 第三个服务
  gateway-zuul:
    image: registry.cn-hangzhou.aliyuncs.com/yuzh/microservice-gateway-zuul:0.0.1-SNAPSHOT
    restart: on-failure
    ports:
      - 8050:8050
    networks:
      - microservice-net
    depends_on:
      - eureka-server
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
      placement:
        constraints: [node.role == manager]

  # 第四个服务
  order-service:
    image: registry.cn-hangzhou.aliyuncs.com/yuzh/microservice-orderservice:0.0.1-SNAPSHOT
    restart: on-failure
    ports:
      - 7900:7900
    networks:
      - microservice-net
    depends_on:
      - mysql
      - eureka-server
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

  # 第五个服务
  user-service:
    image: registry.cn-hangzhou.aliyuncs.com/yuzh/microservice-userservice:0.0.1-SNAPSHOT
    restart: on-failure
    ports:
      - 8030:8030
    networks:
      - microservice-net
    depends_on:
      - mysql
      - eureka-server
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

  # 第六个服务
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - 8081:8080
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - microservice-net

networks:
  microservice-net:
volumes:
  microservice-mysql:
```

_note: Mysql 容器运行的一些细节，可见 《Docker 补救指南（一）—— 基础使用》 的 `Mysql 容器` 部分。_

**三）在 docker-compose.yml 目录下执行：`docker-compose up`**

如需停止：`docker-compose down`


## 集群环境下的部署

**一）登陆远程仓库**

**二）部署服务**

	docker stack deploy -c docker-compose-swarm.yml --with-registry-auth mall-management

- `-c`：指定 docker-compose 文件地址
- `--with-registry-auth`：通知所有服务节点要到指定的私有仓库拉取镜像
- mall-management：集群服务的总名称

**三）管理命令**

查看所有集群服务：

	docker stack ps

删除指定集群服务：

	docker stack rm XX

更多命令：

	docker stack --help
