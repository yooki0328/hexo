---
title: 实现ES7中的async
date: 2017-04-07 15:37:38
tags: [javascript]
---
async函数是为了使得异步操作更加方便而引入的.async是什么？它就是Generator函数的语法糖，即自动执行的generator函数.
***基本用法***
**async**函数返回一个Promise对象，可以使用**then**方法添加回调函数.
在函数执行的时候，遇到**await**就会先返回，等到异步操作完成，再接着执行函数体内后面的语句.<!--more-->
一段示例代码如下:
```js
function timeout(ms){
  return new Promise(function(resolve)=>{
    setTimeout(resolve,ms)
  })
}
async function asyncPrint(value,ms){
  await timeout(ms)
  console.log(value)
}
asyncPrint('hello world',50)
```
上述代码 50ms之后输出**hello world**.
***实现async函数所需要知道的***
1. 首先，我们需要知道执行async函数后发生了什么？
   返回一个promise对象，内部封装了一个自动执行的generator函数
2. 如何实现自动执行的generator函数？
   generator函数返回的是一个iterator对象，有一个value属性和一个next()方法.
3. generator函数内部是怎样的呢？
   generator函数内部是一个带有yield语句的普通函数.yield语句是暂停执行的标记，而next方法可以恢复执行.
***实现async函数的代码***
```javascript
async function fn(args){
  //...
} 
等同于 
function fn(args){
  return genExec(function*(){
  //...
  })
}

```
函数genExec是一个返回值为promise对象的自动执行函数.
```javascript
function genExec(genF){
   return new Promise((resolve,reject)=>{
    var gen = genF()
    function next(nextF){
        try{
          var ne = nextF()
        }catch(e){
          return reject(e)
        }
        if(ne.done){
          resolve(ne.value)
        }else{
          Promise.resolve(ne.value).then(v=>{
            next(function(){return gen.next(v)})
          },e=>{
            next(function(){ return gen.throw(e)})
          })
        }
    }
    next(function(){ return gen.next(undefined)})
   
    })
}
```
综上所述，源代码可以更改为如下:
```javascript
function asyncPrint(value,ms){
  return genExec(function*(){
    yield timeout(ms)
    console.log(value)
  })
}
```
