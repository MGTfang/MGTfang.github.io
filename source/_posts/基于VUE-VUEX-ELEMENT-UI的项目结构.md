title: 基于VUE+VUEX+ELEMENT-UI的项目结构
author: Peng Fang
tags:
  - vue
categories:
  - vue
date: 2017-07-31 17:54:00
---
分析了石头哥的VUE模板工程（[基于VUE+VUEX+ELEMENT-UI](https://github.com/TCL-MIG-FE/vue-web-spa-startkit)）的项目结构，如下：

```
mock-server // 模拟数据
|   api.json // 配置请求响应的数据
|   server.js // mock数据响应
node_modules // 组件安装存放目录
src // 源文件目录
|---actions // 分模块定义接口调用
|   |   articles.js
|   |   index.js
|   |   root.js
|---components // 组件
|   |   Select.vue
|   |   Table.vue
|---constants // 定义一些常量
|   |   actions.js
|   |   api.js
|---containers // 页面
|   |   Article.vue
|---layouts // 布局
    |---css
        |---core
        |   |   base.less
        |   |   common.less
        |   |   font.less
        |   |   index.less
        |   |   normalize.less
        |   |   reset.less
        |---mixins
        |   |   compatibility.less
        |   |   iconfont.less
        |   |   index.less
        |   |   opacity.less
        |   |   size.less
        |---themes
        |   |   default.less
        index.less
    |---fonts
        |---antd
        |   |   iconfont.eot
        |   |   iconfont.svg // ...
    |---img
|---mutations // 更改state
|   |   article.js
|   |   index.js
|   |   root.js
|---plugins
|   |   element.js // 注册需要的element组件
|---store
|   |   index.js // 创建Vuex的Store实例
|---utils
|   |   api.js // 暴露一些请求Api以及createAction方法
|   |   misc.js // 公用方法
|   App.vue // 主页
|   index_dev.html // 开发环境的首页
|   index_prod.html // 生产环境的首页
|   root.js // 入口文件，创建Vue实例
|   router.js // 创建VueRouter实例
.babelrc babel配置文件
.editorconfig IDE编辑器配置文件
package-lock.json npm更改操作自动生成
package.json 项目相关的各种元数据(依赖的模块)
proxy.js 配置代理
webpack.config.js 根据是否是生产环境设置webpack的配置文件
webpack.dev.config.js 开发环境配置文件
webpack.prod.config 生产环境配置文件
```