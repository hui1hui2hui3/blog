---
title: "如何在 Zsh 中使用 tab 键来补全 docker 和 docker-compose 命令"
date: 2016-12-02
tags: ["Zsh"]
draft: false
---

## Zsh
[Zsh](http://zsh.sourceforge.net/) 是一个兼容 `bash` 的更加强大的 shell.

## 方法概括
1. 下载补全脚本
```
mkdir -p ~/.zsh/completion
curl -L https://raw.githubusercontent.com/docker/docker/master/contrib/completion/zsh/_docker > ~/.zsh/completion/_docker
curl -L https://raw.githubusercontent.com/docker/compose/master/contrib/completion/zsh/_docker-compose > ~/.zsh/completion/_docker-compose
```
2. 配置 `~/.zshrc`, 主要添加或修改下面两行
```
fpath=(~/.zsh/completion $fpath)
autoload -Uz compinit && compinit -u
```
3. 重新打开`shell`

## 参考链接
1. [官方Command-line completion](https://docs.docker.com/compose/completion/)



