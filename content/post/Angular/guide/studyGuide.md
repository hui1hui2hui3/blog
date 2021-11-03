---
title: "ng-controller学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---

[TOC]

## 1. ng-controller学习笔记

### 1.1 理解Controller

> **可做的事情**
> > 1. 初始化`$state`状态
> > 2. 给`$state` 添加行为和方法
> 
> **不可做的事情**
> > 1.  操作DOM：Controller应该仅包含业务逻辑，不应该包含表示逻辑，应该用directive去封装手动操作DOM的逻辑
> > 2. 格式输入：用[angular form controls](https://docs.angularjs.org/guide/forms)代替
> > 3. 过滤器输出：用[angular filters](https://docs.angularjs.org/guide/filter)代替
> > 4. 共享状态和代码: 用[angular services](https://docs.angularjs.org/guide/services)代替
> > 5. 管理其他组件的生命周期（例如：创建service实例）

## Services

> **特性：**
> > 1. **延迟初始化**：angular只有当有组件依赖Service时才会实例化Service
> > 2. **单例**： 每一个组件得到都是对单个实例的引用

## Scope




> Written with [StackEdit](https://stackedit.io/).