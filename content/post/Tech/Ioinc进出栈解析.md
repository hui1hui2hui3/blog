---
title: "Ioinc进出栈解析"
date: 2016-12-15
tags: ["Ionic"]
draft: false
---

## `$state.go` 和 `$ionicHistory.goback` 返回时的具体区别

**注明：下面两种演示情况的A,B,C,D页面均为一级页面，也就是不含有`<ion-nav-view>`页面。复杂页面（abstract：true)的go和goBack情况参见结论。**
    
### 1. 跨页面数量为0时（也就是两个相邻的页面返回时)
**Demo：**
功能|结果
------|------
页面顺序|A->B->C->D
历史记录|[A,B,C,D]
cursor|3，表示当前处于D页面

**解析：**
功能|结果
-----|------
go方法|
调用方法|go(C)
历史记录|[A,B,C,D]
cursor|2，表示当前处于C页面，复杂页面时D页面会销毁

功能|结果
-----|------
goBack方法| 
调用方法|goback(-1)
历史记录|[A,B,C,D]
cursor|2，表示当前处于C页面,复杂页面时D页面会销毁

**结论：**
当相邻的两个页面返回时:
如果返回的页面为一级页面：go和goBack无区别，都只改变cursor的值。
如果返回的页面为复杂页面：go和goBack无区别， cursor之后的页面都会销毁掉。

### 2. 跨页面数量为1个以上时（最少间隔一个）
**Demo:**
功能|结果
-----|------
页面顺序|A->B->C->D
历史记录|[A,B,C,D]
cursor|3，此时也为D

**解析：**
功能|结果
-----|------
go方法|
调用方法|go(B)
历史记录|[A,B,C,D,B]
cursor|4，此时页面为B

功能|结果
-----|------
goBack方法|
调用方法|goback(-2)
历史记录|[A,B,C,D]
cursor|1，页面同样为B

**附加情况：此时处于B页面，如果要跳转到G时，也就是B->G时**
第一种情况下：
功能|结果
-----|------
调用方法|go(G)
历史记录|[A,B,C,D,B,G]
cursor|5

第二种情况下：
功能|结果
-----|------
调用方法|go(G)
历史记录|[A,B,G]
cursor|2，页面C,D销毁掉了（可参考[页面何时销毁](https://www.zybuluo.com/huis/note/602072#ionichistory对象的histories堆栈中页面何时销毁)）

**结论：**
当跨页面返回时:
如果返回的页面为一级页面：go向堆栈的末尾添加视图（堆栈视图增加），goBack会更改cursor的值（堆栈视图不变）
如果返回的页面为复杂页面：go向堆栈的末尾添加视图（堆栈视图增加），goBack会更改cursor的值（cursor之后的视图会销毁）

## $ionicHistory对象的histories堆栈中页面何时销毁

### 情况1：  A,B,C,D页面均为一级页面，也就是都不包含有<ion-nav-view>标签的页面。

**Demo:**
功能|结果
-----|------
页面流程| A->B->C->D
历史记录|[A,B,C,D]
cursor|3（从0开始）

调用`goback(-3)`或调用`go`一级一级返回到A时,
功能|结果
-----|------
历史记录|[A,B,C,D]
cursor|0

**下面分三种情况：**
**情况1：**
功能|结果
-----|------
页面调用|A->B
历史记录|[A,B,C,D]不变
cursor|1，无页面销毁

**情况2：**
功能|结果
-----|------
页面调用|A->C（原本存在堆栈中的）
历史记录|[A,C]
cursor|1，原本的B,C,D页面销毁

**情况3：**
功能|结果
-----|------
页面调用|A->F（新的）
历史记录|[A,F]
cursor|1，(B,C,D)页面会销毁，和情况2一样

**结论：**
返回的页面为一级页面，这时cursor不是堆栈的末尾时：
如果这时跳转go到不是当前页面的下一个页面时，就会产生新的页面，且原cursor之后的页面都会销毁
如果这时跳转go到当前页面的下一个页面时，不产生新的页面，cursor增加1，堆栈视图信息不变

### 情况2： A页面为复杂页面，abstract为true,且内部包含有<ion-nav-view>标签， 而 B,C,D页面均为一级页面。

**Demo：**
功能|结果
-----|------
页面流程|A(A1)->B->C->D
历史记录|[A(A1),B,C,D]
cursor|3（从0开始），此时为D页面

调用`goback(-3)`或调用`go`一级一级返回到A(A1)时
功能|结果
-----|------
历史记录|[A(A1)]
cursor|0，页面B,C,D已经销毁掉了。
**注意：这里和情况1不一样**

**结论：**
返回的页面为复杂页面时，cursor之后的页面都会被销毁

### 情况3：state.go 和 goback 返回时传递参数的问题
调用方法：
```
#Method1
state.go(state,params)

#Method2
$ionicHistory.backView().stateParams = {card:true};
$ionicHistory.goBack();
```

**注意： 这时不管是跨页面数量为0还是跨页面数量为1个以上时（最少间隔一个）都会在堆栈中产生新的view，为什么？？？？？？？**

具体原因:由于`stateParams`，因为它的改变导致`$ionicHistory`产生的`card=null`的页面`stateID`和`card=true`的页面`stateID`不一致，所以会产生一个新的View加入堆栈中，不再是之前我们分析的`goBack`一定不会产生新的View的问题

功能|结果
-----|------
具体流程图| A[params:{test:null}] --> B --> C
历史记录|[A,B,C]
cursor|2

#### 情况1：`state.go(B,{test:1})`
功能|结果
-----|------
历史记录|[A,B,C,B]
cursor|3,这时会产生死循环问题

#### 情况2：`$ionicHistory.backView().stateParams ={card:true};$ionicHistory.goBack();`
功能|结果
-----|------
历史记录|[A,B,C,B]
cursor|3,这时会产生死循环问题

#### 情况3：`state.go(B)` 或 `$ionicHistory.goBack()`
功能|结果
-----|------
历史记录|[A,B,C]
cursor|1, 
**注意:C页面没有传递参数到B页面，不符合我们的要求**

解决方法，使用service：
service.data={test:1}, 这样才能完美解决该问题

**结论：**
情况1和情况2，都会出现死循环问题，所以带有参数的页面跳转，只能使用服务的形式去传递参数



