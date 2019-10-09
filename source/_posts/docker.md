---
title: docker
date: 2019-10-09 15:31:54
tags:
---
Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中。
可以统一我们的开发测试发布环境，可以快速搭建我们所需要的环境（结合 docker compose）
![docker是集装箱的意思](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=05f46e0580d4b31cf03c93bdbfed4042/2cf5e0fe9925bc31137974de55df8db1cb13704b.jpg)
Docker Hub是一个云端服务，可以用它共享应用。（类似Github）
## Docker优势
1. 更高效的利用系统资源，榨取物理机的资源（节约成本）
2. 沙箱机制（安全性）
3. 持续交付和部署（敏捷）
4. 更轻松的迁移、维护和扩展（可移植性）

## 镜像和容器
### 镜像
镜像是一个特殊的文件系统，除了提供容器运行时所需要的程序、库、资源、配置等文件外，还包括了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。奖项不包含任何动态数据，其内容在构建之后也不会被改变。
### 容器
镜像（Image）和容器（Container）的关系，就像是面向对象的类和实例一样。
镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
容器的实质是进程，但和宿主的进程不同，容器进程运行于自己的独立的命名空间，运行在一个隔离环境。

## Dokcer 常用命令
1. 获取镜像
``` bash
$ sudo docker pull NAME[:TAG]
$ sudo docker pull centos:latest
```
tag不添加默认为latest
2. 启动Container
``` bash
$ sudo docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
$ sudo docker run -t -i centos /bin/bash
```
使用容器的bash
3. 查看镜像列表，列出本地的所有images
``` bash
$ sudo docker pull NAME[:TAG]
$ sudo docker pull centos:latest
```
4. 查看容器列表，列出创建过的所有Container
``` bash
$ sudo docker ps [OPTIONS]
$ sudo docker ps -a
```
5. 删除镜像，从本地删除一个已经下载的镜像
``` bash
$ sudo docker rmi IMAGE [IMAGE...]
$ sudo docker rmi cenos:latest
```
6. 移除一个或多个容器实例
``` bash
$ sudo docker rm [OPTIONS] CONTAINER [CONTAINER...]
$ sudo docker rm sudo docker ps -aq（移除所有未运行的容器）
```
7. 停止一个正在运行的容器
``` bash
$ sudo docker kill [OPTIONS] CONTAINER [CONTAINER...]
$ sudo docker kill 026e
```
8. 重启一个正在运行的容器
``` bash
$ sudo docker restart [OPTIONS] CONTAINER [CONTAINER...]
$ sudo docker restart 026e
```
9. 启动一个已经停止的容器
``` bash
$ sudo docker start [OPTIONS] CONTAINER [CONTAINER...]
$ sudo docker start 026e
```

## 构建node镜像案例
### 构建Dockerfile
``` bash
FROM node:10.0-alpine 

RUN npm install pm2 -g --registry=https://registry.npm.taobao.org
RUN mkdir -p  /user/src/server/
WORKDIR /user/src/server/
COPY package.json /user/src/server/package.json
COPY ./dist /user/src/server/dist
RUN cd /user/src/server/
RUN npm i --production --registry=https://registry.npm.taobao.org

EXPOSE 3000
CMD pm2 start dist/src/main.js && pm2 log
```
1. Alpine是一个很小的Linux发行版本，可以大幅度减小镜像体积
2. package.json提前，在没有修改得情况是不会重新安装npm包的，会减少部署时间
3. npm i --production 生产环境不打包测试环境的包

### 构建和部署
```bash
docker build -t heshuai326/server:latest .
# docker commit -a "heshuai326" -m "latest" 31f397f03bf1 server:latest
docker tag ddf287a62df5  heshuai326/server:latest
docker push heshuai326/server:latest

docker pull heshuai326/server:latest
docker run -p 3000:3000 heshuai326/server:latest
```

## docker-compose
### 什么是docker-compose
docker-compose是一个用户定义和运行多个容器的Docker应用程序。在Compose中你可以使用YAML文件来配置你的应用服务。
然后只需要一个简单的命令就可以创建并启动你配置的所有服务。
### 启动命令
```bash
docker-compose up -d
```
-d参数是后台进行启动
### mysql脚本
```bash
version: '3.3'
services:
  mysql:
    image: mysql:5.6
    volumes:
      - /root/data/mysql:/var/lib/mysql
      - ./conf/:/etc/mysql/conf.d
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=db
      - MYSQL_ROOT_PASSWORD=123456
volumes:
  mysql:
```
### mongo和redis脚本
```bash
version: '3'
services:
  mongo:
    image: mongo:4.0.0
    command: --bind_ip_all
    restart: always
    ports:
    - "27017:27017"
    volumes:
    - "mongo:/var/mongo"
  redis:
    image: redis:4.0.8
    restart: always
    ports:
    - "6379:6379"
    volumes:
    - "redis:/var/redis"
volumes:
  mongo:
  redis:
```

## 参考链接
[docker从入门到实践](https://yeasy.gitbooks.io/docker_practice/introduction/what.html)
[nodejs技术栈docker入门到实践](https://mp.weixin.qq.com/s/S7ksqF8z4SYJvcG1DOupNA)
[nodejs技术栈Node.js服务 Docker 容器化应用实践](https://mp.weixin.qq.com/s/ZUw_qLk3m77ATkYXpfP08A)