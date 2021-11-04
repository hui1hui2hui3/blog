---
title: "Mysql 初始化Root密码和远程登录授权"
date: 2017-04-19
tags: ["Mysql"]
draft: false
---

> Mysql版本：5.7.17
> 系统环境：Center OS

## 初始化Root密码--CenterOS 
### 获取初始Root密码
`grep 'temporary password' /var/log/mysqld.log`

### 重置Root密码
**1.Stop mysql:**
`systemctl stop mysqld`

**2.Set the mySQL environment option** 
```
systemctl set-environment MYSQLD_OPTS="--skip-grant-tables"
```
**3.Start mysql usig the options you just set**
`systemctl start mysqld`
**4.Login as root**
`mysql -u root`
**5.Update the root user password with these mysql commands**
```
mysql> UPDATE mysql.user SET authentication_string = PASSWORD('MyNewPassword')
    -> WHERE User = 'root' AND Host = 'localhost';
mysql> FLUSH PRIVILEGES;
mysql> quit
```
**6.Stop mysql**
`systemctl stop mysqld`
**7.Unset the mySQL envitroment option so it starts normally next time**
`systemctl unset-environment MYSQLD_OPTS`
**8.Start mysql normally:**
`systemctl start mysqld`
**7.Try to login using your new password:**
`mysql -u root -p`

## 开启远程登录--授权
语法1：授权用户访问所有数据库
`GRANT ALL PRIVILEGES ON *.* TO 'username'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;`

语法2：授权用户访问指定数据库
`GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;`

语法3：授权用户访问指定数据库--精简
`GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'%';`

**问题：语法1和2由于涉及到password，5.7.6以后会出现如下报错**
`ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`
解决方法：
`SET GLOBAL  validate_password_policy='LOW';`

最后记得：刷新权限
`FLUSH PRIVILEGES;`

## 授权清除方法
**1.To list users:**
`select user,host from mysql.user;`

**2.To show privileges:**
`show grants for 'user'@'host';`

**3.To change privileges, first revoke. Such as:**
`revoke all privileges on *.* from 'user'@'host';`

**4.Then grant the appropriate privileges as desired:**
`grant SELECT,INSERT,UPDATE,DELETE ON `db`.* TO 'user'@'host';`

**5.Finally, flush:**
`flush privileges;`


**参考链接1：[How to Grant All Privileges on a Database in MySQL](https://chartio.com/resources/tutorials/how-to-grant-all-privileges-on-a-database-in-mysql/)**
**参考链接2：[How To Create a New User and Grant Permissions in MySQL](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)**
**参考链接3：[Mysql初始化root密码和允许远程访问](http://www.cnblogs.com/cnblogsfans/archive/2009/09/21/1570942.html)**





