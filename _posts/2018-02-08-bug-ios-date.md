---
layout: post
title:  "ios日期"
date:   2018-02-08 18:00:00
categories: ios bug
permalink: /archivers/bug-ios-date
---

在开发中遇到几次ios的`new Date(str)`拿不到日期，主要跟str有关
```
var arr = ['2018-02-08', '2018/02/08', '2018-2-8', '2018/2/8'];
for(let i = 0; i < arr.length; i ++) {
     alert(`${arr[i]}:${new Date(arr[i])}`);
}
```
经测试，在安卓跟pc下都能拿到，ios下只有2018-2-8返回`invalid Date`
固建议开发中使用 ‘/’ 做分隔符