---
title: TypeScript
date: 2021-04-22 15:40:24
author: zhengzheng
cover: true
categories: 前端
tags:
  - 前端
  - TypeScript
---


## 调试环境
- 安装ts
```bash
npm i typescript -g
```
- 配置文件
```bash
tsc --init
```
- 生成package.json
```bash
npm init
```
- 工程化
  - 安装相关工具：webpack,webpack-cli,webpack-dev-server 
  ```js
  npm i webpack webpack-cli webpzck-dev-server ts-loader typescript html-webpack-plugin
  ```


编译ts 
```bash
tsc ./index.ts
```

## ts+vue项目应用
## UI库等第三方库引入
- 声明文件
```js
//常用库都是有声明文件的，如果没有就手动安装
//安装地址形式通常是@types/XXX
//这里是：npm i -D @types/XXX

//假设lodash:npm i -D @types/lodash
```
```js
//以moment为例
import moment from 'moment'

@Component
export default class App extends Vue{
  msg = moment().format('YYYY-MM-DD')
}
```
---------------------------------------------------------

## 知识点：
- ts核心语法
- ts+vue
- 装饰器原理
- 源码

### 准备工作
#### 新建一个基于ts的vue项目
![](ts+Vue.png)
#### 在已存在项目中安装typeScript
```bash
vue add @vue/typescript
```
> 请暂时忽略引发的几处Error,他们不会影响项目运行，我们将在后面处理他们

装完以后，在项目中会多出4个文件：main.ts,shims-tsx.d.ts,shims-vue.d.ts,tsconfig.json

#### TS特点
- 类型注解、类型检测
- 类
- 接口
- 泛型
- 装饰器
- 类型声明
```js
//类型注解
let var1: string;
var1 = 'typeScript'
// var1 = 1

//类型推论
let var2 = true

//原始类型：String, number,boolean,undefined,null,symbol
let var3: String | undefined;
//类型数组

let arr: string[];
arr = ['tom']

//任意类型
let varAny: anny;
varAny = 'XXX'
varAny = 3

//any用于数组
let arrAny: arr[];
arrAny = [1, true, 'free']
arrAny[1] = 100

//函数类型约束
function greet(person: string): string{
  return "hello "+ person
}
const msg = greet('tom')

// void类型
function warn():void{

}

// 对象object,不是原始类型的就是对象类型
function fn1(o: object){}
fn1({prop:0})
// fn1(1) //no ok

// 正确的姿势
function fn2(o: {prop: number}){
  o.prop
}

fn2({prop:0})
// fn2({prop:'tom'}) // no ok

// 类型别名 type, 自定义类型
type Prop = {prop: number} & {foo:String}
function fn3(o: Prop){// 等同于fn2

}
//type和接口interface的区别，基本完全相同
interface Prop2 {
  prop: number
}

// 类型断言
const someValue: any = 'this is a string'
const strLen = (someValue as string).length

// 联合类型
let union: string | number;
union = '1'
union = 1

// 交叉类型
type First = {first:number}
type Second = {second:number}
// 扩展新的type
type FirstAndSecond = First & Second;
function fn4(): FirstAndSecond {
  return {first:1,second:2}
}

//函数
// 1.设置了就是必填参数
// 2.默认值msg = 'abc'
// 3.可选参数？
function greeting(person: string,  msg1 = 'abc', msg2?:string): string{
  return ''
}
greeting('tom')

// 函数重载：场景主要源码和框架，函数使用参数个数、类型或者返回值类型区分同名函数
// 先声明，在实现
//同名声明有多个
function watch(cb1: () => void): void;
function watch(cb1: () => void, cb2: (v1:any, v2:any) => void):void;

//实现
function watch(cb1:() => void, cb2?: (v1:any, v2:any) => void){
  if(cb1 && cb2){
    console.log('执行重载2')
  }else {
    console.log('执行重载1')
  }
}
watch()

// 类
class Parent {
  private _foo = "foo";// 私有属性， 不能在类的外部访问
  protected bar = 'bar'; // 保护属性，可以在子类中访问

  // 参数属性： 构造函数参数加修饰符，能够定义为成员属性
  contructor(public tua = "tua") {}

  // 方法也有修饰符
  private someMethod() {}

  // 存储器：属性方式访问，可添加额外逻辑，控制读写性
  // 存储器可用于计算属性

  get foo() {
    return this._foo;
  }

  set foo(val) {
    this._foo = val 
  }

  

}
const p = new Parent()
  可以访问： p.foo和p.tua
class child extends Parent {
    say() {
      this.bar
    }
}
const c = new Child()



```
## 接口

> 接口仅约束结构，不要求实现，使用更简单

```js
interface Person {
  firstName: string;
  lastName: string;
}
// greeting函数通过Person接口约束参数结构
function greeting(person: Person ) {
  return 'Hello, ' + person.firstName + '' + person.lastName;
}
greeting({firstName: 'Jane', lastName: 'User'}); // 正确
greeting({firstName: 'Jane'}) //错误
```

## 泛型
> 泛型(Generics)是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。以此增加代码通用性

```js
// 不用泛型
interface Result {
  ok: 0 | 1;
  data: Feature[];
}
// 使用泛型
interface Result<T> { 
  ok: 0 | 1;
  data: T;
}
// 泛型方法
function getResult<T>(data: T): Result<T> {
  return {ok:1, data};
}

// 用尖括号方式指定T为String
getResult<string>('hello')
// 用类型推断指定T为number
getResult(1)
```

#### 泛型优点
- 函数和类可以支持多种类型，更加通用
- 不必编写多条重载，冗长联合类型，可读性好
- 灵活控制类型约束

> 不仅通用且能灵活控制，泛型被广泛用于通用库 的编写

范例： 用axios获取数据

安装axios: npm i axios -S

配置一个模拟接口，vue.config.js

```js
module.exports = {
  devServer: {
    before(app) {
      app.get('/api/list',(req,res) => {
        res.json([
          { id: 1, name: '类型注解', version: '2.0'},
          { id: 2, name: '编译型语言', version: '1.0'}
        ])
      })
    }
  }
}
```

创建服务，api/feature.ts

```js
import axios from 'axios'
import Feature from '@/models/feature'
export function getFeatures() {
  // 通过泛型约束返回值类型，这里时Promisr<AxiosResponse<Feature[]>>
  return axios.get<Feature[]>('/api/list')
}
```

使用接口，Hello.vue

```js
created() {
  //getFeatures()返回Promise<AxiosResponse<Feature[]>>
  //res类型推断为AxiosResponse<Feature[]>
  // res.data类型推断为Feature[]
  getFeatures().then(res => {
    this.features = res.data
  })
}
```
## 声明文件
> 使用ts开发时如果要使用第三方js库的同时还想利用ts诸如类型检查等特性就需要声明文件，类似xx.d.ts 同时，vue项目中还可以在shims-vue.d.ts中编写声明，从而扩展模块，这个特性叫模块补充

```js
import Vue from "vue";
import { AxiosInstance } from "axios";
import VueRouter from "vur-router";
import { Store } from "vuex";
//声明后缀.vue文件处理
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}

// 模块扩展vue
declare module "vue/types/vue" {
  interfacce Vue {
    $axios: AxiosInstance;
  }
}

// 扩展ComponentOptions
declare module "vue/types/options" {
  interface ComponentOptions<V extends Vue> {
    router?: VueRouter;
    store?: Store<any>;
  }
}
```

## 装饰器
> 装饰器用于扩展类或者它的属性和方法。@XXX就是装饰器的写法

### 属性声明： @Prop

除了在@Component中声明，还可以采用@Prop的方式声明组件属性

```js
export default class Helloworld extends Vue {
  //Prop()参数是为vue提供属性选项
  //！称为明确赋值断言，它是提供给ts的
  @Prop({type: String, require: true})
  private mag!: string;
}
```

### 事件处理：@Emit 

新增特性时派发事件通知，Hello.vue

```js
//通知父类新增事件，若未指定事件名则函数名作为事件名(羊肉串形式)
@Emit()
private addFeature(event: any) {// 若没有返回值形参将作为事件参数
  const feature = { name: event.target.value, id: this.feature.length + 1};
  this.features.push(feature)
  event.target.value = "";
  return feature;// 若有返回值则返回值作为事件参数
}
```

### 变更检测： @Watch
```js
@Watch('msg')
onMsgChange(val:string, oldVal:any){
  console.log(val, oldVal);
}
```
### vuex推荐使用：vuex-class

vuex-class为vue-class-component提供vuex状态绑定帮助方法

安装
```bash
npm i vuex-class -s
```
使用，Hello.vue
```js
<h3 @click="add"> {{counter}} </h3>
<h3 @click="asycAdd"> {{counter}} </h3>
```

```js
import { Action, State } from "vuex-class";
export default class Hello extends Vue {
  @State counter!: number;
  // add既是type, 类型时函数且无返回值
  @Mutation add!: () => void;
  // add仍是type,但是会和上面重名，需要换个变量名
  // 类型是函数返回值是Promise
  @Action('add') asycAdd!: () => Promise<number>
}
```

## 装饰器原理
> 类装饰器参数只有一个就是类构造函数
> 表达式会在运行是当作函数被调用，类的构造函数作为其唯一的参数
 
```js
// 类装饰器
function log(target: Function) {
  // target是构造函数
  console.log(target === Foo); //true
  target.prototype.log = function() {
    console.log(this.bar)
  }

}
// 方法装饰器有三个参数: 1-实例， 2-方法名
function dong(target: any, name: string, descriptor: any) {
  console.log(target,name,descriptor);
  //这里通过修改descriptor.value扩展了bar方法
  const baz = descriptor.value;
  descriptor.value = function(val: string) {
    console.log('dong~~')
    //原始逻辑
    baz.call(this, val)
  }
}
//属性装饰器 如果包一层，可以传递配置对象进来，更加灵活
function mua(option:string){
  // 返回才是装饰器函数
  // 接受两个参数： 1-实例， 2-属性名称
  return function (target, name){
    target[name] = option
  }
}

@log
class Foo {
  bar = "bar";
  @mua('mua~~')
  ns!:string;
  @dong
  setBar(val: string){
    this.bar = val
  }
}
const foo = new Foo();
// @ts-ignore
foo.log();
foo.setBar('abc')
```






