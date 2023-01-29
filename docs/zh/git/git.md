---
title: git
date: 2023-01-28 05:25:48
permalink: /pages/a986bf/
categories:
  - zh copy
  - git
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
Git使用心得
## 官网

[https://git-scm.com/doc](https://git-scm.com/doc)
===========================
## 克隆库

> git clone 'https://github.com/ifengshai/PHPMailer.git'

## 没有版本控制的，添加文件commit可以

```shell
git commit -m "sdfsf"
## 有新增文件使用：
git add .
git commit -m 'message' -a
```

##指定本地master绑定远程的master，再进行pull和push
```shell
git branch --set-upstream-to=origin/master master
git pull
git push
```
## 避免每次都输账号密码
```shell
git config --global credential.helper store
```
## 修改git缓冲区大小

.git文件夹下conf添加以下配置

```
[core]
	packedGitLimit = 1777m 
	packedGitWindowSize = 1777m 
[pack]
	deltaCacheSize = 1777m 
	packSizeLimit = 1777m 
	windowMemory = 1777m 
[http]
	postBuffer = 924288000
```

## 强制使用远程库文件，覆盖本地文件

```shell
git fetch --all     #只是下载远程的库的内容，不做任何的合并 

git reset --hard origin/master   #把HEAD指向刚刚下载的最新的版本

git pull
```



## .gitignore规则不生效

.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。
解决方法就是先把本地缓存删除（改变成未track状态），然后再提交。

```shell
git rm -r --cached .

git add .

git commit -m 'update .gitignore'
```

## 回退上个版本,一个^退一个版本，HEAD~100退100个版本

> git reset --hard HEAD^

## 获取所有git操作（回退版本错误可根据此命令回滚操作）

> git reflog

## 拉取代码自动合并

> git pull origin master

## 后面加上 --allow-unrelated-histories ， 把两段不相干的 分支进行强行合并

> git pull origin master --allow-unrelated-histories

## Git强制提交

> git push origin master --force

## 关闭https 解决报错443问题

> git config --global http.sslVerify false

## 代理问题取消代理  解决问题unable to access

> git config --global --unset http.proxy 

> git config --global --unset https.proxy 

## 添加代理端口

```shell
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy http://127.0.0.1:1080
```

## 拉取指定远程分支到本地分支

```shell
git checkout -b test origin/test
git checkout -b feature/1002636 origin/feature/1002636
git checkout -b feature/1000743_paypal_subscribe origin/feature/1000743_paypal_subscribe
git checkout -b feature/1002698 origin/feature/1002698
```

## 创建分支
```shell
#查看当前分支
git branch
#创建本地分支并切换到新创建的分支
git checkout -b fengsha
#将新创建的分支信息推送到github
git push origin HEAD -u
#切换到master分支
git checkout master
```
## 删除分支

```shell
git branch -D test
```

## 回滚单个文件
```shell
#首先要找到要回滚文件的版本的hash值，commit的hash值是 2d1ed0e066fd9fde6aef913c102fd808e86161fa
git log main.js 
#利用 hash 回滚特定文件到特定版本，注意，这里为了方便操作，使用 hash 的前六位就可以
git checkout 2d1ed0 main.js 
```
## 添加标签

创建标签，自动把创建tag标签添加到最近的commit_id上
```shell
git tag v1.1
#推送所有的本地tag到库里面
git push origin --tags
#删除tag标签
git tag -d v1.1
```

## 拉取远程所有分支
```shell
git branch -r | grep -v '\->' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done
git fetch --all
git pull --all
```

## 迁移仓库
```shell
git clone --mirror <URL to my OLD repo location>
cd <New directory where your OLD repo was cloned>
git remote set-url origin <URL to my NEW repo location>
git push -f origin
```

## 更换远程仓库之后拉取不到分支，使用下面命令拉取

```shell
git remote set-url origin  git@firmiana-gitlab.yunniao.cn:beeper/beeper.git
git remote rm origin
git remote add origin git@firmiana-gitlab.yunniao.cn:beeper/beeper.git
git fetch --all
git branch -avv
git checkout -b fs_change1 origin/fs_change1
git remote set-url origin git@firmiana-gitlab.yunniao.cn:beeper/spring_cloud_config.git
```

## github使用报错

ssh: connect to host github.com port 22: Connection timed out

下面报错

ssh -T git@github.com

下面正常

ssh -T -p 443 git@ssh.github.com

在.ssh文件夹下建立文件config

```yml
Host github.com
 Hostname ssh.github.com
 Port 443
```



## 生成ssh命令

```shell
ssh-keygen -t rsa -C "你在gitee/github/gitlab上注册帐号时填写的邮箱"
```
