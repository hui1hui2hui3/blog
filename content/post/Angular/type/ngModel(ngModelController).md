---
title: "ngModel-ngModelController学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ngModel-ngModelController学习笔记

[TOC]

> **[ngModelController API 地址](https://docs.angularjs.org/api/ng/type/ngModel.NgModelController)**

## Methods

### `$render();`

触发该方法的情况如下：
1.  调用$rollbackViewValue() 
2.  通过编程的方式更改ng-model, `$modelValue`和`$viewValue` 都和上一次的值不同

### `$isEmpty(value);`

默认的判断empty的是`undefined`, `''`, `null` or `NaN`。
可以重写自定义empty的判断条件

### `$setValidity(validationErrorKey, isValid);`

#### Parameters
Param | Type | Details
-------|-------|----------
validationErrorKey | string | Name of the validator. The validationErrorKey will be assigned to either `$error[validationErrorKey]` or `$pending[validationErrorKey] `(for unfulfilled `$asyncValidators`), so that it is available for data-binding. The validationErrorKey should be in camelCase and will get converted into dash-case for class name. Example: myError will result in `ng-valid-my-error` and `ng-invalid-my-error` class and can be bound to as `{{someForm.someControl.$error.myError}}` .
isValid | boolean | Whether the current state is valid (true), invalid (false), pending (undefined), or skipped (null). Pending is used for unfulfilled `$asyncValidators`. Skipped is used by Angular when validators do not run because of parse errors and when `$asyncValidators` do not run because any of the `$validators` failed.

### `$setPristine();`
### `$setDirty();`
### `$setUntouched();`
### `$setTouched();`
### `$rollbackViewValue();`

取消更新和复位input的value，同时阻止向`$modelValue` 更新
> **注意：** 如果要对ng-model的`$modelValue`进行编程操作，同时如果处于pending状态中，则必须首先调用`$rollbackViewValue()` 用以保持view value 和 model value的值同步

#### 代码演示（官方）
```js
angular.module('cancel-update-example', [])

.controller('CancelUpdateController', ['$scope', function($scope) {
  $scope.resetWithCancel = function(e) {
    if (e.keyCode == 27) {
      $scope.myForm.myInput1.$rollbackViewValue();
      $scope.myValue = '';
    }
  };
  $scope.resetWithoutCancel = function(e) {
    if (e.keyCode == 27) {
      $scope.myValue = '';
    }
  };
}]);
```
```html
<div ng-controller="CancelUpdateController">
  <p>Try typing something in each input.  See that the model only updates when you
     blur off the input.
   </p>
   <p>Now see what happens if you start typing then press the Escape key</p>

  <form name="myForm" ng-model-options="{ updateOn: 'blur' }">
    <p id="inputDescription1">With $rollbackViewValue()</p>
    <input name="myInput1" aria-describedby="inputDescription1" ng-model="myValue"
           ng-keydown="resetWithCancel($event)"><br/>
    myValue: "{{ myValue }}"

    <p id="inputDescription2">Without $rollbackViewValue()</p>
    <input name="myInput2" aria-describedby="inputDescription2" ng-model="myValue"
           ng-keydown="resetWithoutCancel($event)"><br/>
    myValue: "{{ myValue }}"
  </form>
</div>
```
### `$validate();`

> **注意：** 如果ngModel验证不通过，则设置值为undefined, 除非`ngModelOptions.allowInvalid=true`

### `$commitViewValue();`

### `$setViewValue(value, trigger);`

> **注意1：** 如果newValue是Object(不是string或number),则在调用该方法以前必须copy newValue
> **注意2：** 该方法不会触发`$digest` 方法

## 代码演示
```html
<!DOCTYPE html>
<html ng-app="modelModule">

<head>
    <title>ngModel-ngModelController</title>
    <style>
    [contenteditable] {
        border: 1px solid black;
        background: white;
        min-height: 50px;
    }
    
    .ng-invalid {
        border: 1px solid red;
    }
    </style>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript" src="../bower_components/angular-sanitize/angular-sanitize.min.js"></script>
    <script type="text/javascript">
    angular.module('modelModule', ['ngSanitize'])
        .controller('ModelController', ['$scope', function($scope) {
            var modelCtrl = this;
            modelCtrl.divContent = 'change me';
        }])
        .directive('contenteditable', ['$sce', function($sce) {
            var init = false;
            return {
                require: 'ngModel',
                restrict: 'A',
                link: function(scope, element, attrs, ngModel) {

                    ngModel.$render = function() {
                        init = true;
                        element.html($sce.getTrustedHtml(ngModel.$viewValue || ''));
                    };

                    element.on('keyup change blur', function() {
                        scope.$evalAsync(read);
                    });
                    read();

                    function read() {
                        var html = element.html();
                        if (!init && !html) {
                            return;
                        }
                        if (attrs.stripBr && html == '<br>') {
                            html = '';
                        }
                        ngModel.$setViewValue(html);
                        init = true;
                    }
                }
            };
        }]);
    </script>
</head>

<body ng-controller="ModelController as modelCtrl">
    <form name="myForm">
        <div name="myName" contenteditable ng-model="modelCtrl.divContent" strip-br="true" required></div>
        <span ng-show="myForm.myName.$error.required">Required!</span>
        <textarea ng-model="modelCtrl.divContent"></textarea>
    </form>
</body>

</html>

```

> Written with [StackEdit](https://stackedit.io/).