---
title: "no-cloak学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
#no-cloak学习笔记
## 完整代码
```html
<!DOCTYPE html>
<html ng-app="cloakModule">

<head>
    <meta charset="utf-8">
    <title></title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('cloakModule', [])
        .controller('CloakController', ['$scope',
            function($scope) {
                var self = this;
                self.testCloak = "tedt";
            }
        ]);
    </script>
</head>

<body ng-controller="CloakController as cloakCtrl">
    no-cloak:<span>{{cloakCtrl.testCloak}}</span>
    <br>
    ng-cloak:<span ng-cloak>{{cloakCtrl.testCloak}}</span><br>
    <script type="text/javascript">
    for (var i = 0; i < 1000 * 100; i++) {
        document.write("-")
    }
    </script>
</body>

</html>

```

> Written with [StackEdit](https://stackedit.io/).