---
title: "ngPluralize学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ngPluralize学习笔记

> **[ngPluralize API链接](https://docs.angularjs.org/api/ng/directive/ngPluralize)**

> **重点：** 该指令依赖于 en-US localization rules，该规则已经绑定于angular.js当中，重写方法参见**[Angular i18n](https://docs.angularjs.org/guide/i18n)** , 根据配置不同指令的使用参数表示意思参见**[Plural categories](http://unicode.org/repos/cldr-tmp/trunk/diff/supplemental/language_plural_rules.html)**
>   **Angular's default en-US locale: "one" and "other".**

## Arguments

Param | Type | Details
--------|---------|--------
count|`string` `expression` | The variable to be bound to.
when | string | The mapping between plural category to its corresponding strings.
offset(optional) | number | Offset to deduct from the total number.

## 使用方法1 -- count and when (default:'en')
```html
<input type="text" ng-model="personCount">
<ng-pluralize count="personCount"
                 when="{'0': 'Nobody is viewing.',
                     'one': '1 person is viewing.',
                     'other': '{} people are viewing.'}">
</ng-pluralize>
```
> **解释：** 
>> - 0是明确值，所以优先匹配
>> - one匹配1， 所以匹配1
>> - other匹配剩余数字，匹配其他 
>> - **{} 作用：** 相当于{{personCount}}

## 使用方法2 -- count,offset and when(default:'en')

```html
<input type="text" ng-model="personCount">
<ng-pluralize count="personCount" offset=2
              when="{'0': 'Nobody is viewing.',
                     '1': '{{person1}} is viewing.',
                     '2': '{{person1}} and {{person2}} are viewing.',
                     'one': '{{person1}}, {{person2}} and one other person are viewing.',
                     'other': '{{person1}}, {{person2}} and {} other people are viewing.'}">
</ng-pluralize>
```
> **解释：**
> > - **offset作用: 除了明确值，count-offset=rule(规则匹配的值)**
> > - 0,1,2位明确值，则优先匹配显示
> > - one规则匹配1,  根据（count-offset=rule), 则count为3时匹配该规则
> > - other规则匹配其他，同样根据算法，则count>3时匹配该规则

## 完整例子参考

```html
<!DOCTYPE html>
<html ng-app='plur'>

<head>
    <title></title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('plur', []);
    angular.module('plur')
        .controller('plurController', ['$scope', function($scope) {
            $scope.person1 = 'Huis';
            $scope.person2 = 'Yonna';
        }]);
    </script>
</head>

<body ng-controller="plurController">
    <input type="text" ng-model="test.count">
    <br>
    <ng-pluralize count="test.count" when="{
		'0':'Nobody',
		'one':'{} person',
		'12':'What a Heal!',
		'other':'{} other person'
	}"> </ng-pluralize>
    <br>
    <ng-pluralize count="test.count" when="{
		'0': 'Nobody is View',
		'1': '{{person1}} is View',
		'2': '{{person1}} and {{person2}} is View',
		'one': '{{person1}} , {{person2}} and one other person is View',
		'other': '{{person1}} , {{person2}} and {} other person is View'
	}" offset="2"></ng-pluralize>
</body>

</html>

```

> Written with [StackEdit](https://stackedit.io/).