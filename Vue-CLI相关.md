---
title: Vue CLI相关
tags:
  - Vue CLI
categories:
  - Vue
top_img: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp8.jpg'
cover: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp8.jpg'
abbrlink: a9f7
date: 2020-11-09 01:29:03
updated: 2020-11-09 01:29:08
---

# Vue CLI

## 基本介绍

- 如果你只是简单写几个Vue的Demo程序, 那么你不需要Vue CLI。
- 如果你在开发大型项目, 那么你需要, 并且必然需要使用Vue CLI。
  - 使用Vue.js开发大型应用时，我们需要考虑代码目录结构、项目结构和部署、热加载、代码单元测试等事情。
  - 如果每个项目都要手动完成这些工作，那无疑效率比较低效，所以通常我们会使用一些脚手架工具来帮助完成这些事情。

**CLI是什么意思?**

- CLI是Command-Line Interface, 翻译为命令行界面, 但是俗称脚手架。

- Vue CLI是一个官方发布 vue.js 项目脚手架。

- 使用 vue-cli 可以快速搭建Vue开发环境以及对应的webpack配置。

## 安装脚手架

> 使用前提：安装Node和Webpack

- 关于旧版本

  Vue CLI 的包名称由 `vue-cli` 改成了 `@vue/cli`。 如果你已经全局安装了旧版本的 `vue-cli` (1.x 或 2.x)，你需要先通过 `npm uninstall vue-cli -g` 或 `yarn global remove vue-cli` 卸载它。如果卸载不了旧版本，像我使用npm卸载2.9.6版本太久甚至没反应，可以使用cnpm卸载，具体命令参考[我的webpack详解中的2.1](https://nanzx.top/posts/a5d2/)。

- Node 版本要求

  Vue CLI 4.x 需要 [Node.js](https://nodejs.org/) v8.9 或更高版本 (推荐 v10 以上)。

- 可以使用下列任一命令安装这个新的包

  `npm install -g @vue/cli` 或 `yarn global add @vue/cli`

- 可以用这个命令来检查其版本是否正确：

  `vue --version`



# Vue CLI2

## 创建旧版本的2.x模板

上面安装的是Vue CLI 4.x的版本，如果需要想按照Vue CLI2的方式初始化项目时不可以的。![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201107212311.png)

- Vue CLI2创建项目命令：
  	`vue init webpack my-project`
- Vue CLI>=3创建项目命令：
  	`vue create my-project`

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201107212603.png)



## 目录结构详解

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201107213239.png)



## Vue程序运行过程

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201108030136.png)



## Runtime-Compiler和Runtime-only的区别

在使用 vue-cli 脚手架构建项目时，会遇到一个构建选项 Vue build，有两个选项：Runtime + Compiler和Runtime-only：

- **Runtime + Compiler: recommended for most users**

  > 运行程序+编译器:推荐给大多数用户

- **Runtime-only: about 6KB lighter min+gzip, but templates (or any Vue-specificHTML) are ONLY allowed in .vue files - render functions are required elsewhere**

  > 仅运行程序: 比上面那种模式轻大约 6KB min+gzip，但是 template (或任何特定于vue的html)只允许在.vue文件中使用——其他地方用需要 render 函数

---

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201108032301.png)

两种模式生成的区别只有在 main.js 中，其他都是一样的：

- Runtime + Compiler中是使用 template + component
- Runtime-only则是使用 render 函数

---

**Runtime + Compiler 中 Vue 的运行过程:**

(1)首先将vue中的模板解析成abstract syntax tree （ast）抽象语法树

(2)将抽象语法树再编译成render函数

(3)将render函数再转换成virtual dom，也就是虚拟dom

(4)最后将虚拟dom显示在浏览器上

**而 Runtime-only 只需2步：**

(1)将render函数再转换成virtual dom，也就是虚拟dom

(2)最后将虚拟dom显示在浏览器上

**简单总结：**

- Runtime-only 比 Runtime-Compiler 轻 6kb，因为少了编译器
- Runtime-only 运行更快，因为它少了编译的环节
- Runtime-only 只能识别render函数，不能渲染template，`.vue`文件中的template也是被 `vue-template-compiler` 编译成了render函数

- 如果在之后的开发中，你依然使用template，就需要选择Runtime-Compiler

- 如果你之后的开发中，使用的是.vue文件夹开发，那么可以选择Runtime-only



## render函数的使用

**类型**：`(createElement: () => VNode) => VNode`

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20220706215027.png)

Vue 选项中的 `render` 函数若存在，则 Vue 构造函数不会从 `template` 选项或通过 `el` 选项指定的挂载元素中提取出的 HTML 模板编译渲染函数。



## npm run build

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201108233918.png)



## npm run dev

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201108233949.png)



## 修改配置：webpack.base.conf.js起别名

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201108234123.png)

这样起别名后，在引用文件的路径上可以用别名代替，例如：

- .src/components/tabbar/TabBar可变为 ~components/tabbar/TabBar，以后移动代码或文件时，就不会因为忘记修改引用路径而报错。

- `~`只有js不用加，html和css要加。

  

# Vue CLI3

vue-cli 3 与 2 版本有很大区别：

- vue-cli 3 是基于 webpack 4 打造，vue-cli 2 还是 webapck 3
- vue-cli 3 的设计原则是“0配置”，移除的配置文件根目录下的，build和config等目录
- vue-cli 3 提供了 vue ui 命令，提供了可视化配置，更加人性化
- 移除了static文件夹，新增了public文件夹，并且index.html移动到public中



## 创建项目

创建项目命令：
	`vue create my-project`

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201108234604.png)

## 目录结构详解

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201108234638.png)



## 配置

- 通过图形化界面进行vue的配置和管理，启动配置服务器的命令：`vue ui`

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201109012242.png)



- 可以看到目录结构与2的相比少了配置目录，也就是没了build和config，配置文件都到了@vue模块里：

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20220706215028.png)

- 可以自定义vue.config.js



## 自定义配置：起别名

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201109012627.png)



# Vue CLI4

2020年11月9日01:27:38，目前我安装脚手架的最新版本为：4.5.8，与3.x的差异不大。