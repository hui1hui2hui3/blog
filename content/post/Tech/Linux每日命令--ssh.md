---
title: "Linux每日命令--ssh"
date: 2017-04-12
tags: ["Linux"]
draft: false
---

[TOC]

ssh命令用于远程登录上Linux主机。

常用格式：`ssh [-l login_name] [-p port] [user@]hostname`
更详细的可以用`ssh -h`查看。

## SSH config 文件

### 配置文件
SSH 程序可以从以下途径获取配置参数 (Ubuntu 14.04 LTS)：

- 命令行选项
- 用户配置文件 (~/.ssh/config)
- 系统配置文件 (/etc/ssh/ssh_config)

### 常用配置项
下面介绍一些常用的 SSH 配置项：

#### Host
Host 配置项标识配置区段,主要用于ssh登录时使用。
如：
```
HOST home
    ...
    ...
```
登录如下：
```
ssh home
```

#### **GlobalKnownHostsFile**
指定一个或多个全局认证主机缓存文件，用来缓存通过认证的远程主机的密钥，多个文件用空格分隔。默认缓存文件为：/etc/ssh/ssh_known_hosts, /etc/ssh/ssh_known_hosts2.

#### **HostName**
指定远程主机名，可以直接使用数字IP地址。如果主机名中包含 ‘%h’ ，则实际使用时会被命令行中的主机名替换。

#### **IdentityFile**
指定密钥认证使用的私钥文件路径。默认为 ~/.ssh/id_dsa, ~/.ssh/id_ecdsa, ~/.ssh/id_ed25519 或 ~/.ssh/id_rsa 中的一个。文件名称可以使用以下转义符：

    '%d' 本地用户目录
    '%u' 本地用户名称
    '%l' 本地主机名
    '%h' 远程主机名
    '%r' 远程用户名

可以指定多个密钥文件，在连接的过程中会依次尝试这些密钥文件。

#### **Port**
指定远程主机端口号，默认为 22 。

#### **User**
指定登录用户名。

#### **UserKnownHostsFile**
指定一个或多个用户认证主机缓存文件，用来缓存通过认证的远程主机的密钥，多个文件用空格分隔。默认缓存文件为： ~/.ssh/known_hosts, ~/.ssh/known_hosts2.

还有更多参数的介绍，可以参看用户手册：

`$ man ssh config`

### 示例
以下连接为例：

    SSH 服务器(Host)： ssh.test.com
    端口号(Port)： 2200
    账户(User)： user
    密钥文件(IdentityFile)： ~/.ssh/id_rsa_test

####密码认证登录方式为：
```
$ ssh -p 2200 -i ~/.ssh/id_rsa_test user@ssh.test.com
user@ssh.test.com's password:
```
####密钥认证登录方式：

```
$ ssh-copy-id -i ~/.ssh/id_rsa_test user@ssh.test.com
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user@ssh.test.com's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'user@ssh.test.com'"
and check to make sure that only the key(s) you wanted were added.

$ ssh user@ssh.test.com
```
####使用配置文件方式

有如下配置文件：
```
$ vim ~/.ssh/config
Host sshtest
    HostName ssh.test.com
    User user
    Port 2200
    IdentityFile ~/.ssh/id_rsa_test

Host ssttest2
    HostName ssh.test2.com
    User user2
    Port 2345
    IdentityFile ~/.ssh/id_rsa_test2
```
使用配置文件登录：
```
$ ssh sshtest
```

## SSH无密码登录--实例MAC版
**示例：**

    SSH 服务器(Host)： ssh.test.com
    端口号(Port)： 2200
    账户(User)： user
    密钥文件(IdentityFile)： ~/.ssh/id_rsa_test
    密码：123456

想要是实现的效果：`ssh ssh.test.com` 即可登录进入指定的服务器

### **第一步：实现ssh配置文件登录**
修改或新增配置文件`~/.ssh/config`
```
Host ssh.test.com
    User user
    Post 2200
```
尝试登录`ssh ssh.test.com`，**仍需要输入密码**

### 第二步：ssh-keygen生成ssh key
生成方式：`ssh-keygen -f id_rsa_test -C user`
输入登录ssh的帐号的密码: `123456`
生成的密钥位于: `~/.ssh/`下
### 第三步：ssh-copy-id把public key放到ssh服务器上
方法：`ssh-copy-id -i id_rsa_test.pub user@ssh.test.com`
输入密码：`123456`
可以去ssh服务器验证：`cat ~/.ssh/authorized_keys`查看应该会多了public key内容
### 第四步：修改ssh配置文件配置
添加私钥到配置文件中
```
Host ssh.test.com
    User user
    Post 2200
    IdentityFile ~/.ssh/id_rsa_test
```
### 第五步：重启终端
尝试`ssh ssh.test.com`登录成功

## 常见错误
### WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
执行`ssh iot@1.1.1.1` 时出现标题的错误信息。
原因是由于之前生成ssh-keygen，但是目标机器重装过，导致之前的key不可用，所以出现警告如下图：
![image_1bdgc49861vjf6gej031i73u3i9.png-128.1kB][1]

**解决方法：`ssh-keygen -R host`**

## 参考链接

- [使用 SSH config 文件](http://daemon369.github.io/ssh/2015/03/21/using-ssh-config-file)
- [如何优雅地连接ssh](https://segmentfault.com/a/1190000000585526)
- [linux中ssh的config怎么配置可以省略输入密码？](https://segmentfault.com/q/1010000002445731)


[1]: http://static.zybuluo.com/huis/11hhtqbb04sh8jrel3fycwej/image_1bdgc49861vjf6gej031i73u3i9.png