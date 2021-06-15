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


### 组件跨岑寂通讯- Context

