---
title: Elasticsearch
date: 2023-01-28 05:25:46
permalink: /pages/a2d8cf/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
## 安装Elasticsearch
获取链接https://www.elastic.co/cn/downloads/elasticsearch
> wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.2-linux-x86_64.tar.gz

解压文件
> tar xf elasticsearch-7.13.2-linux-x86_64.tar.gz

复制文件夹
> cp -r elasticsearch-7.13.2-linux-x86_64 /home/elasticsearch/elasticsearch_01

修改配置文件
> vi /home/elasticsearch/elasticsearch_01/config/elasticsearch.yml

~~~
#配置集群名称 多台节点服务器保持一致
cluster.name: my-application
#配置单一节点名称，每个节点唯一标识
node.name: node-1
#指定该节点是否有资格被选举成为node，默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。
node.master: true
#设置绑定的ip地址
network.host: 0.0.0.0
#设置对外服务的http端口，默认为9200
http.port: 9200
#设置节点间交互的tcp端口，默认是9300
transport.tcp.port: 9300
#集群节点ip或者主机
discovery.zen.ping.unicast.hosts: ["192.168.56.77:9300", "192.168.56.77:9301"]

# 开启跨域访问支持，默认为false
http.cors.enabled: true
# 跨域访问允许的域名地址
http.cors.allow-origin: "*"
# 通过为 cluster.initial_master_nodes 参数设置符合主节点条件的节点的 IP 地址来引导启动集群
cluster.initial_master_nodes: ["node-1"]
~~~

### 新增elsearch用户组
>groupadd elsearch
### 创建elsearch用户
>useradd elsearch -g elsearch -p elasticsearch
### 给予用户目录权限
chown -R elsearch:elsearch /home/elasticsearch

## 启动elsearch
>su elsearch
>/home/elasticsearch/elasticsearch_01/bin/elasticsearch
>/home/elasticsearch/elasticsearch_02/bin/elasticsearch


### 创建启动脚本
>vi /home/elasticsearch/elasticsearch.sh
~~~shell
#!/bin/sh
#chkconfig: 2345 80 05
#description: elasticsearch

case "$1" in
start)
    su elsearch<<!
    /home/elasticsearch/elasticsearch_01/bin/elasticsearch -d
    /home/elasticsearch/elasticsearch_02/bin/elasticsearch -d
!
    echo "elasticsearch_01 startup"
    echo "elasticsearch_02 startup"
    ;;
stop)
    es_pid=`ps aux|grep elasticsearch | grep -v 'grep elasticsearch' | awk '{print $2}'`
    kill -9 $es_pid
    echo "elasticsearch_01 stopped"
    echo "elasticsearch_02 stopped"
    ;;
restart)
    es_pid=`ps aux|grep elasticsearch | grep -v 'grep elasticsearch' | awk '{print $2}'`
    kill -9 $es_pid
    echo "elasticsearch_01 stopped"
    echo "elasticsearch_02 stopped"
    su elsearch<<!
    /home/elasticsearch/elasticsearch_01/bin/elasticsearch -d
    /home/elasticsearch/elasticsearch_02/bin/elasticsearch -d
!
    echo "elasticsearch_01 startup"
    echo "elasticsearch_02 startup"
    ;;
*)
    echo "start|stop|restart"
    ;;
esac
~~~
### 创建自启动服务
>vi /etc/systemd/system/elasticsearch.service
~~~
[Unit]
Description=elasticsearch
[Service]
#定义启动进程时执行的命令。
ExecStart=/home/elasticsearch/elasticsearch.sh start
#重启服务时执行的命令
ExecReload=/home/elasticsearch/elasticsearch.sh restart
#停止服务时执行的命令
ExecStop=/home/elasticsearch/elasticsearch.sh stop
RemainAfterExit=yes
Type=simple
KillMode=process
Restart=on-failure
RestartSec=30s
[Install]
WantedBy=multi-user.target
~~~
### 添加开机自启动
>systemctl daemon-reload
>systemctl enable elasticsearch.service
### 重启
>reboot now
安装Elasticsearch-Head
/home/elasticsearch/elasticsearch_01/bin/elasticsearch-plugin install mobz/elasticsearch-head


## 问题一：
ERROR: bootstrap checks failed
max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]
原因：无法创建本地文件问题,用户最大可创建文件数太小
解决方案：
切换到root用户，编辑limits.conf配置文件， 添加类似如下内容：
vi /etc/security/limits.conf
添加如下内容:
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
备注：* 代表Linux所有用户名称（比如 hadoop）
保存、退出、重新登录才可生效

## 问题二
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
在/etc/sysctl.conf文件最后添加一行
>vi /etc/sysctl.conf
~~~
vm.max_map_count=262144
~~~
立即生效
>/sbin/sysctl -p
再重新启动

## 问题三
error:
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x000000064d800000, 6215958528, 0) failed; error='Not enough space' (errno=12)
jvm内存限制，具体情况按实际配置这里为了减少虚拟机的压力调最小
>vi /home/elasticsearch/elasticsearch_01/conf/jvm.options
~~~
-Xms1g
-Xmx1g
~~~

elasticsearch/bin/plugin install mobz/elasticsearch-head





# 安装Kibana
下载Kibana安装包https://www.elastic.co/cn/downloads/kibana下载最新的编译过的
> wget https://artifacts.elastic.co/downloads/kibana/kibana-7.13.2-linux-x86_64.tar.gz

解压文件
> tar xf kibana-7.13.2-linux-x86_64.tar.gz

复制文件夹
> mkdir /home/elasticsearch/kibana
> cp -r kibana-7.13.2-linux-x86_64 /home/elasticsearch/kibana

修改配置文件
> vi /home/elasticsearch/kibana/config/kibana.yml
~~~
server.port: 5601　　　　　　　　　　　　　　　　　　　　 #端口，不需要修改
server.host: 192.168.56.77　　　　　　　　　　　　　　　 #本地IP地址　
elasticsearch.hosts: ["http://192.168.56.77:9200"]   #ES集群地址

~~~
### 新增kibana用户组
>groupadd kibana
### 创建kibana用户
>useradd kibana -g kibana -p kibana
### 给予用户目录权限
chown -R kibana:kibana /home/elasticsearch/kibana


## 启动kibana
> su kibana
> nohup ./kibana 2>&1 &

开启防火墙5601
> firewall-cmd --zone=public --add-port=5601/tcp --permanent
> firewall-cmd --reload


# 客户端
浏览器插件：
google/firfox     Elasticvue











































