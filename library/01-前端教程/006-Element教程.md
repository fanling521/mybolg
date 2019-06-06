# Element教程

本教程只讲述项目流程，控件用法看官网，使用技巧看Vue.js教程。

## 什么是Element

一套和vue配合使用的ui控件库，只适用PC端。

### 什么是webpack

webpack是打包工具。

### 什么是vue.js

`Vue.js`是一套构建用户界面的渐进式框架。

## 项目构建

### 环境准备

确保本机装了node.js，npm管理工具，开发工具可以使用IDEA。

### 构建过程

使用webpack创建vue.js项目

```bash
# 卡在checking installable status
npm cache clean --force
# 替换npm的镜像源
C:\Users\fanling>
npm get registry
npm config set registry http://registry.npm.taobao.org/
# 安装webpack
npm install webpack -g
```

（1）安装vue-cli@3

```bash
C:\Users\fanling>
node -v
v10.15.1
# 卸载之前版本
npm uninstall vue-cli -g
# 安装新版本
npm install -g @vue/cli
```

（2）进入你的目录创建vue项目

```bash
E:\MyProject>
npm install -g @vue/cli-init
vue init webpack web-test
# 提示下载模板
# 添加路由，不添加代码规范检查和测试
# 等待下载依赖包
# 登录测试
npm run dev
# 打开可以浏览表示安装成功，注意需要进入项目目录
Your application is running here: http://localhost:8080
```

（3）在项目目录下安装element-ui

```bash
npm i element-ui -S #  i 是install 的简写，-S 即--save，保存在正式依赖包中
```

（4）修改项目

```js
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI);
```

（5）布局组件

新建views文件，存放布局文件`layout.vue`

```vue
<template>
  <el-container>
    <el-header></el-header>
    <el-main></el-main>
    <el-footer></el-footer>
  </el-container>
</template>
```

修改路由文件`/src/route/index.js`

```js
import Vue from 'vue'
import Router from 'vue-router'
import Layout from '@/views/layout'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'Layout',
      component: Layout
    }
  ]
})
```

运行即可。

关于element-ui组件教程，查看[官网](https://element.eleme.cn)

### webpack打包

（1）安装webpack-cli

```bash
npm install webpack-cli -g
```

（2）运行

```bash
npm run build
```

之后会在界面显示资源信息，以及目标目录为/dist

