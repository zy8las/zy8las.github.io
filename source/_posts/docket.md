---
title: docker学习整理
date: 2018-12-19 
tags:
---

docker学习整理

#### 一、介绍

云计算，类似于VM

docker 平台，管理部署应用程序，开发容纳，运行应用程序的平台。在容器中安全的运行应用程序。

docker引擎，cs结构的程序，server长时间运行的守护进程，cli客户端命令

docker对象，同时运行多个虚拟机

image 镜像

containers  容器，进程

network 网络

data values 数据卷

docker daemon 守护进程，docker的守护进程，运行在host之上，不需要直接交互，通过cli访问。

docker client 接收用户指令，并和docker守护进程通信

docker image docker镜像，只读模板

docker registries  注册表，包含镜像库

docker container docker容器，通过镜像创建容器，每个容器是独立的，安全的运行平台，容器中运行的是docker的组件。

#### 二、安装

参考官网 

#### 三、使用

1. 启动/停止
   service docker start/stop
2. 验证docker是否启动成功
   docker run hello-world
3. 查看docker版本
   docker version
4. 查看帮助
   docker --help   / docker run --help    
5. 配置开机自启动
   systemctl enable docker
6. 查看运行的容器
   docker ps -a
   docker  告知os使用docker指令
   run 子命令创建运行容器
   hello-world  加载的镜像
7. 注册docker用户
   https://hub.docker.com/
8. 查看本地镜像
   docker images
9. 构建镜像
   * 创建文件
     touch Dockerfile
   * 编辑文件
     FROM docker/whalesay:latest
     RUN apt-get -y update && apt-get install  -y fortunes
     CMD /usr/games/fortune -a |cowsay
   * 从dockerfile构建镜像文件
     docker build -t docker-whale . 
10.  找到镜像ID
    Image id
11. 使用docker login远程登录到docker hub
    docker login --username=用户名  --email=邮箱
12. 标记镜像
    docker tag 镜像id  用户名/hello-world：latest
13. 使用push命令上传镜像
    docker push 用户名/hello-world
14. 删除本地镜像文件
    docker rmi  -f 镜像id
15. 使用docker run下载镜像并使用
    docker run 用户名/hello-world
16. 运行容器
    docker run ubuntu /bin/each "hello world"
    运行交互式容器
    -t  伪字符终端
    -i  交互式
17. 进入容器中可以使用正常的shell指令，exit /ctrl +D 退出交互式
18. 创建容器以守护进程的方式运行
    -d 以守护进程的方式
19. 查看容器的log
    docker log  容器id
20. 停止容器
    docker stop 容器id
21. 删除容器
    docker rm 容器id
22. 删除多个容器
    docker rm id1 id2
23. 查看容器
    docker ps -a
24. 停止容器
    docker stop 容器id
25. 运行容器，使用training/ webapp
    -d 后台方式
    -p(小p)映射指定端口
    -P（大P）映射所有暴露的端口
    docker run -d -P training/webapp  python app.py
    docker run -d -p 80:5000 training/webapp  python app.py
    打开浏览器使用host主机映射的端口，访问虚拟机的客户机web服务器
26. 查看容器的IP地址
    docker-machine id 容器id
27. 查看port子命令
    docker port --help
28. 查看容器中指定的端口在宿主机上的映射端口
    docker port 容器id 5000
29. 查看web程序的log
    docker logs -f 容器名称
30. 查看容器的进程
    docker top 容器名称/容器id
31. 检查容器
    docker inspect 容器名称
32. 查看容器列表
    docker ps -l
33. 删除容器
    docker rm -f 容器id    （-f强制删除）
34. 删除镜像
    docker rmi -f 镜像id  （-f强制删除）
35. 通过标记运行容器
    docker run -i -t ubuntu:14.04 /bin/bash 如果只指定ubuntu，则默认使用latest版本
36. 下载镜像
    docker pull 镜像名称
37. 搜索镜像
    docker search 镜像名称
38. 提交变化给镜像
    docker commit -m “说明文字” -a “作者名称” 容器id ubuntu:V2
    本地产生新的镜像，提交就是写入，保持变化的部分，产生新的镜像
39. 拷贝文件到docker容器中
    docker cp a.war mytomcat:/usr/local/tomcat/webapps
40. 进入交互模式
    docker exec -it mytomcat /bin/bash

#### 四、应用

docker应用之tomcat容器部署web应用

docker run -d --name mytomcat -p8888(宿主机端口):8080(docker中的端口)

访问docker中的tomcat ：宿主机IP：映射端口

docker cp a.war mytomcat:/usr/local/tomcat/webapps

进入交互模式

docker exec -it mytomcat /bin/bash

