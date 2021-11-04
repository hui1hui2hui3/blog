---
title: "Git 使用笔记--日志查看"
date: 2016-11-21
tags: ["Git"]
draft: false
---

## Oneline
`--oneline`标记把每一个提交压缩到了一行中。它默认只显示提交ID和提交信息的第一行。`git log --oneline`的输出一般是这样的：
```
0e25143 Merge branch 'feature'
ad8621a Fix a bug in the feature
16b36c6 Add a new feature
23ad9ad Add the initial code base
```

## 如何查看的日志

### 按日期

查看一周前的日志：
```
git log --after '1 week ago'
```

查看昨天日志：
```
get log --after="yesterday"
```

查看具体时间之后的日志：
```
git log --after="2016-11-11"
```

查看时间段之间的日志：
```
git log --after="2016-10-1" --before="2014-11-11"
```

**注意：** `--since` 、`--until` 标记和`--after` 、`--before`标记分别是等价的。

### 按数量

查看之前的10个日志：
```
git log -3
```

### 按作者

查看Huis的日志：
```
git log --author="Huis"
```

### 按提交信息

查看提交信息中有mouse的日志：
```
git log --grep="mouse"
```

### 按文件

查看某个或某些文件的提交日志：
```
git log -- foo.py bar.py
```
**注意：**`--`表示的是后面很多的不是分支名称，而是文件名称

### 按内容

查看文件内容中包含hello的日志：
```
git log -S "hello" #字符串查找
```
或
```
git log -G "<regex>" #正则查找
```

### 按范围

基本语法：
```
git log <since>..<until>
```

查看master分支到feature分支提交的日志，也就是feature分支新提交的日志

```
git log master..feature
```

查看feature分支中没有，master分支中有的日志
```
git log feature..master
```

## 自定义格式
使用`--pretty=format:"<string>"`选项
下面命令中的`%cn`、`%h` 和`%cd`这三种占位符会被分别替换为作者名字、缩略标识和提交日期。

如只显示commit id
```
git log --pretty=format:"%h"
```
结果：
```
4549004
5e7054b
f8dd5bc
7e0ce0f
036b73d
6d7ba60
```


## 显示提交差异
常用命令`-p` 和 `--stat`

`--stat`选项显示每次提交的文件增删数量（注意：修改一行记作增加一行且删去一行）

比如：下面这个提交在hello.py文件中增加了67行，删去了38行。
```
commit f2a238924e89ca1d4947662928218a06d39068c3
Author: John <john@example.com>
Date:   Fri Jun 25 17:30:28 2014 -0500

    Add a new feature

 hello.py | 105 ++++++++++++++++++++++++-----------------
 1 file changed, 67 insertion(+), 38 deletions(-)
```

`-p`选项显示每次提交删改的绝对数量
比如：
```
commit 16b36c697eb2d24302f89aa22d9170dfe609855b
Author: Mary <mary@example.com>
Date:   Fri Jun 25 17:31:57 2014 -0500

    Fix a bug in the feature

diff --git a/hello.py b/hello.py
index 18ca709..c673b40 100644
--- a/hello.py
+++ b/hello.py
@@ -13,14 +13,14 @@ B
-print("Hello, World!")
+print("Hello, Git!")
```

## 过滤合并提交

过滤掉合并的日志
```
git log --no-merges
```

只查看合并的日志
```
git log --merges
```

## 显示提交日志统计Shortlog

```
git shortlog
```
显示结果如下：
```
Junrui Chen (5):
      1.新版答题界面已答未答数显示     2.界面样式调整     3.答案保存逻辑调整     参考：YHJY-4574
      Merge branch 'feature/exam-answer-fxh' of http:xxx into feature/exam-answer-fxh
      修改考试结束界面成绩获取方式     参考：YHJY-4602
      修改界面样式     参考：YHJY-4574
      还原被注释的代码     参考：无

chenjunrui (344):
      合并冲突     参考：YHJY-2804
      修改重置密码时的错误提示信息     参考：YHJY-2704
      删除多余代码     参考：YHJY-2704
      修改不规范的代码     参考：YHJY-2704
      修改不规范代码     参考：YHJY-2704
      修改线下考试的可编辑逻辑     参考：YHJY-2870
...
```
默认情况下，`git shortlog`把输出按作者名字排序，但你可以传入`-n`选项来按每个作者提交数量排序。


## 参考
> [Git log高级用法](https://github.com/geeeeeeeeek/git-recipes/wiki/5.3-Git-log%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95)



