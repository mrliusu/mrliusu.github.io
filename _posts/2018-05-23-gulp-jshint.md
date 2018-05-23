---
layout: post
title:  "gulp-jshint 检测js代码"
date:   2018-05-23 15:30:00
categories: gulp 学习笔记
permalink: /archivers/gulp_jshint
---

### 安装 jshint和gulp-jshint

注意gulp-jshint需要依赖jshint

```
npm install jshint gulp-jshint -D
```
然后在gulpfile.js中修改

```
gulp.task('jshint', function () {
    gulp.src('app/src/js/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'))  // 输出结果，可以与其他的插件配合使用，可以更加好看的（比如颜色等）显示出错误信息
})
```
然后执行```gulp jshint```命令
可以看到结果

```
PS C:\wamp64\www\gulpbabel> gulp jshint
[11:56:42] Using gulpfile C:\wamp64\www\gulpbabel\gulpfile.js
[11:56:42] Starting 'jshint'...
[11:56:42] Finished 'jshint' after 8.86 ms
app\src\js\1.js: line 1, col 1, 'let' is available in ES6 (use 'esversion: 6') or Mozilla JS extensions (use moz).
app\src\js\1.js: line 2, col 18, Missing semicolon.

2 errors
app\src\js\2.js: line 1, col 18, Missing semicolon.

1 error
app\src\js\3.js: line 1, col 18, Missing semicolon.

1 error
app\src\js\greet.js: line 4, col 31, Missing semicolon.
app\src\js\greet.js: line 6, col 2, Missing semicolon.

2 errors
app\src\js\index.js: line 1, col 1, 'import' is only available in ES6 (use 'esversion: 6').
app\src\js\index.js: line 1, col 29, Missing semicolon.
app\src\js\index.js: line 2, col 1, 'import' is only available in ES6 (use 'esversion: 6').
app\src\js\index.js: line 2, col 24, Missing semicolon.
app\src\js\index.js: line 3, col 1, 'const' is available in ES6 (use 'esversion: 6') or Mozilla JS extensions (use moz).
app\src\js\index.js: line 3, col 28, Missing semicolon.
app\src\js\index.js: line 4, col 13, 'template literal syntax' is only available in ES6 (use 'esversion: 6').
app\src\js\index.js: line 4, col 30, Missing semicolon.
app\src\js\index.js: line 5, col 18, Missing semicolon.
app\src\js\index.js: line 6, col 37, Missing semicolon.

10 errors
PS C:\wamp64\www\gulpbabel>
```

可以看到提示错误说let属于es6的语法以及缺少分号，未解决这个问题，可以配置.jshintrc文件；或者在package.json文件中添加, 表示es的版本是6,可以解决es6的问题

```
{
  "jshintConfig" : {
    "esversion": 6
  }
}
```

或者新建.jshintrc

```
{
    "esversion": 6,  //可以有es6语法
    "undef": true, // 使用之前必须定义
    "unused": true, // 定义了必须使用
    "devel": true, // 可以使用console, alert；不配置为true是会报console未定义
    "globals": { // 配置全局变量，直接使用不会报错
        "module": true //不配置module为true时，在 module.exports = {} 时报module未定义
    }
}
```
jshint只检测代码不检测逻辑
具体参数参考http://jshint.com/docs/options/
