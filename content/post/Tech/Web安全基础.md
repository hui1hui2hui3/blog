---
title: "Web安全基础"
date: 2016-12-06
tags: ["安全"]
draft: false
---


## 分类
### 以服务器为目标的主动攻击
1. SQL注入攻击
2. OS命令注入攻击

### 以服务器为目标的被动攻击
1. 跨站脚本攻击（Cross-Site Scripting,XSS）
2. 跨站点请求伪造（Cross-Site Request Forgeries,CSRF）


## 跨站脚本攻击（Cross-Site Scripting,XSS）
跨站点脚本攻击是通过存在安全漏洞的web网站注册用户的浏览器内运行非法的HTML标签或JavaScript进行的攻击，也被称为XSS攻击。
这种漏洞(XSS)通常用于发动cookie窃取、恶意软件传播(蠕虫攻击),会话劫持,恶意重定向。在这种攻击中,攻击者将恶意JavaScript代码注入到网站页面中,这样”受害”者的浏览器就会执行攻击者编写的恶意脚本。这种漏洞容易找到，但很难修补。这就是为什么你可以在任何网站发现它的身影

### 演示Demo2--Post
如果网站存在可以输入显示为html的入口，则下面代码有可能导致脚本注入满天飞：
```
<img src="http://snoopyxdy.blog.163.com/blog/err" onerr="alert('xss')" />
<script>alert('xss')</script>
```

### 演示Demo3--基于dom的跨站点脚本攻击 
### 需要编码和过滤的对象
    The URL
    HTTP referrer objects
    GET parameters from a form
    POST parameters from a form
    Window.location
    Document.referrer
    document.location
    document.URL
    document.URLUnencoded
    cookie data
    headers data
    database data

数据是输出到HTML中的那就要进行HtmlEncode，如果数据是输出到javascript代码中进行拼接的，那就要进行javascriptEncode。

### SQL注入攻击
SQL注入（SQL Injection）是指针对Web应用使用的数据库，通过运行非法的SQL而产生的攻击。危害极大。

#### 例子1
搜索名字为plhwin,正常情况如下：
`http://localhost/test/userinfo.php?username=plhwin`,SQL执行如下：
```
SELECT uid,username FROM user WHERE username='plhwin'
```
但是，如果用户在浏览器里把传入的username参数变为 `plhwin';SHOW TABLES-- hack`，也就是当URL变为 `http://localhost/test/userinfo.php?username=plhwin';SHOW TABLES-- hack` 的时候，此时我们程序实际执行的SQL语句变成了：
```
SELECT uid,username FROM user WHERE username='plhwin';SHOW TABLES-- hack'
```
经过上面的SQL注入后，原本想要执行查询名字详情的SQL语句，此时还额外执行了 `SHOW TABLES;` 语句

现在Mysql已经不允许使用分号一次执行多SQL语句

#### 例子2
登录：
```
SELECT uid,username FROM user WHERE username='plhwin' AND password='e10adc3949ba59abbe56e057f20f883e'
```
但如果输入的用户名为 `plhwin' AND 1=1-- hack`，密码随意输入，比如aaaaaa，那么SQL语句为：
```
SELECT uid,username FROM user WHERE username='plhwin' AND 1=1-- hack' AND password='0b4e7a0e5fe84ad35fb5f95b9ceeac79'
```
这意味着只需要知道别人的会员名，无需知道密码就能顺利登录到系统。

#### 预防方法：
1. 检查变量数据类型和格式
2. 过滤转义特殊符号
3. 绑定变量，使用预编译语句
4. 数据库信息加密

### 跨站点请求伪造（Cross-Site Request Forgeries,CSRF）
跨站点请求伪造攻击是指攻击者通过设置好的陷阱，强制对已完成认证的用户进行非预期的个人信息或设定信息等状态的更新。

### 其他攻击方式
1、文件上传漏洞攻击
2、分布式拒绝服务攻击 - DDOS
3、URL漏洞

## 我们网站中的问题
1. 可以重复发布课程
2. 多余的email页面
3. 统计页面可以查看任意用户的学习记录

## 总结
1. 网站包含富文本编辑器的更易受到攻击，比如可以输入html，a超链接，或markdown语法等
2. 虽然现在的代码技术和浏览器等帮我们拦截掉了很多的安全攻击，但是XSS和CSRF一直存在于我们的身边，网站安全仍是我们要时刻警惕的事情。


[1]: http://static.zybuluo.com/huis/c64g5xa6buv42fhxc539ihpn/image_1b370cnth1fnatmolamjt61d6d9.png
[2]: http://static.zybuluo.com/huis/2zmh6kj9d4q7fqgq0x6ku2ed/image_1b371f4ci1ccm1gfh1cmm1lts14j9m.png
[3]: http://static.zybuluo.com/huis/n2gj1w8c67rm2j5m8hayuhvh/image_1b3723i068mo7jn1alo1crgbs413.png