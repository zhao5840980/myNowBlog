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