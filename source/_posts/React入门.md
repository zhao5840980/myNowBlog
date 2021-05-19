---
title: React入门
date: 2021-05-07 10:23:57
author: zhengzheng
cover: true
tags:
  - 前端
  - react
---
  
## React入门

> 用于构建用户界面的javaScript库

1. 掌握create-react-app使用

### 起步
  - 1. 创建项目：```js npx create-react-app my-app```
  - 2.打开项目： ```js cd my-app```
  - 3.启动项目： ```js npm start```
  - 4.暴露配置项： ```js npm run eject```

在index.js中

```js
//React负责逻辑控制，数据-> VDOM
import React from 'react';
//ReactDom渲染实际DOM, VDOM -> DOM
import ReactDOM from 'react-dom';

```

### JSX语法
```js
// index.js
import React from "react";
import ReactDOM from "react-dom"
import "./index.css"  
import logo from './logo.png'
import styles from "./index.module.css"

const name = "React";
const obj = {
  firstName: "Marry",
  lastName: "Potter"
}
function formatName(name) {
  return name.firstName + " "+ name.lastName;
}
const greet = <div>good</div>
const show = false
const a = [0, 1, 2];
const jsx = {
  <div className={styles.app}>
    <div>hello,{name}</div>
    <div>{formatName(obj)}</div>
    {greet}
    {show? greet : '登录'}
    {show && greet}
    <ul>
      {/* diff时候，首先比较type。然后是key.所以同级同类型元素，key值必须得 唯一  */}
      {a.map(item => (<li key={item}>{item}</li>))}
    </ul>
    <img src={logo} className={styles.logo} style={{ width: "50px", height: "30px"}}> 
  </div>
}


ReactDOM.render(jsx,document.getElementById("root"))

//基本使用， 表达式用{}包裹
//函数
//JSX对象
//条件语句
//数组
//属性
//模块化
```
```js
.app {
  .logo {
    width: 100px;
  }
}
```
### 组件
- class组件
- function组件

#### 组件
> 组件，从概念上类似JavaScript函数。它接受任意的入参（即“props”）,并返回用于描述页面展示内容的React 元素

组件有两种形式： class组件和function组件
#### class组件

> class组件通常拥有状态和生命周期，继承于component,实现render方法，用class组件创建一个Clock

```js
import React, { Component } from "react"

export default class ClassConponent extends Component {
  constructor(props){
    super(props);
    //存储状态
    this.state = {
      date: new Date()
    }
  }
  //组件挂载完成之后执行
  componentDidMount(){
    this.timer = setInterval(() => {
      //跟新state,不能用this.state
      this.setState({
        date: new Date()
      });
    }, 1000)
  }
  //组件卸载之前执行
  componentWillUnmount() {
    clearInterval(this.timer)
  }
  render() {
    const {date} =  this.state;
    return (
      <div>
        <h3>ClassConponent</h3>
        <p>{date.toLocaleTimeString()}</p>
      </div>
    )
  }
}
```
```js
//app.js
import React from "react"
import ClassConponent from "./pages/ClassConponent"; 
function App() {
  return (
    <div className="App">
      <ClassComponent />
      <FunctionComponent />
    </div>
  )
}
``` 
```js
//Function Component
import React, { useState, useEffect } from "react"
export function FunctionComponent(props){
  const [date, setDate] = useState(new Date());

  useEffect(() => {
    //相当于componentDidMount, componentDidUpdate, componentWillUnmount的集合
    const timer = setInterval(() => {
      setDate(new Date());
    }, 1000);
    return () => clearInterval(timer)
  }, []);
  return (
    <div>
      <h3>FUnctionComponent</h3>
      <p>{date.toLocaleTimeString()}</p>
    </div>
  )
}
```

### 正确使用setState

#### 关于setState()你应该了解三件事
- 不要直接修改State
- State的更新可能是异步的
- State的更新会被合并
#### 正确使用setState
```js
setState(partialState, callback)
1. partialState:object|function

用于产生与当前state合并的子集

2. callback:function

state更新之后被调用
```

#### 关于setState()你应该了解三件事：

##### 不要直接修改State

例如，此代码不会重新渲染组件：
```js
//错误示范
this.state.commrnt = 'Hello'

而是应该使用setState():

//正确使用
this.setState({comment: 'Hello'});
```

##### State的更新可能是异步的
> 处于性能考虑，React可能会把多个setState()调用合并成一个调用。观察以下例子中log的值和button显示的counter

```js
//SetStatePage.js
import React, { Component } from "react"
export default class SetStatePage extends component {
  constructor(props){
    super(props);
    this.state = {
      counter: 0
    };
  }
  componentDidMount() {
    this.changeValue(1)
    document.getElementById('test').addEventListener('click', this.setCounter);
  }
  changeValue = v => {
    //setState在合成事件和生命周期中是异步的，这里说的异步其实是批量更新,达到了优化性能的目的
    this.setState({
      counter: this.state.counter + V
    }, () => {
      //callback就是在state更新完成之后再执行
          console.log('counter', this.state.counter)
    });

    //避免setState不合并
    this.setState(state => {
      return {
        counter: state.counter + v
      }
    })

  };
  setCounter = () => {
    //setState在setTimeout和原生事件中是同步的
    setTimeout(() => {
      this.changeValue(1)
    }, 0)
    this.changeValue(1);
    this.changeValue(2);
    this.changeValue(3);
    
  }
  render() {
    return (
      <div>
        <h3>SetStatePage</h3>
        <button onClick={this.setCounter}>{this.state.counter}</button>
        <button id="test">原生事件*{this.state.counter}</button>
      </div>
    )
  }
}
```
> 总结： setState只有在合成事件和生命周期函数中是异步的，在原生事件和setTimeout中都是同步的，这里的异步其实是批量更新

##### State的跟新会被合并

## 生命周期

### 知识要点
- 生命周期方法
- ReactV16.3之前的生命周期
  - 新引入的两个生命周期函数
       - getDerivedStateFromProps
       - getSnapshotBeforeUpdate
  - 验证生命周期

### 组件的生命周期
#### 生命周期方法

> 生命周期方法，用于在组件不同阶段执行自定义功能。在组件被创建并插入到DOM时（即挂载中阶段（mounting））,组件更新时，组件取消挂载或从DOM中删除时，都有可以使用的生命周期方法

![](react生命周期.png)

```js
import React, { Component } from 'react'
imoprt PropTypes from 'prop-types';
export default class LifeCyclePage extends Component {
  static defaulrProps = {
    msg: 'omg'
  }
  static propTypes = {
    msg: PropTypes.string.isRequired
  };
  constructor(props){
    super(props)
    this.state = { count: 0} 
    console.log('constructor')
  }
  componentWillMount() {
    console.log("componentWillMount")
  }
  componetnDidMount() {
    console.log("componetnDidMount")
  }
  shouldComponentUpdate(nextProps,nextState) {
    const { count } = nextState
    console.log("componentDidMount", nextState, this.state)

    // return true
    return count !== 3;
  }
  componentWillUpdate(){
    console.log("componentWillUpdate")
  }
  componentDidUpdate(){
    console.log("componentDidUpdate")
  }
  setCount = () => {
    this.setState({count: this.state.count + 1 })
  }

  render() {
    console.log('render', this.props)
    const {count} = this.state
    return (
      <div>
        <h3>LiftCyclePage</h3>
        <p>{count}</p>
        <button onClick={this.setCount}>改变count</button>
        {count % 2 && <Child count={count}>}
        <Child count={count}>
      </div>
    )
  }
}

class Child extends Component {
  // 初次渲染的时候不会执行，只有在已经挂载的组件接收新的props的时候，才会执行
  componentWillReceiveProps() {
    console.log("componentWillReceiveProps ")
  }
  componentWillUnmount() {
    console.log("ComponentWillUnmount")
  }
}
render() {
  console.log("Child render")
  const {count} = this.props;
  return (
    <div>
      <h3>Child</h3>
      <p>{count}</p>
    </div>
  )
}
```
![](React17生命周期.png) 

引入两个新的生命周期：
- static getDerivedStateFormProps
- getSnapshotBeforeUpdate

如果不想手动给将要废弃的生命周期添加UNSAFE_前缀，可以用下面的命令

```js
npx react-codemod rename-unsafe-lifecycles <path>
```

### 新引入的两个生命周期函数
#### getDerivedStateFromProps
```js
static getDerivedStateFromProps(props, state)
```

getDerivedStateFromProps会在调用render方法之前调用，并且在初始挂载及后续更新时都会被调用，它应返回一个对象来更新state，如果返回null则不更新任何内容。

请注意，不管原因是什么，都会在每次渲染钱触发此方法，这与UNSAFE_componentWillReceiveProps形成对比，后者仅在父组件重新渲染时触发，而不是在内部调用setState时

```js
static getDerivedStateFromProps(props, state){
  console.log("getDerivedStateFromProps");
  const {count} = state
  return count > 5? {count:0} : null;
}
```
#### getSnapshotBeforeUpdate

```js
getSnapshotBeforeUpdate(prevProps, prevState)
```

在render之后，在componentDidUpdate之前

getSnapshptBeforeUpdate()在最近一次渲染输出（提交到DOM节点）之前调用，它使得组件能在发生更改之前从DOM中获取一些信息（例如，滚动位置）。此生命周期的任何返回值将作为参数传递给componentDidUpdate(prevProps,prevState,snapshot)

```js
getSnapshotBeforeUpdate(prevProps, prevState){
  console.log("getSnapshotBeforeUpdate", prevProps, prevState)
  return {
    msg: 'getSnapshotBeforeUpdate'
  }
}
componentDidUpdate(prevProps, prevState, snapshot){
  console.log("componentDidUpdate", prevProps, prevState, snapshot)
}
```
## 组件复合
```js
import React, { Component } from "React";
// import TopBar from "../components/TopBar";
// import BottomBar from '../components/BottomBar';
import Layout from "./Layout"
export default class UserPage extends Component {
  render() {
    return (
      <Layout showTopBar={false} showBottomBar={true} title="商城首页">
        <div>
          // <TopBar />
          // <h3>HomePage</h3>
          // <BottomBar>

        </div>
      </Layout>
    )
  }
}
```

```js
// layout.js
import React, { Component } from "react";
export default class Layout extends Component {
  componentDidMount() {
    const { title = "商城" } = this.props;
    document.title = title;
  }
  render() {
    const {children, showTopBar, showBottomBar } = this.rpops
    console.log('children',children)
    return (
      <div>
        <showTopBar && TopBar />
        // <h3>layout</h3>
        {this.props.children}
        <showBottomBar && BottomBar />
      </div>
    )
  }
}

```

模仿vue实现具名插槽
```js
//App.js
import React from "react";

export default function App() {
  return (
    <div>
      <HomePage />
    </div>
  )
}
```

```js
//HomePage.js
import React, { Component } from "react";
import Layout from "./Layout";

export default class HomePage extends Component {
  render() {
    return (
      <Layout showTopBar={false} showBottomBar={true} title="商城首页">
        {
          {
            content: (
              <div>
                <h3>HomePage</h3>
              </div>
            ),
            txt: "这是文本"，
            btnClick: () => {
              console.log("btnClick")
            }
          }
        }
      </Layout>

    )
  }
}
```
```js
//Layout.js
import React, { Component } from "react";
import TopBar from "../component/TopBar"
import BottomBar from "../components/BottomBar"

export default Layout extends Component {
  componentDidMount() {
    const { title = "商城"} = this.props;
    document.title = title
  }
  render() {
    const { children, showTopBar, showBottomBar } = this.props;
    console.log('children', children)
    return (
      <div>
        {showTopBar && <TopBar/>}
        {children.content}
        {dhildren.txt}
        <button onClick={children.btnClick}>button</button>
        {showBottomBar && <BottomBar />}
      </div>
    )
  }
}
```

## redux
- 1.理解我们什么时候才会需要redux
- 2.掌握redux基本使用

### 你可能不需要redux
> Redux是负责组织state的工具，但你也要考虑它是否适合你的情况，不要应为有人告诉你要用Redux就去用，花点时间好好想象使用了Redux会带来的好处或坏处

在下面的场景中，引入Redux是比较明智的：
- 你有着相当大量的、随时间变化的数据
- 你的state需要有一个单一可靠数据来源；
- 你觉的把所有state放在最顶层组件中已经无法满足需要了
- 某个组件的状态需要共享 

### redux
> redux是javaScript应用的状态容器，提供可预测化的状态管理，它保证程序行为一致性且易于测试
![](redux.png)

#### 安装redux
```js
npm install redux --save
```

#### redux上手

用一个累加器举例

1. 需要一个store来存储数据
2. store里的reducer初始化state并定义state修改规则
3. 通过dispatch一个action来提交对数据的修改
4. action 提交到reducer函数里，根据传入的action的type,返回新的state

```js
// 创建store, src/store/ReduxStore.js
import {createStore} from 'redux'

//定义state初始化和修改规则 ,reducer是一个纯函数
const counterReducer = (state = 0, action) => {
  console.log("state", state)
  switch (action.type ) { 
    case "ADD":
      return state + 1;
    case "MINUS":
      return state -1;
    default:
      return state;
  }
}
const store =createStore();
export default store; 
```

```js
//ReduxPage.js
import React, { Component } from "react"
import store from "../store/"

export default class ReduxPage extend Component {
  componentDidMount() {
    store.subscribe(() => {
      console.log("state发生变化了")
      this.forceUpdate()
    })
  }
  render() {
    console.log("store", store)
    return (
      <div>
        <h3>ReduxPage</h3>
        <p>{store.getState()}</p>
        <button onClick={() => store.dispatch({ type: "ADD" })}>add</button>
      </div>
    )
  }
}
```






  


