---
title: 'Docker + Nginx 部署个人静态博客指南'
draft: false
date: 2023-03-24T10:00:00+08:00
draft: false
description: '通过本指南，您可以快速了解如何使用 Docker 和 Nginx 部署自己的静态博客网站。'
tags: ["Docker", "Nginx", "Deployment", "Static Site", "Tutorial"]
author: 'Cassius0924'
---
tags: ["Linux", "Docker", "Nginx", "Server", "Blog"]
---

# Docker + Nginx 部署个人静态博客指南

本文是一个使用 Docker 和 Nginx 部署个人静态博客的指南。通过本指南，您可以快速了解如何使用 Docker 和 Nginx 部署自己的静态博客网站。

## 前提

在开始使用本指南之前，请具备以下前提：

- 首先你得有个服务器
- 服务器已经安装好Git、Vim等工具
- 一份静态博客源码，本文以 [Astro Air Blog](https://github.com/austin2035/astro-air-blog) 为例

## 步骤

###  第一步：安装 Docker

```bash
sudo apt install docker
```

###  第二步：拉取 Nginx 镜像

```bash
docker pull nginx
```

> 镜像名后不加版本号表示拉取最新版，若希望拉取指定版本则需在镜像名后加上tag，例如`docker pull nginx:1.16`。

### 第三步：获取 Nginx 的配置文件

- 先运行一个不挂载的 Nginx 容器

	```bash
	docker run -d --name my-nginx -p 80:80 nginx
	```

	> `-d`：使容器在后台以守护进程模式运行。
	>
	> `--name`：为容器指定一个名称。
	>
	> `-p 80:80`：将Docker容器的80端口映射到主机的80端口，让你可以通过浏览器访问运行在容器内的 Nginx 服务器。80端口是HTTP服务，443端口是HTTPS服务。

- 进入这个 Nginx 容器内部

	```bash
	docker exec -it my-nginx bash
	```

	> `-i`：表示以交互式模式运行容器。
	>
	> `-t`：表示为容器分配一个伪终端。 因此`-it`表示使用交互式终端，允许在容器内交互式地运行命令。
	>
	> `bash`：表示使用Bash shell。

-  了解 Nginx 的目录结构

  Nginx 容器的部分目录结构如下图：

  ```
  /
  ├── etc/
  │   └── nginx/
  │       ├── nginx.conf
  │       └── conf.d/
  │           ├── default.conf
  │           └── other.conf
  ├── var/
  │   └── log/
  │       └── nginx/
  │						├── access.log
  │						└── error.log
  └── usr/
      └── share/
          └── nginx/
              ├── html/
              └── conf/
  ```
  
  > `nginx.conf`是 Nginx 主配置文件，用于设置全局的 Nginx 配置。
  >
  > `default.conf`是 Nginx 的 server 的配置文件。这个文件中通常包含了 HTTP 或 HTTPS 服务器的基本配置信息，如监听端口、虚拟主机等。
  
  > `access.log`是 Nginx 的访问日志。
  >
  > `error.log`是 Nginx 的错误日志。
  
  > `/usr/share/nginx/` 目录用于存放 Nginx 的一些静态文件，我们通常把打包后的网站代码挂载到此处。

- 退出容器

  按下`ctrl + D`或者输入`exit` 即可退出容器。

- 复制容器内部的配置文件到服务器

  ```bash
  docker cp my-nginx:/etc/nginx/nginx.conf ~/nginx/nginx.conf
  docker cp my-nginx:/etc/nginx/conf.d ~/nginx/
  ```

  > 这样就不怕自己写的配置文件不标准而导致容器运行失败啦。

  简单分析一下这两个配置文件：

  ```bash
  cd ~
  vim nginx/nginx.conf
  ```

  > 先看`nginx.conf`：
  >```nginx
  >user  nginx;
  >worker_processes  auto;		# 指定 Nginx 使用的工作进程数，auto 表示 Nginx 会自动根据 CPU 核数设置合适的进程数
  >
  >error_log  /var/log/nginx/error.log notice;		# Nginx 错误日志文件的路径和日志级别，这里的日志级别为 notice，表示只记录警告级别及以上的日志
  >pid        /var/run/nginx.pid;
  >
  >
  >events {
  >    worker_connections  1024;
  >}
  >
  >
  >http {
  >    include       /etc/nginx/mime.types;
  >    default_type  application/octet-stream;
  >
  >  	# 定义访问日志的格式
  >    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
  >                      '$status $body_bytes_sent "$http_referer" '
  >                      '"$http_user_agent" "$http_x_forwarded_for"';
  >
  >    access_log  /var/log/nginx/access.log  main;	# 指定访问日志文件的路径和日志格式，日志格式为上面定义的 main 格式
  >
  >    sendfile        on;		# 启用 sendfile 指令，加快文件传输速度
  >    #tcp_nopush     on;
  >
  >    keepalive_timeout  65;	# 指定客户端连接保持时间的最大值。
  >
  >    #gzip  on;
  >
  >    include /etc/nginx/conf.d/*.conf;		# 加载 /etc/nginx/conf.d/ 目录下的所有以 .conf 结尾的配置文件
  >}
  >```

  > 接着是`default.conf`：
  > ```nginx
  > server {
  >     listen       80;	# 监听 IPV4 的80端口
  >     listen  [::]:80;	# 监听 IPV6 的80端口
  >     server_name  111.222.111.222;		# 这里填入你的服务器地址
  >     #access_log  /var/log/nginx/host.access.log  main;
  > 
  >   	# location 指令用于匹配请求的 URL 路径，并设置对应的处理方式
  >   	# 简单的说就是当有客户端访问 http://111.222.111.222/ 时，Nginx 将会尝试在 			
  >   	# /usr/share/nginx/html 路径下查找 index.html 返回给客户端
  >     location / {
  >         root   /usr/share/nginx/html;		
  >         index  index.html index.htm;
  >     }
  > 
  >   	# error_page 指令定义了当服务器出现 500、502、503、504 错误时的处理方式
  >     error_page   500 502 503 504  /50x.html;	# 统一请求 /50x.html
  >     location = /50x.html {	# '='表示精确匹配，后面不可带参数。
  >         root   /usr/share/nginx/html;
  >     }
  > }
  > ```
  >

- 暂停并删除容器

  暂停容器：

  ```shell
  docker stop my-nginx
  ```

  删除容器：

  ```shell
  docker rm my-nginx
  ```

### 第四步：克隆静态博客源码

- 克隆源码：

	```bash
	git clone git@github.com:austin2035/astro-air-blog.git
	```

- 给你的博客源码改个名：

	```bash
	mv astro-air-blog ~/blog
	```

- 进入项目目录：
	```bash
	cd ~/blog
	```

### 第五步：打包博客源码

-  打包项目：

  ```bash
  npm run build
  ```

  > 一定要打包后才再挂载，Nginx 只能解析 html 文件。

- 查看打包后的文件：

  ```bash
  cd dist
  ls
  ```

  确保目录内存在 index.html 文件。

### 第六步：部署网站

- 先创建一个日志目录：

  ```bash
  cd ~
  mkdir nginx/logs
  ```

- 挂载目录并运行容器：
	
	```bash
	docker run \
	-p 80:80 \
	--name my-nginx \
	-v ~/blog/dist:/usr/share/nginx/html \
	-v ~/nginx/nginx.conf:/etc/nginx/nginx.conf \
	-v ~/nginx/conf.d:/etc/nginx/conf.d \
	-v ~/nginx/logs:/var/log/nginx \
	-d nginx
	```

- 查看容器是否运行成功：

  ```bash
  docker ps
  ```

  若有`my-nginx` 这个镜像表示运行成功，若没有则表示 Docker 运行出错了，检查`nginx.conf`和`default.conf`文件语法是否存在错误。

### 第七步：测试网站

试在浏览器直接访问你的服务器IP，若部署成功你将看到你的网站。
