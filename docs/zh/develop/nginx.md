---
title: nginx
date: 2023-01-28 05:25:47
permalink: /pages/70afd4/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
# 什么是nginx
Nginx（发音同“engine X”）是异步框架的网页服务器，也可以用作反向代理、负载平衡器和HTTP缓存。该软件由伊戈尔·赛索耶夫创建并于2004年首次公开发布。2011年成立同名公司以提供支持。2019年3月11日，Nginx公司被F5 Networks以6.7亿美元收购。
# nginx的应用场景
* [反向代理服务器](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)
* [负载均衡服务器](https://zh.wikipedia.org/wiki/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)
* [HTTP缓存服务器](https://zh.wikipedia.org/wiki/Web%E7%BC%93%E5%AD%98)
* [与PHP集成](https://zh.wikipedia.org/wiki/Nginx#PHP%E6%95%B4%E5%90%88)
# nginx安装
下载`nginx_modules`(ps:编译的时候会用到)
链接: https://pan.baidu.com/s/1MjVdVkF4EAjhPfLImRfWMA  密码: wwgb

```
#!/usr/bin/env bash

DIR=/Users/shiwenyuan/webserver
mkdir -p $DIR
cd $DIR
mkdir run

tar -zxvf /Users/shiwenyuan/totalXbox/project/phpstorm/xlegal/devops/opbin/nginx_modules.tgz -C $DIR

mkdir tmp
cd tmp

wget http://nginx.org/download/nginx-1.8.1.tar.gz -O nginx-1.8.1.tar.gz

tar -zxvf nginx-1.8.1.tar.gz

cd nginx-1.8.1

./configure \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_addition_module \
--add-module=$DIR/nginx_modules/echo-nginx-module-master \
--add-module=$DIR/nginx_modules/headers-more-nginx-module-master \
--add-module=$DIR/nginx_modules/memc-nginx-module-master \
--add-module=$DIR/nginx_modules/nginx-http-concat-master \
--add-module=$DIR/nginx_modules/ngx_devel_kit-master \
--add-module=$DIR/nginx_modules/ngx_http_consistent_hash-master \
--add-module=$DIR/nginx_modules/ngx_http_enhanced_memcached_module-master \
--add-module=$DIR/nginx_modules/ngx_http_upstream_ketama_chash-0.6 \
--add-module=$DIR/nginx_modules/srcache-nginx-module-master \
--with-pcre=$DIR/nginx_modules/pcre-8.38 \
--prefix=$DIR
if [[ $? -ne 0 ]];then
	echo 'error occured\n'
    exit 1
fi
make && make install
cd $DIR
/bin/rm -rf tmp
/bin/rm -rf node_modules

/sbin/nginx -t
```

# nginx命令行常用命令
* nginx  # 启动nginx
* nginx -s reload  # 向主进程发送信号，重新加载配置文件，热重启
* nginx -s reopen	 # 重启 Nginx
* nginx -s stop    # 快速关闭
* nginx -s quit    # 等待工作进程处理完成后关闭
* nginx -t         # 查看当前 Nginx 配置是否有错误
* nginx -t -c <配置路径>    # 检查配置是否有问题，如果已经在配置目录，则不需要-c
# nginx配置文件详解
线上应用常常都是一个nginx上面会配置好几个域名，每个域名都会放到一个单独的配置文件里。然后在nginx.conf中引用这些文件，所以可以理解为每次nginx启动的时候都会默认加载nginx.conf，nginx.conf会把相关的server配置都引用进来形成一个大的nginx文件。

![20200622144019](https://cdn.learnku.com/uploads/images/202006/22/43503/MDHBRdJswh.png!large)

* **main**:全局设置
* **events**:配置影响`Nginx`服务器或与用户的网络连接
* **http**:http模块设置
* **upstream**:负载均衡设置
* **server**:http服务器配置,一个http模块中可以有多个server模块
* **location**:url匹配配置,一个server模块中可以包含多个location模块

一个`nginx`配置文件的结构就像`nginx.conf`显示的那样，配置文件的语法规则：
1. 配置文件由模块组成
2. 使用`#`添加注释
3. 使用`$`使用变量
4. 使用`include`引用多个配置文件

# nginx与php通信
## 访问路径

```

www.example.com/index.php
        |
        |
      Nginx
        |
        |
php-fpm监听127.0.0.1:9000地址
        |
        |
www.example.com/index.php请求转发到127.0.0.1:9000
        |
        |
nginx的fastcgi模块将http请求映射为fastcgi请求
        |
        | 
  php-fpm监听fastcgi请求
        |
        | 
php-fpm接收到请求，并通过worker进程处理请求
        |
        | 
php-fpm处理完请求，返回给nginx
        |
        | 

```

## nginx与php通信方式
### tcp-socket
tcp socket通信方式，需要在nginx配置文件中填写php-fpm运行的ip地址和端口号，该方式支持跨服务器，即nginx和php-fpm不再同一机器上时。

```
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
}
```

### unix-socket
unix socket通信方式，需要在nginx配置文件中填写php-fpm运行的pid文件地址。unix socket又叫IPC（inter process communication进程间通信）socket，用于实现统一主机上进程间通信。

```
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
}
```

### 两者间的区别
Unix socket 不需要经过网络协议栈，不需要打包拆包、计算校验和、维护序号和应答等，只是将应用层数据从一个进程拷贝到另一个进程。所以其效率比 tcp socket 的方式要高，可减少不必要的 tcp 开销。不过，unix socket 高并发时不稳定，连接数爆发时，会产生大量的长时缓存，在没有面向连接协议的支撑下，大数据包可能会直接出错不返回异常。而 tcp 这样的面向连接的协议，可以更好的保证通信的正确性和完整性。

所以，如果面临的是高并发业务，则考虑优先使用更可靠的tcp socket，我们可以通过负载均衡、内核优化等手段来提供效率。
# nginx配置动静分离
## 什么是动静分离
在Web开发中，通常来说，动态资源其实就是指那些后台资源，而静态资源就是指HTML，JavaScript，CSS，img等文件。
在使用前后端分离之后，可以很大程度的提升静态资源的访问速度，同时在开发过程中也可以让前后端开发并行可以有效的提高开发时间，也可以有效的减少联调时间 。
## 动静分离方案
* 直接使用不同的域名，把静态资源放在独立的云服务器上，这个种方案也是目前比较推崇的。
* 动态请求和静态文件放在一起，通过nginx配置分开

```
server {
  location /www/ {
  	root /www/;
    index index.html index.htm;
  }
  
  location /image/ {
  	root /image/;
  }
}

```

# nginx配置反向代理
反向代理常用于不想把端口暴露出去，直接访问域名处理请求。

```
server {
    listen    80;
    server_name www.phpblog.com.cn;
    location /swoole/ {
        proxy_pass http://127.0.0.1:9501;
    }
    location /node/ {
        proxy_pass http://127.0.0.1:9502;
    }
        
}
```

# nginx配置负载均衡

```
upstream phpServer{
    server 127.0.0.1:9501;
    server 127.0.0.1:9502;
    server 127.0.0.1:9503;
}
server {
    listen    80;
    server_name www.phpblog.com.cn;
    location / {
        proxy_pass http://phpServer;
        proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_next_upstream error timeout invalid_header;
        proxy_max_temp_file_size 0;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
    }
}
```

## 常见的负载均衡策略
### round-robin/轮询： 到应用服务器的请求以round-robin/轮询的方式被分发

```
upstream phpServer{
    server 127.0.0.1:9501 weight=3;
    server 127.0.0.1:9502;
    server 127.0.0.1:9503;
}
```

在这个配置中，每5个新请求将会如下的在应用实例中分派： 3个请求分派去9501,一个去9502,另外一个去9503.
### least-connected/最少连接：下一个请求将被分派到活动连接数量最少的服务器

```
upstream phpServer{
    least_conn;
    server 127.0.0.1:9501;
    server 127.0.0.1:9502;
    server 127.0.0.1:9503;
}
```

当某些请求需要更长时间来完成时，最少连接可以更公平的控制应用实例上的负载。

### ip-hash/IP散列： 使用hash算法来决定下一个请求要选择哪个服务器(基于客户端IP地址)

```
upstream phpServer{
    ip_hash;
    server 127.0.0.1:9501;
    server 127.0.0.1:9502;
    server 127.0.0.1:9503;
}
```

将一个客户端绑定给某个特定的应用服务器;
# nginx配置跨域
由于浏览器同源策略的存在使得一个源中加载来自其它源中资源的行为受到了限制。即会出现跨域请求禁止。
所谓同源是指：域名、协议、端口相同。

![20200622140049](https://cdn.learnku.com/uploads/images/202006/22/43503/9tc85FiS98.jpeg!large)

```
server {
		listen       80;
		server_name  www.phpblog.com.cn;
		root   /Users/shiwenyuan/blog/public;
		index  index.html index.htm index.php;
		location / {
			try_files $uri $uri/ /index.php?$query_string;
		}
        add_header 'Access-Control-Allow-Origin' "$http_origin";
        add_header 'Access-Control-Allow-Credentials' 'true';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, DELETE, PUT, PATCH';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,X-XSRF-TOKEN';

		location ~ \.php$ {
			fastcgi_pass   127.0.0.1:9000;
			fastcgi_index  index.php;
			fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include        fastcgi_params;
		}
		error_page  404              /404.html;
		error_page   500 502 503 504  /50x.html;
		location = /50x.html {
		    root   html;
		}
}
```

* Access-Control-Allow-Origin：允许的域名，只能填 *（通配符）或者单域名。
* Access-Control-Allow-Methods: 允许的方法，多个方法以逗号分隔。
* Access-Control-Allow-Headers: 允许的头部，多个方法以逗号分隔。
* Access-Control-Allow-Credentials: 是否允许发送Cookie。
# nginx配置https
此处可以参考我之前写的一篇文章
[nginx配置https证书认证](https://shiwenyuan.github.io/post/ck9so9sli000q1370k0dfisi8.html)
# nginx伪静态
## 应用场景
* seo优化
* 安全
* 流量转发

```
location ^~ /saas {
    root /home/work/php/saas/public;
    index index.php;
    rewrite ^/saas(/[^\?]*)?((\?.*)?)$ /index.php$1$2 last;
    break;
}
```

# 日常工作中的奇淫技巧
## 日志切割脚本

```
#!/bin/bash
#设置你的日志存放的目录
log_files_path="/mnt/usr/logs/"
#日志以年/月的目录形式存放
log_files_dir=${log_files_path}"backup/"
#设置需要进行日志分割的日志文件名称，多个以空格隔开
log_files_name=(access.log error.log)
#设置nginx的安装路径
nginx_sbin="/mnt/usr/sbin/nginx -c /mnt/usr/conf/nginx.conf"
#Set how long you want to save
save_days=10

############################################
#Please do not modify the following script #
############################################
mkdir -p $log_files_dir

log_files_num=${#log_files_name[@]}
#cut nginx log files
for((i=0;i<$log_files_num;i++));do
    mv ${log_files_path}${log_files_name[i]} ${log_files_dir}${log_files_name[i]}_$(date -d "yesterday" +"%Y%m%d")
done
$nginx_sbin -s reload
```

## 图片防盗链

```
server {
  listen       80;        
  server_name  *.phpblog.com.cn;
  
  # 图片防盗链
  location ~* \.(gif|jpg|jpeg|png|bmp|swf)$ {
    valid_referers none blocked server_names ~\.google\. ~\.baidu\. *.qq.com;
    if ($invalid_referer){
      return 403;
    }
  }
}
```

## nginx访问控制

```
location ~ \.php$ {
    allow 127.0.0.1;  #只允许127.0.0.1的访问，其他均拒绝
    deny all;
	fastcgi_pass   127.0.0.1:9000;
	fastcgi_index  index.php;
	fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
	include        fastcgi_params;
}
```

## 丢弃不受支持的文件扩展名的请求

```
location ~ \.(js|css|sql)$ {
    deny all;
}
```

# nginx常用配置

配置文件主要由四部分组成：main(全区设置)，server(主机配置)，upstream(负载均衡服务器设置)，和location(URL匹配特定位置设置)。

## 全局变量

~~~
#Nginx的worker进程运行用户以及用户组
#user  nobody nobody;
#Nginx开启的进程数
worker_processes  1;
#worker_processes auto;
#以下参数指定了哪个cpu分配给哪个进程，一般来说不用特殊指定。如果一定要设的话，用0和1指定分配方式.
#这样设就是给1-4个进程分配单独的核来运行，出现第5个进程是就是随机分配了。eg:
#worker_processes 4     #4核CPU 
#worker_cpu_affinity 0001 0010 0100 1000;
nets    
#定义全局错误日志定义类型，[debug|info|notice|warn|crit]
#error_log  logs/error.log  info;
#指定进程ID存储文件位置
#pid        logs/nginx.pid;
#一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n的值保持一致。
#vim /etc/security/limits.conf
#  *                soft    nproc          65535
#  *                hard    nproc          65535
#  *                soft    nofile         65535
#  *                hard    nofile         65535
worker_rlimit_nofile 65535;
~~~

## 事件配置

~~~
events {
    #use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
    use epoll;
    #每个进程可以处理的最大连接数，理论上每台nginx服务器的最大连接数为worker_processes*worker_connections。理论值：worker_rlimit_nofile/worker_processes
    #注意：最大客户数也由系统的可用socket连接数限制（~ 64K），所以设置不切实际的高没什么好处
    worker_connections  65535;    
    #worker工作方式：串行（一定程度降低负载，但服务器吞吐量大时，关闭使用并行方式）
    #multi_accept on; 
}
~~~

## http参数

~~~
   #文件扩展名与文件类型映射表
    include mime.types;
    #默认文件类型
    default_type application/octet-stream;
 
#日志相关定义
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #定义日志的格式。后面定义要输出的内容。
    #1.$remote_addr 与$http_x_forwarded_for 用以记录客户端的ip地址；
    #2.$remote_user ：用来记录客户端用户名称；
    #3.$time_local ：用来记录访问时间与时区；
    #4.$request  ：用来记录请求的url与http协议；
    #5.$status ：用来记录请求状态； 
    #6.$body_bytes_sent ：记录发送给客户端文件主体内容大小；
    #7.$http_referer ：用来记录从那个页面链接访问过来的；
    #8.$http_user_agent ：记录客户端浏览器的相关信息
    #连接日志的路径，指定的日志格式放在最后。
    #access_log  logs/access.log  main;
    #只记录更为严重的错误日志，减少IO压力
    error_log logs/error.log crit;
    #关闭日志
    #access_log  off;
 
    #默认编码
    #charset utf-8;
    #服务器名字的hash表大小
    server_names_hash_bucket_size 128;
    #客户端请求单个文件的最大字节数
    client_max_body_size 8m;
    #指定来自客户端请求头的hearerbuffer大小
    client_header_buffer_size 32k;
    #指定客户端请求中较大的消息头的缓存最大数量和大小。
    large_client_header_buffers 4 64k;
    #开启高效传输模式。
    sendfile on;
    #防止网络阻塞
    tcp_nopush on;
    tcp_nodelay on;    
    #客户端连接超时时间，单位是秒
    keepalive_timeout 60;
    #客户端请求头读取超时时间
    client_header_timeout 10;
    #设置客户端请求主体读取超时时间
    client_body_timeout 10;
    #响应客户端超时时间
    send_timeout 10;
 
#FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
 
#gzip模块设置
    #开启gzip压缩输出
    gzip on; 
    #最小压缩文件大小
    gzip_min_length 1k; 
    #压缩缓冲区
    gzip_buffers 4 16k;
    #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_http_version 1.0;
    #压缩等级 1-9 等级越高，压缩效果越好，节约宽带，但CPU消耗大
    gzip_comp_level 2;
    #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_types text/plain application/x-javascript text/css application/xml;
    #前端缓存服务器缓存经过压缩的页面
    gzip_vary on;
~~~

## 虚拟主机基本设置

~~~
#虚拟主机定义
    server {
        #监听端口
        listen       80;
        #访问域名
        server_name  localhost;
        #编码格式，若网页格式与此不同，将被自动转码
        #charset koi8-r;
        #虚拟主机访问日志定义
        #access_log  logs/host.access.log  main;
        #对URL进行匹配
        location / {
            #访问路径，可相对也可绝对路径
            root   html;
            #首页文件。以下按顺序匹配
            index  index.html index.htm;
        }
 
#错误信息返回页面
        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 
#访问URL以.php结尾则自动转交给127.0.0.1
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
#php脚本请求全部转发给FastCGI处理
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
 
#禁止访问.ht页面 （需ngx_http_access_module模块）
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
#HTTPS虚拟主机定义
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
#vue配置
    server {
        listen       80;
        server_name  jcsd-cdn-monitor.jdcloud.com;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        root /root/dist;
 
        location / {
            try_files $uri $uri/ /index.html;
        }
 
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
~~~

## Nignx状态监控

~~~
#Nginx运行状态，StubStatus模块获取Nginx自启动的工作状态（编译时要开启对应功能）
        #location /NginxStatus {
        #    #启用StubStatus的工作访问状态    
        #    stub_status    on;
        #    #指定StubStaus模块的访问日志文件 可off
        #    access_log    logs/Nginxstatus.log;
        #    #Nginx认证机制（需Apache的htpasswd命令生成）
        #    #auth_basic    "NginxStatus";
        #    #用来认证的密码文件
        #    #auth_basic_user_file    ../htpasswd;    
        #}
~~~

## 反向代理

~~~
#以下配置追加在HTTP的全局变量中

#启动代理缓存功能
proxy_buffering on;
#nginx跟后端服务器连接超时时间(代理连接超时)
proxy_connect_timeout      5;
#后端服务器数据回传时间(代理发送超时)
proxy_send_timeout         5;
#连接成功后，后端服务器响应时间(代理接收超时)
proxy_read_timeout         60;
#设置代理服务器（nginx）保存用户头信息的缓冲区大小
proxy_buffer_size          16k;
#proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
proxy_buffers              4 32k;
#高负荷下缓冲大小（proxy_buffers*2）
proxy_busy_buffers_size    64k;
#设定缓存文件夹大小，大于这个值，将从upstream服务器传
proxy_temp_file_write_size 64k;
#反向代理缓存目录
proxy_cache_path /data/proxy/cache levels=1:2 keys_zone=cache_one:500m inactive=1d max_size=1g;
#levels=1:2 设置目录深度，第一层目录是1个字符，第2层是2个字符
#keys_zone:设置web缓存名称和内存缓存空间大小
#inactive:自动清除缓存文件时间。
#max_size:硬盘空间最大可使用值。
#指定临时缓存文件的存储路径(必须在同一分区)
proxy_temp_path /data/proxy/temp;
 
#服务配置
server {
    #侦听的80端口
    listen       80;
    server_name  localhost;
    location / {
        #反向代理缓存设置命令(proxy_cache zone|off,默认关闭所以要设置)
        proxy_cache cache_one;
        #对不同的状态码缓存不同时间
        proxy_cache_valid 200 304 12h;
        #设置以什么样参数获取缓存文件名
        proxy_cache_key $host$uri$is_args$args;
        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
        #代理设置
        proxy_pass   http://IP; 
        #文件过期时间控制
        expires    1d;
    }
    #配置手动清楚缓存(实现此功能需第三方模块 ngx_cache_purge)
    #http://www.123.com/2017/0316/17.html访问
    #http://www.123.com/purge/2017/0316/17.html清楚URL缓存
    location ~ /purge(/.*) {
        allow    127.0.0.1;
        deny    all;
        proxy_cache_purge    cache_one    $host$1$is_args$args;
    }
    #设置扩展名以.jsp、.php、.jspx结尾的动态应用程序不做缓存
    location ~.*\.(jsp|php|jspx)?$ { 
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
        proxy_pass http://IP;
    }
~~~

## 负载均衡

~~~
#负载均衡服务器池
upstream my_server_pool {
    #调度算法
    #1.轮循（默认）（weight轮循权值）
    #2.ip_hash：根据每个请求访问IP的hash结果分配。（会话保持）
    #3.fair:根据后端服务器响应时间最短请求。（upstream_fair模块）
    #4.url_hash:根据访问的url的hash结果分配。（需hash软件包）
    #参数：
    #down：表示不参与负载均衡
    #backup:备份服务器
    #max_fails:允许最大请求错误次数
    #fail_timeout:请求失败后暂停服务时间。
    server 192.168.1.109:80 weight=1 max_fails=2 fail_timeout=30;
    server 192.168.1.108:80 weight=2 max_fails=2 fail_timeout=30;
}
#负载均衡调用
server {
    ...
    location / {
    proxy_pass http://my_server_pool;
    }
}
~~~

## URL重写

~~~
#根据不同的浏览器URL重写
if($http_user_agent ~ Firefox){
rewrite ^(.*)$  /firefox/$1 break; 
}
if($http_user_agent ~ MSIE){
rewrite ^(.*)$  /msie/$1 break; 
}
 
#实现域名跳转
location / {
    rewrite ^/(.*)$ https://web8.example.com$1 permanent;
}
~~~

## IP限制

~~~
#限制IP访问
location / {
    deny 192.168.0.2；
    allow 192.168.0.0/24;
    allow 192.168.1.1;
    deny all;
}
~~~

# 高并发nginx需要配置：

## 服务器运行下面命令

### 允许等待中的监听

> #echo 50000 >/proc/sys/net/core/somaxconn  

#### tcp连接快速回收

> #echo 1 >/proc/sys/net/ipv4/tcp_tw_recycle  

### tcp连接重用 

> #echo 1 >/proc/sys/net/ipv4/tcp_tw_reuse   

#### 不抵御洪水攻击

> #echo 0 >/proc/sys/net/ipv4/tcp_syncookies

#### 设置每个用户最大允许打开文件数量

> #vi /etc/security/limits.conf

~~~
 * soft nofile 100000  
 * hard nofile 100000
~~~

## nginx.conf 进行修改优化

### 查看cpu核数

> #lscpu

> #vi /usr/local/nginx/conf/nginx.config

~~~
user www www;
worker_processes 2;#官方的建议是修改成CPU的内核数，实践表明，nginx的这个参数在一般情况下开4个或8个就可以了，nginx开启太多的进程，会影响主进程调度，所以占用的cpu会增高

error_log /data/wwwlogs/error_nginx.log crit;
pid /var/run/nginx.pid;
worker_rlimit_nofile 50000;#理论值应该是系统的最多打开文件数（ulimit-n）与nginx进程worker_processes数相除

events {
  use epoll;
  worker_connections 50000; #设置worker进程最大打开的连接数,理论值：worker_rlimit_nofile/worker_processes
  multi_accept on;
}

http {
  include mime.types;
  default_type application/octet-stream;
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 1024m;
  client_body_buffer_size 10m;
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 120;#客户端连接超时时间，单位是秒,高并发下设为0  上传文件等长连接要保持连接
  server_tokens off;
  tcp_nodelay on;

  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  fastcgi_intercept_errors on;

  #Gzip Compression
  gzip on;
  gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";

  ##Brotli Compression
  #brotli on;
  #brotli_comp_level 6;
  #brotli_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript image/svg+xml;

  ##If you have a lot of static files to serve through Nginx then caching of the files' metadata (not the actual files' contents) can save some latency.
  #open_file_cache max=1000 inactive=20s;
  #open_file_cache_valid 30s;
  #open_file_cache_min_uses 2;
  #open_file_cache_errors on;
~~~

### 配置服务
~~~
  server {
    listen 80;
    server_name _;
    access_log /data/wwwlogs/access_nginx.log combined;
    root /data/wwwroot/default;
    index index.html index.htm index.php;
    #error_page 404 /404.html;
    #error_page 502 /502.html;
    location /nginx_status {
      stub_status on;
      access_log off;
      allow 127.0.0.1;
      deny all;
    }
    location ~ [^/]\.php(/|$) {
      #fastcgi_pass remote_php_ip:9000;
      fastcgi_pass unix:/dev/shm/php-cgi.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
      expires 30d;
      access_log off;
    }
    location ~ .*\.(js|css)?$ {
      expires 7d;
      access_log off;
    }
    location ~ ^/(\.user.ini|\.ht|\.git|\.svn|\.project|LICENSE|README.md) {
      deny all;
    }
  }
~~~

### 负载均衡服务器池
~~~
upstream serverpool.com{
    #调度算法
    #1.weight:轮循（默认）（weight轮循权值,权值越大执行的次数越高）
    #2.ip_hash:根据每个请求访问IP的hash结果分配。（会话保持,可解决session问题，需要nginx在最前端分发服务器）
    #3.fair:根据后端服务器响应时间最短请求。（upstream_fair模块）
    #4.url_hash:根据访问的url的hash结果分配。（需hash软件包）
    #参数：
    #down：表示不参与负载均衡
    #backup:备份服务器
    #max_fails:允许最大请求错误次数
    #fail_timeout:请求失败后暂停服务时间。
    #ip_hash;
    server 192.168.6.77:80;
    server 192.168.3.177:80;
    server 192.168.6.55:90;
    server 127.0.0.1:8080;
    #server 192.168.1.109:80 weight=1 max_fails=2 fail_timeout=30;
    #server 192.168.1.108:80 weight=2 max_fails=2 fail_timeout=30;
}
~~~
### 负载均衡调用
~~~
server {
    listen 8080;
    server_name serverpool.com; 
    root /data/wwwroot/serverpool.com;
    location / {
        proxy_pass http://serverpool.com;
    }
}
server {
    listen 90;
    server_name serverpool.com;
    root /data/wwwroot/serverpool.com;
    index index.html index.htm index.php;
  }

  include vhost/*.conf;
}
~~~



















































