---
title: "Git使用笔记--显示修改文件列表"
date: 2016-11-21
tags: ["Git"]
draft: false
---

## 显示文件的关键选项

`--name-only` 这个只显示文件名
`--name-status` 这个显示文件名和修改状态（比如：M/A/D）
两个文件的使用方式相同，只是结果不同


## 使用log显示文件修改列表

### name-only
```
git log --name-only --oneline --no-merges
```
结果如下：
```
src/styles/print/_paperPrint.scss
036b73d 修复案例题批阅时自动定位到输入框 参考：YHJY-6000
src/views/common/kyQuestion.html
6d7ba60 添加试题时只能选择联动的父类和其子类，却不能选择单独的子类文件夹 参考:YHJY-5398
src/scripts/common/utils/angular-tree-control.js
c875b55 创建试卷，第二部分添加试题，然后将第二部分整体删除后，再添加第二部分，试题没清空 参考:YHJY-6079
src/scripts/paper/create/controller/paperCreateController.js
```

### name-status
```
git log --name-status --oneline --no-merges
```
结果：
```
9a28ce2 修复案例题转码问题
M       src/scripts/questions/controllers/caseQuestionController.js
5e7054b 修复试题转码失败不能继续更新试题问题,修 修复转码中不让播放视频问题 修复选项子题资源名称不提示
M       src/scripts/questions/controllers/askQuestionController.js
M       src/scripts/questions/controllers/caseQuestionController.js
M       src/scripts/questions/controllers/fillBlankQuestionController.js
M       src/scripts/questions/services/uploadImageService.js
M       src/views/questions/caseQuestion.html
f8dd5bc 改进打印试卷打印答案时的界面 参考：YHJY-6077
M       src/styles/print/_paperPrint.scss
```


## 使用diff显示文件列表

### name-status
```
git diff --name-status hash1 hash2
```
或
```
git diff --name-status branch1 branch2
```
结果如下：
```
M       src/scripts/questions/controllers/askQuestionController.js
M       src/scripts/questions/controllers/caseQuestionController.js
M       src/scripts/questions/controllers/fillBlankQuestionController.js
M       src/views/questions/caseQuestion.html
```

### name-only
```
git diff --name-only hash1 hash2
```
或
```
git diff --name-only branch1 branch2
```
使用`--name-only`结果如下：
```
gulp-tasks/fonts.js
gulp-tasks/images.js
gulp-tasks/inject.js
gulp-tasks/jshint.js
gulp-tasks/oss-upload.js
gulp-tasks/rev.js
gulp-tasks/script-business.js
gulp-tasks/script-vendor.js
gulp-tasks/scripts.js
gulp-tasks/server.js
gulp-tasks/style-vendor.js
gulp-tasks/styles.js
```

## 只显示commitid
```
git log --after '1 week ago' --oneline --pretty=format:'%h'
```
结果如下：
```
7e0ce0f
036b73d
6d7ba60
c875b55
80d9915
560f863
c631da1
ee13556
28769f0
```

## 使用python的实现
```
import commands

logCmd = "git log --after '1 week ago' --pretty=format:'%h'"
diffCmd_template = "git diff --name-only %s %s^"

log_status, log_put = commands.getstatusoutput(logCmd)
if len(log_put) > 0:
    hashIds = log_put.split("\n")
    diffCmd = diffCmd_template % (hashIds[0], hashIds[len(hashIds) - 1])
    diff_status, diff_put = commands.getstatusoutput(diffCmd)
    files = diff_put.split("\n")
    print files
```