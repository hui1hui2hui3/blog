---
title: "ng-csp学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---

# ng-csp学习笔记

## Affect Rules

Name | Descriptions
---------|-------------
unsafe-eval |  this rule forbids apps to use eval or Function(string) generated functions (among other things). Angular makes use of this in the $parse service to provide a 30% increase in the speed of evaluating Angular expressions.
unsafe-inline | this rule forbids apps from inject custom styles into the document. Angular makes use of this to include some CSS rules (e.g. ngCloak and ngHide). To make these directives work when a CSP rule is blocking inline styles, you must link to the angular-csp.css in your HTML manually.

> **解释：** ngShow/Hide, ngCloak等在CSP Mode下将不会生效，除非自己手动引入angular-csp.css(在angular目录下）

## 写法

1. `<body ng-csp/>`
2. `<body ng-csp="no-unsafe-eval">`
3. `<body ng-csp="no-inline-style">`
4. `<body ng-csp="no-inline-style;no-unsafe-eval">` like **1**

## 完整例子

```html
<!DOCTYPE html>
<html ng-app="cspModule" ng-csp>

<head>
    <title>ng-csp</title>
    <link rel="stylesheet" type="text/css" href="../bower_components/angular/angular-csp.css">
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('cspModule', [])
        .controller('CspController', ['$scope', function($scope) {
            var cspCtrl = this;
            cspCtrl.counter = 0;
            cspCtrl.normalFn = function() {
                cspCtrl.counter++;
            };

            cspCtrl.evalFn = function() {
                try {
                    eval('1+2');
                    cspCtrl.isCsp = !cspCtrl.isCsp;
                } catch (e) {
                    cspCtrl.evalMsg = "error" + e.message;
                }
            };
        }]);
    </script>
</head>

<body ng-controller="CspController as cspCtrl">
    <input type="button" value="increment" ng-click="cspCtrl.normalFn()" />
    <div>counter:{{cspCtrl.counter}}</div>
    <br>
    <input type="button" value="evalClick" ng-click="cspCtrl.evalFn()" />
    <div>evalMsg: {{cspCtrl.evalMsg}}</div>
    <div ng-show="cspCtrl.isCsp">HHJKHJK</div>
</body>

</html>
```


----------
> Written with [StackEdit](https://stackedit.io/).