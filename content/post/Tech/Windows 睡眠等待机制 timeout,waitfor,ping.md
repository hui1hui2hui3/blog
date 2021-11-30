---
title: "Windows 睡眠等待机制 timeout,waitfor,ping"
date: 2017-10-30
tags: ["Windows","Batch"]
draft: false
---

[TOC]

    尝试实现window batch脚本中等待机制，即不在命令行输出等待倒计时，直接等待到时间结束

## Timeout

### 语法： 
`TIMEOUT [/T] timeout [/NOBREAK]`

### 描述:
    这个工具接受超时参数，等候一段指定的时间(秒)或等按任意键。它还接受
    一个参数，忽视按键。

### 参数列表:
    /T        timeout       指定等候的秒数。有效范围从 -1 到 99999 秒。
    
    /NOBREAK                忽略按键并等待指定的时间。
    
    /?                      显示此帮助消息。

**注意: 超时值 -1 表示无限期地等待按键。**

### 普通示例:
    TIMEOUT /?
    TIMEOUT /T 10
    TIMEOUT /T 300 /NOBREAK
    TIMEOUT /T -1

### 高级实例：
```
timeout /t 10 /nobreak > NUL
```
**实际等待时间：10s**

### Jenkins运行结果：
**错误: 不支持输入重新定向，立即退出此进程。**

## Waitfor

WaitFor 有两种运行方式:

### 语法 1: 发送信号
    WAITFOR [/S system [/U user [/P [password]]]] /SI signal

### 语法 2: 等候信号
    WAITFOR [/T timeout] signal

### 描述:
    此工具在系统上发送或等待信号。当没有指定 /S 时，信号会被广播到一个
    域的所有系统上。如果指定了/S，信号只发送到指定的系统上。

### 参数列表:
    /S     system         指定远程系统以便发送信号。
    
    /U     [domain\]user  指定用户上下文，命令在此上下文中执行。
    
    /P     [password]     指定给定用户上下文的密码。
    
    /SI                   把信号发送到网络上正在等待的机器。
    
    /T     timeout        等待信号的秒数。有效范围是 1 - 99999。默认值
                          是永远等待信号。
    
    signal                等待或发送的信号名称。
    
    /?                    显示此帮助消息。
    
    注意: 系统可以等待多个唯一的信号名称。信号名不能超过 225 个字符，
    而且不能含有 a-z、A-Z、0-9 和范围为 128-255 的 ASCII 字符之外的
    字符。

### 普通示例：
```
WAITFOR /?
WAITFOR SetupReady
WAITFOR CopyDone /T 100
WAITFOR /SI SetupReady
WAITFOR /S system  /U user /P password /SI CopyDone
```
**注意：waitfor 的事件如果等不到信号，则会出现“错误: 等候 'SomethingThatIsNeverHappening' 时超时”**

### 高级示例：
```
waitfor SomethingThatIsNeverHappening /t 10 2>NUL
```
**实际等待时间：10s**

### Jenkins运行结果：
```
Finished: FAILURE //可以正常等待，但是结果Failure
```
改进：
```
waitfor SomethingThatIsNeverHappening /t 10 2>NUL
...//其他代码
EXIT /b 0
```

## Ping

### 高级示例：
```
ping 127.0.0.1 -n 6 > nul
```
**实际等待时间5秒，也就是n-1秒**

### Jenkins运行结果：
```
OK
```

### Ping 如何判断网络有问题
```
Ping 192.168.60.217 -n 4 -w 1000 | find "TTL=" >nul
if errorlevel 1 (
    echo "no connect"
    exit -1
)
```
