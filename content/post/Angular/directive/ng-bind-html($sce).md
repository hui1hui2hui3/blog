---
title: "ng-bind-html & $sce学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
#ng-bind-html & $sce学习笔记

> **注意事项：** 用该指令bind内容时，必须依赖ngSanitize模块，同时调用$sce.trustAsHtml等方法，否则会出现 `Attempting to use an unsafe value in a safe context.` 异常

## ng-bind-html源码
**注意：** 可以发现内部是调用的`$sce.getTrustedHtml` 方法
```js
var ngBindHtmlDirective = ['$sce', function($sce) {
  return function(scope, element, attr) {
    element.addClass('ng-binding').data('$binding', attr.ngBindHtml);
    scope.$watch(attr.ngBindHtml, function ngBindHtmlWatchAction(value) {
      element.html($sce.getTrustedHtml(value) || '');
    });
  };
}];
```

## trustAs Vs getTrusted
理解不是透彻，先把想法写一下

- **trustAs** 参数是信任内容，调用该方法后会得到一个继承自TrustedValueHolderType对象的对象包裹住真正的内容，该对象须调用getTrusted才可得到包装的内容
- **getTrusted** 参数是包裹内容的trusted对象或者是普通内容，如果是trusted内容则不会做过多处理，如果是普通内容则按照一定的规则删除不信任的内容，该规则可自定义（详情参见[`$sceDelegateProvider`](https://docs.angularjs.org/api/ng/provider/$sceDelegateProvider))

## 官方代码Code
```html
<div ng-controller="AppController as myCtrl">
  <i ng-bind-html="myCtrl.explicitlyTrustedHtml" id="explicitlyTrustedHtml"></i><br><br>
  <b>User comments</b><br>
  By default, HTML that isn't explicitly trusted (e.g. Alice's comment) is sanitized when
  $sanitize is available.  If $sanitize isn't available, this results in an error instead of an
  exploit.
  <div class="well">
    <div ng-repeat="userComment in myCtrl.userComments">
      <b>{{userComment.name}}</b>:
      <span ng-bind-html="userComment.htmlComment" class="htmlComment"></span>
      <br>
    </div>
  </div>
</div>
```
```js
angular.module('mySceApp', ['ngSanitize'])
.controller('AppController', ['$http', '$templateCache', '$sce',
  function($http, $templateCache, $sce) {
    var self = this;
    $http.get("test_data.json", {cache: $templateCache}).success(function(userComments) {
      self.userComments = userComments;
    });
    self.explicitlyTrustedHtml = $sce.trustAsHtml(
        '<span onmouseover="this.textContent=&quot;Explicitly trusted HTML bypasses ' +
        'sanitization.&quot;">Hover over this text.</span>');
  }]);
```
```json
[
  { "name": "Alice",
    "htmlComment":
        "<span onmouseover='this.textContent=\"PWN3D!\"'>Is <i>anyone</i> reading this?</span>"
  },
  { "name": "Bob",
    "htmlComment": "<i>Yes!</i>  Am I the only other one?"
  }
]
```

## 完整代码例子
```html
<!DOCTYPE html>
<html ng-app="bindSceModule">

<head>
    <title>ng-bind-html--$sce</title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript" src="../bower_components/angular-sanitize/angular-sanitize.min.js"></script>
    <script type="text/javascript">
    angular.module('bindSceModule', ['ngSanitize'])
        .controller('SceController', ['$scope', '$sce', '$http', '$templateCache',
            function($scope, $sce, $http, $templateCache) {
                var self = this;
                $http.get('../data/test_sce.json').success(function(result) {
                    self.comments = result;
                });

                var html = '<span onmouseover="this.textContent=&quot;Explicitly trusted HTML bypasses ' +
                    'sanitization.&quot;">Hover over this text.</span>';

                self.html = html;
                self.trustedHtml = $sce.getTrustedHtml(html);
                self.explictTrustedHtml = $sce.trustAsHtml(html);
                self.myTrustedHtml = $sce.getTrustedHtml(self.explictTrustedHtml);
            }
        ])
        .directive('ngBindHtmlSafe', ['$compile', function($compile) {

            return {
                link: function(scope, element, attrs, controller) {
                    var compile = function(html) {
                        var htmlDom = $compile(html)(scope);
                        element.html('').append(htmlDom);
                    };

                    var htmlName = attrs.ngBindHtmlSafe;

                    scope.$watch(htmlName, function(newHtml) {
                        if (!newHtml) return;
                        compile(newHtml);
                    });
                }
            };
        }]);
    </script>
</head>

<body ng-controller="SceController as sceCtrl">
    <i ng-bind-html="sceCtrl.explictTrustedHtml"></i>
    <br>
    <i ng-bind-html="sceCtrl.trustedHtml"></i>
    <br>
    <i ng-bind-html="sceCtrl.html"></i>
    <br>
    <i ng-bind-html-safe="sceCtrl.myTrustedHtml"></i>
    <br>
    <div ng-repeat="comment in sceCtrl.comments">
        <b>{{comment.name}}</b>
        <span ng-bind-html="comment.htmlComment"></span>
        <br>
    </div>
</body>

</html>

```

------------
> Written with [StackEdit](https://stackedit.io/).