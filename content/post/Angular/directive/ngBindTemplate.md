---
title: "ngBindTemplate学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ngBindTemplate学习笔记

## 官方Code
```html
<script>
  angular.module('bindExample', [])
    .controller('ExampleController', ['$scope', function($scope) {
      $scope.salutation = 'Hello';
      $scope.name = 'World';
    }]);
</script>
<div ng-controller="ExampleController">
 <label>Salutation: <input type="text" ng-model="salutation"></label><br>
 <label>Name: <input type="text" ng-model="name"></label><br>
 <pre ng-bind-template="{{salutation}} {{name}}!"></pre>
</div>
```

> Written with [StackEdit](https://stackedit.io/).