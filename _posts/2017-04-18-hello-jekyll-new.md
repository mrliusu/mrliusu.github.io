---
layout: post
title: 'gulp学习笔记'
date: 2018-02-08
author: mrliusu
cover: ''
tags: gulp 学习笔记
---

### 需要用到的基本插件有
- gulp
- gulp-sass （scss转成css）
- browser-sync （创建本地服务器，并自动刷新）
- gulp-useref （多个引用文件合并成一个）
- gulp-if （if判断）
- gulp-uglify （js压缩）
- gulp-minify-css （css压缩）
- gulp-htmlmin （html压缩）
- gulp-imagemin （图片压缩）
- gulp-cache （缓存，无修改的不压缩，节约时间）
1. 先安装
>  npm install gulp-sass --save-dev
2. 再引用
```js
var sass = require('gulp-sass')
```
3. 然后调用
```js
sass()
```
### 基本语法如下
```js
gulp.task('useref', function () {
    var htmlOption = {
        collapseWhitespace: true,
        minifyCSS: true,
        removeComments: true
    }
    return gulp.src('app/*.html')
        .pipe(useref())
        .pipe(gulpif('*.css', minifyCss()))
        .pipe(gulpif('*.js', uglify()))
        .pipe(gulpif('*.html', htmlmin(htmlOption)))
        .pipe(gulp.dest('dist'));
})
```
- task（新建） 
- src （读取资源）
- pipe （管道即要执行的操作）
- dest （输出）
最后在cmd执行
```
 gulp useref
```
### 基本参数
uglyfi与minifycss自动的去掉注释压缩代码，但是htmlmin需要做参数配置。
- **html的参数配置**
```js
gulp.task('htmlmin', function () {
    var options = {
        removeComments: true, /*清除HTML注释*/
        collapseWhitespace: true, /*压缩HTML*/
        collapseBooleanAttributes: true, /*省略布尔属性的值 <input checked="true"/>=><input />*/
        removeEmptyAttributes: true, /*删除所有空格作属性值 <input id="" /> ==> <input />*/
        removeScriptTypeAttributes: true, /*删除<script>的type="text/javascript"*/
        removeStyleLinkTypeAttributes: true, /*删除<style>和<link>的type="text/css"*/
        minifyJS: true, /*压缩页面的JS*/
        minifyCSS: true /*压缩页面写的CSS*/
    };
    gulp.src('src/html/*.html')
        .pipe(htmlmin(options))
        .pipe(gulp.dest('dist/html'));
});
```
### 踩过的坑
1. **安装gulp之后，执行`gulp -v`显示gulp不是内部或外部命令，有时候安装其他的插件也会遇到这个问题**
是缺少环境变量或者环境变量错误的原因.
cmd执行`npm config get prefix`拿到环境变量，然后 `计算机>属性>高级系统设置>高级>环境变量>编辑`，把拿到的环境变量添加或者修改进去，重启cmd再执行
2. **pipe方法有先后执行顺序**