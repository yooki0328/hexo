---
title: Jackblog-vue源码分析
date: 2016-10-18 03:35:34
tags: [javascript]
---
最近开始学习vue.js,在CNode社区发现了一篇Jackblog的帖子，觉得这是个值得学习的项目，通过这个项目来更好的学习vue.js。<!--more-->
* 技能
1. gulp:基于流的自动化构建工具.
2. webpack模块加载器兼打包工具.
3. vue.js构建数据驱动的前端框架.
* 流程
通过gulp.task启动webpack-dev-server和webpack,webpack对入口文件index.js以及import的模块进行打包,生成bundle.js。webpack-dev-server用模板插件生成index.html，并引用bundle.js 。
然后是生成Vue实例，利用vuex和vue-router进行组件状态管理和路由跳转.
* 详细分析：
  项目目录：
### gulpfile.js
```javascript	
gulp.task('server',function(){})
```
1. server:直接调用new webpackDevServer(webpack(config),{‘对devserver的配置’}).listen(port,’localhost’,funtion(err){}).
2. webpackDevServer:启动webpack-dev-server以及webpack.
将webpack-dev-server添加到webpack的入口文件需要添加：’webpack-dev-server/client?http://localhost:8080‘.
3. 将HMR(热替换文件)添加到webpack的入口文件需要添加：’webpack/hot/dev-server’.
4. webpack-dev-server配置:inline:true—server对入口文件的自动打包和刷新,publicPath:服务器的静态文件目录,hot:true—HMR(热替换):无需刷新整个页面，只把变化的部分替换掉.
### webpack.config.js
这里主要说一下用到的插件以及loader.
1. HtmlWebpackPlugin 用来简化创建服务于webpack bundle的HTML文件。配置信息的话可以自行百度.
2. ExtractTextPlugin:用来独立出css样式.
```javascript
loaders:[
    {test:/\.css$/,loader:ExtractTextPlugin.extract("style-loader","css-loader")} //第一个参数是经过编译后通过style-loader单独提取出来，第二个参数用来编译代码的loader
]
plugins:[
    new ExtractTextPlugin("style.css",{allChunks:true}) //所有独立样式打包成一个css文件
]
```
3. webpack.optimize.OccurenceOrderPlugin())为组件分配ID,通过这个插件webpack可以分析和优先考虑使用最多的模块
4. 缓存
```javascript
output:{
    path:__dirname,
    filename:'[name].[hash:5].js' //在文件名字后面加md5
}
```
### Vue
index.js
* Vue-Router
```javascript
	
Vue.use(VueRouter)//安装路由功能
const router=new VueRouter({})//生成vuerouter实例
configRouter(router)//对router进行路由配置
configRouter()调用了router.map({
    '/':{
     	name:"...",
       	component:require('...')
    } 
})
router.start(Vue.extend(App),"#root")//启动一个启用了此路由的应用.创建Vue实例(以App为根组件)挂载到id=root的元素上.
```
* VueX
Vuex: state management pattern +library for Vue.js
vuex把应用的数据和修改数据的方法放在了一个store对象里面统一管理，对数据的获取和修改通过配置属性vuex的getters和actions来进行
![](http://of8m1pnnt.bkt.clouddn.com/vuex.png)
目录图：
核心：store.js actions.js
Vue.use(Vuex) //注册Vuex 必须一步
export default new Vuex.Store({}) //暴露出生成的store对象
**注意**
在根组件引入 store对象 ：import store from “../vuex/store”
export default{store} // 被vue-loader暴露出去被JS加载器加载。
简单一句 Vue.use(Vuex) 然后在根组件内部引入 store 属性就能将store引入到vue的子组件里，通过在vue组件内部调用 this.$store 就能访问到store，而不是每个子组件引入一次store
### api
vue-resource 提供网络请求服务和处理响应通过XMLHttpRequrest
支持 Promise API
```javascript
this.$http.get('/someUrl').then((res)=>{
    //success callback
},(res)=>{
    //error callback
})
2.Vue.resource('url'+{/id}).get({id:'someId'}).then((res)=>{
    //succss callback
},(res)=>{
    //error callback
}) //{/id} 占位符
```
ndex.js 暴露出来的是前端向后端发送请求的API.供vuex中的actions调用。

### 总结
项目主要逻辑就是如此：
gulp控制流程构建
webpack打包/加载(css-loader,image-loader,vue-loader…)/调用插件/优化请求
webpack-dev-server:详细配置：简书
vue-loader:参考Thinksaas
vue+vue-router+vuex+vue-resource 前端逻辑搭建
vue-strap vue.js的bootstrap组件
plugin，loader，参考github

