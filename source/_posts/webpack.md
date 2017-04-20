---
title: webpack入门
date: 2016-10-22 03:07:07
tags: [webpack]
---
### Webpack
Webpack 是模块加载器以及打包工具，能把各种资源作为模块来使用和处理
工作方式为:把项目当做一个整体，通过一个给定的主文件，Webpack将从这个文件开始找到项目的所有依赖文件，使用loaders处理它们，最后打包成一个浏览器可识别的JavaScript文件。<!--more-->
### 使用webpack
* 安装
```javascript
//全局安装
npm install -g webpack
//安装到项目目录 文件夹中应有package.json (npm init)
npm install --save-dev webpack  //写入到package.json中的devdependencies
```
* 使用
在项目目录里 创建两个文件夹分别为src 和 dest
```javascript	
mkdir src dest 
cd src
vim main.js 
cd dest
vim index.html
cd ..
vim webpack.config.js //webpack配置文件
//main.js
var $=require('jquery')
$('body').html("yeah!")
//index.html
<!DOCTYPE html>
<html>
<body>
</script src="bundle.js"> //顺带一提 script放在此处的原因是 浏览器遇到body呈现内容，此时html 以及css 已经加载完毕
</body>
</html>
//webpack.config.js
module.exports={
    entry:__dirname+"/src/main.js", //webpack入口文件
    output:{
        path:__dirname+"/dest", 
        filename:"bundle.js" //输出文件
    }
}
//执行代码
1.webpack
2.打开index.html
```
* Loaders
通过使用不同的loader，webpack调用外部的脚本或工具对各种各样的文件进行处理并把它转换为浏览器可以识别的js文件。
1. Loaders的配置 在modules关键字下配置
2. -test:loaders处理的文件的拓展名的正则表达式
3. -loader:loader的名称
4. -include/exclude:手动添加必须处理的文件或屏蔽不需要处理的文件
5. -query:为loaders提供额外的设置选项
```javascript
//webpack.config.js
module.exports={
    entry:__dirname+"/src/main.js", //webpack入口文件
    output:{
        path:__dirname+"/dest", 
        filename:"bundle.js" //输出文件
    },
    module:{
        loaders:[
            {test:/\.json$/,loader:"json"} 
        ]
    }
}
```
### webpack dev server
WEBPACK DEV SERVER 是一个轻量的EXPRESS服务器，使用WEBPACK-DEV-MIDDLEWARE服务WEBPACK BUNDLE
```javascript
var path = require('path')
module.exports={
    entry:[
       webpack-dev-server/client?http://localhost:8080 /
    ],
    hot:true,
    plugins:[
        new webpack.HotModuleReplacementPlugin()
    ]
}
```
