
---
title: "Jmeter学习笔记--无GUI模式运行详解"
date: 2019-10-14
tags: ["Jmeter"]
draft: false
--- 

## 普通命令
`jmeter -n -t test.jmx -l test.jtl`
> 备注：这种方式的运行，最好把脚本中的监听器全部删掉或禁用，防止影响性能

## 参数大全
```
-h, --help
                print usage information and exit
　　　　　　　　　#打印帮助信息　
        -v, --version
                print the version information and exit
　　　　　　　　　 #打印版本信息
        -p, --propfile {argument}
                the jmeter property file to use
　　　　　　　　　 #运行时指定property文件，默认是使用JMETER_HOME/bin目录下的jmeter.properties，如果用户自定义有其它的配置，在这里加上
　　　　　　　　　 #用法如下： -p user.properties
        -q, --addprop {argument}
                additional property file(s)
　　　　　　　　　 #其它配置文件，如JVM参数等等
        -t, --testfile {argument}
                the jmeter test(.jmx) file to run
　　　　　　　　  #要运行的jmeter脚本
        -j, --jmeterlogfile {argument}
                the jmeter log file
　　　　　　　　  #指定记录jmeter log的文件，默认为jmeter.log
        -l, --logfile {argument}
                the file to log samples to
　　　　　　　　  #记录采样器Log的文件
        -n, --nongui
                run JMeter in nongui mode
　　　　　　　　  #以nongui模式运行jmeter
        -s, --server
                run the JMeter server
　　　　　　　　  #运行JMeter server
        -H, --proxyHost {argument}
                Set a proxy server for JMeter to use
　　　　　　　　  #代理服务器地址
        -P, --proxyPort {argument}
                Set proxy server port for JMeter to use
　　　　　　　　  #代理服务器端口
        -u, --username {argument}
                Set username for proxy server that JMeter is to use
　　　　　　　　  #代理服务器的用户名
        -a, --password {argument}
                Set password for proxy server that JMeter is to use
　　　　　　　　  #代理服务器用户名对应的密码
        -J, --jmeterproperty {argument}={value}
                Define additional JMeter properties
　　　　　　　　  #定义额外的Jmeter属性
        -G, --globalproperty (argument)[=(value)]
                Define Global properties (sent to servers)
                e.g. -Gport=123
                 or -Gglobal.properties
　　　　　　　　  #定义发送给server的全局属性
　　　　　　　　　#如：-Gport=123 或者-Gglobal.properties（指定监听server的端口）
        -D, --systemproperty {argument}={value}
                Define additional System properties
　　　　　　　　  #定义系统属性
        -S, --systemPropertyFile {filename}
                a property file to be added as System properties
　　　　　　　　　#通过指定的property文件定义系统属性
        -L, --loglevel {argument}={value}
                Define loglevel: [category=]level 
                e.g. jorphan=INFO or jmeter.util=DEBUG
　　　　　　　　  #定义日志等级
        -r, --runremote (non-GUI only)
                Start remote servers (as defined by the jmeter property remote_hosts)
　　　　　　　　  #启动远程server（在jmeter property中定义好的remote_hosts），公在non-gui模式下此参数才生效
        -R, --remotestart  server1,... (non-GUI only)
                Start these remote servers (overrides remote_hosts)
　　　　　　　　  #启动远程server（如果使用此参数，将会忽略jmeter property中定义的remote_hosts）
        -d, --homedir {argument}
                the jmeter home directory to use
                #Jmeter运行的主目录
        -X, --remoteexit
                Exit the remote servers at end of test (non-GUI)
　　　　　　　　  #测试结束时，退出（在non-gui模式下）
```

## 运行并生成报告
`jmeter -n -t test.jmx -l test.jtl -e -o ./reportFolder`
> 备注：-o 生成报告的目录必须不存在

## 日志转报告
`jmeter -g result.jtl -o ./resultReport`
> 把生成的日志转化为报告

## 限制流量运行，模拟手机2G，3G，4G网络
```
jmeter -n -t test.jmx -l test.jtl -Jhttpclient.socket.http.cps=21888 -Jhttpclient.socket.https.cps=21888
```

**21888表示:  171*1024/8 
171表示:    171Kbit/s(下行带宽）=21.375KB/s（下行速度）**

> 备注：8Kbit/s = 1KB/s

所以CPS计算公式为：
$CPS=模拟速度*1024$

下面给出常用的网络cps值：
网络 | Cps值
------|-------
GPRS | 21888
3g | 2688000
4g | 19200000
wifi(802.11a/g) | 6912000
adsl | 1024000
100m | 12800000
Gigabit | 128000000

## 命令行控制脚本中的可变变量
```
jmeter -n -t rls_exam.jmx -l test.jtl -Jexamcode=675867 -Jlaunchtime=300 -Jlazytime=5 -Jthreadsnum=2 -Jramptime=10
```
* `-Jexamcode` : 表示脚本中需要examcode变量，脚本中写法是`${__P(examcode,)}`
* `-Jthreadsnum`: 表示脚本中的线程数变量，可以有默认值，比如`${__P(threadsnum,1)}`表示默认为1

示例截图：
![image_1avt8ctqra841m6a1d41lrv1il8g.png-145.1kB][1]

思考问题：
1. 如何控制是否开启循环次数
> -1 表示无线循环

2. 如何控制是否开启调度器

## Out Of Memery
默认的JVM配置是`-Xms512m -Xmx512m`，所以如果超过150个线程执行时有可能会内存溢出，下面是配置JVM方法：
```
JVM_ARGS="-Xms1024m -Xmx1024m" jmeter -t test.jmx [etc.]
```
## 参考
> [JMeter-自动生成测试报告](http://www.jianshu.com/p/c9f9a06df5cb)
> [在jmeter测试中模拟不同的带宽环境](http://www.cnblogs.com/landhu/p/5969632.html)


[1]: http://static.zybuluo.com/huis/nbpg1o943zlbeh4a8yg0rplk/image_1avt8ctqra841m6a1d41lrv1il8g.png