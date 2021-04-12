---
title: Vue组件化实践
date: 2021-04-09 09:07:07
author: zhengzheng
cover: true
categories: 前端
tags:
   - 前端
---

## 组件通讯常用方式

- prop
- eventbus(事件总线)
- VueX
-自定义事件
  - 边界情况
      - $parent
      - $children
      - $root
      - $refs
      - provide/inject
  - 非prop特性  
    - $attrs
    - $listeners

## 组件通讯

1. props

   父给子传值
  ```js
  //child
  prop: {msg:String}
  //parent
  <HelloWorld msg="Welcome to your vue.js App">
  ```
  2. 自定义事件

     子给父传值

  ```js
    //child
    this.$emit('add', good)
    //parent
    <Cart @add="CartAdd($event)"></Cart>
  ```
  3. 事件总线
  > 任意两个组件之间传值常用事件总线或VueX的方式
  ```js
  class Bus {
    constructor() {
      this.callback = {}
    }
    $on(name, fn) {
      this.callback[name] = this.callback[name] : []
      this.callback[name].push(fn)
    }
    $emit(name, args) {
      if(this.callback[name]){
        this.callback[name].forEach(cb => cb(args))
      }
    }
  }

  //main.js
  Vue.prototype.$bus = new Bus()//在Vue原型上添加新的属性$bus
  // child2
  this.$bus.$emit('foo')

  ```
  > 实际项目中通常用Vue代替Bus,因为Vue已经实现了相应接口
  4. VueX
  > 创建唯一的全局数据管理者store,通过它管理数据并通知组件状态变更
  5. $parent/$root
  > 兄弟组件之间通讯可通过共同祖辈搭桥，$parent/$root
   ```js
   //brother1 -> 发布订阅模式（事件的派发者和监听者都是一个）
   this.$parent.$on('foo', handle)
   //brother2
   this.$parent.$emit('foo')
   ```
   6. $children
   > 父组件可以通过$children访问子组件实现父子通讯
```js
//parent
this.$children[0].xx = 'xxx'
//注意： $children不能保证子元素顺序
```
7. $attrs/$listeners 
> 使用场景：父组件引用的子组件传参，子组件没有设置props,可以在子组件中使用$attrs获取对应的值，包含了父作用域中不作为prop被识别（且获取）的特性绑定（class和style除外），当一个组件没有声明任何prop时，这里会包含所有父作用域的绑定（class和style除外），并且可以通过v-bind="$attrs" 传入内部组件 --在创建高级别的组件时非常有用

  >  $listeners会被展开并监听

  > $listeners:应用场景:父组件中引入的子组件绑定了事件方法，在父组件执行对应的事件方法，子组件调用了父组件的回调方法，可以用在子组件v-on="$listeners"

```js
//children: 并未在props中声明foo
<p>{{ $attrs.foo }}</p>
//parent
<HelloWorld foo = "foo"/>

// refs -> 获取子节点引用  
//parent
<HelloWorld ref = 'hw'>
mounted(){
  this.$refs.hw.xx = "XXX"
}
```
8. provide(提供)/inject（注入，适合通用组件库开发）-> 能够实现祖先和后代之间传值（只限于UI库的开发使用）
> 使用场景：老祖宗的组件想给子孙组件传值，依赖注入

```js
//ancestor
provide() {
  return {foo: 'foo'}
}
//descendant
inject: ['foo']
    父：provide() {
      return {
        foo: 'f000000'
        comp: this //当前组件的实例
      }
    }
    子：inject: ['foo'] | inject: {bar: {from}}
// 如果provide儿子也声明了，孙子组件也一样会报错？解决方案： 起别名
```
## 2. 插槽

> 插槽语法是Vue实现的内容分发API,用于复合组件开发，该技术在通用组件库开发中有大量应用

1. 匿名插槽（相当于一个坑位）
```js
//comp1
<div>
  <slot></slot>
</div>
//parent
<comp>hello</comp>
```
2. 具名插槽
> 将内容分发到子组件指定位置
```js
//conp2
<div>
  <slot></slot>
  <slot name = "content"></slot>
</div>
//parent
<comp2>
  <!--默认插槽default做参数-->
  <template v-slot: default>具名插槽</template>
  <!--具名插槽用插槽名做参数-->
  <template v-slot:content>内容。。。。</template>
</comp2>
```
3. 作用域插槽：
> 应用场景： 分发内容要用到子组件中的数据，

> 在父组件引用中，作用域插槽会给带有子组件的那个插槽，传一个上下文对象包含对应的数据

```js
//com3
<div>
  <slot :foo = "foo"></slot>
</div>
//parent
<com3>
  <!--把v-slot的值指定为作用域上下文对象-->
  <template>
    来自子组件数据：{{slotProps.foo}}
  </template>
</com3>
```
> 什么时候使用具名插槽？作用域插槽？

> 在声明的template数据中插槽的数据是来自腹肌组件还是子组件，如果来自子组件，则采用作用域插槽，反之使用其他类型的插槽




