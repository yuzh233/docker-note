---
title: Hexo迁移至Docker
date: 2019-06-18
tags:
- hexo
- docker
thumbnail: http://img.yuzh.xyz/docker-note/65.jpg
toc: true
---

# 一、本地迁移

**一）将之前的博客地址根文件夹拷贝一份复制到指定位置，用作与容器共享的数据卷。**

```sh
~/Workspace/docker-hexo  $ pwd
/Users/harry/Workspace/docker-hexo
~/Workspace/docker-hexo  $ ll
total 8
-rw-r--r--@  1 harry  staff   482B Jun 17 17:10 Dockerfile
drwxr-xr-x@ 15 harry  staff   480B Jun 17 17:19 harrys-blog # 根目录
~/Workspace/docker-hexo  $
```

**二）编写 Dockerfile 脚本，构建镜像。**

**docker build -f Dockerfile -t harrys-blog:latest .**

```sh
FROM centos
MAINTAINER harry<yuzh233@gmail>
EXPOSE 4000
RUN cd / && mkdir docker-hexo && cd docker-hexo

WORKDIR /docker-hexo

# install latest node
RUN yum install -y gcc-c++ make \
&& curl -sL https://rpm.nodesource.com/setup_12.x | bash - \
&& yum -y install nodejs \
&& node -v

# install git
RUN yum -y install git \
&& git --version

# install hexo
RUN npm install -g hexo-cli

# CMD ["sh","-c","hexo g && hexo s"]
ENTRYPOINT ["sh","-c","cd /harrys-blog && hexo g && hexo s"]
```

**三）运行容器，指定数据卷文件路径**

**docker run -d -p 4000:4000 -v ~/Workspace/docker-hexo/harrys-blog/:/harrys-blog harrys-blog:latest**

# 二、迁移到远程服务器

**一）创建阿里云私有仓库并推送镜像**

```sh
$ sudo docker login --username=yuzh233 registry.cn-hangzhou.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/yuzh/harrys-blog:[镜像版本号]
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/yuzh/harrys-blog:[镜像版本号]
```

**二）使用 source copy 上传文件到服务器**

```sh
scp hexo.gz root@120.78.69.82:/hexo.gz
```

**三）运行容器，指定数据卷文件路径**

**docker run -d -v ~/harrys-blog:/harrys-blog -p 4000:4000 registry.cn-hangzhou.aliyuncs.com/yuzh/harrys-blog**

**四）运行 nginx 容器，将其中 /etc/nginx 目录拷贝到宿主机，停止容器**

**五）运行一个新的容器，挂载本地 nginx 配置到容器中**

`docker run -d -p 80:80 -v ~/nginx-container-config/:/etc/nginx nginx`

**nginx.conf:**

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

    upstream hexo {
        server 120.78.69.82:4000;
    }

    server {
        # 默认监听80端口，这样就不需要指定端口访问。
        listen       80;
# listen 443;
        # 用户访问时的域名。
        server_name  yuzh.xyz;

# ssl on;
listen 443 ssl;
 ssl_certificate   cert/yuzh.xyz.pem;
 ssl_certificate_key  cert/yuzh.xyz.key;
 ssl_session_timeout 5m;
 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  #使用此加密套件。
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #使用该协议进行配置。
 ssl_prefer_server_ciphers on;

        location / {
            # nginx通过前面的 hexo 转发到指定的网站，访问 hexo 就是访问 120.78.69.82:4000
            proxy_pass   http://hexo;
            index  index.html index.htm;
        }
    }

}
```
