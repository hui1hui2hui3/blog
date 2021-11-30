---
title: "Windows Batch 编程--For，Exist"
date: 2017-07-03
tags: ["Batch"]
draft: false
---

## 例子解析：实现检测分支同时新建的功能
```
@echo off
set branch_name=origin/test
set result=0

FOR /F %%G IN ('git branch -r') DO (
   if %%G==%branch_name% (
	echo find branch %%G
        set result=1
   )
)
if %result%==0 (
   echo "prepare test branch"
   call git checkout -B test
   call git push http://%stash_user%:%stash_pass%@s.kyee.com.cn/scm/iotbc/iot-server.git test -f
) else (
   echo "another test branch exists!"
   exit 1
)
```

## 例子解析2： 实现文件检测，删除和重命名文件
```
@echo off
cd server
if exist "dist_one*.jar" (
	echo found_dist_one
	del iot-server-snapshot.jar
	ren dist_one*.jar iot-server-snapshot.jar
) else (
	echo just_run_iot
)
java -Dfile.encoding=utf-8 -jar iot-server-snapshot.jar 1>server.log 2>&1

```
## 例子3：实现for循环内赋值问题
### 代码1：正常实现方式,这种你会发现无法赋值
```
@echo off
set num=0
for %%i in (mysql-bin.0*) do (
	echo %%i
	set /a num=num+1
	echo %num%
)
echo %num%
pause
```
结果如下：
![image_1bk3t3f6g124chuq1ptao9u1v2v9.png-25.3kB][1]
会发现结果for循环中的变量都是0，没有+1
What A Hell！！！

### 代码2：实现方式
```
@echo off
SETLOCAL ENABLEDELAYEDEXPANSION
set num=0
for %%i in (mysql-bin.0*) do (
	echo %%i
	set /a num=num+1
	echo !num!
)
echo %num%
pause
```
结果如下：
![image_1bk3t73891dbha7l1i9g1p2h1fkkm.png-23.3kB][2]
正常了，为啥累！！！仔细看会发现，方法2的代码方法1的多了一行
`SETLOCAL ENABLEDELAYEDEXPANSION`，这个是啥意思，就是“启用延缓环境变量”，另外还有就是`echo !num!`这里用的不再是`echo %num%`注意了


## 参考链接
[Windows 批处理脚本学习教程](http://docs.30c.org/dosbat/chapter04/)
[CMD](https://ss64.com/nt/set.html)
[Guide to Windows Batch Scripting](http://steve-jansen.github.io/guides/windows-batch-scripting/index.html)
[windows批处理 (cmd/bat) 编程详解](https://my.oschina.net/superkangning/blog/528881)
[Windows批处理(cmd/bat)常用命令小结](https://wsgzao.github.io/post/windows-batch/)
[Windows命令行和批处理技巧](http://netwjx.github.io/blog/2012/07/29/windows-shell-and-bat-skills/)


[1]: http://static.zybuluo.com/huis/y6mogjuwv8u819orc120t2dy/image_1bk3t3f6g124chuq1ptao9u1v2v9.png
[2]: http://static.zybuluo.com/huis/mllsbpqrf7ecjrqty5itihvf/image_1bk3t73891dbha7l1i9g1p2h1fkkm.png