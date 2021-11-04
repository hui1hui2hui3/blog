---
title: "Ionic问题笔记"
date: 2016-12-15
tags: ["Ionic"]
draft: false
--- 


## 1. 使用ion-nav-back-button和tabs时，子页面的返回问题
    **实现效果：** /index/page2/tab1 点击进入 /index/page2/tab2
                    这时点击返回按钮想要的是返回到 /index/page2
                    而不是/index/page2/tab1(这个是现实的问题效果），如图：


### 解决方法1：这个可能有问题
```js
var backView = $ionicHistory.backView();
$state.go('app.patInfo.baseInfo').then(function(){
$ionicHistory.backView(backView);
}); 
```

### 解决方法2：重写ion-nav-back-button的click方法,核心代码如下：
```js
function goBackSmart() {
var currentView = $ionicHistory.currentView(),
backView = $ionicHistory.backView();
if (backView) {
var newBackView = _getBackView(currentView, backView);
goBack(newBackView.index - currentView.index);
}
}
 
function _getBackView(currentView, backView) {
var curP = _getParentStateName(currentView.stateName),
backP = _getParentStateName(backView.stateName),
newBackView = backView;
 
//判断是否是不为app的同一父类
while (curP == backP && backP.split(".").length > 1) {
        newBackView = $ionicHistory.getViewById(newBackView.backViewId);
curP = backP;
backP = _getParentStateName(newBackView.stateName);
}
return newBackView;
}
 
//获取父类的状态名：例：app.test.test1 -> app.test
function _getParentStateName(stateName) {
return stateName.substring(0, stateName.lastIndexOf("."));
}
```
## 2.  $ionHistory对象的histories堆栈中页面何时销毁
    当页面流程走 A->B->C->D 时，历史记录为[A,B,C,D]，cursor为3（从0开始）
         比如：当调用goback(-3)返回或调用go一级一级返回时，
                     返回到A ,                这时历史记录为[A,B,C,D]不变，但cursor为0，
                    下面分三种情况：
                    1.这时如果调用A->B,这时历史记录为[A,B,C,D]不变，但cursor为1，
                    2.这时如果调用A->C（原本存在堆栈中的）,这时历史记录为[A,C],cursor为1，原本的B,C,D页面销毁
                    3.这时如果调用A->F（新的）, 这时历史记录为[A,F],(B,C,D)页面就会销毁掉，和A->C情况一样
**也就是说cursor不是堆栈的末尾时，如果这时跳转想其他不是当前的下一个页面时，就会产生新的页面，且原cursor之后的页面都会销毁**
## 3.  state.go 和 goback 返回时的具体区别--1
		1.跨页面数量为0时（也就是两个相邻的页面返回时），go和goback无区别，都是只改变cursor的值
					比如： A->B->C->D ，  历史记录为[A,B,C,D],cursor为3
						go情况下： go(C) ，历史记录为[A,B,C,D],cursor为2
	                                            goBack情况下： goback(-1), 历史记录为[A,B,C,D],cursor为2
		1.跨页面数量为1个以上时（最少间隔一个），
	                                    比如： A->B->C->D  ，历史记录为[A,B,C,D],cursor为3
						go情况下，go(B)后，这时历史记录是[A,B,C,D,B],cursor为4
						goBack情况下,goback(-2),这时历史记录是[A,B,C,D]不变，但cursor为1
					附加情况：这时都在B页面，如果要跳转到G时，也就是B->G时，
						第一种情况下go(G): 就会变成， [A,B,C,D,B,G],cursor为5
						第二种情况下go(G): 就会变成，[A,B,G],cursor为2，页面C,D销毁掉了（可**参考页面何时销毁** ）
## 4. ion-nav-bar 样式控制
> 在应用中我们经常会要自定义nav-bar的样式，下面说一下问题：

### 更改样式：行内样式（无效）
```html
<ion-nav-bar style="background:red"></ion-nav-bar>
```

### 更改样式：class样式（有效）
```css
.custom-class {
background-color: red;
}
```
```html
<ion-nav-bar class="custom-class"></ion-nav-bar>
```

## 5.  state.go 和 goback 返回时传递参数的问题
1. state.go
```js
 state.go(state,params)
```
2. goback
```js
$ionicHistory.backView().stateParams = {card:true};
$ionicHistory.goBack();
```

> **注意：** 
>>  这时不管是**跨页面数量为0**还是**跨页面数量为1个以上时（最少间隔一个）**都会在堆栈中产生新的view，**为什么？？？？？？？**
>> 具体原因是由于stateParams，因为它的改变导致$ionicHistory产生的card=null的页面stateID和card=true的页面stateID不一致，所以会产生一个新的View加入堆栈中，不再是之前我们分析的goBack一定不会产生新的View的问题

> **具体流程如图：**
A --params:{test:null}-> B---> C  ，历史记录为[A,B,C]，cursor:2
>> 1.  state.go(B,{test:1}),  历史记录为[A,B,C,B]，cursor:3，这时会产生死循环问题
>> 2.  `$ionicHistory.backView().stateParams = {card:true};
$ionicHistory.goBack();`, 历史记录为[A,B,C,B]，cursor:3，这时会产生死循环问题
>> 3.  `state.go(B) or $ionicHistory.goBack()`，历史记录为[A,B,C]，cursor:1, 但需要注意这时我们C页面没有传递参数到B页面

> **解决方法：service**
>
> > 4.  `state.go(B) or $ionicHistory.goBack(); service.data={test:1}`, 这样才能完美解决该问题

## 6. 页面切换方向问题
> 有时我们写的页面经常会发现页面切换的动画会有问题，本来要退出的结果变为进入了等等，解决方法如下：
```js
//进入
$ionicViewSwitcher.nextDirection('forward');
$state.go(state)
//后退
$ionicViewSwitcher.nextDirection('back');
$state.go(state)
```

## 7. collection-repeat
### 7.1 错误问题
> 1. TypeError: Cannot set property 'webkitTransform' of undefined
> 2. TypeError: clone[0].removeAttribute is not a function
>> **原因：同一属性上使用了ng-if指令，改为ng-show/ng-hide**

### 7.2 使用
1. 使用是要指定item-height和item-width,所以说这个指令适用于等宽和等高的列表进行优化，会变动的高度和宽度则不适合
2. 内部不能使用bononce或者:: 因为需要动态更新值

## 8. ion-infinite-scroll如何避免调用2次on-infinite
> `immediate-check="false"`
```javascript
           var avoidLoadTwice = false;
            $scope.loadMore = function() {
                if (!avoidLoadTwice) {
                    avoidLoadTwice = true;
                    $scope.$broadcast('scroll.infiniteScrollComplete');
                } else {
                     loadData().finally(funciton(){
                          avoidLoadTwice  = false;
                           $scope.$broadcast('scroll.infiniteScrollComplete');
                    })
                } 
            };
```
