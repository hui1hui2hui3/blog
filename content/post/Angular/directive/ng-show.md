---
title: "ng-show学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---

# ng-show学习笔记

## Overriding .ng-hide
```css
.ng-hide:not(.ng-hide-animate) {
  /* this is just another form of hiding an element */
  display: block!important;
  position: absolute;
  top: -9999px;
  left: -9999px;
}
```

## Animations 完整例子

```html
<!DOCTYPE html>
<html ng-app="showModule">

<head>
    <title>ng-show</title>
    <style>
    .animate-show {
        line-height: 20px;
        opacity: 1;
        padding: 10px;
        border: 1px solid black;
        background: white;
    }
    
    .animate-show.ng-hide-add.ng-hide-add-active,
    .animate-show.ng-hide-remove.ng-hide-remove-active {
        -webkit-transition: all linear 0.5s;
        transition: all linear 0.5s;
    }
    
    .animate-show.ng-hide {
        line-height: 0;
        opacity: 0;
        padding: 0 10px;
    }
    
    .check-element {
        padding: 10px;
        border: 1px solid black;
        background: white;
    }
    </style>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript" src="../bower_components/angular-animate/angular-animate.js"></script>
    <script type="text/javascript">
    angular.module('showModule', ['ngAnimate']);
    </script>
</head>

<body>
    Click me:
    <input type="checkbox" ng-model="checked" aria-label="Toggle ngHide">
    <br/>
    <div>
        Show:
        <div class="check-element animate-show" ng-show="checked">
            I show up when your checkbox is checked.
        </div>
    </div>
    <div>
        Hide:
        <div class="check-element animate-show" ng-hide="checked">
            I hide when your checkbox is checked.
        </div>
    </div>
</body>

</html>

```


> Written with [StackEdit](https://stackedit.io/).