---
title: Docker 补救指南（五）—— dockerfile-maven-plugin
date: 2019-05-30
tags:
- docker
- maven
categories:
- 问题解决
thumbnail: http://img.yuzh.xyz/docker-note/65.jpg
toc: true
---

# dockerfile-maven-plugin 插件介绍

该插件帮助 Maven 集成 Docker

- 不需要任何花里胡哨的操作🤪。这个插件使用 Dockerfile 构建镜像，并且是强制性的。
- 使 Docker 的构建过程集成 Maven 的构建过程，如果绑定了默认「阶段（phases）」，当输入 `mvn package` 时，将会构建一个镜像；当输入 `mvn deploy` 时，该镜像将会被推送到远程仓库。
- 让目标「goals」记住你要做什么（通过 goals 标签定制处理过程）。可以通过输入 `mvn dockerfile:tag`、`mvn dockerfile:build`、`mvn dockerfile:push` 来构建并推送一个镜像，作为替代的可以使用：`mvn dockerfile:build dockerfile:push`。

reference：https://github.com/spotify/dockerfile-maven

以下是简单的实例配置，在此配置中，执行 `mvn package` 将构建一个镜像，执行 `mvn deploy` 将推送镜像。当然可以通过执行 `mvn dockerfile:build` 明确的说明构建。

```xml
<plugin>
  <groupId>com.spotify</groupId>
  <artifactId>dockerfile-maven-plugin</artifactId>
  <version>${dockerfile-maven-version}</version>
  <executions>
    <execution>
      <id>default</id>
	  <!-- 这里没有指定 phase 就是默认 mvn package 时执行 build 操作，mvn deploy 时执行 push操作 -->
      <goals>
        <goal>build</goal>
        <goal>push</goal>
      </goals>
    </execution>
  </executions>

  <configuration>
    <repository>spotify/foobar</repository>
    <tag>${project.version}</tag>
    <buildArgs>
      <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
    </buildArgs>
  </configuration>
</plugin>
```

**该插件的所有可用的构建目标（goals）**

Goal         | Description                | Default Phased |
-------------|----------------------------|----------------|--
docker:build | 从 Dockerfile 构建一个镜像 | package        |
docker:tag   | 为镜像设置一个标签         | package        |
docker:push  | 推送镜像到远程仓库         | deploy         |

**使 maven 的某个构建过程跳过 dockerfile-plugin 的某个目标**

Maven Option          | What Does it Do?         | Default Value |
----------------------|--------------------------|---------------|--
dockerfile.skip       | 关闭全部 dockerfile 插件 | fals          |
dockerfile.build.skip | 关闭 build 目标          | false         |
dockerfile.tag.skip   | 关闭 tag 目标            | false         |
dockerfile.push.skip  | 关闭 push 目标           | false         |

默认 mvn package 会执行 dockerfile:build 操作（Goal），如果我们在执行 mvn package 时指定参数：`mvn package -Ddockerfile.build.skip` 那么该过程不会执行镜像的构建。

reference: https://github.com/spotify/dockerfile-maven/blob/master/docs/usage.md

**认证**

See authentication docs.

reference：https://github.com/spotify/dockerfile-maven/blob/master/docs/authentication.md


**依靠 Dockerfile 插件的其他 Docker 工具**

- [ ] TODO

# 使用 dockerfile-maven-plugin 插件遇到的一些问题

完整配置：

```xml
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
        <repository>registry.cn-shanghai.aliyuncs.com/yuzh/microservice-userservice</repository>
        <!--
            如果不想在 pom 中指定 username password
            可以在 setting.xml 中添加 server 节点。
            注意：server 节点中的 id 值必须和 这里的 repository 仓库前缀地址保持一致！如
            <server>
                <id>registry.cn-shanghai.aliyuncs.com</id>
                <username></username>
                <password></password>
            </server>
        -->
        <username>${username}</username>
        <password>${password}</password>
        <tag>${project.version}</tag>
        <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
    </configuration>
</plugin>
```

**0）如果不想指定 push 到远程仓库的用户名和密码的话，可以先登陆远程仓库。（否则，需要在 pom 中或 setting.xml 中配置远程仓库用户名和密码）**

```
e.g: locker login --username=yuzh233 registry.cn-shanghai.aliyuncs.com
```

**1） 使用最新的 `1.4.10` 版本 push 会有点问题，可能缺少什么配置，从官方 github 没有找出问题出来。目前使用 `1.4.3` 没问题。**

**2） 需要注意命令行执行 mvn 命令和 IDE 图形化操作 mvn 时所使用的 setting.xml 文件不是一致的。** terminal 默认加载的配置文件是全局的：`${MAVEN_HOME/conf/setting.xml`，如有需要可以在命令行执行携带参数设置备用加载路径：`--settings=/User/xxxx/settings.xml`

**3）如果不指定 phase 标签，dockerfile-plugin 将会在 deploy 时 push Docker 镜像**，需要注意的是：往往这个时候会报错，提示👇

```
Deployment failed: repository element was not specified in the POM inside distributionManagement element
```

检查后发现，docker 远程仓库的配置看起来没问题（在 setting.xml 中配置了私有仓库用户名和密码或者在 pom 中指明了仓库地址和用户名密码），但是就是一直报错？！

手动执行 `mvn dockerfile:push` 发现成功推送到了远程仓库，所以可以确定的是 **这里报错是 jar 包推送到 maven 的远程仓库时没有配置好环境导致的，而不是 docker 仓库的配置问题**。所以要么配置好maven远程仓库，要么指定 phase 不等于 deploy。 maven 打包到远程仓库相关细节，先晾着吧🙄。
