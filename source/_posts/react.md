---
title: react
date: 2021-05-28 10:41:55
author: zhengzheng
cover: true
tags:
  - 前端
  - react
---

## React组件化

### 快速开始
```js
npx create-react-app lesson1
cd lesson1
npm start 
```

### 组件优化点
1. 增强代码重用性，提高开发效率
2. 简化调试步骤，提升整个项目的可维护性
3.便于协同开发
4.注意点：降低耦合性

### 高阶组件-HOC
> 为了提高组件复用率，可测试性，就要保证组件功能单一性；但是若要满足复杂需求就要扩展功能单一的组件，在React里就有HOC(Higher-OrderComponents)的概念

> 定义： 高阶组件时参数为组件，返回值为新组件的函数

```js
//HocPage.js
import React, {Component} from "react";

//HOC: 是个函数，接收一个组件，返回一个新的组件
function Child(props) {
  return <div>Child</div>
}

// Cmp这里是function或者class组件 
// const foo = Cmp => props => {
//   return (
//     <div className="border">
//       <Cmp {...props}/>
//     </div>
//   )
// };
const foo = Cmp => {
  return props => {
    return (
      <div className = "border">
        <Cmp {...props} />
      </div>
    )
  }
}
const foo2 = Cmp => props => {
  return (
    <div  className = "greenBorder">
      <Cmp {...props} />
    </div>
  )
}

// const Foo = foo(Child)
const Foo = foo2(foo(foo(Child))); // 链式调用
export default class HocPage extends Component {
  return() {
    return (
      <div>
        <h3>HocPage </h3>
        <Foo />
      </div>
    )
  }
}
```
```js
import React from "react"
import HocPage from "./pages/HocPage"

function App() {
  return (
    <div className="App">
      {/*高阶组件*/}
      <HocPage />
    </div>
  )
}
```
#### 装饰器写法
> 高阶组件本身是对装饰器模式的应用，自然可以利用es7中出现的装饰器语法来更优雅的书写代码

```js
npm install -D @babel/plugin-proposal-decorators

```

更新config-overrides.js
```js
//配置完成后记得重启下
const { addDecoratorsLegacy } = require('customize-cra')

module.exports = override(
  ...,
  addDecoratorsLegacy()//配置装饰器
)
```

如果vscode对装饰器有warning,vscode设置里加上
```js
javascript.imlicitProjectConfig.experimentalDecorators: true
``` 

```js
//HocPage.js
import React, {Component} from "react";

//HOC: 是个函数，接收一个组件，返回一个新的组件
// function Child(props) {
//   return <div>Child</div>
// }
@foo
class Child extends Component {
  render() {
    return <div>Child</div>
  }
}

// Cmp这里是function或者class组件 
// const foo = Cmp => props => {
//   return (
//     <div className="border">
//       <Cmp {...props}/>
//     </div>
//   )
// };
const foo = Cmp => {
  return props => {
    return (
      <div className = "border">
        <Cmp {...props} />
      </div>
    )
  }
}
const foo2 = Cmp => props => {
  return (
    <div  className = "greenBorder">
      <Cmp {...props} />
    </div>
  )
}

// const Foo = foo(Child)
// const Foo = foo2(foo(foo(Child))); // 链式调用
export default class HocPage extends Component {
  return() {
    return (
      <div>
        <h3>HocPage </h3>
        <Child />
      </div>
    )
  }
}


```

组件是将props转换为UI,而高阶组件是将组件转换为另一个组件。

HOC是React的第三方库中很常见。例如Redux的connect

#### 使用HOC的注意事项

高阶组件（HOC）是React中用于复用组件逻辑的一种高级技巧。HOC自身不是React API的一部分，它是一种基于React的组合特征而形成的设计模式

- ##### 不要在render方法中使用HOC

React的diff算法(称为协调) 使用组件表示来确认它是应该更新现有子树还是将其丢弃并挂载新子树，如果从render返回的组件与前一个渲染中的组件相同（===）,则React通过将子树与新子树进行区分来递归更新子树，如果他们不相等，则完全卸载前一个子树

这不仅仅是性能问题，重新挂载组件会导致改组件及其所有子组件的状态丢失

##### 表单设计

```js
import React from "react";
import HocPage from "./pages/HocPage";
import FormPage from "./pages/FormPage";
import FormPage2 from "./pages/FormPage";

function App() {
  return (
    <div className="App">
      {/* 表单组件 */}
      <FormPage />

      {/* 表单组件 使用create */}
      <FormPage2 />
    </div>
  )
}

export default App
```
```js
import React, {Component} from "react";
import {Form, Input, Icon, Button} from "antd"

export default class FormPage extends Component {
  constructor(props) {
    super(props);
    this.state = {
      name: '',
      password: ''
    }

  }
  submit = () => {
    console.log("submit",this.state)
  }
  render() {
    const {name, password} = this.state;
    return (
      <div>
        <h3>FormPage</h3>
        <Form>
          <Form.Item>
            <Input placeholder="please input ur name" value={name} onChange={e => this.setState({
              name: e.target.value
            })}/>
          </Form.Item>
          <Form.Item>
            <Input type="password" placeholder="please input ur password" value={password} onChange={e => this.setState({
              password:
            })}/>
          </Form.Item>
          <Button type="primary" onClick={this.submit}>提交</Button>
        </Form>
      </div>

    )
  }
}
```
```js
// FormPage2.js

import React, {Component} from "react"
import {Form, Input, Icon, Button} from "antd";

const nameRules = {required: true, message: 'please input ur name'}
const passwordRules = {required: true, message: 'please input ur password'}

@Form.create({})
export default class FormPage2 extends Component {
  submit = () => {
    console.log("submit");
    const {getFieldsValue, getFieldValue, validateFields} = this.props.form;
    validateFields((err, values) => {
      if(err) {
        console.log('err', err)
      }else {
        console.log('success', values)
      }
    })
    console.log("submit", getFieldsValue(), getFieldValue("name"))
  }
  render(){
    // const {name, password} = this.state;
    console.log("props", this.props)
    const {getFieldDecorator} = this.props.form;
    return (
      <div>
        <h3>FormPage</h3>
        <Form>
          <Form.Item>
            {getFieldDecorator("name", {rules: [nameRules]})(
              <Input placeholder="please input ur name"/>
            )}
            
          </Form.Item>
          <Form.Item>
          {getFieldDecorator("password", {rules: [passwordRules]})(
            <Input type="password" placeholder="please input ur password"/>
          )}
            
          </Form.Item>
          <Button type="primary" onClick={this.submit}>提交</Button>
        </Form>
      </div>
    )
  }
  export default FormPage2
}

```

#### 实现表单

```js
// MyFormPage.js
import React, {Component} from "react"
import kFormCreate from "../components/kFormCreate";

const nameRules = {required: true, message: 'please input ur name'}
const passwordRules = {required: true, message: 'please input ur password'}
@KFormCreate
class MyFormPage extends Component {
  submit = () => {
    const {getFieldsValue, getFieldValue, validateFields} = this.props;


    console.log('submit', getFieldsValue(), getFieldValue('password'))
  };
  render() {
    console.log('props', this.props)
    const {getFieldDecorator} = this.props;
    return (
      <div>
        <h3>MyFormPage</h3>
        {getFieldDecorator(
          "name",
          {rules: [nameRules]}
        )(<input type="text" placeholder="please input ur name"/>)}
        {getFieldDecorator(
          "password",
          {rules: [passwordRules]}
        )(<input type="password" placeholder="please input ur password"/>}
       
      
        <button onClick={this.submit}>提交</button>



      </div>
    )
  }

}

```
```js
// KFormCreate.js
import React, {Component} from "react"

export default function lFormCreate(Cmp){
  return class extends Component {
    constructor(props) {
      super(props)
      this.state = {}；
      this.option = {}
    }
    handleChange=(e) => {
      // setState name value
      let {name, value} = e.target;
      this.setState({[name]: value}); 

    }
    getFieldDecorator = (filed, option) => {
      this.options[field] = option
        return InputCmp => {
          //克隆一份
          return React.cloneElement(InputCmp, {
            name: field,
            value: this.state[filed] || "",
            onChange: this.handleChange
          })
        }
      }
      getFieldsValue=()=> {
        return {...this.state}
      }
      getFieldValue = field => {
        return this.state[field];
      }
      validateFields = callback => {
        // 校验错误信息
        const errors = {};
        const state = {...this.state}
        for (let name in this.options) {
          if (state[name] === undefined) {
            // 没有输入， 判断为不合法
            errors[name] = "error"
          }
        }
        if (JSON.stringify(errors) === "{}") {
          // 合法
          callback(undefined, state);
        } else {
          callback(errors, state)
        }
      }
    render() {
      return (
        <div className="border">
          <Cmp 
            getFieldDecorator = {this.getFieldDecorator}
            getFieldsValue={this.getFieldsValue}
            getFieldValue={this.getFieldValue}
            validateFields={this.validateFields}
            />
        </div>
      )
    }
  }
}
```

```js
import React, {Component} from "react"
import Dialog from "../components/Dialog";

export default class DialogPage extends Component {
  constructor(props) {
    super(props);
    this.state = {
      showDialog: false
    }
  }
  render() {
    const {showDialog} = this.state;
    return (
      <div>
        <h3>DialogPage</h3>
        <button onClick={() => {
          this.setState({showDialog: !showDialog})
        }}>toggle</button>
        {showDialog && <DIalog>
          <p>我是一段文本</p>
        <Dialog />}
      </div>
    )
  }
}
```

```js
// Dialog.js
import React, {Component} from "react"
import {createPortal} from "react-dom"

export default class Dialog extends Component {
  constructor(props) {
    super(props)
    const doc = window.document; 
    this.node = doc.createElement("div")
    doc.body.appendChild(this.node)
  }
  componentWillUnmount() {
    window.document.body.removeChild(this.node);
  }
  render() {
    return createPortal(
      <div className="dialog">
        <h3>Dialog</h3>
        {this.props.children}
      </div>,
      this.node
    )
  }
}
```
#### 具体实现：Portal
> 传送门，react16之后出现的portal可以实现内容传送功能

#### 使得react项目加载typescript进行开发
```js
Npx create-react-app --typescript
```

### React全家桶01-redux
#### Reducer
##### 什么是reducer
> reducer就是一个纯函数，接受旧的state 和 action ， 返回新的state
```js
;(previousState, action) => newState
```

之所以将这样的函数称之为reducer，是因为这种韩式与传入Ayyay.prototype.reduce(reducer, ?initialValue)里的回调函数属于相同的类型。保持reducer纯净非常重要，永远不要再reducer里做这些操作：
- 修改传入参数；
- 执行有副作用的操作，如API请求和路由跳转
- 调用非纯函数， 如Date.noe()或Math.random().

```js
const array1 = [1,2,3,4]
const reducer = (accumulator, currentValue) => accumulator + currentValue;

// 1+2+3+4
console.log(array1.reduce(reducer));
// expected output: 10

// 5+1+2+3+4
console.log(array1.reduce(reducer, 5))
// expected output: 15
```


### 组件跨层级通讯- Context
![context](./Context.png)

在一个典型的React应用中，数据是通过props属性自上而下（由父及子）进行传递的，但这种做法对于某些类型的属性而言是及其繁琐的（例如：地区偏好，UI主题），这些属性是应用程序中许多组件都需要的。Context提供了一种在组件之间共享此类值的方式，而不必显式的通过组件树的逐层传递props. 

React中使用COntext实现祖代组件向后代组件跨层级传值，Vue中的provide & inject来源于Context

#### Context API
##### React.createContext

创建一个COntext对象。当React渲染一个订阅了这个Context对象的组件，这个组件会从组件树中离自身最近的那个匹配的Provider中读取到当前的context值

```js
// App.js
import React from 'react'
import ContextPage from "./pages/ContextPage";

function App() {
  return (
    <div className="App">
      {/* context 上下文*/}
     <ContextPage/>
    </div>
  );
}
```

```js
// ContextPage.js
import React, {Component} from "react"
import ContextTypePage from "./ContextTypePage";
import ConsumerPage from "./ConsumerPage";
import MultipleContextPage from "./MultipleContextPage";
import {ThemeProvider} from "../ThemeContext"
import {UserProvider} from "../UserContext"
// 创建context 农民种菜
// const ThemeContext = React.createContext()
// // 接收者 批发商批发菜
// const ThemeProvider = THemeContext.Provider

 export default class ContextPage extends Component {
   constructor(props) {
     super(props);
     this.state = {
       theme: {
         theme: {
           themeColor: "red"
         },
         user: {
           name: 'xiaoming'
         }
       }
     };
   }
   changeColor = () => {
     const {themeColor} = this.state.theme;
     this.setState({
       theme: {
         themeColor: themeColor === "red" ? "green" : "red"
       }
     })
   }
   render() {
     const {theme, user} = this.state
     return (
       <div>
        <button onClick={this.changeColor}>change color</button>
        <h3>ContextPage</h3>
        <ThemeProvider value={theme}>
        {/* 只能订阅一个context */}
          <ContextTypePage />
          <ConsumerPage />
          <UserProvider value={user}>
            <MultipleContextPage />
          </UserProvider>
        </ThemeProvider>
          
       </div>
     )
   }
 }
```
```js
// ContextTypePage.js
import React, {Component} from "react";
import {ThemeContext} from "../ThemeContext";

 class ContextTypePage extends Component {
  // static contextType = ThemeContext;
  render() {
    console.log("this", this)
    const {themeColor} = this.context;
    return (
      <div className="border">
        <h3 className={themeColor}>
          ContextTypePage
        </h3>
      </div>
    )
  }
}
// 只能订阅一个context 并且是类组件
ContextTypePage.contextType = ThemeContext;
export default ContextTypePage;
```
```js
// themeContext.js
import React from "react"

// 创建context 农名种菜 如果没有匹配到Provider,取值默认值
export const ThemeContext = React.createContext({themeColor: "pink"});
// 接收者 批发商批发菜
export const ThemeProvider = ThemeContext.Provider;
// 消费者 吃菜
export const ThemeConsumer = ThemeContext.Consumer;
```

```js
// COnsumerPage.js
import React, {Component} from "react";
import {ThemeConsumer} from "../ThemeContext";

export default function ConsumerPage(props) {
  return (
    <div className="border">
      <h3>ConsumerPage</h3>
      <ThemeConsumer>
        {ctx => <div className={ctx.themeColor}>文本</div>}
      </ThemeConsumer>
    </div>
  )   
}
```
```js
// userContext
import React from "react"
// 创建context 农民种菜， 
export const UserContext = React.createCOntext()
// 接收者 批发商批发菜
export const UserProvider = UserContext.Provider;
// 消费者 吃菜
export const UserConsumer = UserContext.Consumer
```
```js
// MultipleContextPage.js
import React, {Component} from "react";
import {ThemeContext} from '../ThemeContext';
import {UserContext} from '../userContext';

export default class MultioleContextPage extends Component {
  render() {
    <div>
      <h3>MultipleContextPage</h3>
      <ThemeConsumer>
        {theme => (
          <UserConsumer>
            {user => <div className={theme.themeColor}>{user.name}</div>}
          </UserConsumer>
        )}
      <ThemeConsumer/>
    </div>
  }
}
```

##### 使用context步骤
- 1. 创建createContext
- 2. Provider接收value,以保证有传下去的数据
- 3. 接收Consumer或者class.contextType

##### 注意事项
> 因为context会使用参考表示（reference identity）来决定何时进行渲染，
这里可能会有一些陷阱，当provider的父组件进行重渲染时，可能会在consumers组件中触发以外的渲染。举个例子，当每一次Provider重渲染时，以下的代码会重渲染所有下面的consumers组件，因为value属性总是被赋值为新的对象：

```js
class App extends React.Component {
  render() {
    return (
      <Provider value={something: 'something'}>
        <Toolbar />
      </Provider>
    )
  }
}
```

为了防止这种情况，将value状态提升到父节点的state里：

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: {something: 'something'}
    }
  }
  render() {
    return (
      <Provider value={this.state.value}>
        <Toolbar />
      </Provider>
    )
  }
}
```
##### 总结
> 在React的官方文档中，COntext被归类为高级部分(Advanced),属于React的高级API,但官方并不建议在稳定版的App中使用Context

> 不过，这并非意味这我们不需要关注Context.事实上，很多优秀的React组件都通过Context来完成自己的功能，比如React-redux的<Provider />,就是通过Context提供一个全局态的store，路由组价react-router通过Context管理路由状态等等，在React组件开发中，如果用好Context,可以让你的组件变的强大，而且灵活

> 函数组件中可以通过useContext引入上下文，后面hooks部分介绍

### Reducer
#### 什么是reducer
##### reducer 就是一个纯函数，接收旧的state和action，返回新的state
> (previousState, action) => newState

之所以将这样的函数称之为reducer,是因为这种函数与传入Array.prototype.reduce(reducer, ?initialvalue),里的回调函数属于相同的类型，保持reducer纯净非常重要，永远不要在reducer里做这些操作

- 修改传入参数
- 执行有副作用的操作，如API请求和路由跳转；
- 调用非纯函数，如Date.now() 或Math.random().

```js
// reduce案例
const array1 = [1,2,3,4]
const reducer = (accumulator, currentValue) => accumulator + currentValue;

//1+2+3+4
console.log(array1.reduce(reducer))
// expected output: 10
// 5+1+2+3+4
console.log(array1.reduce(reducer, 5 ))

```

## Redux上手

> Redux是JavaScript应用的状态容器，它保证程序行为一致性且易于测试

![redux](./redux.png)

### 安装redux
```npm
npm install redux --save
```

#### 手写Redux
```js
// KRedux.js
export function createStore(reducer) {
  let currentState = undefined; 

  function getState() {
    return currentState;
  }
  function dispatch(action) {
    currentState = reducer(currentState, action)
    // 监听函数是一个数组，那就循环吧
    currentListeners.map(listener => listener())

  }
  // 订阅， 可以多次订阅
  function subscribe(listener) {
    // 每次订阅， 把回调放入回调函数
    currentListeners.push(listener);
  }

  // 取值的时候，注意一定要保证不和项目中的会重复
  dispatch({type: "@INIT/REDUX-KKB"})

  return {
    getState,
    dispatch,
    subscribe
  }
}
 
```
```js
// index.js
 import {createStore, combineReducers} from "../kRedux";

 //定义修改规则
 function countReducer(state=0, action) {
   switch(action.type) {
     case "ADD":
        return state + 1;
     case "MINUS":
        return state - 1;
     default:
        return state;

   }
 }
 const store = createStore(combineReducers({count:countReducer}));

 export default store

```
#### 异步

> Redux只是个纯粹的状态管理器，默认只支持同步，实现异步任务， 比如延迟、网络请求，需要中间件的支持，比如我们试用最简单的redux-thunk和redux-logger.

> 中间件就是一个函数，对store.dispatch方法进行改造，在发出Action和执行Reducer这两步之间，添加了其他功能

```js
npm install redux-thunk redux-logger --save
```
```js
import {createStore} from "redux"
import thunk from 'redux-thunk'
import logger from 'redux-logger'
// 定义修改规则


```
