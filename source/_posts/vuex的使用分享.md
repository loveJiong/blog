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

### 2.1 什么是vuex
先看个例子：
```javascript
const store = new Vuex.Store({
    state: {
        name: 'thb',
        age: 18,
        profession: 'Move Bricks',
    },
    getters: {
        personInfo(state) {
            return `My name is ${state.name}, I am ${state.age}`;
        }
    }
    mutations: {
        SET_AGE(state, age) {
            state.age = age;
        }
    },
    actions: {
        nameAsyn({commit}) {
            setTimeout(() => {
                commit('SET_AGE', 18);
            }, 1000);
        }
    },
    modules: {
        a: modulesA
    }
}
```
简单来说就是一个闭包，闭包里面的有一个state对象，它保存了我们许许多多的全局变量，但是我们不能直接修改它，只能通过闭包对外开出来的mutations去修改这些state，其中还开放了例如把state打扮一番再返回的getters，要绕一圈（异步）才能让mutations修改state的actions，还有包含所有内容的子孙后代modules。


### 2.2 为什么要使用vuex
在vue的组件化开发中，经常会遇到需要将当前组件的状态传递给其他组件。父子组件通信时，我们通常会采用 props + emit 这种方式。但当通信双方不是父子组件甚至压根不存在相关联系，或者一个状态需要共享给多个组件时，就会非常麻烦，数据也会相当难维护，这对我们开发来讲就很不友好，vuex 这个时候就很实用。

## 3 如何定义vuex

### 3.1 代码中的目录结构
├── build                                       // webpack配置文件
├── config                                      // 项目打包路径
├── src                                         // 源码目录
│   ├── components                              // 组件
│   ├── store                                   // vuex的状态管理
│   │   ├── modules                             // modules目录
│   │       ├── person-center                   // modules子目录
│   │           ├── person-center               // 配置module
│   │   ├── actions-types.js                    // 定义常量actions名
│   │   ├── action.js                           // 配置actions
│   │   ├── index.js                            // 引用vuex，创建store
│   │   ├── mutation-types.js                   // 定义常量muations名
│   │   └── mutations.js                        // 配置mutations
├── index.html                                  // 入口html文件

### 3.2 index.js

```javascript
import Vue from 'vue';
import Vuex from 'vuex';
import actions from './actions'; // 引入actions
import mutations from './mutations'; // 引入mutations
// 引入personCenter子模块
import personCenter from './modules/person-center/person-center'; 

Vue.use(Vuex);

// 定义state的属性
const state = {
  appInfo: {},
  loginInfo: {},
};

export default new Vuex.Store({
  state,
  actions,
  mutations,
  modules: {
    personCenter,
  },
});

```

### 3.3 mutations-types.js
``` javascript
export const SET_APPINFO = 'SET_APPINFO';
export const SET_LOGININFO = 'SET_LOGININFO';
export const UPDATE_LOGININFO = 'UPDATE_LOGININFO';
```

### 3.4 mutations.js
``` javascript
// 只能进行同步操作
import { SET_APPINFO, SET_LOGININFO, UPDATE_LOGININFO } from './mutations-types';

export default {
  [SET_APPINFO](state, appInfo) {
    state.appInfo = {
      appId: appInfo.appId,
      mac: appInfo.appMac,
      version: appInfo.appVer,
      locationCode: appInfo.locationCode,
    };
  },
  [SET_LOGININFO](state, loginInfo) {
    state.loginInfo = loginInfo;
  },
  [UPDATE_LOGININFO](state, updateInfo) {
    const keys = Object.keys(updateInfo);
    keys.forEach(key => {
      state.loginInfo[key] = updateInfo[key];
    });
  },
};
```

### 3.5 actions-types
```javascript
export const UPDATE_LOGININFO_ASYNC = 'UPDATE_LOGININFO_ASYNC';
```

### 3.6 actions
```javascript
// 专门处理异步操作
import { UPDATE_LOGININFO } from './mutations-types';
import { UPDATE_LOGININFO_ASYNC } from './actions-types';

export default {
  [UPDATE_LOGININFO_ASYNC]({ commit }, data) {
    setTimeout(() => {
      commit(UPDATE_LOGININFO, data);
    }, 1000);
  },
};

```

### 3.7 modules
```javascript
const SET_PORTRAIT = 'SET_PORTRAIT';

export default {
  namespaced: true,
  state: {
    portrait: '', // 头像
  },
  mutations: {
    [SET_PORTRAIT](state, v) {
      state.portrait = v;
    },
  },
};

```

## 4 如何使用vuex