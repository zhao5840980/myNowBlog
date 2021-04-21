---
title: new Date().getTime()在IE下的兼容问题
date: 2021-04-21 11:03:40
author: zhengzheng
cover: true
categories: 前端
tags: 前端
---

## 背景

项目中碰到后台返回时间类似2018-01-30 09:07:15这样的格式，使用new Date().getTime()将时间格式转成时间戳，但在IE出现兼容问题，具体错误为带“-”格式的时间无法被new Date()转成时间格式，返回NaN.
 
## 解决

代码如下：

```js
const date = '2018-01-30 09:07:15';
const timestamp = new Date(date).getTime(); //这里在ie下返回NaN,具体场景使用的是11.0版本

// 将上诉修改如下
const date = '2018-01-30 09:07:15'.replace(/-/g, '/');
const timestamp = new Date(date).getTime();
```

问题解决，重点是将时间字符串的“-”替换成“/”就可以了
