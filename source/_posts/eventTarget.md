---
title: Javascript自定义事件
date: 2017-03-23 09:22:15
tags: [javascript]
---
###原生事件监听器注册方法
首先，Js中自带的事件监听函数是window.addEventListener(type,callback,false/true),IE可能不支持需要用element.attachEvent(type,callback)
但是，这个监听器只可以对特定的时间进行监听，无法实现自定义事件监听。
所以我们可以写出一个具有兼容性的addlistener函数，代码如下：<!--more-->
```javascript
function addlistener(el,type,callback,flag){
  if(el.addEventListener){
    el.addEventListener(type,callback,flag)
  }else if(el.attachEvent){
    el.attachEvent('on'+type,callback)
  }
}
```
###自定义事件
因此，我们需要自己去实现一个事件监听器注册的函数，解析在代码的注释中，直接上代码：
```javascript
function EventTarget(){
  this.handlers={} //用来存储事件相对应的callback
}
EventTarget.prototype={
  constructor:EventTarget,
  add(type,callback){
      if(!this.handlers[type]){
        this.handlers[type]=[callback]
        
      }else{
        this.handlers[type].push(callback)
      }
  },
  fire(type){
    console.log(this.handlers[type])
    if(this.handlers[type].length!=0 ){
      var handlers = this.handlers[type]
      for(var i=0;i<handlers.length;i++){
        handlers[i]()
      }
    }else{
      console.log(`no this ${type} function`)
    }
    
  },
  remove(type,handle){
    if(this.handlers[type] !=undefined){
      var handlers=this.handlers[type]
      
      for(var i=0;i<handlers.length;i++){
        if(handlers[i]==handler){
          break;
        }
      }
      handlers.splice(i,1)
    }else{
      console.log('error')
    }
  }
}
var event = new EventTarget()
var handler = function(){
  console.log('haha')
}
event.add('test',handler)
event.fire('test')
event.remove('test',handler)
event.fire('test')
```
