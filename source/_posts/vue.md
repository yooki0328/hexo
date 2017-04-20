---
title: vue源码分析
date: 2017-04-16 14:54:41
tags: [javascript]
---
想要真正的掌握一个框架就应该了解它的源码，知道整个框架是如何工作的.Vue是一个轻量化但非常好用的前端框架.它是以数据驱动为核心的javascript框架.Vue2.0更是重写之前的vue代码。本次要看的vue版本是2.03,线上包只有12Kb，性能也是非常不错的.
接下来从生命周期的各个过程去分析vue源码.<!--more-->
## 生命周期
我们首先需要对Vue的生命周期有一个完整的理解，在下面放两张图。
![](http://of8m1pnnt.bkt.clouddn.com/3504099265-580628fd03258_articlex.png)
![](http://of8m1pnnt.bkt.clouddn.com/3346068135-580822cd52898_articlex.png)
从这个生命周期图我们可以得知vue的声明周期主要是4个过程：
* create
* mount
* update
* destroy

### Create
new Vue()的时候发生了哪些事情呢？先从 vue/src/core/instance/index.js看起。
index.js
主要代码片段如下: 解析写在注释里
```javascript

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }  // 这个if判断是用来说明 Vue这个函数是个构造函数只能够通过 new的方式调用
  this._init(options) // 用来初始化vue的函数(这个函数是在vue.prototype上的)
}
//以下是对Vue这个构造函数的初始化
initMixin(Vue) 
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
``` 
接下来是对每个函数的解析

### this._init(vm)

文件为 vue/src/core/instance/init.js
主要代码片段为
```javascript
export function initMixin (Vue: Class<Component>) {
  //初始化vue的函数 也就是上文中调用的this._init函数
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    //...中间一部分省略掉
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
      vm.$mount(vm.$options.el)
  }
}
```
在这个_init函数中主要做了什么事情呢？没错，就是主要执行以下这些函数：

*    initLifecycle(vm)
*   initEvents(vm)
*    initRender(vm)
*    callHook(vm,”..”)
*    initInjections(vm)
*    initState(vm)
*    initProvide(vm)
*    vm.$mount(vm.$options.el)

### initLifecycle(vm)
文件路径 vue/src/core/instance/state.js
主要代码如下:
```javascript
export function initLifecycle (vm: Component) {
  const options = vm.$options
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }
  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm
  vm.$children = []
  vm.$refs = {}
  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

### initEvents(Vm)
文件路径 vue/src/core/instance/event.js
主要代码如下:
```javascript
export function initEvents (vm: Component) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
```

### initRender(vm)
文件位于 vue/src/core/instance/render.js
主要代码如下：
```javascript
export function initRender (vm: Component) {
  vm.$vnode = null // the placeholder node in parent tree
  vm._vnode = null // the root of the child tree
  vm._staticTrees = null
  const parentVnode = vm.$options._parentVnode
  const renderContext = parentVnode && parentVnode.context
  vm.$slots = resolveSlots(vm.$options._renderChildren, renderContext)
  vm.$scopedSlots = emptyObject
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
}
```

### callHook(vm,'beforeCreate')
首先callHook函数定义在 lifestyle.js中
```javascript

export function callHook (vm: Component, hook: string) {
  const handlers = vm.$options[hook]
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        handlers[i].call(vm)
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
}
```

### initInjections(vm)
文件代码位于 vue/src/core/instance/inject.js
主要代码如下：
```javascript
export function initInjections (vm: Component) {
  const result = resolveInject(vm.$options.inject, vm)
  if (result) {
    Object.keys(result).forEach(key => {
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production') {
        defineReactive(vm, key, result[key], () => {
          warn(
            `Avoid mutating an injected value directly since the changes will be ` +
            `overwritten whenever the provided component re-renders. ` +
            `injection being mutated: "${key}"`,
            vm
          )
        })
      } else {
        defineReactive(vm, key, result[key])
      }
    })
  }
}
```

### initState(vm)

文件位于 vue/src/core/state.js
代码如下:
```javascript
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch) initWatch(vm, opts.watch)
}
```
initState(vm)做了什么呢？

1.    初始化vm._watchers这个数组
2.    initProps 初始化options.props
3.    initMethods 初始化options.methods
4.    initData 初始化options.data

### callHook(vm,’created’)
调用options.created这个对象里的函数，表示这些函数在vue实例created之后调用

以上几个函数代表的是，.init()初始化，依次调用lifecycle，events，render，state模块中的初始化函数
接下来的函数是在.init()末尾调用的vm.$mount()

### vm.$mount(vm.$options.el)
具体相关代码如下：
```javascript
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)
  const options = this.$options
  //...省略一些代码
  if (!options.render) {  //如果存在render方法，直接运行mount
    let template = options.template
    if (template) { //如果有template，获取template
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) { //如果获取到了模板，则将模板转化为render方法
      //....省略一些代码
      const { render, staticRenderFns } = compileToFunctions(template, {
        shouldDecodeNewlines,
        delimiters: options.delimiters
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns
    }
  }
  return mount.call(this, el, hydrating)
}
```
这个函数做了什么呢？
根据el，template，以及render去生成Vue的AST render方法
解释一下:vue.prototype.$mount这个方法在 web-runtime.js和web-runtime-with-compiler.js均有定义,执行顺序是：先const mount = web-runtiome.js中的vue.prototype.$mount，然后在web-runtime-with-compiler.js中重写vue.prototype.$mount,最后在重写的vue.prototype.$mount末尾处调用mount方法.接着继续调用mountComponent()这个方法.这个方法定义在lifecycle.js中。

### mountComponent

主要代码如下：
```javascript
updateComponent = () => {
      vm._update(vm._render(), hydrating)
}
vm._watcher = new Watcher(vm, updateComponent, noop)
```
这段代码传入render方法生成的vNode对象，生成一个watcher.
接下来看生成watcher的过程

### wathcer

Wathcer构造函数定义在../observer/watcher.js中
主要逻辑代码如下:
```javascript
this.vm = vm
    vm._watchers.push(this)
    //...省略的部分
    this.value = this.lazy
      ? undefined
      : this.get()
  }
  get () {
    pushTarget(this)
  	let value = this.getter.call(vm, vm)
    const vm = this.vm
    if (this.deep) {
      traverse(value)
    }
    popTarget()
    this.cleanupDeps()
    return value
  }
```
中间的一大部分初始化操作可以省略不看，重点在于最后一句this.value ，this.lazy初始为false，则调用了this.get().
看看get()发生了什么？主要是调用了this.getter，在上面省略的代码中可知this.getter就是在上文中传入的第二个参数即 updateCompount这个函数，内部是两个函数的调用，先调用vm._render(),这个函数的返回值即vNode作为参数传入 vm._update()，来更新vDom。
接下来在看一看vm._render()和vm._update()
代码如下:
```javascript
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const prevActiveInstance = activeInstance
    activeInstance = vm
    vm._vnode = vnode
    if (!prevVnode) { //初次加载
      vm.$el = vm.__patch__(
        vm.$el, vnode, hydrating, false /* removeOnly */,
        vm.$options._parentElm,
        vm.$options._refElm
      )
    } else {
      // 更新vnode
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    //...后续省略.
  }
```
vm._render()这个函数之后再进行赘述，这里只需要知道这个函数返回值是vNode就可以了。
vm._update()这个函数的主要逻辑就是执行了vm.patch(prevVnode,vnode),对Virtual DOM 进行更新。
***综合以上三步***
1. 初始化之后调用vm.$mount(),生成AST render方法
2. 调用mount()，即调用mountComponent()这个函数，生成watcher
3. 生成watcher的过程中，调用了this.get(),这个函数内部通过调用this.getter()即传递进来的第二个参数updateComponent()来调用vm._render()生成vnode节点，调用vm._update更新vDom。


## 如何实现数据绑定的?
vue的数据绑定重中之重就是把数据(Model)和视图(view)建立起关系
回顾一下watcher构造函数中，调用了get(),这个函数中调用了pushTarget()和popTarget()
代码位于 src/core/observer/dep.js
代码如下：
```javascript
export function pushTarget (_target: Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}
export function popTarget () {
  Dep.target = targetStack.pop()
}
```
这两个函数用于处理Dep.target,分别是数组的push()和pop()，
Dep.target调用方式为 Dep.target.addDep(this) 调用的时候会涉及到依赖收集，从而建立数据绑定的关系。
这里我们得知，在使用数据进行渲染的过程中，会对数据属性进行“读”操作，从而触发dep.depend(),进而收集到这个依赖关系。

我们需要找到这个dep.depend()调用所在处，在observer/index.js中，defineReactive()
代码如下：
```javascript
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: Function
) {
  const dep = new Dep()
  const getter = property && property.get
  const setter = property && property.set
  let childOb = observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
        //...省略
        dep.depend()
    },
    set: function reactiveSetter (newVal) {
        //...省略
      dep.notify()
    }
  })
}
```
当数据的属性的get()被调用时，会通过dep.depend()收集依赖关系,记录到vm中
在数据的属性的set()被调用时，会通过dep.notify()来通知vm，从而触发vm的更新操作。
回头来看state.js,是在initProps()中调用了defineReactive().而initProps()是在initState()中调用，而initState()是在this._init()中被调用

总结如下：

1.    初始化：new Vue()->vm._init()
2.    数据劫持: initState(vm)->initProps(),initData()->dep.depend()
3.    依赖收集: vm.$mount()->vm._mount()->new Watcher()->vm._render()->vm._update()


