---
layout: post
title:  "sublime text 3 + jshint检查代码"
date:   2018-05-23 15:30:00
categories: sublimetext jslint 学习笔记
permalink: /archivers/sublimetext3_jshint
---

很多时候代码不用gulp搭建环境，这个时候如果想要检查代码，就需要再编辑器里面添加检测了。而且在编辑器里面检测就不需要每个项目都去安装了。
### 安装sublimelinter
sublime Text安装sublimelinter跟sublimelinter-jshint插件。具体步骤 
1. ``` ctrl + shif + p ``` 或者选择菜单栏的 ``` Preferences=> package control``` 打开插件面板 
2. 搜索 ``` install package ``` 安装插件
3. 搜索 ``` sublimelinter ``` 插件安装
 查看安装列表
1. ``` ctrl + shif + p ``` 或者选择菜单栏的 ``` Preferences=> package control``` 打开插件面板 
2. 搜索 ``` list package ``` 列出已安装插件
### 安装sublimelinter-jshint
与以上步骤相同

**我之前安装的汉化版发现package control的时候提示```There are no packages available for installation ``` 然后我卸载之后去官网下载一个安装么就好了**

### 安装jshint

因为sublimelinter-jshint实际是调用nodejs的jshint的接口检查代码的，所以需要安装jshint
```
npm i jshint -g
```
然后重启sublime text可以看到会有错误提示

![image.png](https://upload-images.jianshu.io/upload_images/2406529-7e4898ca0b419ef7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

黄色是警告，红色是错误
鼠标放到错误位置，会有错误提示

![image.png](https://upload-images.jianshu.io/upload_images/2406529-3f146ca7f6a43d75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 自定义jshint配置
可以根据需要进行自定义的配置
``` preferences => package Setting => sublimeLinter => setting ``` 会打开两个页面，左边的是默认的不可修改，右边是用户自定义的可修改；如下图

![image.png](https://upload-images.jianshu.io/upload_images/2406529-b420aaf8bc5b39a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
设置user setting代码

```
// SublimeLinter Settings - User
{
    "linters": {
        "jshint": {
            "@disable": true,
            "args": [
                "--config",
                "D:\\Program Files\\Sublime Text 3\\.jshintrc" // 这个是配置的.jshintrc文件
            ],
            "excludes": []
        }
    },
    "lint_mode": "save only"  // lint_mode设为save only只有在保存的时候才检查代码错误
}
```

注意：sublimelinter是按照文件类型检测的，jshint只能检测.js文件，就是说内嵌在html里面的js将不会被检测到
