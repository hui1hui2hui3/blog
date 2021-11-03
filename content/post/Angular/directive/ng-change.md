---
title: "ng-change学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ng-change学习笔记

> **注意事项1：** 当值改变时，ng-change绑定的expression会立即执行，而不像on-change事件（需要用户离开Form元素或点击回车）
> **注意事项2：** 不会触发ng-change的情况有以下几种：
> > 1. 如果通过`$parsers`返回的值没有改变，则不会触发（if the value returned from the $parsers transformation pipeline has not changed）
> 2. 如果当前input处于invalid状态，model的值为null，则不会触发（if the input has continued to be invalid since the model will stay null）
> 3. 如果model的值是通过编程的方式进行改变的而不是改变input value，则不会触发(if the model is changed programmatically and not by a change to the input value)

## 官方Code
```html
<script>
  angular.module('changeExample', [])
    .controller('ExampleController', ['$scope', function($scope) {
      $scope.counter = 0;
      $scope.change = function() {
        $scope.counter++;
      };
    }]);
</script>
<div ng-controller="ExampleController">
  <input type="checkbox" ng-model="confirmed" ng-change="change()" id="ng-change-example1" />
  <input type="checkbox" ng-model="confirmed" id="ng-change-example2" />
  <label for="ng-change-example2">Confirmed</label><br />
  <tt>debug = {{confirmed}}</tt><br/>
  <tt>counter = {{counter}}</tt><br/>
</div>
```

> Written with [StackEdit](https://stackedit.io/).