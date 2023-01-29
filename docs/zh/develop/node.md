---
title: node
date: 2023-01-28 05:25:47
permalink: /pages/dbdf51/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---


```shell
#安装npm
wget https://nodejs.org/dist/v16.14.1/node-v16.14.1-linux-x64.tar.xz
tar -xvf node-v16.14.1-linux-x64.tar.xz
mv node-v16.14.1-linux-x64 /docker_data/node
cd /docker_data/node

ln -s /docker_data/node/bin/node /usr/bin/node
ln -s /docker_data/node/bin/npm /usr/bin/npm

npm install -g yarn
npm install -g typescript
npm install --g lerna

ln -s /docker_data/node/bin/npx /usr/bin/npx
ln -s /docker_data/node/bin/yarn /usr/bin/yarn
npm install -g npm@8.8.0

#安装淘宝源
npm install -g cnpm --registry=https://registry.npm.taobao.org
ln -s /docker_data/node/bin/cnpm /usr/bin/cnpm
#安装vue,初始化vue项目
cnpm install vue@next
cnpm install -g @vue/cli
ln -s /docker_data/node/bin/vue /usr/bin/vue
vue init webpack zhupu
cd zhupu && npm install && npm run dev

#安装python
yum -y groupinstall "Development tools"
yum -y install zlib zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel xz-devel libffi-devel
wget https://www.python.org/ftp/python/3.9.9/Python-3.9.9.tgz
tar -zxvf Python-3.9.9.tgz
cd Python-3.9.9
./configure --prefix=/usr/local/python3
make && make install
ln -s /usr/local/python3/bin/python3.9 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3.9 /usr/bin/pip3
```

https://24kcs.github.io/vue3_study/00_%E8%AF%BE%E7%A8%8B%E4%BB%8B%E7%BB%8D.html

https://gitee.com/qiuaiyun/vite-vue3-ts-volar/blob/master/vite+vue3%E6%9C%80%E6%96%B0%E6%8A%80%E6%9C%AF%E6%A0%88%E6%96%87%E6%A1%A3/2021%E5%B9%B4%E6%9C%80%E6%96%B0%E5%AE%8C%E6%95%B4vite+vue3+vuex4.0%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98.md







































































