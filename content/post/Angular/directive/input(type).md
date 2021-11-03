---
title: "Input[type] 系列学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---

#Input[type] 系列学习笔记
[TOC]

## input[checkbox]

### 额外属性 Arguments：
Param | Type | Details
--------|---------|--------
ngTrueValue(optional)|expression|The value to which the expression should be set when selected.
ngFalseValue(optional)|expression|The value to which the expression should be set when not selected.

### 完整代码
```html
<!DOCTYPE html>
<html ng-app="inputCheckBox">

<head>
    <title>ng-input[checkbox]</title>
    <script src="../bower_components/angular/angular.js" type="text/javascript"></script>
    <script type="text/javascript">
    angular.module('inputCheckBox', [])
        .controller('CheckBoxController', ['$scope', function($scope) {
            $scope.check = {
                test1: true,
                test2: 'HEHE'
            }
        }])
    </script>
</head>

<body ng-controller="CheckBoxController">
    <form name="checkboxForm">
        <label>
            Value1:
            <input type="checkbox" name="check1" ng-model="check.test1">
        </label>
        <label>
            Value2:
            <input type="checkbox" name="check2" ng-model="check.test2" ng-true-value="'HEHE'" ng-false-value="'WHAT?'">
        </label>
        <br>
        <tt>Value1:{{check.test1}}</tt><br>
        <tt>Value2:{{check.test2}}</tt>
    </form>
</body>

</html>

```
## input[date] | input[month] | input[time]

> **注意事项：** ng-model 必须是一个date object

### 额外属性 Arguments
Param | Type | Details
--------|-------------|-----------|
min | string | Sets the min validation error key if the value entered is less than min. This must be a valid ISO date string (yyyy-MM-dd).
max | string | Sets the max validation error key if the value entered is greater than max. This must be a valid ISO date string (yyyy-MM-dd).

### 完整代码
```html
<!DOCTYPE html>
<html ng-app="dateModule">

<head>
    <title>ng-input[date]</title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('dateModule', [])
        .controller('DateController', ['$scope', function($scope) {
            $scope.date = {
                value: new Date(2014, 9, 22)
            }
        }])
    </script>
</head>

<body ng-controller="DateController">
    <form name="dateForm">
        <input type="date" name="dateTime" min="2014-02-01" max="2015-03-21" ng-model="date.value" required>
        <div role="alert">
        	<span ng-show="dateForm.dateTime.$error.required">Required!</span>
        	<span ng-show="dateForm.dateTime.$error.date">Time is error!</span>
        </div>
        <code>date.value:{{date.value | date:"yyyy-MM-dd"}}</code>
        <br>
        <code>dateForm.$error.required:{{dateForm.$error.required}}</code>
        <br>
        <code>dateForm.$error.date:{{dateForm.$error.date}}</code>
        <br>
        <code>dateForm.$valid:{{dateForm.$valid}}</code>
    </form>
</body>

</html>

```

## input[datetime-local]

### 额外属性 Arguments

Param | Type | Details
---------|---------|----------
min|string |Sets the min validation error key if the value entered is less than min. This must be a valid ISO datetime format (yyyy-MM-ddTHH:mm:ss).
max| string | Sets the max validation error key if the value entered is greater than max. This must be a valid ISO datetime format (yyyy-MM-ddTHH:mm:ss).

### 代码演示（官方）
```html
<script>
  angular.module('dateExample', [])
    .controller('DateController', ['$scope', function($scope) {
      $scope.example = {
        value: new Date(2010, 11, 28, 14, 57)
      };
    }]);
</script>
<form name="myForm" ng-controller="DateController as dateCtrl">
  <label for="exampleInput">Pick a date between in 2013:</label>
  <input type="datetime-local" id="exampleInput" name="input" ng-model="example.value"
      placeholder="yyyy-MM-ddTHH:mm:ss" min="2001-01-01T00:00:00" max="2013-12-31T00:00:00" required />
  <div role="alert">
    <span class="error" ng-show="myForm.input.$error.required">
        Required!</span>
    <span class="error" ng-show="myForm.input.$error.datetimelocal">
        Not a valid date!</span>
  </div>
  <tt>value = {{example.value | date: "yyyy-MM-ddTHH:mm:ss"}}</tt><br/>
  <tt>myForm.input.$valid = {{myForm.input.$valid}}</tt><br/>
  <tt>myForm.input.$error = {{myForm.input.$error}}</tt><br/>
  <tt>myForm.$valid = {{myForm.$valid}}</tt><br/>
  <tt>myForm.$error.required = {{!!myForm.$error.required}}</tt><br/>
</form>
```

## input[email]
### 额外属相（Arguments）
Param | Type | Details
---------|-----------|------------
pattern | string | Similar to ngPattern except that the attribute value is the actual string that contains the regular expression body that will be converted to a regular expression as in the ngPattern directive.

### 代码演示（官方）
```html
<script>
  angular.module('emailExample', [])
    .controller('ExampleController', ['$scope', function($scope) {
      $scope.email = {
        text: 'me@example.com'
      };
    }]);
</script>
  <form name="myForm" ng-controller="ExampleController">
    <label>Email:
      <input type="email" name="input" ng-model="email.text" required>
    </label>
    <div role="alert">
      <span class="error" ng-show="myForm.input.$error.required">
        Required!</span>
      <span class="error" ng-show="myForm.input.$error.email">
        Not valid email!</span>
    </div>
    <tt>text = {{email.text}}</tt><br/>
    <tt>myForm.input.$valid = {{myForm.input.$valid}}</tt><br/>
    <tt>myForm.input.$error = {{myForm.input.$error}}</tt><br/>
    <tt>myForm.$valid = {{myForm.$valid}}</tt><br/>
    <tt>myForm.$error.required = {{!!myForm.$error.required}}</tt><br/>
    <tt>myForm.$error.email = {{!!myForm.$error.email}}</tt><br/>
  </form>
```

## input[number]

> **注意事项：** input[number]不是不能输入number,比如 **'e'**

### 额外属性（Arguments)
Param | Type | Details
---------|-----------|------------
pattern | string | Similar to ngPattern except that the attribute value is the actual string that contains the regular expression body that will be converted to a regular expression as in the ngPattern directive.

### 代码演示（官方）
```html
<script>
  angular.module('numberExample', [])
    .controller('ExampleController', ['$scope', function($scope) {
      $scope.example = {
        value: 12
      };
    }]);
</script>
<form name="myForm" ng-controller="ExampleController">
  <label>Number:
    <input type="number" name="input" ng-model="example.value"
           min="0" max="99" required>
 </label>
  <div role="alert">
    <span class="error" ng-show="myForm.input.$error.required">
      Required!</span>
    <span class="error" ng-show="myForm.input.$error.number">
      Not valid number!</span>
  </div>
  <tt>value = {{example.value}}</tt><br/>
  <tt>myForm.input.$valid = {{myForm.input.$valid}}</tt><br/>
  <tt>myForm.input.$error = {{myForm.input.$error}}</tt><br/>
  <tt>myForm.$valid = {{myForm.$valid}}</tt><br/>
  <tt>myForm.$error.required = {{!!myForm.$error.required}}</tt><br/>
 </form>
```

## input[radio]
### 额外属性（Arguments)

Param | Type | Details
--------|------|-------
value | string | The value to which the ngModel expression should be set when selected. Note that value only supports string values, i.e. the scope model needs to be a string, too. Use ngValue if you need complex models (number, object, ...).
ngValue | string | Angular expression to which ngModel will be be set when the radio is selected. Should be used instead of the value attribute if you need a non-string ngModel (boolean, array, ...).

> **注意事项：** value属性只支持string类型，其他类型或Angular expression则要用ngValue

### 完整代码演示
```html
<!DOCTYPE html>
<html ng-app="radioModule">
<head>
	<title>ng-input[radio]</title>
	<script type="text/javascript" src="../bower_components/angular/angular.js"></script>
	<script type="text/javascript">
	angular.module('radioModule',[])
	.controller('RadioController',['$scope',function($scope){
		$scope.complex = {
			value: 'complex'
		}
		$scope.color = {
			name: 'red'
		}
	}]);
	</script>
</head>
<body ng-controller="RadioController">
<form name="radioForm">
	<input type="radio" name="simpleRadio" value="red" ng-model="color.name">Red
	<input type="radio" name="simpleRadio" ng-value="complex" ng-model="color.name">Green
	<br>
	<code>color.name {{color.name | json}}</code>
</form>
</body>
</html>
```

## input[text]
### 额外属性（Arguments)

Param | Type | Details
-------|------|------
pattern(optional) | string | Similar to ngPattern except that the attribute value is the actual string that contains the regular expression body that will be converted to a regular expression as in the ngPattern directive.
ngTrim(optional) | boolean | If set to false Angular will not automatically trim the input. This parameter is ignored for input[type=password] controls, which will never trim the input.(default=true)

### 完整代码演示
```html
<!DOCTYPE html>
<html ng-app="textModule">
<head>
	<title>ng-input[text]</title>
	<script type="text/javascript" src="../bower_components/angular/angular.js"></script>
	<script type="text/javascript">
	angular.module('textModule',[])
	.controller('TextController', ['$scope', function($scope){
		$scope.example = {
			value: 'Guest',
			pattern: /^\s*\w*\s*$/
		}
	}])
	</script>
</head>
<body ng-controller="TextController">
	<form name="textForm">
		<input type="text" name="textName" placeholder=""
		ng-model="example.value" ng-pattern="example.pattern" ng-trim="false">

		<br><code>example.value:{{example.value}}</code>
		<br><code>textForm.$error.pattern:{{textForm.$error.pattern}}</code>

	</form>
</body>
</html>
```

## input[url]

> 参考链接[Input[url] API](https://docs.angularjs.org/api/ng/input/input%5Burl%5D)

## input[week]
> **注意事项：** ng-model必须是一个date object

### 额外属性（Arguments）

Param | Type | Details 
--------|----------|---------
min | string | Sets the min validation error key if the value entered is less than min. This must be a valid ISO week format (yyyy-W##).
max | string | Sets the max validation error key if the value entered is greater than max. This must be a valid ISO week format (yyyy-W##).

### 代码演示（官方）
```html
<script>
angular.module('weekExample', [])
  .controller('DateController', ['$scope', function($scope) {
    $scope.example = {
      value: new Date(2013, 0, 3)
    };
  }]);
</script>
<form name="myForm" ng-controller="DateController as dateCtrl">
  <label>Pick a date between in 2013:
    <input id="exampleInput" type="week" name="input" ng-model="example.value"
           placeholder="YYYY-W##" min="2012-W32"
           max="2013-W52" required />
  </label>
  <div role="alert">
    <span class="error" ng-show="myForm.input.$error.required">
        Required!</span>
    <span class="error" ng-show="myForm.input.$error.week">
        Not a valid date!</span>
  </div>
  <tt>value = {{example.value | date: "yyyy-Www"}}</tt><br/>
  <tt>myForm.input.$valid = {{myForm.input.$valid}}</tt><br/>
  <tt>myForm.input.$error = {{myForm.input.$error}}</tt><br/>
  <tt>myForm.$valid = {{myForm.$valid}}</tt><br/>
  <tt>myForm.$error.required = {{!!myForm.$error.required}}</tt><br/>
</form>
```

----------
> Written with [StackEdit](https://stackedit.io/).