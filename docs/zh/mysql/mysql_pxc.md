---
title: mysql_pxc
date: 2023-01-28 05:25:48
permalink: /pages/b211c7/
categories:
  - zh
  - mysql
tags:
  - 
author: 
  name: fengsha
  link: https://github.com/ifengshai/
---


# **前言**

​		数据库是用来存储数据的，那么数据的重要性不言而喻，那么只用一台数据库存储数据是非常危险的，比如，如果这台数据库或者部署这台数据库的服务器崩溃了，那么这段时间的系统崩溃可能会造成不良的用户体验，进而导致经济效益的下滑。 

​		于是，这样设想，如果有多台数据库提供服务支持，即使有一台崩溃了，还有其他的数据库顶上去干活那不就完事了，确实，这些若干个数据库共同组成了一个集群，称为数据库集群。它包括两种常见结构：一主多从，多主多从；



## ***\*PXC特性和优点\****

A. 完全兼容 MySQL。

B. 同步复制，事务要么在所有节点提交或不提交。

C. 多主复制，可以在任意节点进行写操作。

D. 在从服务器上并行应用事件，真正意义上的并行复制。

E. 节点自动配置，数据一致性，不再是异步复制。

F. 故障切换：因为支持多点写入，所以在出现数据库故障时可以很容易的进行故障切换。

G. 自动节点克隆：在新增节点或停机维护时，增量数据或基础数据不需要人工手动备份提供，galera cluster会自动拉取在线节点数据，集群最终会变为一致；

PXC最大的优势：强一致性、无同步延迟

## 开放端口

systemctl start firewalld
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload


## 安装Percona-XtraDB-Cluster
>wget https://downloads.percona.com/downloads/Percona-XtraDB-Cluster-LATEST/Percona-XtraDB-Cluster-8.0.23/binary/redhat/7/x86_64/Percona-XtraDB-Cluster-8.0.23-rd3b9a1d-el7-x86_64-bundle.tar
>tar -xvf Percona-XtraDB-Cluster-8.0.23-rd3b9a1d-el7-x86_64-bundle.tar
>yum localinstall *.rpm

## 启动mysql
修改配置
~~~conf
# Template my.cnf for PXC
# Edit to your requirements.
[client]
socket=/var/lib/mysql/mysql.sock
a
[mysqld]
server-id=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Binary log expiration period is 604800 seconds, which equals 7 days
binlog_expire_logs_seconds=604800

######## wsrep ###############
# Path to Galera library
wsrep_provider=/usr/lib64/galera4/libgalera_smm.so

#pxc集群所有节点之间通讯的加密与否，OFF表示不加密，ON表示加密
pxc-encrypt-cluster-traffic=OFF

# Cluster connection URL contains IPs of nodes
#If no IP is found, this implies that a new cluster needs to be created,
#in order to do that you need to bootstrap this node
#注意第一个节点一定是第一个启动的主节点
wsrep_cluster_address=gcomm://192.168.56.78,192.168.56.79

# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW

# Slave thread to use
wsrep_slave_threads=8

wsrep_log_conflicts

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# Node IP address
#wsrep_node_address=192.168.70.63
wsrep_node_address=192.168.56.78

# Cluster name
wsrep_cluster_name=pxc-cluster

#If wsrep_node_name is not specified,  then system hostname will be used
wsrep_node_name=pxc-cluster-node-1

#pxc_strict_mode allowed values: DISABLED,PERMISSIVE,ENFORCING,MASTER
pxc_strict_mode=ENFORCING

# SST method
wsrep_sst_method=xtrabackup-v2

~~~

第一个启动或者主节点启动需要执行下面命令
>systemctl start mysql@bootstrap.service
其它节点使用,其余节点会同步主节点数据
>systemctl start mysqld



### 修改mysql的配置
查找初始密码
>cat /var/log/mysqld.log
>mysql -u root -p
~~~sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'fengshapxc1';
#设置远程登录
use mysql;
update user set host = '%' where user ='root';
exit;
~~~


## 报错缺少需：qpress
yum -y install https://repo.percona.com/yum/release/7/RPMS/x86_64/qpress-11-1.el7.x86_64.rpm

################################不需要###################################################### 

## 卸载mysql
rpm -qa | grep mysql
rpm -e --nodeps mysql
yum clean all

## 安装mysql
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
rpm -ivh mysql80-community-release-el7-3.noarch.rpm
yum update
yum install mysql-server

### 创建mysql用户,给与mysql权限
>groupadd mysql
>useradd mysql -g mysql -p mysql
>chmod +w /etc/sudoers
>vim /etc/sudoers
>chmod -w /etc/sudoers
~~~
root    ALL=(ALL)       ALL
mysql   ALL=(ALL)  ALL
~~~
>chmod -R 777 /var/lib/mysql
>chown mysql:mysql -R /var/lib/mysql

### 启动mysql
>mysqld --initialize
>systemctl start mysqld
>systemctl status mysqld

### 修改mysql的配置
查找初始密码
>cat /var/log/mysqld.log
>mysql -u root -p
~~~sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'fengshapxc1';
#设置远程登录
use mysql;
update user set host = '%' where user ='root';
exit;
~~~

























































