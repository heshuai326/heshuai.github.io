---
title: docker
date: 2019-10-09 15:31:54
tags:
---
Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用一级依赖到一个可移植的镜像中。
可以统一我们的开发测试发布环境，可以快速搭建我们所需要的环境（结合 docker compose）
Docker Hub是一个云端服务，可以用它共享应用。（类似Github）
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