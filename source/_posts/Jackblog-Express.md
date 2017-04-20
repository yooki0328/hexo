---
title: Jackblog-express源码分析
date: 2016-10-21 03:18:51
tags: [javascript]
---
Jackblog是一个前后端分离的项目，前两天把前端vue-blog的源码看完了，本人对express比较熟悉，所以这篇文章只是对代码的逻辑以及中间件的使用进行分析.<!--more-->
### 整体流程
通过gulp.src()加载gulp.task()的配置文件，加载nodemon启动app.js，express服务器打开. 用mongoose 以及es6原生promise以脱离回调函数的方式来写数据库的初始化以及CRUD. 在逻辑处理部分:require进来express的配置部分，require进来router的配置，很值得学习的地方，令人看着清晰明了.在router的配置部分,是通过对每个路由挂载一个require进来的router对象.对路由的处理函数分离出到api的文件夹里，每个函数都用module.exports暴露出来。 整个流程就是如此！
### 技术栈
* express4.X
* mongoose
* 各种中间件的使用
### 中间件分析
此处贴出express.js代码
![]http://of8m1pnnt.bkt.clouddn.com/express.jpg
[compression](https://github.com/expressjs/compression)
Node.js的压缩中间件，支持deflate，gzip的编码
[body-parser](https://github.com/expressjs/body-parser)
对请求request的body部分进行解析，可通过req.body获取值
useage:
app.use(bodyParser.urlencoded({extended:false})) //只解析urlencoded body，当extended是true的时候解析任意类型的键值对，当extended是false的时候，解析value是string或array的键值对
app.use(bodyParser.json())//对json解析
[cookie-parser](https://github.com/expressjs/cookie-parser)
解析cookie header 并且用这个cookie对象来填充req.cookies
usage:
app.use(cookieParser())
[cors](https://github.com/expressjs/cors)
解决跨域问题的中间件，跨域是指不同域名之间相互访问，二级域名、端口、协议必须与主页面完全相同，否则就算是跨域.
**usage:**
```javascript
app.use(cors()) // 允许任意来源的网站跨域访问
app.use(cors({
    origin:'http://example.com'  //只允许这个网站跨域访问
}))
```
流程：　对于简单请求，浏览器发出CORS请求，即在请求的头信息中增加Origin字段。
服务器根据这个字段指定的源来响应，若在范围内，响应的头信息添加以下几个头信息字段
Access-Control-Allow-Origin：表示接受某域名的请求
Access-Control-Allow-Credentials：表示是否允许发送Cookie，默认为false
Access-Control-Allow-Expose-Headers： CORS请求时，XMLHttpRequest对象的getResponseHeader()只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，必须在这个字段里指定.[参考链接](http://www.ruanyifeng.com/blog/2016/04/cors.html)
[method-override](https://github.com/expressjs/method-override)
Lets you use HTTP verbs such as PUT or DELETE in places where the client doesn’t support it.
在表单中 只能用get或者post提交，运用这个中间件的话，就可以实现delete或者put的提交了～
usage:
```javascript
app.use(methodOverride('X-HTTP-Method-Override'))
or
app.use(methodOverride(function(req,res){
    if(req.body&&typeof req.body==='object'&&'_method'in req.body){
        var method = req.body._method
        delete req.body._method
        return method
    }
}))
```
[passport](https://github.com/jaredhanson/passport)
这个中间件用来进行身份验证。你提供Passport一个认证请求，Passport返回一个钩子控制回调函数(成功或者失败)
usage:
以QQ为例
```javascript
var passport=require(‘passport’)
var qqStrategy=require(‘passport-qq’).Strategy

passport.use(new qqStrategy({
clientID:xxxx,
clientSecret:xxxx,
callbackURL:xxx,
passReqToCallbacl:true
}),function(req,accessToken,refreshToken,profile,done){})
```
[express-session](https://github.com/expressjs/session)
usage:
app.use(session({
    secret: ‘any key’,
    resave: true, //即使session没有被修改，也重新保存，默认为true
    saveUninitialized:true,//每次请求重新设置session cookie 默认给哥connect.sid
    cookie:{maxAge:6000*5}，
    store:  //session的存储方式，默认存储在内存中，可以使用redisstore
    store:new RedisStore({
        host:xxx
        port:xxx
        pass:xxx//password 认证Redis
    })

}))
[cookie与session](http://kirochen.com/2015/07/09/about-cookie-session/#)
### 路由逻辑
routes.js
![](http://of8m1pnnt.bkt.clouddn.com/routes.jpg)
user.js
![](http://of8m1pnnt.bkt.clouddn.com/controller.jpg)
controller.js
![](http://of8m1pnnt.bkt.clouddn.com/user.jpg)
这种思路很值得学习，具体没什么特别说的，对路由的请求处理～
