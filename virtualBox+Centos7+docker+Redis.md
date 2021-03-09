# virtualBox+Centos7+docker+Redis

## 安装virtualBox，导入centos7.ova镜像文件

## xShell远程连接工具

### 虚拟机设置网络为桥接，ip addr 命令查看虚拟机ip

### 连接工具设置主机为虚拟机ip，端口默认22

### 连接成功

## 安装docker

### 查看内核版本：uname -r

### 更新内核版本：yum update

### 安装docker：yum install docker

### 启动docker：systemctl start docker（如果启动docker失败，重启计算机重试，还是不行的话卸载docker重新安装，卸载命令自行百度）

### 设置docker为开机启动：systemctl enable docker

## docker安装Redis

### 查看redis镜像版本：docker search redis

### 获取镜像：docker pull redis:latest

### 查看docker安装的镜像：docker images

### 运行redis容器：docker run -itd --name redis-test -p 6379:6379 redis

### 测试redis服务：docker exec -it redis-test /bin/bash

## docker容器操作

### 查看运行中的容器：docker ps

### 停止你当前运行中的容器：docker stop 容器名或容器id

### 启动容器：docker start 容器名或容器id

### 端口映射：-p 6379:6379

### 容器日志：docker logs 容器名或容器id

*XMind: ZEN - Trial Version*