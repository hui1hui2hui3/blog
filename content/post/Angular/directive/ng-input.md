---
title: "Input(ng-input) 学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---

#Input(ng-input) 学习笔记

> **参考链接**：[Input API](https://docs.angularjs.org/api/ng/directive/input)

> **注意事项: **
> 1. 不是所有的input都支持ng-model, *input[file]*不支持
> 2. 使用ng-model属性时，必须使用a.b形式，不能单独使用b的形式

##Arguments

| Param	| Type	| Details|
|--------|-----------|---------|
|ngModel	|string	|Assignable angular expression to data-bind to.|
|name(optional)|string	|Property name of the form under which the control is published.|
|required(optional)|string	|Sets required validation error key if the value is not entered.|
|ngRequired(optional)|boolean	|Sets required attribute if set to true|
|ngMinlength(optional)|number	|Sets minlength validation error key if the value is shorter than minlength.|
|ngMaxlength(optional)|number	|Sets maxlength validation error key if the value is longer than maxlength. Setting the attribute to a negative or non-numeric value, allows view values of any length.|
|ngPattern(optional)|string	|Sets pattern validation error key if the ngModel value does not match a RegExp found by evaluating the Angular expression given in the attribute value. If the expression evaluates to a RegExp object, then this is used directly. `If the expression evaluates to a string, then it will be converted to a RegExp after wrapping it in ^ and $ characters. For instance, "abc" will be converted to new RegExp('^abc$').`**Note:** Avoid using the g flag on the RegExp, as it will cause each successive search to start at the index of the last search's match, thus not taking the whole input value into account.|
|ngChange(optional)|string	|Angular expression to be executed when input changes due to user interaction with the input element.|
|ngTrim(optional)|boolean	|If set to false Angular will not automatically trim the input. This parameter is ignored for input[type=password] controls, which will never trim the input. *(default:true)* |


----------
##完整参考代码
```
<!DOCTYPE html>
<html ng-app="inputModule">
<head>
	<title>ng-input</title>
	<script src="../bower_components/angular/angular.js" type="text/javascript"></script>
	<script type="text/javascript">
	angular.module('inputModule',[])
	.controller('inputController',['$scope',function($scope) {
		$scope.user = {name:'huis',password:'123'};
	}])
	</script>
</head>
<body ng-controller="inputController">
<form name="myForm">
	userName: <input name="userName" type="text" ng-model="user.name" required>
	<div role="alert">
		<span ng-show="myForm.userName.$error.required">Required</span>
	</div>
	password: <input type="password" name="password" ng-model="user.password"
	ng-minlength="2" ng-maxlength="11">
	<div role="alert">
		<span ng-show="myForm.password.$error.minlength">Too Short</span>
		<span ng-show="myForm.password.$error.maxlength">Too Long</span>
	</div>
	<br>
	<code>userName:{{user.name}}</code><br>
	<code>password:{{user.password}}</code><br>

	<code>myForm.userName.$valid:{{myForm.userName.$valid}}</code><br>
	<code>myForm.userName.$error:{{myForm.userName.$error}}</code><br>


	<code>myForm.password.$valid:{{myForm.password.$valid}}</code><br>
	<code>myForm.password.$error:{{myForm.password.$error}}</code><br>

	<code>myForm.$valid:{{myForm.$valid}}</code><br>
	<code>myForm.$error.required:{{myForm.$error.required}}</code><br>	
	<code>myForm.$error.minlength:{{myForm.$error.minlength}}</code><br>
	<code>myForm.$error.maxlength:{{myForm.$error.maxlength}}</code><br>
	
</form>
</body>
</html>
```
>  输出结果：
> `userName:
password:
myForm.userName.$valid:false
myForm.userName.$error:{"required":true}
myForm.password.$valid:false
myForm.password.$error:{"maxlength":true}
myForm.$valid:false
myForm.$error.required:[{"$viewValue":"","$validators":{},"$asyncValidators":{},"$parsers":[],"$formatters":[null],"$viewChangeListeners":[],"$untouched":false,"$touched":true,"$pristine":false,"$dirty":true,"$valid":false,"$invalid":true,"$error":{"required":true},"$name":"userName","$options":null}]
myForm.$error.minlength:
myForm.$error.maxlength:[{"$viewValue":"123456789011","$validators":{},"$asyncValidators":{},"$parsers":[],"$formatters":[null],"$viewChangeListeners":[],"$untouched":false,"$touched":true,"$pristine":false,"$dirty":true,"$valid":false,"$invalid":true,"$error":{"maxlength":true},"$name":"password","$options":null}]`

-----------
> **注：**可以发现当出现错误时，**ng-model对应的值为空**

----------
> Written with [StackEdit](https://stackedit.io/).