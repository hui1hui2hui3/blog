---
title: "Angular animation学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---

[TOC]
# Angular animation 总结
> [参考文章](http://www.cnblogs.com/wushangjue/p/4529063.html)

AngularJS中实现动画效果有两大种方式

 - 基于CSS的动画效果
   - CSS Transition Animation
   - CSS Class-based Animation
 - 基于Javascript的动画效果

官方给出的能支持动画效果的Directives：
Directive | Supported Animations
---------|-------------
ngRepeat	| enter, leave and move
ngView	 | enter and leave
ngInclude	 | enter and leave
ngSwitch	| enter and leave
ngIf	| enter and leave
ngClass	| add and remove (the CSS class(es) present)
ngShow&ngHide	| add and remove (the ng-hide class value)
form&ngModel	| add and remove (dirty, pristine, valid, invalid & all other validations)
ngMessages	| add and remove (ng-active & ng-inactive)
ngMessage	| enter and leave

## 基于CSS的动画效果
### 1. CSS Transition Animation
示例（官方DEMO）：
```html
<!DOCTYPE>
<html>
<head>
    <style type="text/css">
        /* 开始时的样式 */
        .fade.ng-enter {
            transition: 5s linear all; /* 当使用css transition实现动画效果时，在开始时的样式中必须包含transition的设置 */
            opacity: 0;
        }

            /* 结束时的样式 */
            .fade.ng-enter.ng-enter-active {
                opacity: 1;
            }
    </style>

    <script src="/Scripts/angular.js"></script>
    <script src="/Scripts/angular-animate.js"></script>
    <script type="text/javascript">
        (function () {
            var app = angular.module('cssBasedAnimationTest', ['ngAnimate']);
        })();
    </script>
</head>
<body ng-app="cssBasedAnimationTest">
    <div ng-if="bool" class="fade">
        Fade me in out
    </div>
    <button ng-click="bool=true">Fade In!</button>
    <button ng-click="bool=false">Fade Out!</button>
</body>
</html>
```
> 使用CSS Transition时，ng-EVENT（动画开始前的样式）和ng-EVENT-active（动画执行完毕后的样式）这两组样式必须同时出现，且在ng-EVENT中必须包含transition的设置。
> **注意：这种写法需要使用ng-Event和ng-Event-active**

另一种CSS transition的写法是使用CSS的Keyframe关键字
```html
<style type="text/css">
    /* 开始时的样式，使用keyframes不需要定义结束时的样式 */
    .fade.ng-enter {
        animation: my_fade_animation 0.5s linear;
        -webkit-animation: my_fade_animation 0.5s linear;
    }

    @keyframes my_fade_animation {
        from {
            opacity: 0;
        }

        to {
            opacity: 1;
        }
    }

    @-webkit-keyframes my_fade_animation {
        from {
            opacity: 0;
        }

        to {
            opacity: 1;
        }
    }
</style>
```
> **注意：这种写法不需要ng-EVENT-active**

### 2. CSS Class-based Animation
Class-based Animation即为通过ngClass、ngShow、ngHide等Directives执行动画效果。  
示例2：
```html
<!DOCTYPE>
<html>
<head>
    <style type="text/css">
        .fade.ng-hide {
            transition: 3s linear all;
            opacity: 0;
        }

        .fade.ng-show {
            transition: 3s linear all;
            opacity: 1;
        }
    </style>

    <script src="/Scripts/angular.js"></script>
    <script src="/Scripts/angular-animate.js"></script>
    <script type="text/javascript">
        (function () {
            var app = angular.module('cssClassBasedAnimationTest', ['ngAnimate']);
        })();
    </script>
</head>
<body ng-app="cssClassBasedAnimationTest" ng-init="bool=true">
    <div ng-show="bool" class="fade">
        This is ng-show.
    </div>
    <div ng-hide="bool" class="fade">
        This is ng-hide.
    </div>
    <button ng-click="bool=!bool">Toggle</button>
</body>
</html>
```
实际观察Html的变化，无论是ngShow还是ngHide，其实都是在隐藏元素时，默认添加ng-hide-animate、ng-hide-add、ng-hide-add-active样式。也就是针对像ngHide、ngShow等这些可以感知动画的Directives，由AngularJS的ngAnimate模块自动添加了CSS Transition动画。

ngAminate能检测的行为是样式的add或者remove， 那如何显式的指定add和remove的样式呢？

示例3：
```html
<!DOCTYPE>
<html>
<head>
    <style type="text/css">
        .highlight {
            transition: 3s linear all;
        }

            .highlight.on-add {
                background: white;
            }

            .highlight.on {
                background: yellow;
            }

            .highlight.on-remove {
                background: black;
            }
    </style>

    <script src="/Scripts/angular.js"></script>
    <script src="/Scripts/angular-animate.js"></script>
    <script type="text/javascript">
        (function () {
            var app = angular.module('cssClassBasedAnimationTest', ['ngAnimate']);
        })();
    </script>
</head>
<body ng-app="cssClassBasedAnimationTest" ng-init="bool=true">
    <div ng-class="{on:onOff}" class="highlight">
        Highlight this box
    </div>
    <button ng-click="onOff=!onOff">Toggle</button>
</body>
</html>
```
[Live Demo](http://codepen.io/anon/pen/QjwXLp)
我们让ng-class随着点击Toggle按钮变化，当onOff=true时样式on会被ngAnimate执行on-add的过程，反之则执行on-remove的过程。由于显式指定了样式，当我们运行示例3时，这个过程就一目了然了。

## 基于Javascript的动画效果
使用基于Javascript的动画效果可以让你在脚本中使用其他的Service甚至引用第三方的脚本进行动画的制作，使动画效果更丰富多变。

与基于CSS的动画效果相似，基于Javascript的动画效果也会由AngularJS自动添加一些指定的样式到元素上，但基于Javascript的动画效果还需要使用module.animation()添加动画脚本。

示例4：
```html
<!DOCTYPE>
<html>
<head>
    <script src="/Scripts/angular.js"></script>
    <script src="/Scripts/angular-animate.js"></script>
    <script src="/Scripts/jquery-1.9.1.js"></script>
    <script type="text/javascript">
        (function () {
            var app = angular.module('javascriptBasedAnimationTest', ['ngAnimate']);

            app.animation('.slide', [function () {
                return {
                    enter: function (element, doneFn) {
                        jQuery(element).fadeIn(1000, doneFn);
                    },

                    move: function (element, doneFn) {
                        jQuery(element).fadeIn(1000, doneFn);
                    },

                    leave: function (element, doneFn) {
                        jQuery(element).fadeOut(1000, doneFn);
                    }
                }
            }]);

            app.controller('myController', ['$scope', function ($scope) {
                $scope.students = ["Tom","Jack","Alice","May","Thomas"];
            }]);
        })();
    </script>
</head>
<body ng-app="javascriptBasedAnimationTest" ng-controller="myController">
    <div ng-if="isshow" ng-repeat="stu in students" class="slide">
        {{ stu }}
    </div>
    <input type="button" value="Toggle" ng-click="isshow=!isshow" />
</body>
</html>
```
> Written with [StackEdit](https://stackedit.io/).