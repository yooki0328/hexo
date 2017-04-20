---
title: 手动实现ES6中的promise
date: 2017-02-28 13:35:21
tags: [javascript]
---
Promise是ES6中的一个重要的概念。promise是一个对象，简单来说，是一个容器，里面保存着某个未来才会结束的事件的结果.从它可以获取异步操作的消息。我们要是实现Promise，就要了解Promise的运行机制以及特点。<!--more-->
## Promise的特点及用法
1. 特点：对象有三种状态:Pending,Resolved,Rejected.状态从pending到Resolved，或者 从pending到Rejected。只有这两种状态改变且改变之后不会再改变.
2. 用法：ES6规定，Promise对象是一个构造函数，用来生成Promise实例.
3. 实例：
```javascript
function fn1(resolve,reject){
  setTimeout(function(){
    console.log('步骤一:执行')
    resolve('1')
  },1000)
}
function fn2(resolve,reject){
  setTimeout(function(){
    console.log('步骤二:执行')
    resolve('2')
  },500)
}
new Promise(fn1).then(function(val){
  console.log(val)
  return new Promise(fn2)
}).then(function(val){
  console.log(val)
  return 1111
}).then(function(val){
  console.log(val)
})
```
运行结果如下图:![]()
这里解释一下:promise的运行机制:then()是好比一个入队列的操作，把待执行的函数，压入队列中.resolve()好比是一个出队列的操作,取出要执行的函数并执行，且then的返回值是一个新的promise对象.于是乎，我们可以大致写出这么一个对象promise：
```javascript
function promise(fn){
  var value = null
  var self = this
  self._resolves=[]
  self._rejects =[]
  self.then=function(done){
        self._resolves.push(done)
        return self
  }
  function resolve(value){
    setTimeout(function(){     //确保执行之前then全部执行完
      self._resolves.forEach(function(callback){
        value = callback&&callback(value)
      })
    },0)
  }
  function reject(reson){
  
  }
  try{
    fn(resolve,reject)
  }catch(e){
    throw e
  }
}
```
先把fn1添加到resolves中，接着调用then方法依次把function放进队列中，执行到任务队列末尾，调用fn1的settimeout函数，输出'步骤一:执行'，接着调用resolve('1')。
可以发现这里出现了个问题：如果返回值是promise对象的话，则上一个promise的_resolves数组中的callback无法调用，为了解决这个问题，把then函数改为返回一个promise对象。并且对then中的函数用handle包装起来，如果返回值是promise对象的话，就使 promise.then调用上一个promise的resolve()。这样就可以保证有序的执行，得到我们想要的结果了
```javascript
function promise(fn){
  var value = null
  var self = this
  self._resolves=[]
  self._rejects =[]
  self._status = 'PENDING'
  this.then=function(onfulfilled){
    return new promise(function(resolve){
      function handle(value){
       var ret = typeof onfulfilled === 'function' &&onfulfilled(value)||value
        if(ret && ret['then'] === 'function'){
            ret.then(function(value){
              resolve(value)
            })
        }else {
          resolve(ret)
        }
      }
      if(self._status === 'PENDING')
        self._resolves.push(handle)
      else if(self._status === 'FULFILLED')
        handle(value)
    })
  }
  function resolve(val){
    setTimeout(function(){     //确保执行之前then全部执行完
      self._status = 'FULLFILED'
      self._resolves.forEach(function(callback){
	  callback(val)
	})
	},0)
  }
  function reject(reson){
  
  }
  try{
    fn(resolve,reject)
  }catch(e){
    throw e
  }
}
```
接下来就是把reject添加进去就大致完成了promise
```javascript
function promise(fn){
  var value = null
  var self = this
  self._resolves=[]
  self._rejects =[]
  self._status = 'PENDING'
  self.then=function(onfulfilled,onRejected){
    return new promise(function(resolve){
      function handle(value){
       var ret = typeof onfulfilled === 'function' &&onfulfilled(value)||value
        if(ret && ret['then'] === 'function'){
            ret.then(function(value){
              resolve(value)
            })
        }else {
          resolve(ret)
        }
      }
      function errback(reason){
	reason = isFunction(onRejected)&&onRejected(reason)||reason
	reject(reason)
      }
      if(self._status === 'PENDING'){
        self._resolves.push(handle)
	self._rejects.push(errback)
      }
      else if(self._status === 'FULFILLED')
        handle(value)
    })else if(self._status === 'REJECTED'){
	errback(self._reason)
	}
  }
  function resolve(val){
    setTimeout(function(){     //确保执行之前then全部执行完
      self._status = 'FULLFILED'
      self._resolves.forEach(function(callback){
	  callback(val)
	})
	},0)
  }
  function reject(reson){
      setTimeout(function(){
	self._status = 'REJECTED'
	self._rejects.forEach(function(callback){
		self._reason = callback(value)
	})
	},0)
  }
  try{
    fn(resolve,reject)
  }catch(e){
    throw e
  }
}
```
这样的话，promise大致就简单的实现了～～～yeah
***后序更新***
### promise.all()的实现
promise.all(args) 参数为一个promise数组，当且进当数组中所有的promise都执行完才执行后续的回调函数.内部的实现原理为:对数组进行遍历，对每个promise元素添加一个回调函数(通过then())，这个回调函数是一个闭包函数，记录着promise元素的返回值和promise执行完的个数，当个数等于数组的长度的时候 代表所有的promise都已经执行完毕.如果有一个reject则promise.all()直接调用reject返回。
### promise.race()的实现
promise.race(args) 与all(),差不太多。区别就是当且仅当有一个promise元素resolve之后promise.all()则执行resolve()函数.内部的实现原理为：对数组进行遍历，对每隔promise元素添加一个回调函数(通过then())，这个回调函数同样是一个比包函数，记录着promise元素的返回值和promise失败的个数，当个数等于数组的长度的时候，promise.all()执行reject()函数，当任意一个promise执行过resolve之后，则promise.all()直接调用resovle函数.
