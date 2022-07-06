---
title: Vue Router详解
tags:
  - Vue Router
categories:
  - Vue
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp9.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp9.jpg'
abbrlink: 426d
date: 2020-11-12 15:05:35
updated: 2020-11-19 10:20:45
---



# 认识路由

## 什么是路由

路由（routing）就是通过互联的网络把信息从源地址传输到目的地址的活动。 --- 维基百科

在生活中, 我们也有听说过路由的概念： 路由器。

- 路由器提供了两种机制: 路由和转发。
  - 路由是决定数据包从源头到目的地的路径。
  - 转发是将输入端的数据转移到合适的输出端。
- 路由中有一个非常重要的概念叫路由表。
  - 路由表本质上就是一个映射表, 决定了数据包的指向。



## 后端路由阶段

早期的网站开发，整个HTML页面是由**服务器来渲染**的，服务器直接生产渲染好且对应的HTML页面，返回给客户端进行展示。

但是一个网站这么多页面，**服务器如何处理呢？**

- 一个页面有自己对应的网址, 也就是URL。
- URL会发送到服务器，服务器会通过正则对该URL进行匹配，并且最后交给一个Controller进行处理。
- Controller进行各种处理， 最终生成HTML或者数据， 返回给前端展示。
- 这就完成了一个IO操作。

上面的这种操作，就是**后端路由**：

- 当我们页面中需要请求不同的路径内容时，交给服务器来进行处理，服务器渲染好整个页面，并且将页面返回给客户端。
- 这种情况下渲染好的页面，不需要单独加载任何的js和css，可以直接交给浏览器展示，这样也有利于SEO的优化。

**后端路由的缺点：**

1. 整个页面的模块由后端人员来编写和维护的。
2. 前端开发人员如果要开发页面，需要通过PHP和Java等语言来编写页面代码。
3. 通常情况下HTML代码和数据以及对应的逻辑会混在一起，编写和维护都是非常糟糕的事情。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201110094425.png)



## 前端路由阶段

**前后端分离阶段：**

- 随着Ajax的出现，有了前后端分离的开发模式。
- 后端只提供API来返回数据，前端通过Ajax获取数据，并且可以通过JavaScript将数据渲染到页面中。
- 这样做最大的优点就是前后端责任的清晰，后端专注于数据上，前端专注于交互和可视化上。
- 并且当移动端(iOS/Android)出现后，后端不需要进行任何处理，依然使用之前的一套API即可。
- 目前很多的网站依然采用这种模式开发。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201110094819.png)



**单页面富应用阶段（Single Page Application）：**

- 其实SPA最主要的特点就是在前后端分离的基础上加了一层前端路由。也就是前端来维护一套路由规则。
- 整个网站只有一个HTML页面。
- 前端路由的核心是：改变URL，但是页面不进行整体的刷新。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201110095255.png)





# 前端路由的规则

## URL的hash

URL的hash也就是锚点(#)， 本质上是改变 window.location 的 href 属性。

我们可以通过直接赋值 location.hash 来改变 href , 但是页面不发生刷新。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201110100537.png)

## HTML5的history模式：pushState

history接口是HTML5新增的, 它有五种模式改变URL而不刷新页面。

改变URL后浏览器左上方可以前进和后退。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201110100732.png)



## HTML5的history模式：replaceState

改变URL后浏览器左上方不可以前进和后退，按钮是灰色的。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201110101225.png)



## HTML5的history模式：go

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201110101451.png)

> 上面只演示了三个方法，因为 history.back()  等价于 history.go(-1)，history.forward() 则等价于 history.go(1)，这三个接口等同于浏览器界面的前进后退。



#  Vue Router起步

> vue-router是基于路由和组件的：
>
> 路由用于设定访问路径，将路径和组件映射起来。
>
> 在vue-router的单页面应用中, 页面的路径的改变就是组件的切换。

我们可以在使用Vue CLI创建项目时安装vue-router或者通过命令安装：`npm install vue-router --save`

---

## （一）创建router实例

在src/router/index.js中配置如下：

```javascript
import VueRouter from 'vue-router'
import Vue from 'vue'

//1.通过Vue.use(插件)，安装注入插件
Vue.use(VueRouter)

//2.定义路由映射
const routes = []

//3.创建router实例
const router = new VueRouter({
  //配置路由和组件之间的应用关系
  routes,
})

//4.将router实例传入到vue实例当中
export default router
```

## （二）挂载到Vue实例中

在src/main.js中配置如下：

```javascript
import Vue from 'vue'
import App from './App.vue'
import router from './router'

Vue.config.productionTip = false

new Vue({
  //在Vue实例中挂载创建的路由实例
  router,
  render: h => h(App)
}).$mount('#app')
```

## （三）创建组件

在src/components中新建.vue文件：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111111059.png)

## （四）配置组件和路径的映射关系

在src/router/index.js中配置如下：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111111614.png)

## （五）使用路由

在src/App.vue中配置如下：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111112112.png)

> `<router-link>`：该标签是一个vue-router中已经内置的组件, 它会被渲染成一个`<a>`标签。
> `<router-view>`：该标签会根据当前的路径, 动态渲染出不同的组件。
> 网页的其他内容，比如顶部的标题/导航，或者底部的一些版权信息等会和`<router-view>`处于同一个等级，
> 在路由切换时，切换的是`<router-view>`挂载的组件，其他内容不会发生改变。

## （六）最终效果

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111113241.png)



# 细节处理

## 路由的默认路径

默认情况下进入网站的首页，我们希望`<router-view>`渲染首页的内容。

但是我们的实现中，默认没有显示首页组件，必须让用户点击首页才可以。

如何可以让路径默认跳到到首页, 并且`<router-view>`渲染首页组件呢？

我们只需要配置多一个默认映射就可以了。

```javascript
const routes = [
  {
    path: '',
    redirect: '/home'
  },
  ...
]
```

配置解析：我们在routes中又配置了一个映射. 

- path配置的是根路径（/）或者可以为空
- redirect是重定向，也就是我们将根路径重定向到/home的路径下，这样就可以得到我们想要的结果了。



## 更改router的模式为history模式

我们前面说过改变路径的方式有两种：URL的hash和HTML5的history。

默认情况下，router路径的改变使用的是URL的hash。

如果希望使用HTML5的history模式， 进行如下配置即可：

```javascript
const router = new VueRouter({
  routes,
  mode: 'history'
})
```

**可以看到url地址中没有了#**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111114735.png)



## router-link的补充

在前面的`<router-link>`中，我们只是使用了一个属性：to，用于指定跳转的路径。

`<router-link>`还有一些其他属性：

- tag：可以指定`<router-link>`之后渲染成什么组件，`<router-link to='/home' tag='button'>`会被渲染成一个按钮，而不是`<a>`标签。
- replace：不会留下history记录，所以指定replace的情况下，点击后退键返回并不能返回到上一个页面中。
- active-class：当`<router-link>`对应的路由匹配成功时，会自动给当前元素设置一个**router-link-active**的class，而我们可以通过设置 active-class 来修改默认的class属性。
  - 在进行高亮显示的导航菜单或者底部tabbar时会使用到该类。但是通常不会修改类的属性，会直接使用默认的**router-link-active**即可。
  - ![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111121425.png)

`<router-link to="/home" tag="button" replace active-class="active">首页</router-link>`



## 修改linkActiveClass

该class具体的属性也可以通过router实例的属性进行修改 

```javascript
const router = new VueRouter({
  routes,
  mode: 'history',
  linkActiveClass: 'active'
})
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111121500.png)



## 路由代码跳转

有时候页面的跳转可能需要执行对应的JavaScript代码，这个时候就可以使用第二种跳转方式了。

在src/App.vue中配置如下：

```javascript
<template>
  <div id="app">
    <button @click="linkToHome">首页</button>
    <button @click="linkToAbout">关于</button>
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'App',
  methods: {
    linkToHome() {
      // this.$router.push('home')
      this.$router.replace('home')
    },
    linkToAbout() {
      // this.$router.push('about')
      this.$router.replace('about')
    }
  }
}
</script>

</style>
```



# 动态路由

在某些情况下，一个页面的path路径是不确定的，比如我们进入用户界面时，希望是如下的路径：/user/james或/user/kobe，除了有前面的/user之外，后面还跟上了用户的ID。

这种path和Component的匹配关系，我们称之为动态路由(也是路由传递数据的一种方式)。

`<router-link :to="'/user/'+ userId">用户</router-link>`

`<router-link: to="{name: 'User',params: {userId: userId}}">`

```javascript
const routes = [
  ...
  {
    path: '/user/:userId',
    name: 'User',
    component: User
  }
]
```

```html
<template>
  <div>
    <h2>我是用户：{{$route.params.userId}}</h2>
  </div>
</template>
```



# 路由懒加载

## 认识路由的懒加载

- 官方给出了解释:
  - 当打包构建应用时，Javascript包会变得非常大，影响页面加载。
  - 如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。
- 官方在说什么呢?
  - 首先，我们知道路由中通常会定义很多不同的组件。
  - 这些组件最后都被打包放在一个js文件中。
  - 但是这么多组件放在一个js文件中，必然会造成这个文件非常的大，影响加载。
  - 如果我们一次性从服务器请求下来这个文件，可能需要花费一定的时间，甚至用户的电脑上还可能会出现短暂空白的情况。
  - 如何避免这种情况呢? 使用路由懒加载就可以了.
- 路由懒加载做了什么?
  - 路由懒加载的主要作用就是将每个路由对应的组件打包成一个个的js代码块。
  - 只有在这个路由被访问到的时候，才加载对应的组件。



## 路由懒加载的效果

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111125039.png)



## 懒加载的方式

方式一：结合Vue的异步组件和Webpack的代码分析

const Home = resolve => { require.ensure(['../components/Home.vue'], () => {resolve(require('../components/Home.vue')) })};

方式二：AMD写法

const About = resolve => require(['../components/About.vue'], resolve);

方式三：在ES6中，我们可以有更加简单的写法来组织Vue异步组件和Webpack的代码分割

const Home = () => import('../components/Home.vue')



# 路由嵌套

## 认识嵌套路由

嵌套路由是一个很常见的功能，比如在home页面中,，我们希望通过/home/news和/home/message访问一些内容。
路径和组件的关系如下：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111125526.png)

## 嵌套路由实现

实现嵌套路由有两个步骤：

1. 创建对应的子组件, 并且在路由映射中配置对应的子路由（嵌套路由也可以配置默认的路径）。
2. 在组件内部使用`<router-view>`标签。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111125920.png)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111141215.png)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111141436.png)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111142015.png)



# 传递参数

传递参数主要有两种类型：params和query

- params的类型：
  配置路由格式：/router/:id
  传递的方式：在path后面跟上对应的值
  传递后形成的路径：/router/123，/router/abc
- query的类型:
  配置路由格式：/router，也就是普通配置
  传递的方式：**对象**中使用query的key作为传递方式
  传递后形成的路径：/router?id=123，/router?id=abc
-  有两种方式使用： `<router-link>`的方式和JavaScript代码方式（路由代码跳转）

`下图12行才是params的传递方式`

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201111152139.png)

---

获取参数通过**$route**对象获取的。

在使用了 vue-router 的应用中，路由对象会被注入每个组件中，赋值为 this.$route ，并且当路由切换时，路由对象会被更新。

```html
<template>
  <div>
    <h2>{{ $route.query }}</h2>
  </div>
</template>
```

**$route和$router是有区别的：**

- $router为VueRouter实例，想要导航到不同URL，则使用$router.push等方法。
- $route为当前router跳转对应的对象，可以获取name、path、query、params等 



# 导航守卫

我们来考虑一个需求：在一个SPA应用中，如何改变网页的标题呢？

网页标题是通过`<title>`来显示的，但是SPA只有一个固定的HTML，切换不同的页面时，标题并不会改变。

但是我们可以通过JavaScript来修改`<title>`的内容：window.document.title = '新的标题'

那么在Vue项目中，我们比较容易想到的修改标题的位置是每一个路由对应的组件.vue文件中，通过mounted声明周期函数，执行对应的代码进行修改即可。但是当页面比较多时, 这种方式不容易维护(因为需要在多个页面执行类似的代码)。

有没有更好的办法呢? 使用导航守卫即可.

什么是导航守卫?

- vue-router提供的导航守卫主要用来监听路由的进入和离开

- vue-router提供了beforeEach和afterEach的钩子函数，它们会在路由即将改变前和改变后触发

---

我们可以利用beforeEach来完成标题的修改：

- 首先，我们可以在route当中定义一些标题，可以利用meta来定义：

```javascript
const routes = [
  ...
  {
    path: '/about',
    component: About,
    meta: {
      title: "关于"
    },
  }
]
```

- 其次，利用导航守卫，修改我们的标题：

```javascript
router.beforeEach((to, from, next) => {
  document.title = to.matched[0].meta.title
  next()
})
```

导航钩子的三个参数解析：
to：即将要进入的目标的路由对象
from：当前导航即将要离开的路由对象
next：调用该方法后, 才能进入下一个钩子

---

补充一：如果是后置钩子，也就是afterEach，不需要主动调用next()函数。
补充二：上面我们使用的导航守卫，被称之为全局导航守卫。

更多内容，可以查看[官网](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E8%B7%AF%E7%94%B1%E7%8B%AC%E4%BA%AB%E7%9A%84%E5%AE%88%E5%8D%AB)进行学习。


# keep-alive

keep-alive 是 **Vue 内置**的一个组件，可以使被包含的组件保留状态，或避免重新渲染。

它们有两个非常重要的属性：

- include - 字符串或正则表达，只有匹配的组件会被缓存（组件名就是export中的name）
- exclude - 字符串或正则表达式，任何匹配的组件都不会被缓存

router-view 是 **Vue Router** 的一个组件，如果直接被包在 keep-alive 里面，所有路径匹配到的视图组件都会被缓存，不会被销毁，可通过 create 函数或 destroyed 函数验证：

```html
    <keep-alive includ="Home">
      <router-view/>
    </keep-alive>
```

---

在我们上面的案例实现中，当我们访问完消息组件后，再去访问用户组件时，可以通过首页组件的destroyed函数发现首页组件被销毁，再次回到首页组件时，并没有显示我们上次浏览的消息组件，解决途径如下：

1. 我们可以在App.vue中配置keep-alive，使我们首页的组件被缓存而不是销毁。
2. 注释掉原本我们在index.js中配置的首页**子组件中**的默认路由。
3. 在首页组件data中创建path数据，通过activated函数，使每次首页组件**活跃时**跳转到对应的路由。
4. 当首页组件**准备跳转**到其他路由时，通过**组件内的守卫**beforeRouteLeave将当前路由保存到path中。
5. 这样当我们从其他组件返回首页组件时，可以通过activated函数返回到我们上次浏览保存的路由。

`注意：activated函数和deactivated函数只有在组件被keep-alive标签包裹时才生效，同时第一次创建并访问组件时，activated函数也会执行。keep-alive只缓存父组件，如果要使子组件不被销毁，则在父组件中使用keep-alive。`

```javascript
<template>
  <div>
    <h2>我是首页页面</h2>
    <router-link to="/home/news">新闻</router-link>
    <router-link to="/home/messages">消息</router-link>
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      path: '/home/news'
    }
  },
  created() {
    console.log("home create")
  },
  destroyed() {
    console.log("home destroy")
  },
  activated() {
    this.$router.push(this.path)
  },
  beforeRouteLeave(to, from, next) {
    this.path = this.$route.path
    next()
  }
}
</script>

<style scoped>

</style>
```



# TabBar练习

## 思路

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201112143446.png)



如果有一个单独的TabBar组件，你如何封装？

- 自定义TabBar组件，在APP中使用
- 让TabBar位于底部，并且设置相关的样式

- TabBar中显示的内容由外界决定，定义插槽
- flex布局平分TabBar

TabBar组件里通过子组件TabBarItem，可以传入图片和文字：

- 自定义TabBarItem组件，并且定义两个插槽：图片、文字
- 给两个插槽外层包装div，用于设置样式
- 填充插槽，实现底部TabBar的效果
- 由于要传入高亮图片，定义另外一个插槽，插入active-icon的数据
- 定义一个变量isActive，通过v-show来决定是否显示对应的icon

TabBarItem绑定路由数据：

- 安装路由：npm install vue-router —save
- 完成router/index.js的内容，以及创建对应的组件
- main.js中注册router
- APP中加入`<router-view>`组件

点击item跳转到对应路由，并且动态决定isActive：

- 监听item的点击，通过this.$router.replace()替换路由路径
- 通过this.$route.path.indexOf(this.link) !== -1来判断是否是active

动态计算active样式：

- 封装新的计算属性：this.isActive ? {'color': 'red'} : {}

---

## 实现

TabBarItem组件：

```html
<template>
  <div id="tab-bar-item" @click="itemClick">
    <div v-if="!isActive" class="item-icon">
      <slot name="icon"></slot>
    </div>
    <div v-else class="item-active-icon">
      <slot name="active-icon"></slot>
    </div>
    <div class="item-text" :style="activeStyle">
      <slot name="text"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "TabBarItem",
  props: {
    link: {
      type: String,
      required: true
    }
  },
  computed: {
    isActive() {
      return this.$route.path.indexOf(this.link) !== -1
    },
    activeStyle() {
      return this.isActive ? {'color': 'deepPink'} : {}
    }
  },
  methods: {
    itemClick() {
      this.$router.replace(this.link)
    }
  }
}
</script>

<style scoped>
#tab-bar-item {
  /*对flex均等分*/
  flex: 1;
  text-align: center;
  height: 49px;
  font-size: 14px;
}

.item-icon img, .item-active-icon img {
  width: 24px;
  height: 24px;
  margin-top: 3px;
  vertical-align: middle;
  margin-bottom: 2px;
}

.item-text {
  font-size: 12px;
  margin-top: 3px;
  color: #333;
}
</style>
```

TabBar组件：

```html
<template>
  <div id="tab-bar">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "Tabbar"
}
</script>

<style scoped>
#tab-bar {
  display: flex;
  background-color: #f6f6f6;

  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;

  box-shadow: 0 -1px 1px rgba(100, 100, 100, .2);
}
</style>
```

App.vue：

```html
<template>
  <div id="app">
    <router-view></router-view>
    <tab-bar>
      <tab-bar-item link="/home">
        <template #icon>
          <img src="./assets/img/tabbar/home.svg">
        </template>
        <template #active-icon>
          <img src="./assets/img/tabbar/home_active.svg">
        </template>
        <template #text>
          <div>首页</div>
        </template>
      </tab-bar-item>
      <tab-bar-item link="/category">
        <template #icon>
          <img src="./assets/img/tabbar/category.svg">
        </template>
        <template #active-icon>
          <img src="./assets/img/tabbar/category_active.svg">
        </template>
        <template #text>
          <div>分类</div>
        </template>
      </tab-bar-item>
      <tab-bar-item link="/cart">
        <template #icon>
          <img src="./assets/img/tabbar/shopcart.svg">
        </template>
        <template #active-icon>
          <img src="./assets/img/tabbar/shopcart_active.svg">
        </template>
        <template #text>
          <div>购物车</div>
        </template>
      </tab-bar-item>
      <tab-bar-item link="/profile">
        <template #icon>
          <img src="./assets/img/tabbar/profile.svg">
        </template>
        <template #active-icon>
          <img src="./assets/img/tabbar/profile_active.svg">
        </template>
        <template #text>
          <div>我的</div>
        </template>
      </tab-bar-item>
    </tab-bar>
  </div>
</template>

<script>
import TabBar from "./components/tabbar/TabBar";
import TabBarItem from "./components/tabbar/TabBarItem";

export default {
  name: 'App',
  components: {
    TabBar,
    TabBarItem
  }
}
</script>

<style>
@import "assets/css/base.less";
</style>

```

