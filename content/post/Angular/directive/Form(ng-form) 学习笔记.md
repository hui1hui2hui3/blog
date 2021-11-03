---
title: "Form(ng-form) 学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
#Form(ng-form) 学习笔记
[TOC]

##1. CSS classes

> **详情参见**[Form API](https://docs.angularjs.org/api/ng/directive/form)

1. **ng-valid** is set if the form is valid.
2. **ng-invalid** is set if the form is invalid.
3. **ng-pristine** is set if the form is pristine.
4. **ng-dirty** is set if the form is dirty.
5. **ng-submitted** is set if the form was submitted.

上面的CSS名称表示的是当form处于某种状态时，form会触发的样式。

**例子：想要实现的是form当中有为通过校验的控件时，显示背景色为红色。**
CSS实现代码：
```css
.my-form {
	transition: all linear 0.5s;
	background: transparent;
}
.my-form.ng-invalid {
	background: red;
}
```
HTML实现代码：
```html
<form class="my-form">
	name:<input type="text" value="huis1" required>
</form>
```
用以上的代码你会发现，**.ng-invalid样式不会生效**。
如果想要生效怎么办？看下面的代码

HTML实现代码：
```html
<form class="my-form">
	name:<input type="text" ng-model="name" required>
</form>
```
总结：**如果想要form下的css classes生效，则必须绑定ng-model**

##2. form.FormController

> **详情参见** [FormController API](https://docs.angularjs.org/api/ng/type/form.FormController)

这里主要说属性：
> - $pristine         **form未修改过**
> - $dirty              **form修改过**
> - $valid              **验证通过**
> - $invalid           **验证不通过**
> - $submitted      **已提交**
> - $error              **验证不通过的错误**

> > Built-in validation tokens:
> 1. email
> 2. max
> 3. maxlength
> 4. min
> 5. minlength
> 6. number
> 7. pattern
> 8. required
> 9. url
> 10. date
> 11. datetimelocal
> 12. time
> 13. week
> 14. month

--------
访问属性的方法，主要依赖属性：**name**
例如：
1. 针对指定的input验证, 形式：**formName.inputName.attr[key(only error)]**
`formName.inputName.$error.required` `formName.inputName.$pristine`
2. 针对整个form的验证，形式：**formName.attr[key(only error)]**

## 完整代码参考
```html
<!DOCTYPE html>
<html ng-app="ng-form">
<head>
	<title>ng-form</title>
	<script type="text/javascript" src="../bower_components/angular/angular.js"></script>
	<script type="text/javascript">
	angular.module('ng-form',[])
	.controller('NgFormController',['$scope',function($scope) {
		$scope.name = 'huis';
	}]);
	</script>
	<style type="text/css">
		.my-form {
			transition: all linear 0.5s;
			background: transparent;
		}

		.my-form.ng-invalid {
			background: red;
		}
	</style>
</head>
<body ng-controller="NgFormController">
	<form name="myForm" class="my-form">
		name:<input name="name" type="text" ng-model="name" required>
		<span ng-show="myForm.name.$error.required">Required</span><br>
		<span ng-show="myForm.name.$pristine">Pristine</span><br>

		<code>name:{{name}}</code><br>
		<code>myForm.name.$valid:{{myForm.name.$valid}}</code><br>
		<code>myForm.name.$error:{{myForm.name.$error}}</code><br>
		<code>myForm.$valid:{{myForm.$valid}}</code><br>
		<code>myForm.$error.required:{{!!myForm.$error.required}}</code><br>

	</form>
</body>
</html>
```

---------
> Written with [StackEdit](https://stackedit.io/).