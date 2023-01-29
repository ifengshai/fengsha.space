---
title: docker
date: 2023-01-28 05:25:47
permalink: /pages/69ad66/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
## 安装docker
```shell
wget -qO- https://get.docker.com | sh
```

如果失败使用下面命令安装

```shell
yum -y install yum-utils

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum makecache fast

yum -y install docker-ce
```

如果依然失败执行

```shell
sudo yum install --allowerasing docker-ce docker-ce-cli containerd.io
```



## 安装docker-composer

查考命令地址：https://docs.docker.com/compose/install/

```shell
curl -SL https://github.com/docker/compose/releases/download/v2.7.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
 
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker compose version

```

#国内上面无法下载使用下面命令

```shell
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker --version
docker-compose --version
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```



## docker镜像加速

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ov29upwv.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```







查看Dockerfile和docker-compose语法可以参考
里面是搭建php环境，和一些运维工具以及本人使用的所有软件docker。

https://gitee.com/ifengshai/dnmp


查看php地址
docker inspect fs_php | grep "IPAddress"

修改nginx的相关配置 nginx容器内配置文件路径/etc/nginx/conf.d/default.conf

docker exec -it fs_nginx /bin/bash

删除容器
docker rm -f <容器id>

删除镜像
docker rmi <镜像idIMAGE ID>

常见问题
在启动时如果没有添加restart参数，比如1a7a3b5112fd这个容器在启动的时候是没有添加–restart=always参数的，针对这种情况我们可以使用命令进行修改。

docker container update --restart=always <容器名字>




杀死所有正在运行的容器 docker kill $(docker ps -a -q)

删除所有已经停止的容器 docker rm $(docker ps -a -q)

删除所有未打 dangling 标签的镜 docker rmi $(docker images -q -f dangling=true)

删除所有镜像 docker rmi $(docker images -q)

强制删除 无法删除的镜像 docker rmi -f <IMAGE_ID> docker rmi -f $(docker images -q)







```
cd /etc/yum.repos.d
vi CentOS-AppStream.repo

baseurl=https://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
vi CentOS-Base.repo

baseurl=https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
vi CentOS-Extras.repo

baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os/
```