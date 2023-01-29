---
title: Jenkins
date: 2023-01-28 05:25:46
permalink: /pages/d7b323/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
## 安装jenkins相关依赖 https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum upgrade
sudo yum -y install epel-release java-11-openjdk-devel
sudo yum -y install jenkins
sudo systemctl daemon-reload
sudo systemctl start jenkins



## 国内源修改baidu.com
vi /var/lib/jenkins/updates/default.json

vi /var/lib/jenkins/hudson.model.UpdateCenter.xml
~~~
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>http://mirror.esuni.jp/jenkins/updates/update-center.json</url>
  </site>
</sites>

~~~

## 好用的插件
Active Choices
Git Parameter Plug-In
build user vars

## 修改Jenkins以root用户运行的方法
打开配置文件
vim /etc/sysconfig/jenkins
~~~
$JENKINS_USER="root"
~~~

修改Jenkins相关文件夹用户权限
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins

重启Jenkins服务并检查运行Jenkins的用户是否已经切换为root
systemctl restart jenkins
# 查看Jenkins进程所属用户
ps -ef | grep jenkins




## 卸载jenkins
service jenkins stop
yum clean all 
yum -y remove jenkins
rpm -e jenkins
rpm -ql jenkins 
find / -iname jenkins | xargs -n 1000 rm -rf





















