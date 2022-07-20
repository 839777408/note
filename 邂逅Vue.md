---
title: 邂逅Vue
tags:
  - Vue
categories:
  - Vue
top_img: 'https://unpkg.com/nan-picture/img/wp3.jpg'
cover: 'https://unpkg.com/nan-picture/img/wp3.jpg'
abbrlink: a5b0
date: 2020-10-12 16:41:39
updated: 2020-10-12 16:41:46
---

>参考视频：最全最新Vue、Vuejs教程，从入门到精通 https://www.bilibili.com/video/BV15741177Eh?p=1

# 什么是Vue.js

**Vue.js**（/vjuː/，或简称为**Vue**）是一个用于创建用户界面的**开源JavaScript框架**，也是一个创建单页应用的Web应用框架，是一个**渐进式和响应式**的框架。

- Vue有很多特点和Web开发中常见的高级功能
  - 解耦视图和数据
  - 可复用的组件
  - 前端路由技术
  - 状态管理
  - 虚拟DOM

【Vue中文官网，观看WHY VUE.JS?】：https://cn.vuejs.org/



# Vue.js安装

- 方式一：直接CDN引入
  你可以选择引入开发环境版本还是生产环境版本

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 --> 
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

- 方式二：下载和引入
  - 开发环境 https://vuejs.org/js/vue.js
  - 生产环境 https://vuejs.org/js/vue.min.js
  - `<script src="../js/vue.js"></script>`

- 方式三：NPM安装
  `$ npm install vue`



# Vue初体验

## HelloVuejs

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">{{message}}</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '下午好，阿楠'
        }
    })
</script>
</body>
</html>
```

我们来阅读JavaScript代码，会发现创建了一个**Vue对象**。

创建Vue对象的时候，传入了一些options：{}

- {}中包含了**el属性**：该属性决定了这个Vue对象挂载到哪一个元素上，很明显我们这里是挂载到了id为app的元素上。
- {}中包含了**data属性**：该属性中通常会存储一些数据，这些数据可以是我们直接定义出来的，比如像上面这样。也可能是来自网络，从服务器加载的。

> 浏览器执行代码的流程：
> 执行到10~13行代码显然出对应的HTML
> 执行第16行代码创建Vue实例，并且对原HTML进行解析和修改。



## Vue列表显示

```html
<div id="app">
    <ul>
        <li v-for="item in star">{{item}}</li>
    </ul>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: "#app",
        data: {
            star: ['刘亦菲','程潇','IU','林允儿']
        }
    })
</script>
```

它还是响应式的。
也就是说，当我们数组中的数据发生改变时，界面会自动改变。
打开开发者模式的console试一下

![](https://unpkg.com/nan-picture/blog/20201011173203.png)



## 案例：计数器

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <h2>当前计数为：{{count}}</h2>
    <button v-on:click="add">+</button>
    <button v-on:click="sub">-</button>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            count: 0
        },
        methods: {
            add: function () {
                console.log('add被执行');
                this.count++
            },
            sub: function (){
                console.log('sub被执行');
                this.count--
            }
        }
    })
</script>
</body>
</html>
```

![](https://unpkg.com/nan-picture/blog/20201011173424.png)



# Vue中的MVVM

![](https://unpkg.com/nan-picture/blog/20201011175627.png)

- View层：
  视图层
  在我们前端开发中，通常就是DOM层。
  主要的作用是给用户展示各种信息。
- Model层：
  数据层
  数据可能是我们固定的死数据，更多的是来自我们服务器，从网络上请求下来的数据。
  在我们计数器的案例中，就是后面抽取出来的obj，当然，里面的数据可能没有这么简单。
- ViewModel层：
  视图模型层
  视图模型层是View和Model沟通的桥梁。
  一方面它实现了Data Binding，也就是数据绑定，将Model的改变实时的反应到View中
  另一方面它实现了DOM Listener，也就是DOM监听，当DOM发生一些事件(点击、滚动、touch等)时，可以监听到，并在需要的情况下改变对应的Data。

我们的计数器中就有严格的MVVM思想，View依然是我们的DOM，Model就是我们data中的数据，可以抽离出来成为obj，ViewModel就是我们创建的Vue对象实例。

> 找了一篇相对好理解的博客，了解MVVM以及MVVM和MVC的区别：[MVC和MVVM的区别](https://blog.csdn.net/qq_42068550/article/details/89480350)



# 创建Vue实例传入的options

我们在创建Vue实例的时候，传入了一个对象options。
这个options中可以包含哪些选项呢？
详细解析： https://cn.vuejs.org/v2/api/#%E9%80%89%E9%A1%B9-%E6%95%B0%E6%8D%AE
目前掌握这些选项：

- el: 
  类型：string【常用】 | HTMLElement
  作用：决定之后Vue实例会管理哪一个DOM。
- data: 
  类型：Object | Function （组件当中data必须是一个函数）
  作用：Vue实例对应的数据对象。
- methods: 
  类型：{ [key(string类型)]: Function }
  作用：定义属于Vue的一些方法，可以在其他地方调用，也可以在指令中使用。



# Vue的生命周期

![](https://unpkg.com/nan-picture/blog/20201011182227.png)
![](https://unpkg.com/nan-picture/blog/20201011182212.png)