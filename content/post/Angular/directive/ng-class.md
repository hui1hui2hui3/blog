---
title: "ng-class学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ng-class学习笔记
[TOC]

> 解析表达式expression的5种方式：
> > 1.  **String Syntax** 
> 如果expression是string，则用1个或多个空格风格class names（If the expression evaluates to a string, the string should be one or more space-delimited class names.）, 比如：ng-class="expression"
> 2. **Map Syntax**
> 如果expression是object, 则key-value形式的对应关系，如果value为true,则应用key作为class类名, 比如（{'red': isImportant,'bold':isBold}）
> 3. **Array Syntax**
> 如果expression是array, 则数组的元素可以是类型1的string或类型2的object, 可以混用, 比如：[expression1,expression2]
> 4.  **Expression Syntax**
> 方式： expression ? 'class1' : 'class2'
> 5. **Select Map Syntax**
> 方式比如： {true: 'has-error',false:'has-normal'}[isError], 注意这里的Map形式和第二种的Map相反, 另外需要注意的是isError必须===true 或者 false，不能是undefined | null，否则无法显示样式

**参考链接：[How to use ng-class](https://scotch.io/tutorials/the-many-ways-to-use-ngclass#ngclass-using-the-ternary-operator)**

## Animations
name | details
------|-------
add | happens just before the class is applied to the elements(ng-add)
remove | happens just before the class is removed from the element(ng-remove)

### 代码演示Animation（官方）
```html
<input id="setbtn" type="button" value="set" ng-click="myVar='my-class'">
<input id="clearbtn" type="button" value="clear" ng-click="myVar=''">
<br>
<span class="base-class" ng-class="myVar">Sample Text</span>
```
```css
.base-class {
  -webkit-transition:all cubic-bezier(0.250, 0.460, 0.450, 0.940) 0.5s;
  transition:all cubic-bezier(0.250, 0.460, 0.450, 0.940) 0.5s;
}

.base-class.my-class {
  color: red;
  font-size:3em;
}
```

## 代码演示（官方）
```html
<p ng-class="{strike: deleted, bold: important, 'has-error': error}">Map Syntax Example</p>
<label>
   <input type="checkbox" ng-model="deleted">
   deleted (apply "strike" class)
</label><br>
<label>
   <input type="checkbox" ng-model="important">
   important (apply "bold" class)
</label><br>
<label>
   <input type="checkbox" ng-model="error">
   error (apply "has-error" class)
</label>
<hr>
<p ng-class="style">Using String Syntax</p>
<input type="text" ng-model="style"
       placeholder="Type: bold strike red" aria-label="Type: bold strike red">
<hr>
<p ng-class="[style1, style2, style3]">Using Array Syntax</p>
<input ng-model="style1"
       placeholder="Type: bold, strike or red" aria-label="Type: bold, strike or red"><br>
<input ng-model="style2"
       placeholder="Type: bold, strike or red" aria-label="Type: bold, strike or red 2"><br>
<input ng-model="style3"
       placeholder="Type: bold, strike or red" aria-label="Type: bold, strike or red 3"><br>
<hr>
<p ng-class="[style4, {orange: warning}]">Using Array and Map Syntax</p>
<input ng-model="style4" placeholder="Type: bold, strike" aria-label="Type: bold, strike"><br>
<label><input type="checkbox" ng-model="warning"> warning (apply "orange" class)</label>
```
```css
.strike {
    text-decoration: line-through;
}
.bold {
    font-weight: bold;
}
.red {
    color: red;
}
.has-error {
    color: red;
    background-color: yellow;
}
.orange {
    color: orange;
}
```

# ng-class-even | ng-class-odd

> **注意事项:** 必须结合使用**ng-repeat**

## 代码演示（官方）
```html
<ol ng-init="names=['John', 'Mary', 'Cate', 'Suz']">
  <li ng-repeat="name in names">
   <span ng-class-odd="'odd'" ng-class-even="'even'">
     {{name}} &nbsp; &nbsp; &nbsp;
   </span>
  </li>
</ol>
```
```css
.odd {
  color: red;
}
.even {
  color: blue;
}
```

---------
> Written with [StackEdit](https://stackedit.io/).