---
title: Nuxt使用介绍
date: 2018/12/25
---

## 说明

本文档是基于[nuxt-demo](https://github.com/loveJiong/nuxt-demo)的一篇对于nuxt框架的使用、特性介绍、以及服务端部署的文章。


下载demo食用效果更佳。
``` bash
# 克隆项目
$ git clone https://github.com/loveJiong/nuxt-demo.git

# 安装依赖
$ npm install

# 开发环境服务 0.0.0.0:7017
$ npm run dev

# 生产环境服务
$ npm run build
$ npm start

# 静态站点构建
$ npm run generate
```

## 1.Nuxt简介

Nuxt.js 是一个基于 Vue.js 的通用应用框架。

通过对客户端/服务端基础架构的抽象组织，Nuxt.js 主要关注的是应用的 UI渲染。

Nuxt.js 预设了利用Vue.js开发服务端渲染的应用所需要的各种配置。

除此之外，还提供了一种命令叫：nuxt generate，为基于 Vue.js 的应用提供生成对应的静态站点的功能。

作为框架，Nuxt.js 为 客户端/服务端 这种典型的应用架构模式提供了许多有用的特性，例如异步数据加载、中间件支持、布局支持等。

[官方文档](https://zh.nuxtjs.org/guide/)介绍的很清楚了。

## 2.目录结构

{% asset_img nuxt2-1.png 目录结构 %}

## 3.路由

Nuxt.js是根据我们`pages`下的目录结构自动生成vue-router模块的路由配置。

如图所示是demo中的目录结构：

{% asset_img nuxt3-1.png pages %}

Nuxt.js会自动生成如果的路由配置：
```javascript
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'demo-meta',
      path: '/demo/meta',
      component: 'pages/demo/meta.vue'
    },
    {
      name: 'demo-ssr',
      path: '/demo/ssr',
      component: 'pages/demo/ssr.vue'
    }
  ]
}
```

Nuxt.js关于路由还有更多如动态路由、嵌套路由、动态嵌套路由等等的配置，移步[Nuxt.js路由](https://zh.nuxtjs.org/guide/routing)。

## 4.页面

页面即 `pages` 下面的vue组件，只不过nuxt.js为其提供了许多有用的特殊配置项，如下图：

{% asset_img nuxt4-1.png pages %}

我们对其中的`asyncData`、`head`、`layout`、`middleware`几个特性进行简单的介绍。

### asyncData 方法

`asyncData`方法会在组件（限于页面组件）每次加载之前被调用。它可以在服务端或路由更新之前被调用。在这个方法被调用的时候，第一个参数被设定为当前页面的[上下文对象](https://zh.nuxtjs.org/api/context)，你可以利用`asyncData`方法来获取数据并返回给当前组件(`data`方法)。

```javascript
  export default {
    async asyncData({ $axios }) {
      let res = await $axios.$get('http://192.168.150.189/hswy-basic-web/basic/account/serverDate')
      return {
        res,
      }
    },
    data() {
    },
    mounted() {
      console.log(this.res); // 请求返回的res
    },
  };
```

### head 方法

使用 `head` 方法设置当前页面的头部标签，并与`data`中的属性相结合实现动态设置。

### layout 属性

Nuxt.js 允许你扩展默认的布局，或在 layout 目录下创建自定义的布局。

layouts/default.vue 文件为默认布局，所有未配置layout属性的都使用默认布局。

`<nuxt/>`组件用于显示页面的主体内容。

可以在layouts目录下自定义你所需要的布局，demo中以`layouts-demo.vue`为例。

### head方法和layout属性的demo

pages/demo/meta.vue

```html
<template>
  <section class="container">
    <div>
      <h1 class="title">
        {{ title }}
      </h1>
      <h2 class="subtitle">
        前页面的title:{{ title }}
      </h2>
      <h2 class="subtitle">
        当前页面的head的children数:{{ num }}
      </h2>
      <a class="button--grey" @click="change">变一下</a>
    </div>
  </section>
</template>

<script>
  export default {
    layout: 'layouts-demo',
    data() {
      return {
        title: '动态meta标签',
        meta: [ { hid: 'description', name: 'description', content: 'My custom description' } ],
        link: [],
        num: 0,
      };
    },
    computed: {
    },
    head() {
      return {
        title: this.title,
        meta: this.meta,
        link: this.link,
      };
    },
    mounted() {
      this.num = document.getElementsByTagName('head')[0].children.length;
    },
    methods: {
      change() {
        this.title = '变了一下';
        this.link = [ { rel: 'stylesheet', href: 'https://fonts.googleapis.com/css?family=Roboto' } ];
        setTimeout(() => {
          this.num = document.getElementsByTagName('head')[0].children.length;
        }, 100);
      }
    },
  };
</script>
```

layouts/layouts-demo.vue

```html
<template>
  <div class="layouts-demo">
    <div class="left-menu">
      <a class="links button--green" @click="goBack">返回</a>
      <h2>
        我是layouts
      </h2>
      <ul>
        <li>假装菜单1</li>
        <li>假装菜单2</li>
        <li>假装菜单3</li>
        <li>假装菜单4</li>
        <li>假装菜单5</li>
      </ul>
    </div>
    <nuxt/>
  </div>
</template>

<script>
  export default {
    methods: {
      goBack() {
        this.$router.go(-1);
      },
    }
  }
</script>
```

### middleware

中间件允许定义一个自定义函数运行在一个页面或一组页面渲染之前。

每一个中间件应放置在 middleware/ 目录。文件名的名称将成为中间件名称(middleware/auth.js将成为 auth 中间件)。

middleware/strong.js

```javascript
export default function ({
  store,
  redirect
}) {
  console.log(store.state);
  if (!store.state.isStrong) {
    return redirect('/error');
  }
}
```

pages/demo/middleware.vue

```html
<template>
  <section class="container">
    <div>
      <h1 class="title">只有强者才可以看到这个页面</h1>
      <div class="links">
        <nuxt-link class="button--green" to="/">回到首页</nuxt-link>
      </div>
    </div>
  </section>
</template>

<script>
  export default {
    middleware: 'strong'
  }
</script>
```

## 5.插件

所有的插件都是写在**plugins目录**下，这里以**element-ui为例。**

plugins/element-ui.js

```javascript
// 全局使用
import Vue from 'vue'
import Element from 'element-ui'

// 按需使用
import { Button, Loading, MessageBox } from 'element-ui'

export default () => {
  // 全局使用
  Vue.use(Element)
  // 按需使用
  Vue.use(Button)
  // 注入 $root
  Vue.prototype.$loading = Loading.service
  Vue.prototype.$msgBox = MessageBox
}
```

nuxt.config.js

```javascript
module.exports = {
  css: ['element-ui/lib/theme-chalk/index.css'], // 在 Nuxtjs 里配置全局的 CSS 文件、模块、库。 (每个页面都会被引入)
  plugins: [
    {
      src: "@/plugins/element"
    }
  ]
}
```

想了解更多关于使用插件的信息，移步[插件使用指引](https://zh.nuxtjs.org/guide/plugins)。


## 6.使用Axios,并配置全局拦截器

Nuxtjs推荐使用@nuxtjs/axios

安装依赖

```bash
npm install @nuxtjs/axios --save
```

nuxt.config.js配置

```javascript
module.exports = {
  modules: [ '@nuxtjs/axios' ],
}
```

设置全局拦截器

plugins/axios.js
```javascript
export default function ({ $axios }) {
  let axios = $axios;
  // 基本配置
  axios.defaults.timeout = 10000
  axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded'

  // 请求回调
  axios.onRequest(config => {})

  // 返回回调
  axios.onResponse(res => {
    console.log(res.data);
  })

  // 错误回调
  axios.onError(error => {})
}
```

组件中使用
```javascript
async asyncData({ $axios, query}) {
  let res = await $axios.$get('http://192.168.150.189/hswy-basic-web/basic/account/serverDate')
  return {
    query: res,
  }
},
```

## 7.服务端渲染

Nuxt是基于Vue.js的 vue-server-renderer 模块实现服务端渲染功能，它为我们提供了最佳SSR默认配置。只要我们按照框架的规则去开发和配置，无需做额外的事情，只要将服务启动即可实现服务端渲染。

运行命令
```bash
npm run dev
```
打开调试，查看network，即可看到首屏返回的是渲染好的html页面。

本文章仅介绍如何使用Nuxt开发服务端渲染的项目，具体服务端渲染的实现原理移步[Vue SSR 指南](https://ssr.vuejs.org/zh/)

```javascript
<template>
  <section class="container">
    <div>
      <logo/>
      <h1 class="title">
        nuxt-demo
      </h1>
      <h2 class="subtitle">
        基础和特性demo
      </h2>
      <div class="links">
        <nuxt-link class="button--green" to="/demo/ssr">异步数据</nuxt-link>
        <nuxt-link class="button--green" to="/demo/meta">动态meta标签</nuxt-link>
      </div>
    </div>
  </section>
</template>
```

这是没有数据交互的首页，只要起了服务首屏访问该页面的时候，即会返回渲染好的html页面。
```javascript
<template>
  <section class="container">
    <div>
      <h1 class="title">
        {{ title }}
      </h1>
      <h2 class="subtitle">
        我在服务端发了服务器请求并渲染
      </h2>
      <span>{{ query }}</span>
    </div>
  </section>
</template>

<script>
  export default {
    layout: 'layouts-demo',
    async asyncData({ $axios, query}) {
      let res = await $axios.$get('http://192.168.150.189/hswy-basic-web/basic/account/serverDate')
      return {
        query: res,
      }
    },
    data() {
      return {
        title: '服务端渲染',
      };
    },
  };
</script>
```
在需要发生数据交互的页面中，只需要利用4、5的组合拳，就可以实现在服务端获取好数据后，并合并到data中，在服务端渲染完成后返回客户端。

## 8.项目部署

nuxt.config.js,配置host和端口号

```javascript
module.exports = {
    server: {
    port: 7017, // default: 3000
    host: '0.0.0.0', // default: localhost,
  }
}
```

本地打包
```bash
npm run build
```

将打包后的下列文件夹上传到服务器空间里

```
.nuxt
static
nuxt.config.js
package.json
```

在服务器上部署

1.安装依赖

```bash
npm install
```

2.运行服务

```bash
npm run start
```
这样就可以通过ip:port访问到你的项目了

## 8.利用pm2管理进程

1.安装pm2模块

```bash
npm install pm2 -g
```

2.进入对应目录，执行以下命令

```bash
pm2 start npm --name "nuxt-demo" -- run start
```

3.查看进程列表

```bash
pm2 list
```

{% asset_img nuxt8-1.png 进程列表 %}

4.停止进程、再启动进程
```bash
pm2 stop nuxt-demo
pm2 start nuxt-demo
```

## 结语

还有许多实用的特性会在接下来的文章中介绍，还有一些问题没有想到办法解决，比如开发、测试、生产环境的配置，以及服务端部署的一些荷载之类的问题都没有接触过，这里只是把我在部署nuxt服务中觉得有价值的点以及流程介绍了一下，并提供一个demo，让大家在看更能形象的理解官方文档上的一些解释。