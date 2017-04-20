---
title: Express的简单实现
date: 2016-11-13 03:07:07
tags: [javascript]
---
最近在写express框架的简单实现，其中的next，开始的时候总是想不明白，仔细看了源码后，理解了代码的含义。<!--more-->
### Express
Exress 是 基于connect的Nodejs框架，核心功能就是中间件.
简单的express框架实现主要是对app.use,app.hindle,app.listen的实现
先给出express.js app的定义
```javascript
let _proto={}
function createServer(){
    function app(req,res,next){
        app.handle(req,res,next)
    }
    Object.assign(app,_proto)
    Object.assign(app,EventEmitter.prototype)
    app._route='/'
    app._stack=[]
    return app
}
```
app是一个函数，参数列表为req,res,next。我开始纠结的是next是在什么时候被作为参数传递进来的，源码看了好久，最后才明白。当存在子app的时候，next在上一个handle函数传给子app.handle(req,res,next).开始的时候next参数为空. 通过不断调用handle内部的next()函数 传递next
### app.use
代码如下：
```javascript
_proto.use=function(route,fn){
    if(typeof route!=='string'){
        fn=route
        route='/'
    }
    if(typeof fn.handle==='function'){
        fn._route=route
        fn=fn.handle.bind(fn)
    }
    if(route[route.length-1]==='/')
        route = route.slice(0,-1)
    this._stack.push({route:route,handle:fn})
    return this
}
```
app.use这个函数的参数列表为route和fn，在app函数内定义了一个数组stack用来存储中间件，即路由和对应的回调函数。有一点需要注意，就是fn为子app的时候。
### app.handle
代码如下
```javacript
 console.log(out)
    let index = 0,
        removed=''
    let next= err =>{
        
        if(removed.length){
            req.url = removed+req.url
            removed = ''
        }
        let layer = this._stack[index++]
        if(!layer){
        console.log('out')
            out && out(err)
            return 
        }
        let route = layer.route
        if(req.url.substr(0,route.length).toLowerCase()!==route.toLowerCase())
            return next(err)
        let border = req.url[route.length]

        if(border !== undefined && border !=='/')
            return next(err)
        if(route.length !==0 && route!=='/'){
            req.url = req.url.substr(route.length)
            removed = route
            
        }
        let handle = layer.handle
        if(err&&handle.length ===4)
            return handle(err,req,res,next)
        else if(!err && handle.length <4)
            return handle(req,res,next)
        next(err)
    }
    next()
}
```
### app.listen
代码如下:
```javascript
_proto.listen=function(...args){
    const server = http.createServer(this)
    return server.listen.apply(server,args)
}
```
代码很简单 调用http.createServer(this) // app 是一个函数 。
最后一句 server.listen.apply(server,args) //这里用apply的原因是 listen内部可能用到server的this指代，所以用apply() 用 server代替this.

总结： 通过看源码收获了很多，不仅仅知道了next（）什么时候用，怎么用，更是明白了express底层的机制原理！
