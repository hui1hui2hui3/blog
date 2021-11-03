---
title: "select学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# select学习笔记

> **重点：** 生成select的两种方式**ng-options**和**ng-model**
> **区别：** ng-model绑定的值会转换为string，如果想要绑定不是string的值可以自己用directive convert

## 官方例子（绑定数字）
```html
<select ng-model="model.id" convert-to-number>
  <option value="0">Zero</option>
  <option value="1">One</option>
  <option value="2">Two</option>
</select>
{{ model }}
```
```js
angular.module('nonStringSelect', [])
.run(function($rootScope) {
  $rootScope.model = { id: 2 };
})
.directive('convertToNumber', function() {
  return {
    require: 'ngModel',
    link: function(scope, element, attrs, ngModel) {
      ngModel.$parsers.push(function(val) {
        return parseInt(val, 10);
      });
      ngModel.$formatters.push(function(val) {
        return '' + val;
      });
    }
  };
});
```
> **解释：** 如果不用convertToNumber指令，则会出现select选中空白的情况，因为两者不相等。

## 完整例子
```html
<!DOCTYPE html>
<html ng-app="selectModule">

<head>
    <title>select</title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('selectModule', [])
        .controller('SelectController', ['$scope', function($scope) {
            var selectCtrl = this;

            // 内容为字符串则不用转换
            // selectCtrl.menus = ['test1','test2','test3'];

            selectCtrl.menus = [1, 2, 3];
            selectCtrl.selMenu = selectCtrl.menus[0];
        }])
        .directive('convertToNumber', [function() {
            return {
                require: 'ngModel',
                link: function(scope, element, attrs, ngModel) {
                    ngModel.$parsers.push(function(viewView) {
                        return parseInt(viewView, 10);
                    });
                    ngModel.$formatters.push(function(modelValue) {
                        return '' + modelValue;
                    });
                }
            };
        }]);
    </script>
</head>

<body ng-controller="SelectController as selectCtrl">
    <select ng-model="selectCtrl.selMenu" convert-to-number>
        <option ng-repeat="menu in selectCtrl.menus" ng-value="menu">{{menu}}</option>
    </select>
    {{selectCtrl.selMenu}}
</body>

</html>

```

> Written with [StackEdit](https://stackedit.io/).