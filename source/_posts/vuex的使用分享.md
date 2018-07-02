---
title: vuex的使用分享
date: 2018/6/29
---

## 1 安装[vue-devtools](https://github.com/vuejs/vue-devtools)

### 1.1 谷歌浏览器插件安装
[谷歌商店](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)（需要翻墙）

### 1.2 运行在任何环境下的vue-devtools
可以运行在任何环境下的tools，这样就解决了一些使用自己外壳的项目也可以集成vue-devtools

#### 1.2.1 安装devtools
`npm install --save-dev @vue/devtools`

#### 1.2.2 使用devtools
启动devtools程序

`./node_modules/.bin/vue-devtools`
或者直接加入项目中的package.json的开发依赖中，`npm install`的时候会自行安装

关联到对应的vue项目中,在入口文件中引入devtools
```javascript
import devtools from '@vue/devtools'
// import Vue from 'vue'
```
<font color="red">注意要在vue之前import</font>

将项目与工具进行关联
```javascript
if (process.env.NODE_ENV === 'development') {
  devtools.connect(/* host, port */)
}
```

效果展示：

{% asset_img vuex1-1.png 效果展示 %}

#### 1.2.3 如何在跑项目的同时启动devtools
上文提到的启动devtools的命令`./node_modules/.bin/vue-devtools`，每次启动的时候都要执行，就很繁琐，所以我把它写入了dev环境的webpack配置文件中，这样每次启动dev环境的时候，就会自动把该工具启动。

在dev的webpack配置文件中加入下面代码
```javascript
const process = require('child_process'); // 创建node子进程
const path = require('path') // 引入path模块

path.resolve(__dirname, '..') // 获取node_modules的绝对路径
// 执行命令
process.exec(`${path.resolve(__dirname, '..')}/node_modules/.bin/vue-devtools`,
(error, stdout, stderr) => {
    if (error !== null) {
      console.log('exec error: ' + error);
    }
});
```
这样当你执行npm run xxx的时候，并且也执行了
`./node_modules/.bin/vue-devtools`

## 2 vuex基础知识

