---
title: "ngModel学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ngModel学习笔记

> **建议1：** 比较的是引用而不是值，所以对于值为数组或对象时需要特别注意
> **注意1：** ngModel自动创建的对象，是创建到当前$scope中的，所以需要注意创建的区域是否有其他Scope对象

## getter/setter

> **建议2：** 由于angular会频繁的调用getter,所以get方法尽量要速度快

# ngModelOptions

> **注意1：** ng-submit提交时处于pending状态的ng-model会立刻触发pending，然后才会提交updated model，而ng-click方式则不会触发ng-model的pending状态，会把当前的ng-model立即提交，**详细例子参见ng-form文档**
> **注意2：** 该指令会影响当前元素和其子类元素的ng-model

## Arguments

Param | Type | Details
-------|---------|-------------
ngModelOptions | Object | options to apply to the current model

**Details:**

-  **updateOn**: string specifying which event should the input be bound to. You can set several events using an space delimited list. There is a special event called default that matches the default events belonging of the control.
-  **debounce**: integer value which contains the debounce model update value in milliseconds. A value of 0 triggers an immediate update. If an object is supplied instead, you can specify a custom value for each event. For example: `ng-model-options="{ updateOn: 'default blur', debounce: { 'default': 500, 'blur': 0 } }"`
- **allowInvalid**: boolean value which indicates that the model can be set with values that did not validate correctly instead of the default behavior of setting the model to undefined.
- **getterSetter**: boolean value which determines whether or not to treat functions bound to ngModel as getters/setters.
- **timezone**: Defines the timezone to be used to read/write the Date instance in the model for `<input type="date">, <input type="time">`, ... . It understands UTC/GMT and the continental US time zone abbreviations, but for general use, use a time zone offset, for example, `'+0430'` (4 hours, 30 minutes east of the Greenwich meridian) If not specified, the timezone of the browser will be used.

# 完整代码

```html
<!DOCTYPE html>
<html ng-app="modelModule">

<head>
    <title>ng-model(options)</title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('modelModule', [])
        .controller('ModelController', ['$scope', function($scope) {
            var modelCtrl = this,
                _name = 'huis';

            //getter/setter
            modelCtrl.user = {
                name: function(newName) {
                    return arguments.length ? (_name = newName) : _name;
                }
            }

            modelCtrl.cancel = function(event) {
                if (event.keyCode == 27) {
                    $scope.rollbackForm.rollbackName.$rollbackViewValue();
                    //注意这里不能是用modelCtrl,也就是说View中的form 
                    //对象仍然处于$scope中
                }
            }
        }])
    </script>
</head>

<body ng-controller="ModelController as modelCtrl">
    <caption>
        getterSetter, updateOn, debounce
    </caption>
    <br>
    <input type="text" ng-model="modelCtrl.user.name" ng-model-options="{getterSetter: true, updateOn: 'default blur', 
	debounce: {default: 500, blur: 0}}"> {{modelCtrl.user.name()}}
    <br> $rollbackViewValue
    <form name="rollbackForm">
        <input name="rollbackName" type="text" ng-model="modelCtrl.user.name" ng-model-options="{getterSetter: true, updateOn: 'blur'}" ng-keyup="modelCtrl.cancel($event)">
    </form>
    <br>
    <form name="userForm">
        <label>Name:
            <input type="text" name="userName" ng-model="modelCtrl.user.name" 
            ng-model-options="{ debounce: 1000,getterSetter:true }" />
        </label>
        <button ng-click="userForm.userName.$rollbackViewValue(); modelCtrl.user.name('')">Clear</button>
        <br />
    </form>
    <pre>user.name = <span ng-bind="modelCtrl.user.name()"></span></pre>
</body>

</html>
```

> Written with [StackEdit](https://stackedit.io/).