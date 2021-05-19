---
title: vue项目最佳实践
date: 2021-04-28 17:09:54
author: zhengzheng
cover: true
categories: 前端
tags:
  - 前端
  - Vue
---

## 目标
- 代码规范
- 项目配置
- 权限
- 导航
- 数据mock
- 环境变量
- 测试
- 优化、告警、发布和部署

## 知识点
### 项目配置策略
> 基础配置：指定应用上下文、端口号、vue.config.js

```js
const port = 7070;
module.exports = {
  publicPath:'/best-partice',// 部署应用包时的基本URL
  devServer: {
    port, 
  }
}
```


