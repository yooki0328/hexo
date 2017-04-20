---
title: express整理
date: 2016-09-20 03:55:35
tags: [nodejs]
---
之前对express粗糙的用过，写过express+mongoose个人博客，上周看了深入浅出node.js，现在打算花费一周时间来整体捋一下express<!--more-->
### Express基于Node.Js平台，快速开放、极简的web开发框架
* npm install express –save  //安装express</li>
* npm install express=gengerator -g //通过Express生成器创建express应用</li>
* express路由 :路由是由一个URI(路径)和一个特定的http方法(GET,POST)组成<a id="more"></a>
　　每个路由都可以有一个或者多个处理器函数，当匹配到路由时，这个函数将被执行
　　路由的定义由如下结构组成:
**app.METHOD(path,handler)**
<pre>
 app.get(‘/‘,function(req,res){}})
 app.post(‘/‘,function(req,res){})
</pre>
**app.route()**//定义了链式路由句柄
<pre>
app.route(‘/‘).get(function(req,res){}).post(function(req,res){})
</pre>
**express.Router** //Router实例是一个完整的中间件和路由系统
<pre>
var express = require(‘express’)
var router = express.Router()
//该路由使用的中间件
router.use(function timeLog(req,res,next){
console.log(‘Time’,Date.now())
next()
})
//定义网站主页的路由
router.get(‘/‘,function(req,res){})
module.exports=router
//在应用中加载路由模块
var route = require(‘./route.js’)
app.use(‘/‘,route)</pre></li>
* 利用Express托管静态文件
app.use(express.static(‘public’)) //调用express.static中间件

## [](#MVC模型 "MVC模型")MVC模型
M:Model V：view C：Control

## [](#express响应方法 "express响应方法")express响应方法
<table><thead><th>方法</th><th>描述</th></thead><tbody><tr><td>res.download()</td><td>提示下载文件</td></tr><tr><td>res.end()</td><td>终结响应处理流程</td>
</tr><tr><td>res.json()</td><td>发送一个JSON形式的响应</td></tr><tr><td>res.redirect()</td><td>重定向请求</td></tr><tr><td>res.render()</td><td>渲染视图模板</td></tr><tr><td>res.send()</td><td>发送各种类型的响应</td></tr></tbody></table>
</li>
</ol>


* 使用中间件
中间件(Middleware)是一个函数，它可以访问请求对象req，响应对象res和web应用中处于请求响应循环流程中的中间件，一般被命名为next的变量。如果当前中间件没有终结请求响应循环则必须调用next()方法将控制权交给下一个中间件，否则请求就会挂起
**错误处理中间件** //四个参数
app.use(function(err,req,res,next){
 console.error(err.stack)
 res.status(500).send(‘Something broke’)
})
**内置中间件**
从4.x版本开始，除了express.static,Express以前的中间件已经全部单独作为模块安装使用
* 在Express中使用模板引擎
在应用中进行如下设置才能让Express渲染模板文件

* views,放模板文件的目录，app.set(‘views’,’./views’)
* view engine ,模板引擎 ,app.set(‘view engine’,’jade’)
* 调试Express </br>内部使用debug模块调试，在启动应用时，设置DEBUG环境变量为express:_ 代码为:
DEBUG=express:_ node index.js

### Express API

* **Properties**
**app.locals***:the app.locals object is a JavaScript object,and its properties are local variables within the application.
* Events
**app.on(‘mount’,callback(parent))**//监听挂载事件
* **Methods**
app.all(path,callback)
app.disable(name) =&gt; app.set(name,fasle)
app.disabled(name) if app.disable(name) then app.disabled(name)=&gt;true
app.enable(name) =&gt;app.set(name,true)
app.enabled(name)
app.engine(ext,callback)// ex: app.engine(‘jade’,require(‘jade’).__express)
app.get(name)//return the value of name app setting
app.param([name],function(req,res,next,id){}) called only once and the priority is the highest
app.path() // return the cannonical path of the app,a string
app.route(path)
app.use([path],function{}) //mount the middleware at the path
* **Request properties**
req.app =&gt; return the instance of express
req.baseUrl=&gt; return the path of mounted router instance
req.body=&gt; Contains key-value which is submitted in the request body.But: require(‘body-parser’), then app.use(bodyParse.json()),app.use(bodyParse.urlencoded({extended:false}))
req.rookies =&gt; require(‘cookie-parser’)
req.fresh //Indicates whether the request is “fresh”.THe follows is true
　　if-modified-since == last-modified
req.hostname ,req.ip,req.originalUrl
req.params example: route: /user/:name =&gt;req.params.name
req.path =&gt;return the path part of the request URL
req.query example:route: /search?q=1 =&gt;req.query.q = 1
req.param =&gt; req.params req.query req.body
* **Response properties**
res.headersSent Indicate if the app sent HTTP headers for the response
res.locals.An object that contains response local variablesscoped to the request.
res.cookie(name,value,[options]) =&gt;Sets cookie name to value
res.clearCookie(name,[options])
res.download(path,[filename])
res.end() =&gt;Ends the response process
res.get(field) =&gt; returns the HTTPresponse header
res.location() == res.redirect([status],path)
res.render()  // 内部调用app.render() =&gt;生成视图的工具
res.send([body]) the parameter can be Buffer object, String,Array,Object
res.sendStatus() 200=&gt;OK 403=&gt;Forbidden 404=&gt;NOt FOund 500 =&gt;Internal Server Err

* * *

## [](#Mongoose "Mongoose")Mongoose

1.名词解释
　　<font style="background=#f1f1f1">Schema</font>:**_一种以文件形式存储的数据库模型骨架，不具备数据库的操作能力_**
　　<font style="background=#f1f1f1">Model</font>：**_由Schema发布生成的模型，具有抽象属性和行为的数据库操作对_**
　　<font style="background=#f1f1f1">Entity</font>: **_由Model创建的实体，操作会影响数据库_**

1.  使用
<pre style="background=#f1f1f1">
var mongoose = require(‘mongoose’)//引用mongoose模块
var db = mongoose.createConnection(‘localhost’,’test’)//创建数据库连接
//定义一个**_Schema_**
var PersonSchema = new mongoose.Schema({
 name:String //定义一个属性name，类型为String
})
//将Schema发布为Model
var PersonModel = db.model(‘Person’,PersonSchema)
//用Model创建Entity
var PersonEntity = new PersonModel({name:’Krouky’})
//Entity是具有具体的数据库操作CRUD的
PersonEntity.save()
//执行查询
PersonModel.find(function(err,persons){})</pre>

1.  具体语法
更新：
1.PersonModel.findById(id,function(err,person){
 person.name=’Mdragon’
 person.save=(function(err){})
})
2.PersonModel.findById(id,function(err,person){
 person.name=’MDragon’
 var _id=person._id
 delete person._id
 PersonModel.update({_id:_id},person,function(err){})
}) // update 第一个参数是查询条件，第二个参数是更新的对象，不可以更新主键
PersonModel.update({_id:_id},{$set:{name:’MDragon’}},function(err){})
新增：1.使用Entity增加数据
var krouky = new PersonModel({name:’krouky’})
krouky.save(callback)
2.使用Model来增加一条数据
var MDragon = {name:’MDragon’}
PersonModel.create(MDragon,callback)
查询
1.PersonModel.findOne({‘name’:’MDragon’},’select’,function({err,person}))
2.var query = PersonModel.find({name:’MDragon’})
query.select(‘select’)
query.exec(function(err,person))
验证
1.required:非空验证 2.min/max: 范围验证 3. enum/match： 枚举验证 4.validate:自定义验证

## [](#Cheerio-superagent-爬虫 "Cheerio + superagent 爬虫")Cheerio + superagent 爬虫

## [](#cheerio-API "cheerio API")**_cheerio API_**

1.  npm install cheerio
2.  将html告诉你的服务器
<pre style="background=#f1f1f1">
var cheerio = require(‘cheerio’)
$ = cheerio.load(html)
$data = $(data)
//接下来即跟jquery用法相同了
</pre>
**_superagent API_**
3.  npm install superagent
4.  使用<pre style="background=#f1f1f1">
var superAgent =require("superagent")
superAgent.get(url).end(function(err,sres){}) // get
superAgent.post(url).send({name:'haha',species:'cat'}).end(function(res))
superAgent.del(url).end(function(res))
</pre>

* * *

## [](#使用eventproxy控制并发 "使用eventproxy控制并发")使用eventproxy控制并发

**_eventproxy API_**

1.  npm install eventproxy
2.  使用<pre style="background=#f1f1f1">
var ep = new EventProxy()
ep.all('data1_event','data2_event','data3_event',function(data1,data2,data2){})
//监听了3个事件，当三个事件全部完成后调用回调函数
ep.after('got_file',files.length,function(list){})
//after方法适合重复异步协作 // files数组 list数组存储的是文件的内容
for(var i =0,i<files.length,i++) {="" fs.readfile(files[i],'utf-8',function(err,content){="" ep.emit('got_file',content)="" })="" }="" ep.tail('tp1','data',function(tp1,data))="" 在所有制定事件出发后，将会被调用执行="" <="" pre="">
</files.length,i++)></pre>

* * *

## [](#使用async控制并发 "使用async控制并发")使用async控制并发

async的mapLimit(arr,limit,iterator,callback)接口
arr:数组 limit:并发量 iterator: 迭代函数 callback:回调函数

## [](#mocha-should "mocha should")mocha should

1.  sudo npm install mocha -g
2.  npm install should
3.  使用
<pre>
//新建一个main.js
var fibonacci=function(n){
if(n==0)
return 0
if(n==1)
return 1
return fibonacci(n-2)+fibonacci(n-1)
}
exports.fibonacci=fibonacci
//新建一个test.js
var main = require(‘./main’)
var should = require(‘should’)
descript(‘/test.js’,function(){
 it(‘should equal 55 when n ===10’,function(){

        main.fibonacci(10).should.equal(55)
 })
})
//执行mocha即可
descript:描述的是你要测试的主题，
it:描述的是具体的case内容
should模块：是一个断言库
</pre>

    ### [](#关于exports和module-exports "关于exports和module.exports")关于exports和module.exports

4.  exports是指向<font style="color:blue;">module.exports</font>的引用
5.  module.exports初始值是一个空对象{}
6.  require()返回的是module.exports

    ### [](#ejs "ejs")ejs
&lt;%=code%&gt;会对code进行html转义
&lt;%-code%&gt;不会对code进行html转义
&lt;%code%&gt;执行其中的js代码
基础知识仔细回顾一边之后，做个项目来理一理express开发的流程
代码在[github](https://github.com/yooki0328)
