---
title: canal
date: 2023-01-28 05:25:47
permalink: /pages/1ebfb5/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
## mysql配置修改
> vi /home/mysql/mysql_01/my.cnf
~~~
[mysql]
log-bin=mysql-bin #添加这一行就 ok   
binlog-format=ROW #选择 row 模式    
server_id=1       #配置 mysql replaction 需要定义，不能和canal的slaveId重复
~~~
## 安装canal 参考https://github.com/alibaba/canal/wiki/AdminGuide
下载canal安装包https://github.com/alibaba/canal/releases下载最新的编译过的
> wget https://github.com/alibaba/canal/releases/download/canal-1.1.5/canal.deployer-1.1.5.tar.gz

解压文件
> tar xf canal.deployer-1.1.5.tar.gz

复制文件夹
> cp -r canal /home/canal/canal_01

修改配置文件
> vi /home/canal/canal_01/conf/example/instance.properties
~~~
#mysql地址
canal.instance.master.address=192.168.56.77:3306
#用户名
canal.instance.dbUsername=copy
#密码
canal.instance.dbPassword=copy888
#指定编码方式
canal.instance.connectionCharset=UTF-8
~~~

启动canal
> /home/canal/canal_01/bin/startup.sh




# 安装canal.adapter 参考地址https://github.com/alibaba/canal/wiki/ClientAdapter

下载canal安装包https://github.com/alibaba/canal/releases下载最新的编译过的
> wget https://github.com/alibaba/canal/releases/download/canal-1.1.5/canal.deployer-1.1.5.tar.gz

解压文件
> tar xf canal.adapter-1.1.5.tar.gz

修改adapter配置文件
> vim /home/canal/canal_adapter/conf/application.yml
~~~
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  mode: tcp #tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: 0
  timeout:
  accessKey:
  secretKey:
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 192.168.56.77:11111
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:
    # kafka consumer
    kafka.bootstrap.servers: 127.0.0.1:9092
    kafka.enable.auto.commit: false
    kafka.auto.commit.interval.ms: 1000
    kafka.auto.offset.reset: latest
    kafka.request.timeout.ms: 40000
    kafka.session.timeout.ms: 30000
    kafka.isolation.level: read_committed
    kafka.max.poll.records: 1000
    # rocketMQ consumer
    rocketmq.namespace:
    rocketmq.namesrv.addr: 127.0.0.1:9876
    rocketmq.batch.size: 1000
    rocketmq.enable.message.trace: false
    rocketmq.customized.trace.topic:
    rocketmq.access.channel:
    rocketmq.subscribe.filter:
    # rabbitMQ consumer
    rabbitmq.host:
    rabbitmq.virtual.host:
    rabbitmq.username:
    rabbitmq.password:
    rabbitmq.resource.ownerId:

  srcDataSources:
    defaultDS: 
          url: jdbc:mysql://192.168.56.77:3306/canal_demo?useUnicode=true
          username: copy
          password: copy888
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
#      - name: rdb
#        key: mysql1
#        properties:
#          jdbc.driverClassName: com.mysql.jdbc.Driver
#          jdbc.url: jdbc:mysql://127.0.0.1:3306/mytest2?useUnicode=true
#          jdbc.username: root
#          jdbc.password: 121212
#      - name: rdb
#        key: oracle1
#        properties:
#          jdbc.driverClassName: oracle.jdbc.OracleDriver
#          jdbc.url: jdbc:oracle:thin:@localhost:49161:XE
#          jdbc.username: mytest
#          jdbc.password: m121212
#      - name: rdb
#        key: postgres1
#        properties:
#          jdbc.driverClassName: org.postgresql.Driver
#          jdbc.url: jdbc:postgresql://localhost:5432/postgres
#          jdbc.username: postgres
#          jdbc.password: 121212
#          threads: 1
#          commitSize: 3000
#      - name: hbase
#        properties:
#          hbase.zookeeper.quorum: 127.0.0.1
#          hbase.zookeeper.property.clientPort: 2181
#          zookeeper.znode.parent: /hbase
      - name: es7
        hosts: 192.168.56.77:9200 # 127.0.0.1:9200 for rest mode
        key: es7key
        properties:
          mode: rest # or rest
          #          # security.auth: test:123456 #  only used for rest mode
          cluster.name: my-application
#        - name: kudu
#          key: kudu
#          properties:
#            kudu.master.address: 127.0.0.1 # ',' split multi address
~~~
在/home/canal/canal_adapter/conf/es7/目录下新建一个配置文件user_info.yml
> vi /home/canal/canal_adapter/conf/es7/user.yml
~~~
dataSourceKey: defaultDS
outerAdapterKey: es7key
destination: example
groupId: g1
esMapping:
  _index: index_canal3
  _id: _id
  _type: _doc
  sql: "select id as _id,id,name,title from user"
  commitBatch: 3000
~~~
启动canal_adapter进行同步
> /home/canal/canal_adapter/bin/startup.sh


curl http://localhost:8081/etl/es7/es7key/user.yml -X POST

curl http://localhost:8081/etl/es7/user.yml -X POST











































