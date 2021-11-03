---
title: "ng-repeat 学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ng-repeat 学习笔记

> **重点1：** ng-repeat的每一个子项都会产生一个scope
> **重点2：** ng-repeat虽然可以使用Object,如：`ng-repeat="(key, value) in myObj"` 但这不是推荐的方式，推荐的是Array, 参考链接**[toArrayFilter](http://ngmodules.org/modules/angular-toArrayFilter)**

## properties 

Variable | Type | Details
-------|-------|----------
`$index` | number | iterator offset of the repeated element (0..length-1) 
`$first` | boolean | true if the repeated element is first in the iterator.
`$middle` | boolean | true if the repeated element is between the first and last in the iterator.
`$last` |boolean | true if the repeated element is last in the iterator.
`$even` | boolean	| true if the iterator position $index is even (otherwise false).
`$odd` | boolean | true if the iterator position $index is odd (otherwise false).

## DOM Changes

- When an item is **added**, a new instance of the template is added to the DOM.
- When an item is **removed**, its template instance is removed from the DOM.
- When items are **reordered**, their respective templates are reordered in the DOM.

## Track By

> **注意1：** ngRepeat中是不允许存在重复的item的，如果想要重复，则自定义修改通过track by
> **注意2：** track by 必须是最后的表达式

###默认方式
```html
<div ng-repeat="obj in collection track by $id(obj)">
  {{obj.prop}}
</div>
```
###自定义方式
```html
<div ng-repeat="n in [42, 42, 43, 43] track by $index">
  {{n}}
</div>
```
```html
<div ng-repeat="n in [42, 42, 43, 43] track by myTrackingFunction(n)">
  {{n}}
</div>
```
```html
<div ng-repeat="model in collection track by model.id">
  {{model.name}}
</div>
```
```html
<div ng-repeat="model in collection | orderBy: 'id' as filtered_result track by model.id">
    {{model.name}}
</div>
```

## ngRepeat

> - **variable in expression**. 
> >1. **For example**: `album in artist.albums`
> 
> - **(key, value) in expression**. 
> >1. **For example**: (name, age) in `{'adam':10, 'amalie':12}`
> 
> - **variable in expression track by tracking_expression**. 
> >1. **For example**: `item in items` is equivalent to `item in items track by $id(item)`. 
> >2. **For example**: `item in items track by item.id`.
> >3. **For example**: `item in items | filter:searchText track by item.id`
> 
> - **variable in expression as alias_expression**. 
> >1. **For example**: `item in items | filter:x as results`
> >2. **For example**: `item in items | filter : x | orderBy : order | limitTo : limit as results` .

## 完整例子
```html
<!DOCTYPE html>
<html ng-app="repeatModule">

<head>
    <title>ng-repeat</title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('repeatModule', [])
        .controller('RepeatController', ['$scope', function($scope) {

        }]);
    </script>
</head>

<body ng-controller="RepeatController as repeatCtrl">
    <div ng-init="friends = [
  {name:'John', age:25, gender:'boy'},
  {name:'Jessie', age:30, gender:'girl'},
  {name:'Johanna', age:28, gender:'girl'},
  {name:'Joy', age:15, gender:'girl'},
  {name:'Mary', age:28, gender:'girl'},
  {name:'Peter', age:95, gender:'boy'},
  {name:'Sebastian', age:50, gender:'boy'},
  {name:'Erika', age:27, gender:'girl'},
  {name:'Patrick', age:40, gender:'boy'},
  {name:'Samantha', age:60, gender:'girl'}
]">
        I have {{friends.length}} friends. They are:
        <input type="search" ng-model="q" placeholder="filter friends..." aria-label="filter friends" />
        <ul class="example-animate-container">
            <li class="animate-repeat" ng-repeat="friend in friends | filter:q as results track by friend.name">
                [{{$index + 1}}] {{friend.name}} who is {{friend.age}} years old.
            </li>
            <li class="animate-repeat" ng-if="results.length == 0">
                <strong>No results found...</strong>
            </li>
        </ul>
    </div>
</body>

</html>

```

> Written with [StackEdit](https://stackedit.io/).