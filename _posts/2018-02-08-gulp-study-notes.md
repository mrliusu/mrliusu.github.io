---
layout: post
title:  "gulp学习笔记"
date:   2018-02-08 16:00:00
categories: gulp 学习笔记
permalink: /archivers/gulp-study-notes
---

### 需要用到的基本插件有
```
# 基本的
npm install gulp --save-dev
# sass转为css
npm install gulp-sass --save-dev
# 本地服务器
npm install browser-sync --save-dev
# 多个引用文件合成一个
npm install gulp-useref --save-ref
# 判断
npm install gulp-if --save-dev
# js压缩
npm install gulp-uglify --save-dev
# css压缩
npm install gulp-minify-css --save-dev
# html压缩
npm install gulp-htmlmin --save-dev
# 图片压缩
npm install imagemin --save-dev
# 缓存，无修改的图片不压缩，节约时间
npm install gulp-cache --save
# 删除多余文件
npm install del --save-dev
# 任务队列执行
npm install run-sequence --save-dev

```
### 步骤
1. 先安装

``` 
 npm install gulp-sass --save-dev
 ```
2. 再引用

```
var sass = require('gulp-sass')
```
3. 然后调用

```
 sass()
```

### 基本语法如下

```
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
**useref使用**
```
<!-- build:文件类型 合并之后的目录文件 -->
```
```html
<!-- build:js js/main.min.js -->
<script src="js/lib/a.js"></script>
<script src="js/lib/b.js"></script>
<script src="js/main.js"></script>
<!-- endbuild -->
```
useref()之后，a.js、b.js、main.js合并成js目录下的main.min.js

**uglyfi与minifycss自动的去掉注释压缩代码，但是htmlmin需要做参数配置**
html的参数配置
```javascript
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

### 代码实例
**执行default任务只是创建本地服务器，并监听实时变化，执行build任务是才合并压缩代码**
```javascript
var gulp = require('gulp');
var sass = require('gulp-sass');
var browserSync = require('browser-sync');
var useref = require('gulp-useref');
var uglify = require('gulp-uglify');
var gulpif = require('gulp-if');
var minifyCss = require('gulp-minify-css');
var imagemin = require('gulp-imagemin');
var cache = require('gulp-cache');
var htmlmin = require('gulp-htmlmin');
var del = require('del');
var runSequence = require('run-sequence');

/* 本地服务器 */
gulp.task('browserSync', function () {
    browserSync({
        server: {
            /* 根目录设置为app目录 */
            baseDir: 'app'
        }
    })
})

/* sass转为css */
gulp.task('sass', function () {
    return gulp.src('app/scss/*.scss')
        .pipe(sass())
        .pipe(gulp.dest('app/css'))
        .pipe(browserSync.reload({
            stream: true
        }))
})

/* 压缩合并 */
gulp.task('useref', function () {
    /* 压缩html的参数选项 */
    var htmlOption = {
        /* 压缩js */
        minifyJS: true,
        /* 压缩css */
        minifyCSS: true,
        /* 压缩html */
        collapseWhitespace: true,
        /* 清除注释 */
        removeComments: true
    }
    return gulp.src('app/*.html')
        /* 合并引用的文件 */
        .pipe(useref())
        /* 压缩css */
        .pipe(gulpif('*.css', minifyCSS()))
        /* 压缩js */
        .pipe(gulpif('*.js', uglify()))
        /* 压缩html */
        .pipe(gulpif('*.html', htmlmin(htmlOption)))
        .pipe(gulp.dest(dist))
})

/* 压缩图片 */
gulp.task('imagemin', function () {
    return gulp.src('app/image/*.+(jpg|png|svg|gif)')
    .pipe(cache(imagemin({
        interlaced: true
    })))
    .pipe(gulp.dest('dist/image'))
})

/* 删除文件 */
gulp.task('clean:dist', function () {
    return del.sync(['dist/**/*', '!dist/image', '!dist/image/**/*']);
})

/* 默认事件 */
gulp.task('default', function (callback) {
    runSequence(['sass', 'browserSync'], 'watch', callback)
})

/* 监听 */
gulp.task('watch', function () {
    /* scss文件有变化，执行sass任务转化及刷新 */
    gulp.watch('app/scss/*.scss', ['sass']);
    /* 任何一个文件有变化， 刷新浏览器 */
    gulp.watch('app/*.html', browserSync.reload);
    gulp.watch('app/js/*.js', browserSync.reload);
})

/* 到处任务 */
gulp.task('build', function (callback) {
    runSequence(
        /* 清除dist文件夹 */
        'clean:dist',
        /* 转换css */
        'sass',
        /* 合并压缩 */
        ['useref', 'imagemin'],
        callback
    )
})
```



