---
title: "Windows 查询进程和杀进程--taskkill,tasklist,tskill"
date: 2017-05-22
tags: ["Windows"]
draft: false
---

## tasklist
### 功能：
命令用来显示运行在本地或远程计算机上的所有进程，可以监控用户的操作。 

### 命令格式： 
```
Tasklist [/S system [/U username [/P [password]]]] [/M [module] | /SVC | /V] [/FI filter] [/FO format] [/NH]
```

### 参数列表:

       /S     system           指定连接到的远程系统。
    
       /U     [domain\]user    指定应该在哪个用户上下文执行这个命令。
    
       /P     [password]       为提供的用户上下文指定密码。如果省略，则
                               提示输入。
    
       /M     [module]         列出当前使用所给 exe/dll 名称的所有任务。
                               如果没有指定模块名称，显示所有加载的模块。
    
       /SVC                    显示每个进程中主持的服务。
    
       /V                      显示详述任务信息。
    
       /FI    filter           显示一系列符合筛选器指定的标准的任务。
    
       /FO    format           指定输出格式。
                               有效值: "TABLE"、"LIST"、"CSV"。
    
       /NH                     指定列标题不应该在输出中显示。
                               只对 "TABLE" 和 "CSV" 格式有效。
    
       /?                      显示帮助消息。



### 筛选器:
    筛选器名        有效操作符                有效值
    -----------     ---------------           --------------------------
    STATUS          eq, ne                    RUNNING |
                                              NOT RESPONDING | UNKNOWN
    IMAGENAME       eq, ne                    映像名称
    PID             eq, ne, gt, lt, ge, le    PID 值
    SESSION         eq, ne, gt, lt, ge, le    会话编号
    SESSIONNAME     eq, ne                    会话名
    CPUTIME         eq, ne, gt, lt, ge, le    CPU 时间，格式为
                                              hh:mm:ss。
                                              hh - 时，
                                              mm - 分，ss - 秒
    MEMUSAGE        eq, ne, gt, lt, ge, le    内存使用量，单位为 KB
    USERNAME        eq, ne                    用户名，格式为 [domain\]user
    SERVICES        eq, ne                    服务名称
    WINDOWTITLE     eq, ne                    窗口标题
    MODULES         eq, ne                    DLL 名称

**说明: 当查询远程机器时，不支持 "WINDOWTITLE" 和 "STATUS"
      筛选器。**
      
###示例:

    TASKLIST
    TASKLIST /M
    TASKLIST /V /FO CSV
    TASKLIST /SVC /FO LIST
    TASKLIST /M wbem*
    TASKLIST /S system /FO LIST
    TASKLIST /S system /U domain\username /FO CSV /NH
    TASKLIST /S system /U username /P password /FO TABLE /NH
    TASKLIST /FI "USERNAME ne NT AUTHORITY\SYSTEM" /FI "STATUS eq running"

### 高级示例：
```

```

### 实例分析： 
　　如果我们只是查看本地主机进程信息，直接办入命令即可。下面的实例是从客户机远程查看内网中某台主机时程信息。 
　　假如我们有一台服务器： 
　　内网地址：`192.168.0.1` 
　　管理员帐号：`administrator `
　　管理员密码：`password `
　　我们需要在CMD窗口输入： 
　　`Tasklist /s 192.168.0.1 /u administrator /p password`
　　这条命令可以使我们方便的查看到远程主机的运行情况，当然前提是保证RPC服务正常启动。

## taskkill

### 语法：
```
TASKKILL [/S system [/U username [/P [password]]]]
         { [/FI filter] [/PID processid | /IM imagename] } [/T] [/F]
```

### 描述:
    使用该工具按照进程 ID (PID) 或映像名称终止任务。

### 参数列表:
    /S    system           指定要连接的远程系统。
    
    /U    [domain\]user    指定应该在哪个用户上下文执行这个命令。
    
    /P    [password]       为提供的用户上下文指定密码。如果忽略，提示
                           输入。
    
    /FI   filter           应用筛选器以选择一组任务。
                           允许使用 "*"。例如，映像名称 eq acme*
    
    /PID  processid        指定要终止的进程的 PID。
                           使用 TaskList 取得 PID。
    
    /IM   imagename        指定要终止的进程的映像名称。通配符 '*'可用来
                           指定所有任务或映像名称。
    
    /T                     终止指定的进程和由它启用的子进程。
    
    /F                     指定强制终止进程。
    
    /?                     显示帮助消息。

### 筛选器:
筛选器名    |  有效运算符        |        有效值
-----------  | ---------------      | -------------------------
STATUS     |   eq, ne          |          RUNNING ,NOT RESPONDING , UNKNOWN
IMAGENAME  |   eq, ne       |             映像名称
PID        |   eq, ne, gt, lt, ge, le  |  PID 值
SESSION     |  eq, ne, gt, lt, ge, le  |  会话编号。
CPUTIME    |   eq, ne, gt, lt, ge, le  |  CPU 时间，格式为hh:mm:ss。hh - 时， mm - 分，ss - 秒
MEMUSAGE   |   eq, ne, gt, lt, ge, le   | 内存使用量，单位为 KB
USERNAME    |  eq, ne         |           用户名，格式为 [domain\]user
MODULES     |  eq, ne        |            DLL 名称
SERVICES     | eq, ne        |            服务名称
WINDOWTITLE  | eq, ne           |         窗口标题


    说明
    ----
    1) 只有在应用筛选器的情况下，/IM 切换才能使用通配符 '*'。
    2) 远程进程总是要强行 (/F) 终止。
    3) 当指定远程机器时，不支持 "WINDOWTITLE" 和 "STATUS" 筛选器。

### 例如:
    TASKKILL /IM notepad.exe
    TASKKILL /PID 1230 /PID 1241 /PID 1253 /T
    TASKKILL /F /IM cmd.exe /T
    TASKKILL /F /FI "PID ge 1000" /FI "WINDOWTITLE ne untitle*"
    TASKKILL /F /FI "USERNAME eq NT AUTHORITY\SYSTEM" /IM notepad.exe
    TASKKILL /S system /U domain\username /FI "USERNAME ne NT*" /IM *
    TASKKILL /S system /U username /P password /FI "IMAGENAME eq note*"
    taskkill /FI "WINDOWTITLE eq 管理员Test" /S Win-g72hl4k1onh /U Administrator /P 1qaz@WSX  /IM cmd.exe

### 高级示例：
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

### 使用方法：
`taskkill /S jenkins /F /IM notepad.exe`
`taskkill /F /FI "IMAGENAME eq notepad.exe"`

**Jenkins注意：/S 命令会导致出现问题：`错误: RPC 服务器不可用`**


## tskill

### 功能：用来关掉进程的 

### 命令格式： 
```
TSKILL processid | processname [/SERVER:servername] [/ID:sessionid | /A] [/V] 
```

### 参数含义 

    　　processid 要结束的进程的 Process ID。 
    　　processname 要结束的进程名称。 
    　　/SERVER:servername 含有 processID 的服务器(默认值是当前值)。 
    　　使用进程名和 /SERVER 时，必须指定 
    　　/ID 或 /A 
    　　/ID:sessionid 结束在指定会话下运行的进程。 
    　　/A 结束在所有会话下运行的进程。 
    　　/V 显示正在执行的操作的信息。 
    　　这个Tskill用法很简单，直接输入Tskill 图象名或PID就可以了。 
    　　偶尔碰上Tskill无法结束的进程，还可以试试Ntsd命令， 
    　　格式为: ntsd -c q -pn {进程名｝ 
    　　参数含义： 
    　　-c是表示执行debug命令; 
    　　q表示执行结束后退出; 
    　　-p 表示后面紧跟着是你要结束的进程对应的PID; 
    　　-pn 表示后面紧跟着是你要结束的进程名;

### 使用方法：
`tskill java(进程名）`
`tskill pid`

### 问题
1. 在psexec 下执行`tskill notepad`命令，可能会出现无法找到进程的问题

## 参考链接
1. [DOS或命令行下查看进程,结束进程命令](http://blog.csdn.net/wuquwer/article/details/3908351)
2. [WINDOWS下kill进程的命令](http://www.cnblogs.com/netflu/archive/2010/01/07/1641399.html)



