---
title: Mac 添加自定义脚本
date: 2018-05-30 09:19:05
category: Mac
tags: [Mac,shell]
---

# Mac 添加自定义脚本

> 将常用的命令做成脚本，添加到 terminal 会大大提高效率。

### 1、建立脚本文件

建立 xxx.sh 文件，内容为命令，如

```
## 添加 spring boot 默认目录
mkdir Controller Dao Entity Service
```

### 2、添加快捷键

查看bash文件

```
echo $SHELL
```
有可能是 ~/.bash_profile、~/.zshrc等，我的是zsh，因此应该添加到~/.zshrc

```
open ~/.zshrc
```

添加快捷命令，保存文件

> alias mkspdir='.sh文件路径'

重启 terminal，或者更新源

```
source ~/.zshrc
```

此时即可使用快捷命令执行脚本了。


#### 附：登录的一个脚本例子

```
##命令登录ssh 用户名@ip
spawn ssh root@xxx.xxx.xxx.xxx
##这里是执行上一步后希望出现的文字提示，通常是密码输入提示
expect "**password:"
##利用send命令，发送你的server密码并回车即可
send "yourpassword\r"

##最后加上允许交互的命令
interact
```