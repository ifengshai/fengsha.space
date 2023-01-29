---
title: composer
date: 2023-01-28 05:25:47
permalink: /pages/f27dd1/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
windows下需要配置环境变量，path里添加php.exe路径和php/ext扩展路径 php.ini开启php_openssl.dll扩展，失败请查看extension_dir当前扩展路径地址。 执行：
php -r "readfile('https://getcomposer.org/installer');" | php
新建文件 composer.bat 内容如下：
@php "%~dp0composer.phar" %*
把composer.bat 和 composer.phar 放入 php.exe同级路径 执行：
composer -v


本镜像与 Packagist 官方实时同步，推荐使用最新的 Composer 版本。
最新版本: 1.9.3
下载地址: https://mirrors.aliyun.com/composer/composer.phar

全局配置（推荐）
所有项目都会使用该镜像地址：
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
取消配置：
composer config -g --unset repos.packagist

项目配置
仅修改当前工程配置，仅当前工程可使用该镜像地址：
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
取消配置：
composer config --unset repos.packagist
调试
composer 命令增加 -vvv 可输出详细的信息，命令如下：
composer -vvv require alibabacloud/sdk

安装源码
composer require jaeger/querylist


遇到问题？
1. 建议先将Composer版本升级到最新：
composer self-update
2. 执行诊断命令：
composer diagnose
3. 清除缓存：
composer clear
4. 若项目之前已通过其他源安装，则需要更新 composer.lock 文件，执行命令：
composer update --lock


composer.json文件最初内容

{
  "repositories": {
     "packagist": {
      "type": "composer",
      "url": "https://mirrors.aliyun.com/composer/"
    }
   }
}

更新自动加载
composer dump-autoload
