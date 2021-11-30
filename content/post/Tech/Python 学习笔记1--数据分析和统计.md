---
title: "如何查看git提交近期的所有的改动或新增的文件-Python实现"
date: 2016-09-22
tags: ["Git","Python"]
draft: false
---

> 环境要求：
> 1. `pip install matplotlib`用于生成图表

代码如下：
```python
# coding=utf-8
import commands
import matplotlib.pyplot as plt
import re
import numpy


def find_all_files(target):
    table = {}
    files = re.findall(r"[M|A|D][^u].*?\.\w*", target)
    for file in files:
        real_files = file.split('/')
        real_file = real_files[len(real_files) - 1]
        real_file = real_file.split('\t')
        if len(real_file) > 1:
            real_file = real_file[len(real_file) - 1]
        else:
            real_file = real_file[0]
        if real_file in table.keys():
            table[real_file] += 1
        else:
            table[real_file] = 1
    return table


def get_words_graphic(wordlist):
    keylist = wordlist.keys()
    vallist = wordlist.values()
    xVal = numpy.arange(len(keylist))
    plt.barh(xVal, vallist, align='center', alpha=0.5)
    plt.yticks(xVal, keylist)
    plt.xlabel("Numbers")
    plt.title(u'文件分析图')
    plt.show()


remove_files = ["__init__.py"]


def remove_file(file_list):
    key_list = file_list.keys()
    for key in key_list:
        if key in remove_files:
            file_list.pop(key)


def get_commit_record():
    target = commands.getoutput("git whatchanged --since='2 days ago'")
    file_list = find_all_files(target)
    remove_file(file_list)
    get_words_graphic(file_list)


get_commit_record()


```



