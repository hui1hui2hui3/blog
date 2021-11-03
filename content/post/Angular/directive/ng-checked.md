---
title: "ng-checked学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ng-checked学习笔记

> **注意事项：** 该指令不能与**ngModel**同时使用，否则可能会出现意外的行为。 

## 官方Code
```html
<label>Check me to check both: <input type="checkbox" ng-model="master"></label><br/>
<input id="checkSlave" type="checkbox" ng-checked="master" aria-label="Slave input">
```

> Written with [StackEdit](https://stackedit.io/).