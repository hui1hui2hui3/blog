---
title: "rootScope.Scope学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# rootScope.Scope学习笔记
[TOC]

> **注意：** scope对象的`$parent`并不代表该scope一定继承自该`$parentScope`(或者说可以调用该`$parentScope`中的方法或属性)，而仅仅表示具有DOM中的上下级关系。真正能够访问的方法或属性是来自原型继承的parentScope(_proto_)

## Methods

### `$new(isolate,parent);`

Parameters

Param | Type | Details
-------|--------|----------
**isolate** | boolean | If true, then the scope does not prototypically inherit from the parent scope. The scope is isolated, as it can not see parent scope properties. When creating widgets, it is useful for the widget to not accidentally read parent state.
**parent**(*optional*) | Scope | The Scope that will be the $parent of the newly created scope. Defaults to this scope if not provided. This is used when creating a transclude scope to correctly place it in the scope hierarchy while maintaining the correct prototypical inheritance.(简单说就是为了transclude能够指定parent prototype继承)(***default:this***)


### `$watch(watchExpression, listener, [objectEquality]);`

> **注意1：** 默认比较方式是newValue === oldValue, 但是当objectEquality = true时，会调用angular.copy保存上一次的值，并调用angular.equals方法比较newValue和oldValue。
> **注意2：** 第一次初始化时会调用一次listener，newValue和oldValue都为undefined

Parameters

Param | Type | Details
------|------|--------
**watchExpression** | function() string | Expression that is evaluated on each $digest cycle. A change in the return value triggers a call to the listener. - `string`: Evaluated as expression. - `function(scope)`: called with current scope as a parameter.
**listener** | function(newVal,oldVal,scope | Callback called whenever the value of watchExpression changes. - newVal contains the current value of the watchExpression. - oldVal contains the previous value of the watchExpression. - scope refers to the current scope
**objectEquality**(*optional*) | boolean | Compare for object equality using `angular.equals` instead of comparing for reference equality.


### `$watchGroup(watchExpressions, listener);`

> **注意1：** 这时用来同时检测多个值的，其中任意一个值变化都会触发listener
> **注意2：** 初始化时数组中虽然有多个值，但是仍然只会触发一次listener，而不是多次，这时的newValues和oldValues是[undefined, undefined...], 

Parameters

Param | Type | Details
--------|--------|---------
**watchExpressions** | Array(String,Function(Scope)) | Array of expressions that will be individually watched using $watch()
**listener** | function(newVals,oldVals,scope) | Callback called whenever the return value of any expression in watchExpressions changes The newValues array contains the current values of the watchExpressions, with the indexes matching those of watchExpression and the oldValues array contains the previous values of the watchExpressions, with the indexes matching those of watchExpression The scope refers to the current scope.

### `$watchCollection(obj, listener);`

> **注意1：** 浅监测 object 和 array
> **注意2：** 
> > - obj 触发$digest的条件是added, removed, moved
> > 如1： var obj = {a:1}  
> > add:  obj.b = 1
> > removed: delete obj.b
> > moved: 无
> > 如2：var arr = [1]
> > add: arr[1] = 2
> > removed: arr.splice(1,1)
> > moved: arr.sort(reverse)

Parameters

Param | Type | Details
-------|--------|---------
**obj** | string function(scope) | Evaluated as expression. The expression value should evaluate to an object or an array which is observed on each $digest cycle. Any shallow change within the collection will trigger a call to the listener.
**listener** | function(newCollection,oldCollection,scope) | a callback function called when a change is detected.


### `$digest();`

> **注意1：** digest会执行watchers中的所有的listener，直到没有可以执行的listener或者重复执行超过10次（提示错误`'Maximum iteration limit exceeded.'`)
>  **注意2：** 不应该在controllers和directives中调用$digest,而应该调用$apply方法(通常在directive中）


### `$destroy();`
> **注意：** 销毁前，会先发出boroadcast，type类型是`$destroy`, 可以用于通知子类首先完成destroy

### `$eval([expression], [locals]);`

**Example**
```js
var scope = ng.$rootScope.Scope();
scope.a = 1;
scope.b = 2;
expect(scope.$eval('a+b')).toEqual(3);
expect(scope.$eval(function(scope){ return scope.a + scope.b; })).toEqual(3);
```

### `$evalAsync([expression], [locals]);`

>  **注意1：** 它不能确保何时执行
>  **注意2：** 如果在`$digest` 循环外调用，则或重新触发一次`$digest`，但建议改变model还是放在`$apply`中

**执行条件：**
1.  方法安排后，最好在DOM渲染之前执行
2.  最后一个$digest circle中的最后执行


### `$apply([exp]);`

> **重点：** 用于在其他框架中执行angular中的expression，包括（DOM events,setTimeout,XHR or third party librarys)

**Example**
```js
document.getElementById('btnApply').onclick= function(){
            	$scope.$apply(function(){
            		scopeCtrl.name = 'applyName';
            	});
            };

```

**伪代码：**
```js
function $apply(expr) {
  try {
    return $eval(expr);
  } catch (e) {
    $exceptionHandler(e);
  } finally {
    $root.$digest();
  }
}
```

### `$applyAsync([exp]);`

> **重点1：**  延迟执行$apply方法，通常在10ms左右
> **重点2：** 可以在一次digest方法中执行多个expressions

### `$on(name, listener);`

**listener** function is `function(event,args...)`
**event** object attribures:

- **targetScope** - {Scope}: the scope on which the event was `$emit`-ed or `$broadcast`-ed.
- **currentScope** - {Scope}: the scope that is currently handling the event. Once the event propagates through the scope hierarchy, this property is set to null.
- **name** - {string}: name of the event.
- **stopPropagation** - {function=}: calling stopPropagation function will cancel further event propagation (available only for events that were `$emit`-ed).
- **preventDefault** - {function}: calling preventDefault sets defaultPrevented flag to true.
- **defaultPrevented** - {boolean}: true if preventDefault was called.

### `$emit(name, args);`

**重点1：**根据scope层级结构向上传播，直到rootScope，如果中途有scope调用stopPropagation方法，则终止事件的传播

### `$broadcast(name, args);`

**重点2：** 根据scope层级结构向下遍历所有子类传播，该事件不能被取消和终止

## 完整代码例子
```html
<!DOCTYPE html>
<html ng-app="scopeModule">

<head>
    <title>rootScope.Scope</title>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('scopeModule', [])
        .controller('ScopeController', ['$scope','$timeout', function($scope,$timeout) {
            var scopeCtrl = this;
            scopeCtrl.changeNumber = 0;
            scopeCtrl.items = ['a', 'b'];
            scopeCtrl.complexItems = [{
                name: 'a'
            }, {
                name: 'b'
            }];

            /**
             * ######################################
             * $watch
             */
            //这里这样写会报错
            // $scope.$watch(scopeCtrl.name, function(newValue, oldValue) {
            //     if (newValue === oldValue) {
            //         return;
            //     }
            //     console.log('newValue:' + newValue + ' oldValue:' + oldValue);
            //     scopeCtrl.newName = newValue;
            // });
            $scope.$watch(function() {
                return scopeCtrl.name;
            }, function(newValue, oldValue) {
                if (newValue === oldValue) {
                    return;
                }
                scopeCtrl.newName = newValue;

                /**
                 * 这里是为了演示比较这两个方法的
                 * -----------------------------
                 * $apply and $evalAsync
                 */
                //该方法可以正确执行
                /*$scope.$evalAsync(function(){
					scopeCtrl.name = 'evalAsyncName';
            	});*/

                // 该方法会报错：$digest already in progress
                /*$scope.$apply(function(){
					scopeCtrl.name = 'applyName';
            	});*/
            });


            /**
             * ######################################
             * $watchGroup
             */
            $scope.$watchGroup([function() {
                return scopeCtrl.name;
            }, function() {
                return scopeCtrl.age;
            }], function(newValue, oldValue, scope) {
                if (newValue === oldValue) {
                    return;
                }
                console.log(newValue);
                console.log(oldValue);
                console.log('-----------');
                scopeCtrl.changeNumber++;
            });


            /**
             * ######################################
             * $watchCollection
             */
            $scope.$watchCollection(function() {
                return scopeCtrl.items;
            }, function(newValues, oldValues, scope) {
                if (newValues === oldValues) {
                    return;
                }
                console.log(newValues);
                console.log(oldValues);
                scopeCtrl.changeNumber++;
            });

            $scope.$watchCollection(function() {
                return scopeCtrl.complexItems;
            }, function(newValues, oldValues, scope) {
                if (newValues === oldValues) {
                    return;
                }
                console.log(newValues);
                console.log(oldValues);
                scopeCtrl.changeNumber++;
            });

            /**
             * ######################################
             * $eval
             */
            $scope.a = 1;
            $scope.b = 2;
            console.log($scope.$eval('a+b'));
            console.log($scope.$eval(function(scope) {
                return scope.a * scope.b;
            }));


            /**
             * ######################################
             * $apply
             */
            document.getElementById('btnApply').onclick = function() {
                $scope.$apply(function() {
                    scopeCtrl.name = 'applyName';
                });
            };


            /**
             * ######################################
             * $evalAsync
             */
            scopeCtrl.evalAsyncFn = function() {
                $scope.$evalAsync(function() {
                    scopeCtrl.name = 'evalAsyncName';
                });
            };

            /**
             * ######################################
             * $evalAsync
             */
            document.getElementById('btnApplyAsync').onclick = function() {
                $scope.$applyAsync(function() {
                    scopeCtrl.name = 'ApplyAsync';
                });
                $scope.$applyAsync(function() {
                    scopeCtrl.age = 20;
                });
            };

            /**
             * ######################################
             * $on
             */
            $scope.$on('emitTest', function(event, value) {
                scopeCtrl.name = value;
            });

            $scope.$on('broadcastTest', function(event, value) {
                scopeCtrl.age = value;
            });
            
            /**
             * ######################################
             * $emit
             */
            setTimeout(function() {
            	$scope.$apply(function(){
            		$scope.$emit('emitTest', 'emitTest value');
            	});
            }, 2000);


            /**
             * ######################################
             * $broadcast
             */
            $timeout(function() {
                $scope.$broadcast('broadcastTest', 25);
            }, 2000);

        }]);
    </script>
</head>

<body ng-controller="ScopeController as scopeCtrl">
    name:
    <input type="text" ng-model="scopeCtrl.name" ng-model-options="{debounce:2000}"> name:{{scopeCtrl.name}}
    <br> newName:{{scopeCtrl.newName}}
    <br> age:
    <input type="text" ng-model="scopeCtrl.age" ng-model-options="{debounce:1000}">
    <br><pre>{{scopeCtrl.changeNumber}}</pre>
    <br> 简单数组['1','2']
    <br>
    <br>
    <input type="button" ng-click="scopeCtrl.items.push(scopeCtrl.name)" value="push(scopeCtrl.name)触发$digest">
    <input type="button" ng-click="scopeCtrl.items[0]='test'" value="items[0]='test'触发$digest"> {{scopeCtrl.items | json}}
    <br>
    <br> 复杂对象数组[{a:b}]
    <br>
    <br>
    <input type="button" ng-click="scopeCtrl.complexItems.push({name:scopeCtrl.name})" value=".push({name:scopeCtrl.name})触发$digest">
    <input type="button" ng-click="scopeCtrl.complexItems[0].name='test'" value="complexItems[0].name='test'不会触发$digest"> {{scopeCtrl.complexItems | json}}
    <br>
    <button id="btnEvalAsync" ng-click="scopeCtrl.evalAsyncFn()">触发evalAsync</button>
    <br>
    <button id="btnApply">触发apply</button>
    <br>
    <button id="btnApplyAsync">触发applyAsync</button>
</body>

</html>

```

> Written with [StackEdit](https://stackedit.io/).