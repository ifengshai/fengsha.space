---
title: supervisor
date: 2023-01-28 05:25:48
permalink: /pages/39844c/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
## php使用supervisor管理进程脚本
supervisor是用python开发的一个在linux系统下的进程管理工具，可以方便的监听，启动，停止一个或多个进程。当一个进程被意外杀死后，supervisor监听到后，会自动重新拉起进程。
### supervisor的安装配置
#### 1、通过pip安装
~~~shell
yum -y install epel-release
yum -y install python-pip
pip install supervisor
~~~
安装好后，会生成三个执行命令，echo_supervisord_conf，supervisorctl，supervisord。
### supervisord配置
Supervisor 相当强大，提供了很丰富的功能，不过我们可能只需要用到其中一小部分。安装完成之后，可以编写配置文件，来满足自己的需求。为了方便，我们把配置分成两部分：
- server端：supervisord(supervisor 是一个 C/S 模型的程序。)
- client端：supervisorctl和应用程序（即我们要管理的程序）。

supervisor的默认配置文件在 /etc/supervisord.conf 下。安装完 supervisor 之后，可以运行echo_supervisord_conf 命令输出默认的配置项，也可以重定向到一个配置文件里：
~~~shell
echo_supervisord_conf > /etc/supervisord.conf
~~~
#### 常用的配置项如下：
~~~conf
[unix_http_server]
file=/var/lock/supervisor.sock   ; unix socket文件，supervisorctl会使用,防止文件被删除自己建个文件夹
;chmod=0700                 ; socket文件权限
;chown=nobody:nogroup       ; socket文件所属用户和用户组
 
[inet_http_server]          ; web管理界面
port=*:9001         ; 管理界面的IP和端口*可指定ip
username=admin              ; 登陆管理界面的用户名
password=123456             ; 登陆管理界面的密码
 
[supervisord]
logfile=/tmp/supervisord.log ; 日志文件
logfile_maxbytes=50MB        ; 日志文件大小，为0表示不限制
logfile_backups=10           ; 日志文件备份数量，为0表示不备份
loglevel=info                ; 日志级别，也可设置为 debug,warn,trace
pidfile=/tmp/supervisord.pid ; PID文件路径
nodaemon=false               ; 是否前台启动，为false表示守护进程方式
minfds=1024                  ; 打开文件描述符的最小值
minprocs=200                 ; 创建进程数的最小值
 
[supervisorctl]
serverurl=unix:///var/lock/supervisor.sock ; 通过 unix sokcet 连接supervisord
;serverurl=http://127.0.0.1:9001 ; 通过http方式连接supervisord
 
[include]                      ;注意这个地方也要取消备注
files=/etc/supervisord.d/*.conf ; 包含其他配置文件，可以是.conf或.ini
~~~
### 配置supervisord服务开机启动
管理进程，需要我们启动 supervisor 服务，这里我们配置 systemctl，开机自动启动 supervisor。
创建/etc/init.d/supervisord文件.
~~~shell
vi /etc/init.d/supervisord
~~~
内容如下
~~~conf
#!/bin/sh
#
# /etc/rc.d/init.d/supervisord
#
# Supervisor is a client/server system that
# allows its users to monitor and control a
# number of processes on UNIX-like operating
# systems.
#
# chkconfig: - 64 36
# description: Supervisor Server
# processname: supervisord

# Source init functions
. /etc/init.d/functions

RETVAL=0
prog="supervisord"
pidfile="/tmp/supervisord.pid"
lockfile="/var/lock/supervisor.sock"

start()
{
        echo -n $"Starting $prog: "
        daemon --pidfile $pidfile supervisord -c /etc/supervisord.conf
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && touch ${lockfile}
}

stop()
{
        echo -n $"Shutting down $prog: "
        killproc -p ${pidfile} /usr/bin/supervisord
        RETVAL=$?
        echo
        if [ $RETVAL -eq 0 ] ; then
                rm -f ${lockfile} ${pidfile}
        fi
}

case "$1" in

  start)
    start
  ;;

  stop)
    stop
  ;;

  status)
        status $prog
  ;;

  restart)
    stop
    start
  ;;

  *)
    echo "Usage: $0 {start|stop|restart|status}"
  ;;

esac
~~~
#### 设置开机启动
~~~
chmod +x /etc/init.d/supervisord
chkconfig --add supervisord
chkconfig supervisord on
service supervisord start
systemctl daemon-reload
~~~
#### 启动 supervisord
~~~
systemctl start supervisord
~~~

### 配置一个脚本进程
我们需要把 [include] 前面的注释打开，并配置 files 的路径。
创建 files 中配置的目录。
~~~
mkdir -p /etc/supervisord.d/
~~~
我们在 /etc/supervisord.d/ 目录下创建一个 demo.conf 文件。
~~~shell
vi /etc/supervisord.d/demo.conf
~~~
里面添加内容
~~~conf
[program:demo]
directory=/data/wwwroot ; 程序的启动目录
command=php demo.php    ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart=true          ; 在 supervisord 启动的时候也自动启动
startsecs=5             ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart=true        ; 程序异常退出后自动重启
startretries=1          ; 启动失败自动重试次数，默认是 3
user=root               ; 用哪个用户启动
redirect_stderr=true    ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups=20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/data/logs/supervisord.demo.stdout.log
;错误日志文件
stderr_logfile=/data/logs/supervisord.demo.stderr.log
; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
;环境变量
;environment=PATH="/data/nmp/php/bin/"
~~~
demo.php的代码如下：
~~~php
<?php
$i = 0;
while(true) {
    $i++;
    echo $i, PHP_EOL;
    sleep(1);
}
~~~

#### 启动，停止，重启，程序。
~~~
supervisorctl start 程序名
supervisorctl stop 程序名
supervisorctl restart 程序名
~~~
成功后，就可以通过supervisorctl交互命令管理进程脚本了。
要开放 9001 端口，并重启 supervisor
~~~
firewall-cmd --zone=public --add-port=9001/tcp --permanent
firewall-cmd --reload
supervisorctl reload
~~~

### 设置监听程序
我们在 /etc/supervisord.d/ 目录下创建一个 demo_listener.conf 文件。
~~~conf
[eventlistener:demolistener]                              ;表明这个是一个关于事件监听者的配置，监听者的进程名称叫做demolistener
directory=/data/wwwroot ; 程序的启动目录
command=php demo_listener.php                             ;监听执行程序
events=PROCESS_STATE_FATAL                                ;监听事件，当前事件意思是：当这个事件发生的时候，意味着supervisor的子进程从BACKOFF 变为了 FATAL状态。意味着supervisor多次（重起次数，看自己的配置）尝试重启子进程，依然失败，决定放弃,官网所有事件介绍http://www.supervisord.org/events.html#event-types
stdout_logfile=/var/log/supervisor/script_log/monitor.log
stderr_logfile=/var/log/supervisor/script_log/monitor@error.log
~~~
#### 报警程序
demo_listener.php文件
~~~
<?php 
	//to do 
	//读取demo程序log文件报错信息，发送到钉钉或者邮箱。
	//......

~~~

