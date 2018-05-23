---
layout: post
title:  "gulp + browserify 搭建es6环境"
date:   2018-05-23 10:30:00
categories: gulp 学习笔记
permalink: /archivers/gulp_browserify
---

### 安装gulp

```
npm install -g gulp
npm install --save-dev gulp
```

题外话：
--save-dev安装到开发环境，对应package.json里面的devDependencies；比如gulp或者browserify等打包及转化插件
--save安装到生产环境，对应package.json里面的，比如vue等包

### 安装gulp-connect（搭建本地服务器）

```
npm install --save-dev gulp-connect
```

然后新建gulpfile.js

```
var gulp = require('gulp'),
    connect = require('gulp-connect');

gulp.task('connect', function () {
    connect.server({
        livereload: true
    })
});

gulp.task('default', ['connect'])
```

在命令行执行gulp会执行default任务开启服务器 
### 监听代码修改

```
var gulp = require('gulp'),
    connect = require('gulp-connect');

gulp.task('connect', function () {
    connect.server({
        livereload: true
    })
});
gulp.task('sync', function () {
    gulp.src('app/index.html')   
        .pipe(connect.reload());
})

gulp.task('default', ['connect'], function () {
    gulp.watch('app/*.html', ['sync'])
})
```
gulp的插件必须在pipe里面调用，若以上代码改成下面那样，则报错
```
...
gulp.task('sync', function () {
    connect.reload()
})
...
```
基本语法为
```
gulp.task('sync', function () {
    gulp.src('app/index.html')  // src文件进入 
        .pipe(connect.reload())  // 传递给插件，可链式调用
        .pipe(gulp.dest('app/dest/index.html')) //输出
})
```
### gulp-babel转换es6
安装gulp-babel跟babel-preset-es2015
```
npm install --save-dev babel-cli
npm install --save-dev gulp-babel
npm install --save-dev babel-preset-es2015
```
然后gulpfile.js配置
```
...
gulp.task('js', function () {
    gulp.src('app/src/js/*.js')
        .pipe(babel({
            presets:['es2015']
        }))
        .pipe(gulp.dest('app/dest/js'))
        .pipe(connect.reload());
})
...
```
测试一下es6的api
转换前代码
```
const name = 'gulp & babel'
console.log(`Hello ${name}.`)
console.log(new Array(3).fill('su'))
```
转换之后代码
```
'use strict';

var name = 'gulp & babel';
console.log('Hello ' + name + '.');
console.log(new Array(3).fill('su'));
```
发现并没有转换，然后引入了babel-polyfill,发现把import 转换成了require然后浏览器报错require未定义
###使用Browserify解决模块化
```
npm install  browserify --save-dev
npm install vinyl-source-stream --save-dev
npm install vinyl-buffer --save-dev
```
gulpfile.js代码如下
```
...
var browserify = require('browserify'),  // 插件，实际是node系
    // 转成stream流，gulp系
    stream = require('vinyl-source-stream'),
    // 转成二进制流，gulp系
    buffer = require('vinyl-buffer');
gulp.task('browserify', function () {
    // 定义入口文件
    return browserify({
        entries: 'app/dest/js/index.js',
        debug: true
    })
    // 转成node readabel stream流，拥有pipe方法（stream流分小片段传输）
    .bundle()
    .on('error', function (error) {
        console.log(error.toString())
    })
    // 转成gulp系的stream流，node系只有content，添加名字
    .pipe(stream('index.js'))
    // 转成二进制的流（二进制方式整体传输）
    .pipe(buffer())
    // 输出
    .pipe(gulp.dest('app/dest/js/'))
})
...
```
browserify只做模块化处理，不能直接转es6的代码，会报错
或者可以做以下处理
安装babelify配合browserify使用
```
npm install babelify --save-dev
```
在package.json文件中添加以下代码,表示在browserify的transform的时候使用插件babelify,规则使用es2015（使用react则转换react的代码）
```
"browserify": {
    "transform": [
      [
        "babelify",
        {
          "presets": [
            "es2015"
          ]
        }
      ]
    ]
  }
```
或者也可以在gulpfile.js中改成一下代码
```
gulp.task('browserify', function () {
    // 定义入口文件
    return browserify({
        // 入口必须是转换过的es6文件，且文件不能是es6经过转换的es5文件，否者会报错
        entries: 'app/src/js/index.js',
        debug: true
    })
    // 在bundle之前先转换es6，因为readabel stream 流没有transform方法
    .transform("babelify", {presets: ['es2015']})
    // 转成stream流（stream流分小片段传输）
    .bundle()
    .on('error', function (error) {
        console.log(error.toString())
    })
    // node系只有content，添加名字转成gulp系可操作的流
    .pipe(stream('index.js'))
    // 转成二进制的流（二进制方式整体传输）
    .pipe(buffer())
    // 输出
    .pipe(gulp.dest('app/dest/js/'))
})
```
**有个坑注意下：**
同目录下的文件，以下引入方式会报```Error: Cannot find module 'greet' from 'C:\wamp64\www\gulpbabel\app\src\js'```找不到相应模块
```
import _greet from 'greet'
```
需要这样引入
```
import _greet from './greet'
```
关于stream跟buffer的注释参考https://segmentfault.com/a/1190000003770541

### 转换sass
```
npm install --save-dev gulp-sass
```
gulpfile.js文件
```

...
gulp.task('sass', function () {
    gulp.src('app/src/scss/*.scss')
    .pipe(sass())
    .pipe(gulp.dest('app/dest/css'))
})
...
```
### 压缩css
因为gulp-minify-css被废弃了，建议使用gulp-clean-css
安装
```
npm install gulp-clean-css --save-dev
```
然后gulpfile.js如下
```
...
gulp.task('sass', function () {
    gulp.src('app/src/scss/*.scss')
    .pipe(sass())
    // 转换之后压缩css；gulp支持链式语法
    .pipe(cleanCss())
    .pipe(gulp.dest('app/dest/css'))
    .pipe(connect.reload());
})
...
```
### 压缩js
使用gulp-uglify
与压缩css同理，安装gulp-uglify，在js转换之后，输出之前进行压缩
### 压缩html
安装gulp-htmlmin
注意：直接运行htmlmin()不会有效果，需要传递配置的参数
比如：
```
...
.pipe(htmlmin({
    collapseWhitespace: true //是否压缩html
}))
...
```
常用参数有
```
collapseWhitespace: true // 是否压缩html
removeComments: true  // 清除HTML注释
removeEmptyAttributes: true  // 删除所有空格作属性值 <input id="" /> ==> <input />
minifyJS: true // 压缩页面的JS
minifyCSS: true // 压缩页面写的CSS
```

其他参数参考https://github.com/kangax/html-minifier#user-content-options-quick-reference
### 合并js与css
**安装gulp-concat**
```
npm install gulp-concat -D
```
然后在gulpfile.js修改
```
gulp.task('concat', function () {
    gulp.src(['app/src/js/1.js', 'app/src/js/2.js']) // 打开两个文件
        .pipe(concat('12.js')) // 合并成12.js
        .pipe(gulp.dest('app/dest/js')) //输出到文件夹目录
})
```
将代码合并到12.js中；html中src需是12.js
*但是这有一个问题，就是如果每次修改html中引入的js文件，就需要再手动修改一次合并的gulpfile.js；并且在html中的script标签中的src地址必须是合并后的js文件文件地址，耦合比较高*
看看另一个插件gulp-useref
### 安装gulp-useref
```
npm i gulp-useref -D
```
```
// html代码
<body>
    <h2>gulp + babel</h2>
    <p class="lab">lab</p>
    <!-- build:js js/123.js -->
    <script src="js/1.js"></script>
    <script src="js/2.js"></script>
    <script src="js/3.js"></script>
    <!-- endbuild -->
    <script src="js/index.js"></script>
</body>
```
将代码合并到js/123.js文件，同时修改html代码为
```
<body>
    <h2>gulp + babel</h2>
    <p class="lab">lab</p>
    <script src="js/123.js"></script>
    <script src="js/index.js"></script>
</body>
```
**仅仅只是修改html的src跟合并代码而已，并不做检测**
比如：
```
<!-- build:css css/all.css -->
    <link rel="stylesheet" href="scss/index.scss">
    <link rel="stylesheet" href="scss/lab.scss">
<!-- endbuild -->
// 仅仅修改html的代码为
<link rel="stylesheet" href="css/all.css">
// 同时合并两个scss文件到all.css 
```

同时需要搭配gulp-if使用
因为useref只做合并，不做转化跟压缩；配合gulp-if在合并前做转化操作
### 安装gulp-if
```
npm install gulp-if --save-dev
```
然后gulpfile.js修改为
```
...
gulp.task('useref', function () {
    gulp.src('app/src/*.html')
        .pipe(useref())
        .pipe(gulpif('*.js', uglify())) // 如果要build.js文件，就先执行uglify()
        .pipe(gulpif('*.css', sass())) // 如果是要build:css文件，就先执行sass()
        .pipe(gulp.dest('app/dest/'))
})
...
```

