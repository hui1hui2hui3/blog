---
title: "Python多环境管理"
date: 2019-10-16
tags: ["Python"]
draft: false
---

## pyenv
> 用于管理Python 版本
也就是说使用使用该工具可以存在多个Python版本

- 安装版本： `pyenv install`
- 查看当前使用版本： `pyenv version`
- 查看所有已安装版本：`pyenv versions`
    - 其中`system`环境为系统安装版本
    - 还会显示`virtualenvs`创建的虚拟环境
- 切换版本：
-   `pyenv local xxx`
    - 激活当前目录使用版本为xxx 
-   `pyenv shell xxx`
    - 激活当前命令行使用版本为xxx  
-   `pyenv global xxx`
    - 切换全局默认版本为xxx  


## virtualenv
> 用于管理Python 依赖库的版本
也就是说使用该工具可以存在一个Python版本使用某个依赖库时，可以使用不同版本的。建议搭配pyenv使用

**注意： 该环境优先级高于pyenv** 

## pyenv-virtualenv
> 是pyenv的插件，用于管理virtualenv

- 创建虚机环境：`pyenv virtualenv <version> <env name>`
    - version 为pyenv 安装的python版本
    - env name 为自定义环境名称
- 查看所有虚拟环境： `pyenv virtualenvs`
    - 列表中有`*`前缀的表示当前激活环境
- 激活虚拟环境：`pyenv active <env name>`
    - 激活后，除非关闭shell，否则无论在哪个目录均使用该版本（包括pyenv local环境）
- 离开环境：`pyenv deactive`





