---
title: git pull报错
date: 2021-06-02 15:02:28
author: zhengzheng
cover: true
tags:
  - git
---

## git pull报错，error: cannot lock ref导致拉流失败

1. 使用git命令删除相应refs文件，```git git update-ref -d refs/remotes/XXX ```,或者手动删除文件

2. 简单粗暴强行git pull，执行 ```git git pull -p```