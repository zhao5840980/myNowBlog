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

- 步骤一:使用vue-router插件（1.应用插件）
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
