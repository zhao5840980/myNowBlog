---
title: Vue全家桶原理
date: 2021-04-12 10:13:51
author: zhengzheng
cover: true
categories: 前端
tags:
  - 前端
---

## Vue全家桶原理
## 1.Vue-router
> vue Router是Vue.js官方的路由管理器，它和Vue.js的核心深度集成，让构建单页面应用变的易如反掌

安装: 
```bash
Vue add router
```
核心步骤：

 步骤一:使用vue-router插件（1.应用插件）
```js
import Router from 'vue-router'
Vue.use(Router)

```

在Vue项目中建有router文件夹 -> index.js
```js
import vueRouter from 'vue-router'
//1.应用插件
Vue.use(vueRouter)
```
步骤二：创建Router实例，router.js(创建实例)
```js
const router = new vueRouter({
  mode:'history',
  base: process.env.BASE_URL,
  routes
})

const routes = [...]//对应配置的路由
```
步骤三：在Vue的入口文件main.js引入router文件（3.挂载router实例）
```js
import router from './router'
new Vue({
  router,(why? 为了全局使用)
  render: h => h(App)
}).$mount('#app')
```

问题：我们在单页面应用中使用Vue-Router时会用到router-link和router-view

问题：路由出口：<router-view/> 在这里面做刷新和替换，router是怎么实现的？

问题：router-link 和route-view可以直接使用？哪来的？说明有组件的声明和注册

猜测：router里面实现了这两个组件的声明和注册

> 通过监听浏览器的onHashChange事件，当url变化是，Router会根据Hash地址到路由表中找到对应的配置对象，在此对象中有对应的vue组件，通过router-view无刷新更新次路径的对应的页面内容

## vue-router源码实现

需求分析：
- 作为一个插件存在：实现vueRouter类和install方法
- 实现两个全局组件：router-view用于显示匹配组件内容，router-link用于跳转
- 监听url变化：监听haschange或popstate事件
- 响应最新url:创建一个响应式的属性current，当它改变时获取对应组件并显示

### 实现一个插件：创建VueRouter类和install方法

任务1. 实现一个插件：挂载$router(只是一个对象（一个插件）)，实现一个install方法
- 声明一个kVUeRouter类
- 保存构造函数，在kVueRouter里面使用
- 挂载$router
- 怎样获取根实例中的router选项，使用vue的混入

在main.js中vue实例时加入router的实现
```js
vue.mixin({
  beforeCreate(){
    if(this.$options.router){//确保根实例的时候才执行
      vue.prototype.$router = this.$options.router
    }
  }
})
```
任务2. 实现两个全局组件router-link和router-view
```js
//使用了vue中的component来创建全局组件，
//runtimer-only没有预编译，在运行代码期间，只能使用render函数
Vue.component('router-link', {
  render(h){
    return h('a',{attrs:{href:'#'+this.to},class:'router-link'},this.$slots.default)
  }
})

//router-view也同理这样实现
```
任务3. 监视URL变化

```js
class KVueRouter{
  constructor(options) {
    this.$options = options
    this.current = '/'
    //监控url变化
    window.addEventListener('hashchange',() => {
      this.current = window.location.hash.slice(1)
    })
  }
}
```

router-view实现：
1. 获取path对应的component(遍历$router中的$options中的routes属性，与当前的current路径做比较，若路径相同，则使用render函数渲染对应的页面)

任务4. 响应最新URL(问题:当前由于路由中的current不是响应的，所以切换URL时，页面不能实时刷新响应)
> 解决方案：在vue-Router中创建实例时构造函数中存储URL的变量需要是响应式的

> 需要创建响应式的current属性
```js
Vue.util.defineReactive(this, 'current','/')
```
将current变成响应式的好处是，不管用在任何组件，都会统一收集，只要一改变，就会通知所有的手机依赖，意味着render函数会重新执行

页面刷新时会存在bug,我们监听浏览器的Load事件获取当前的URL

优化：前面我们使用router-view中的Render函数更新对应组件时，使用的方式是遍历Router中的routes属性来获取对应的组件来进行更新渲染，所以我们需要有化一下
```js
//创建一个路由映射表
this.routeMap = {}
options.routes.forEach(route => {
  this.routeMap[route.path] = route
})
```
## 2. VueX(特点：集中式，可预测)单项数据流
> VueX集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以可预测的方式发生变化

![](vuex.png)

疑问：
- 当状态发生变化时，怎样通知视图更新？
- VueX和Redux最大的不同点在于哪？VueX是依据、依托vue的数据双向响应实现的，Redux是使用一个发布订阅的模式

### 整合VueX
```bash
vue add vuex
```
核心概念：
- state 状态，数据
- mutations 更改状态的函数
- actions 异步操作
- store 包含以上概念的容器
### VueX的实现:(VueX原理解析)
#### 任务分析：
- 实现一个插件：声明store类，挂载$store
##### store具体实现：
- 创建响应式的state,保存mutations、actions和getters
- 实现commit根据用户传入type执行对应mutation
- 实现dispatch根据用户传入type执行对应action,同事传递上下文
- 实现getters,按照getters定义对state做派生

```js
 // 保存构造函数引用,避免import
 let vue;
 class Store{
   constructor(options){
     this._mutations = options.mutations;
     this._actions = options.actions;
      //响应化处理state
      // this.state = new Vue({
      //   data: option.state
      // })
      //优化保护state变量数据
      this._vm = new Vue({
        data:{
          $$state: options.state//加两个$$,vue不做代理
        }
      })
      //绑定commit、dispatch的上下文store实例
      this.commit = this.commit.bind(this)
      this.dispatch = this.commit.bind(this)
   }
   //store.commit('add', |)
   //type:mutation的类型
   //payLoad:载荷，是参数
   commit(type, payload){
     const entry = this._mutations[type]
     if(entry){
       entry(this.state, payload)
     }
   }
   dispatch(type, payload){
     const entry = this._actions[type]
     if(entry){
       entry(this, payload)
     }
   }
   function install(_vue){
     vue.mixin({
       beforeCreate(){
         if(this.$options.store){
           vue.prototype.$store = this.$options.store
         }
       }
     })
   }
   //存储器 store.state
   get state(){
     return this._vm._data.$$state
   }
   set state(v){
      console.log('这样不好')
   }
 }
 //vueX
 export default{
   store,
   install
 }
```
## 理解Vue的设计思想
### MVVM模式

![MVVM](mvvm.png)

MVVM框架的三要素：1. 数据响应式、2. 模板引擎及其渲染

数据响应式：监听数据变化并在视图中更新
- Object.definePrototype()
- Proxy

模板引擎：提供描述视图的模板语法
- 插值：```js {{}} ```
- 指令：v-bind,v-on,v-model,v-for,v-if

渲染：如何将模板转换为html
- 模板：=> Vdom => dom(模板)
## 数据响应式原理
> 数据变更能够响应在视图中，就是数据响应式，vue2中利用object.defineproperty()实现变更检测

```js
1.//响应式
const obj = {}
function defineReactive(obj,key,val){
  //对传入obj进行访问拦截
  Object.defineProperty(obj, key,{
    get() {
      console.log('get' + key);
      return val
    },
    set(newVal){
      if(newVal !== val){
        console.log('set'+ key + ':' + newVal);
        val = newVal
      }
      //如果传入的newVal依然是obj，需要做响应化处理
      observe(newVal)
      //watchers.forEach(w => w.update())
    }

  }
  //递归Observe(val)
  //创建一个Dep和当前key一一对应
  const dep = new Dep()
  //依赖收集在这里
  dep.target && dep.addDep(Dep.target)
  //通知跟新
  dep.notify()//只更新相关节点
}
defineReative(obj, 'foo', 'foo')
obj.foo;
obj.foo = 'foooooo'

```

- 2. 在set函数中调用跟新函数

```js

functionn observe(obj){
  if(typeof obj !== 'object' || obj == null){
    //希望传入的是obj
    return
  }
  Object.keys(obj).forEach(key => {
    defineReactive(obj, key, obj[key])
  })
}
const obj = {foo: 'foo', bar:'bar', baz:{a:1}}
//遍历做响应化处理

```

## Vue中的数据响应处理

### 原理分析：
1. new Vue()首先执行初始化，对data执行响应化处理，这个过程发生在Observer中
2. 同时对模板执行编译，找到其中动态绑定的数据，从data中获取并初始化视图，这个过程发生在complie中
3. 同时定义一个更新函数和watcher，将来对应数据变化时watcher会调用更新函数 
4. 由于data的某个key在一个属兔中可能出现多次，所以每个key都需要一个管家Dep来管理多个watcher
5. 将来data中数据一旦发生变化，会首先找到对应的Dep，通知所有的Watcher执行跟新函数

### 设计类型介绍
- kVue:框架构造函数
- Observer:执行数据响应化（分辨数据是对象还是数组）
- compile: 编译模板，初始化视图，收集依赖，（跟新函数，watcher创建）
- Watcher: 执行跟新函数（跟新Dom）
- Dep: 管理多个watcher,批量更新
```js
//1.创建kVUe的构造函数
class KVue{
  constructor(options){
    //保存选项
    this.$options = options;
    this.$data = options.data;
    //响应化处理
    Observe(this, '$data')
    //代理
    proxy(this,'$data')
    //创建编译器
    new Compiler(options.el, this)
  }
}
```





