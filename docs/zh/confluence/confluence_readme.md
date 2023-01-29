---
title: confluence_readme
date: 2023-01-28 05:25:44
permalink: /pages/72534e/
categories:
  - zh copy
  - confluence
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---

### docker安装
docker network create traefik

chown -R daemon:daemon ./data/confluence

docker-compose up confluence

###### 破解
docker cp  confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar  ./atlassian-extras-2.4.jar

记录id例如：BXZU-P6DZ-4HZO-8VKX

执行破解之后的文件再copy进去
docker cp   ./atlassian-extras-2.4.jar   confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar
####### 重启
docker restart confluence


连接mysql报错：执行下面语句
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;





### 普通安装
wget https://product-downloads.atlassian.com/software/confluence/downloads/atlassian-confluence-7.13.0-x64.bin
chmod +x atlassian-confluence-7.13.0-x64.bin
./atlassian-confluence-7.13.0-x64.bin

B979-A0YC-Q77L-T9JZ

AAABKA0ODAoPeJxtkMluwjAQhu9+Cks9GyUB6gbJUl0nB9ostAkHenPdCUQKTvCCyts3bJeqx1n+T
9/MQ+0BVzDggOIwXsyCRUSxqGocBVGIErDKtINre81Er5vOg1aACr//AlM2awvGMhIiYUCelxLpg
J2TJIhJQNGYcVK5Qu6BNaC3dieRGjmTsdkegTnj4b6U5rLtWEindPY0jx+fD4eJ6vcoPcrOX+Csk
Z2Faz5rFWgL9WmAC1yUeZ5+iCXP0IjRDrQcRdOfoTWnq9R0SkkYkWh+BdxPEJ23DkzRf4NlAarSg
m3KNc75W4rzFHNc8QSveJHwCSrNVurW3mRu51RgjmCWCXuJaUx4sBHkndKM1PHrJ7ppjtNsmdyr/
61W3qidtPDnhb/JnYU3MCwCFE+neYiNjZ4n/Viegtki32noop5yAhRd8y9L5ZJVZS7y2Ac4NHdVD
2qUXA==X02eu

service restart confluence

vim /opt/atlassian/confluence/bin/setenv.sh
~~~
# Set the Java heap size
CATALINA_OPTS="-Xms256m -Xmx256m ${CATALINA_OPTS}"

# Default values for small to medium size instances
CATALINA_OPTS="-XX:ReservedCodeCacheSize=64m ${CATALINA_OPTS}"
~~~


scp  -v -r -P 1357 /var/atlassian/application-data/confluence/plugins-osgi-cache/felix/felix-cache/ root@106.13.25.179:/var/atlassian/application-data/confluence/plugins-osgi-cache/felix/
### confluence迁移
步骤一
#### 拷贝旧Confluence上 /var/atlassian/ 文件夹下的数据到新服务器上
scp -r /var/atlassian/    root@106.13.25.179:/var/

#### 拷贝旧Confluence 上 /opt/atlassian/ 文件夹下的数据到新服务器上
scp -r /opt/atlassian/    root@106.13.25.179:/opt/

#### 拷贝旧 Confluence 上mysql数据
scp -r /usr/local/mysql/   root@106.13.25.179:/usr/local/mysql/

#### 拷贝 Confluence 启动文件
scp -r /etc/init.d/confluence   root@106.13.25.179:/etc/init.d/confluence


步骤二
#### 新服务器上创建 confluence 和 mysql 用户
useradd -u 501 mysql
useradd -i 502 confluence

#### 给 confluence 目录 和 mysql 目录授权
chown -R mysql.mysql /usr/local/mysql
chown -R confluence.confluence /opt/atlassian/confluence/
chown -R confluence.confluence /var/atlassian/
chown -R confluence.confluence /etc/init.d/confluence

#### 修改 confluence rewrite 配置，将下边的域名改成自己迁移后要用的域名
vim /opt/atlassian/confluence/conf/server.xml 
        <Connector className="org.apache.coyote.tomcat4.CoyoteConnector" port="8090" minProcessors="5"
                   maxProcessors="75"
                   enableLookups="false" redirectPort="8443" acceptCount="10" debug="0" connectionTimeout="20000"
                   scheme="https"
                   proxyName="cwiki.xxxx.com"
                   proxyPort="443"
                   useURIValidationHack="false" URIEncoding="UTF-8"/>

#### 启动服务
/etc/init.d/confluence start
#### 这里我的mysql是自己编译安装的，机器的默认配置都差不多，所以拷贝过来可以直接启动
/usr/local/mysql/bin/mysqld_safe --defaults-file=/usr/local/mysql/etc/my.cnf --user=mysql&

























































