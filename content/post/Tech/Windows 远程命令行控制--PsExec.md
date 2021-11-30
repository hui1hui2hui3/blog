---
title: "Windows 远程命令行控制--PsExec"
date: 2017-07-19
tags: ["Windows"]
draft: false
---

## 下载地址
[PsExec 下载地址](https://technet.microsoft.com/en-ca/sysinternals/bb897553.aspx)

## 用法
两台windows都安装psexec即可！并都添加到系统PATH
```
psexec \\remote-desktop-name -u name -p pass cmd
```

## 高级示例
```
@echo off
set iot_server_pid=0
set username=%1
set password=%2
set winname=%3

for /f "tokens=2,5 delims=- " %%a in ('psexec \\%winname% -u %username% -p %password% -s netstat -ano ^| findstr "8080"') do (
	if %%b NEQ 0 set iot_server_pid=%%b
)
echo --------find pid:-----------
echo %iot_server_pid%

if %iot_server_pid% NEQ 0 (
psexec \\%winname% -u %username% -p %password% -s taskkill /F /PID %iot_server_pid%
echo -----start wait----
ping 127.0.0.1 -n 61 > nul
echo -------end wait-------
)
psexec \\%winname% -u %username% -p %password% -s -d -w D:\KYEE-IOT start-service.bat
exit 0
```
**注意：`-s`为了jenkins不出现“句柄无效错误”**
**注意：`^|`是windows命令管道的转义，for语句中要注意**
**注意：`-d`是为了不等待脚本的执行，继续执行**

## 问题
### WinServer 2008系统使用其他非Adminstrator，但是是管理员的账户，拒绝访问的问题
错误如下：
```
拒绝访问
```
解决方法如下：Remote 服务器上执行如下命令
```
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\system /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

### 句柄无效的错误如何处理
```
句柄无效
```
解决方法： 加上`-s`参数
```
psexec -s cmd
```

## 参考
[Psexec语法文档](https://technet.microsoft.com/en-us/sysinternals/bb897553.aspx)
[Run batch scripts on a remote server (windows) from jenkins](http://stackoverflow.com/questions/22553588/run-batch-scripts-on-a-remote-server-windows-from-jenkins)





