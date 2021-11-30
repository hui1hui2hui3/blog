---
title: "Shell 编程入门笔记--运算"
date: 2017-01-11
tags: ["Shell"]
draft: false
---

## 1. 计算字符串或文件中字符出现的次数
> * `echo $str | grep -o "字符串" | wc -l` 查找字符串中字符出现次数
> * `grep -o '字符串' file |wc -l` 查找文件中字符出现次数

下面是查找字符串中字符e出现次数的例子：
```bash
str="im a test str"
count=`echo $str | grep -o "e" | wc -l`  #这里顺便一提的是如何把执行的命令赋值给变量
echo $count #1
```

## 2. 数字运算
> * `let val++` let运算 
```bash
count=1
let count++
echo $count #2
```

## 3. 命令执行的结果赋值给变量
```bash
val=`python test.py 2>&1`
echo $val
```
这个可以顺便说一下`2>&1`的意思
比如`echo log >/dev/null 2>&1`
解释：
`/dev/null`: 表示空设备文件
`>`: 代表重定向
`1`: 表示stout标准输出，系统默认值为1，所以`>/dev/null`表示`1>/dev/null`
`2`: 表示sterr标准错误
`&`: 表示等同的意思，`2>&1`表示2的输出重定向等同于1

`1>/dev/null 2>&1`含义：
`1>/dev/null`: 表示标准输出到空设备文件，也就是不输出任何信息到终端，就是不显示任何信息
`2>&1`: 表示标准错误重定向到标准输出，也就是同样输出到空设备文件
> 参考链接http://blog.csdn.net/ithomer/article/details/9288353


## 4. if条件语句是出现`[: or: binary operator expected`错误
```bash
a=1
if [ $a -gt 1 ]
then
    echo "hello"
fi
```
上面的例子中正常是没有问题的，但是假如a为空的话，则会导致出现这样的错误，或者`-gt`写成了`>`也会出现这样的错误
解决方法：
```
a=1
if [[ $a -gt 1 ]] #注意：这里用了双中括号
then
    echo "haha"
fi
```

## 5. 比较运算
> * `-gt` 大于运算符

gt例子：
```bash
count=1
if [[ $count -gt 0 ]]
then
    echo "pass"
fi
```

## 6. 逻辑运算
如何进行多条件判断？
错误想法1：
```bash
count=1
if [[ $count -gt 0 or $count -lt 11 ]] #错误,这种想法
#想的是python语法
then
    pass
fi
```
错误想法2：
```bash
count=1
if [[ $count -gt 0 -o $count -lt 11 ]] #错误，这种想法一般搜索的是shell的或，结果这种根本不好使
then
    pass
fi
```
正确写法：
```bash
count=1
if [[ $count -gt 0 || $count -lt 11 ]] #写法1,这个是正常的想法
then
    pass
fi
if [[ $count -gt 0 ]] || [[ $count -lt 11 ]] #写法2
then
    pass
fi

```
