---
title: "Angular学习笔记--Oauth2"
date: 2019-10-16
tags: ["Angular","Oauth2"]
draft: false
---
 # Angular学习笔记--Oauth2

为了代码的安全性和易用性需要Oauth2认证，我们知道Oauth2的获取Access_Token必须是`application/x-www-form-urlencoded`格式的，而Angular的$http默认的处理方式是`application/json`,这就导致请求无法成功，那要如何处理才能成功呢？看下面代码：

## 方式1-自定义请求处理，转化JSON为key=value&key=value的形式：
```
$http({
    method: 'POST',
    url: url,
    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
    transformRequest: function(obj) {
        var str = [];
        for(var p in obj)
        str.push(encodeURIComponent(p) + "=" + encodeURIComponent(obj[p]));
        return str.join("&");
    },
    data: {username: $scope.userName, password: $scope.password}
}).success(function () {});
```

## 方式2-使用encodeURIComponent直接拼接
```
$http.post(loginUrl, "userName=" + encodeURIComponent(email) +
                     "&password=" + encodeURIComponent(password) +
                     "&grant_type=password"
).success(function (data) {
```

## 方式3-使用`$httpParamSerializerJQLike`或`$httpParamSerializer`
`$httpParamSerializerJQLike` - a serializer inspired by jQuery's .param() (recommended)
`$httpParamSerializer` - a serializer used by Angular itself for GET requests
```
$http({
  url: 'some/api/endpoint',
  method: 'POST',
  data: $httpParamSerializerJQLike($scope.appForm.data), // Make sure to inject the service you choose to the controller
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded' // Note the appropriate header
  }
}).success(function(response) { /* do something here */ });
```
### `$httpParamSerializerJQLike` 和 `$httpParamSerializer` 区别
**Array:**
```
{sites:['google', 'Facebook']} // Object with array property

sites[]=google&sites[]=facebook // Result with $httpParamSerializerJQLike

sites=google&sites=facebook // Result with $httpParamSerializer
```

**Object**
```
{address: {city: 'LA', country: 'USA'}} // Object with object property

address[city]=LA&address[country]=USA // Result with $httpParamSerializerJQLike

address={"city": "LA", country: "USA"} // Result with $httpParamSerializer
```

## 方式4-使用Jquery的param方法
```
$http({
    method: 'POST',
    url: url,
    data: $.param({fkey: "key"}),
    headers: {'Content-Type': 'application/x-www-form-urlencoded'}
})
```




