---
title: "ng-options学习笔记"
date: 2016-05-26
tags: ["Angular1.4"]
draft: false
---
# ng-options学习笔记

> **重点1：** ng-model的比较的是引用，不是值，所以当比较内容是数组或对象就会容易出现不相等，导致选项中出现**空白选项**
> **重点2：** Do not use **select as** and **track by** in the same expression. They are not designed to work together.

## ngOptions

> - **array data sources**
> > 1. label **for** value **in** array
> > 2. select **as** label **for** value **in** array
> > 3. label **group by** group **for** value **in** array
> > 4. label **disable when** disable **for** value **in** array
> > 5. label **group by** group **for** value **in** array **track by** trackexpr
> > 6. label **disable when** disable **for** value **in** array **track by** trackexpr
> > 7. label **for** value **in** array | orderBy:orderexpr **track by** trackexpr (for including a filter with **track by**)

> - **object data sources**
> > 1. label **for** (key , value) **in** object
> > 2. select **as** label **for** (key , value) **in** object
> > 3. label **group by** group **for** (key, value) **in** object
> > 4. label **disable when** disable **for** (key, value) **in** object
> > 5. select **as** label **group by** group **for** (key, value) **in** object
> > 6. select **as** label **disable when** disable **for** (key, value) **in** object

> 单词解释
> > - **array / object**: an expression which evaluates to an array / object to iterate over.
> > - **value**: local variable which will refer to each item in the array or each property value of object during iteration.
> > - **key**: local variable which will refer to a property name in object during iteration.
> > - **label**: The result of this expression will be the label for `<option>` element. The expression will most likely refer to the value variable (e.g. value.propertyName).
> > - **select**: The result of this expression will be bound to the model of the parent `<select>` element. If not specified, select expression will default to value.
> > - **group**: The result of this expression will be used to group options using the `<optgroup>` DOM element.
> > - **disable**: The result of this expression will be used to disable the rendered `<option>` element. Return true to disable.
> > - **trackexpr**: Used when working with an array of objects. The result of this expression will be used to identify the objects in the array. The trackexpr will most likely refer to the value variable (e.g. value.propertyName). With this the selection is preserved even when the options are recreated (e.g. reloaded from the server).

## 完整例子

```html
<!DOCTYPE html>
<html ng-app="optionsModule">

<head>
    <title>ng-options</title>
    <script type="text/javascript" src="../bower_components/angular/angular.js"></script>
    <script type="text/javascript">
    angular.module('optionsModule', [])
        .controller('OptionsController', ['$scope', function($scope) {
            var optionsCtrl = this;

            optionsCtrl.items = [{
                id: 1,
                label: 'aLabel',
                subItem: {
                    name: 'aSubItem'
                }
            }, {
                id: 2,
                label: 'bLabel',
                subItem: {
                    name: 'bSubItem'
                }
            }];

            // optionsCtrl.selected = optionsCtrl.items[0].subItem;

            //由于ngModel默认比较的是引用，所以当比较的是数组或对象时，items中的值不相等，所以
            //会产生空白的选择。
            optionsCtrl.selected = {
                name: 'aSubItem'
            };

            optionsCtrl.colors = [{
                name: 'black',
                shade: 'dark'
            }, {
                name: 'white',
                shade: 'light',
                notAnOption: true
            }, {
                name: 'red',
                shade: 'dark'
            }, {
                name: 'blue',
                shade: 'dark',
                notAnOption: true
            }, {
                name: 'yellow',
                shade: 'light',
                notAnOption: false
            }];
            optionsCtrl.myColor = optionsCtrl.colors[2]; // red
        }]);
    </script>
</head>

<body ng-controller="OptionsController as optionsCtrl">
    <select ng-options="item.subItem as item.label for item in optionsCtrl.items track by item.id" ng-model="optionsCtrl.selected">
    </select>
    <ul>
        <li ng-repeat="color in optionsCtrl.colors">
            <label>Name:
                <input ng-model="color.name">
            </label>
            <label>
                <input type="checkbox" ng-model="color.notAnOption"> Disabled?</label>
            <button ng-click="optionsCtrl.colors.splice($index, 1)" aria-label="Remove">X</button>
        </li>
        <li>
            <button ng-click="optionsCtrl.colors.push({})">add</button>
        </li>
    </ul>
    <hr/>
    <label>Color (null not allowed):
        <select ng-model="optionsCtrl.myColor" ng-options="color.name for color in optionsCtrl.colors">
        </select>
    </label>
    <br/>
    <label>Color (null allowed):
        <span class="nullable">
    <select ng-model="optionsCtrl.myColor" ng-options="color.name for color in optionsCtrl.colors">
      <option value="">-- choose color --</option>
    </select>
  </span></label>
    <br/>
    <label>Color grouped by shade:
        <select ng-model="optionsCtrl.myColor" ng-options="color.name group by color.shade for color in optionsCtrl.colors">
        </select>
    </label>
    <br/>
    <label>Color grouped by shade, with some disabled:
        <select ng-model="optionsCtrl.myColor" ng-options="color.name group by color.shade disable when color.notAnOption for color in optionsCtrl.colors">
        </select>
    </label>
    <br/> Select
    <button ng-click="optionsCtrl.myColor = { name:'not in list', shade: 'other' }">bogus</button>.
    <br/>
    <hr/>
	<!-- 这里注意写法 -->
    Currently selected: {{ {selected_color:optionsCtrl.myColor} }}<hr/>
    Currently selected: {"selected_color":{{optionsCtrl.myColor}} }
    <!-- 这里注意写法 -->
    <div style="border:solid 1px black; height:20px" ng-style="{'background-color':optionsCtrl.myColor.name}">
    </div>
</body>

</html>

```

> Written with [StackEdit](https://stackedit.io/).