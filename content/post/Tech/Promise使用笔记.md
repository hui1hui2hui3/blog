---
title: "Promise使用笔记"
date: 2016-10-14
tags: ["NodeJs","Promise"]
draft: false
---

> 环境描述：nodejs
> Promise模块：[q](https://www.npmjs.com/package/q) 

## 1. 多层异步Promise的实现
**需求：获取一个目录下指定文件（多个）的数据集合**
从错误思路到正确思路进行分析, 解析错误思路是为了知己知彼，为了不在写出这种错误的代码，而且错误的思路中不是全部都错误的，是在某个分界点才开始错误的，所以这个*分界点*是个关键，需要多加注意
### 1.1 错误思路
先直接上完整代码
```
var jsonParser = function(basePath) {
    var deferred = q.defer();
    var resultData = [];
    try {
        fs.readdir(basePath, function(err, files) {
            if (err) {
                console.error(err);
                deferred.reject(err);
            }
            if (files.length < 1) {
                deferred.resolve(resultData);
            }
            for (var i = 0, n = files.length; i < n; i++) {
                var file = files[i];
                var fileExtIndex = file.indexOf(".json");
                if (fileExtIndex > -1) {
                    fs.readFile(path.join(basePath, file), function(err, data) {
                        if (err) {
                            console.error(err);
                            deferred.reject(err);
                        }
                        try {
                            var jsonData = JSON.parse(data);
                            resultData[resultData.length] = jsonData;
                            if (i >= n - 1) {
                                deferred.resolve(resultData);
                            }
                        } catch (e) {
                            console.log(e);
                            deferred.reject(e);
                        }
                    })
                }
            }
        });
    } catch (e) {
        deferred.reject(e)
    }
    return deferred.promise;
}
```
* 一步一步解析错误思路1：
定义一个promise对象返回（**正确**）
```
var deferred = q.defer();
return deferred.promse;
```
* 一步一步解析错误思路2：
`fs.readdir`是个异步的操作，同时可能会出现异常，所以使用`try...catch...`异常捕捉，失败则直接`reject`（**正确**）
```
try {
    fs.readdir(basePath, function(err, files) {})
} catch (e) {
    deferred.reject(e)
}
```
* 一步一步解析错误思路3：
由于读取文件夹会有错误返回，则直接`reject`，或者文件夹下为空，则`resolve`空数组（**正确**）
```
if (err) {
    console.error(err);
    deferred.reject(err);
}
if (files.length < 1) {
    deferred.resolve(resultData);
}
```
* 一步一步解析错误思路4：
这一步对文件列表进行遍历读取，咋一看是没问题的（**无**）
```
for (var i = 0, n = files.length; i < n; i++) {
    fs.readFile(path.join(basePath, file), function(err, data) {}
}
```
* 一步一步解析错误思路5：
读取文件时会有错误，所以直接`reject`，细想会发现，如果读取某个文件时错误会导致这个方法直接结束掉了，其他读取成功的文件也结束了，但是目前思路还有些不清，但就在这段区域来说是对的，在往下分析（**正确**）
```
if (err) {
    deferred.reject(err);
}
```
* 一步一步解析错误思路6：
对读取成功的数据进行解析，解析可能存在异常所以`reject`（**正确**）
```
try {
    var jsonData = JSON.parse(data);
    ...
} catch (e) {
    deferred.reject(e);
}
```
* 一步一步解析错误思路7：
判断是否是最后一个文件`resolve`最终的结果，明显有问题，这里是异步操作的结果中，也就是外围的`for`循环此时已经循环完毕，i一定是等于n了，这个判断毫无用处，对于多个文件的读取可能刚读完一个文件就结束了，另外，如果写成正常思路的 `i==n-1`反而会导致死循环，引起数据永远无法返回的问题（**错误**）
```
resultData[resultData.length] = jsonData;
if (i >= n - 1) { // i == n-1
    deferred.resolve(resultData);
}
```
下面是回过头去在想想不对劲的地方和代码根本没法写的地方

* 回头分析错误思路1：
我想要在读取文件时对文件类型进行筛选，会发现有的文件会进if，有的会进else，else中是`resolve`还是不呢，如果`resolve`那可能会导致第一个文件就不符直接结束了，如果没有`resolve`那假设没有文件是符合的，仍然会导致永远无法返回数据的问题，怎么写？
```
var fileExtIndex = file.indexOf(".json");
if (fileExtIndex > -1) {
    //readfile
} else {
    
}
```
* 回头分析错误思路2：
读取文件时可能会报错，那是不是应该`try...catch...`，那到底写不写`reject`呢？
```
fs.readFile(path.join(basePath, file), function(err, data) {}
```
* 回头分析错误思路3：
对文件循环进行读取，假设`for`循环结束后没有`resolve`或`reject`咋办？代码如何写？
```
for (var i = 0, n = files.length; i < n; i++) {
    fs.readFile(path.join(basePath, file), function(err, data) {}
}
```
**总结：通过上面的分析会发现根本的问题就是出现在循环处理`fs.readFile`这个异步方法,另外可能还有有其他想法就是，打算一个一个文件的进行读取，这个好知道啥时候读到最后的文件（可能可以实现，但是仍然不可取）**

### 1.2 正确思路
同理，先上完整代码
```
function readFiles(basePath, files) {
    var promises = [];
    for (var i = 0, n = files.length; i < n; i++) {
        var file = files[i];
        if (file.indexOf(".json") > -1) { //必须是.json的扩展文件
            promises.push(q.Promise(function(resolve, reject, notify) {
                fs.readFile(path.join(basePath, file), function(err, data) {
                    if (err) {
                        reject(err);
                    }
                    try {
                        resolve(JSON.parse(data));
                    } catch (e) {
                        reject(e);
                    }
                })
            }));
        }
    }
    return q.all(promises);
}

//获取目录下面所有文件，并解析为json格式的数组
var jsonParser = function(basePath) {
    var deferred = q.defer();
    try {
        fs.readdir(basePath, function(err, files) {
            if (err) {
                deferred.reject(err);
            }
            if (files.length < 1) {
                deferred.resolve([]);
            }
            readFiles(basePath, files).then(function(data) {
                deferred.resolve(data)
            }, function(error) {
                deferred.reject(data)
            })
        });
    } catch (e) {
        deferred.reject(e)
    }
    return deferred.promise;
}
```
会发现重大改变的地方就是我们分析的第4步开始改变的
```
readFiles(basePath, files).then(function(data) {
    deferred.resolve(data)
}, function(error) {
    deferred.reject(data)
})
```
这个会发现很爽，我不管结果如何，我就是监听then的结果，进行`resolve`或者`reject`
**核心代码：**
```
function readFiles(basePath, files) {
    var promises = [];
    for (var i = 0, n = files.length; i < n; i++) {
        var file = files[i];
        if (file.indexOf(".json") > -1) { //必须是.json的扩展文件
            promises.push(q.Promise(function(resolve, reject, notify) {
                fs.readFile(path.join(basePath, file), function(err, data) {
                    if (err) {
                        reject(err);
                    }
                    try {
                        resolve(JSON.parse(data));
                    } catch (e) {
                        reject(e);
                    }
                })
            }));
        }
    }
    return q.all(promises);
}
```
**分析：这不是单纯的进行的代码的封装，重点是返回的`q.all(promises)`，代码中把所有的`fs.readFile`异步处理封装成Promise对象，用`q.all()`等待机制，等待所有的文件读取好以后统一进行返回，会发现这个更加严谨，对个各个角度的都有响应的`resolve`或`reject`，而不会导致卡死问题**