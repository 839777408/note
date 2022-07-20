---
title: Vuex详解
tags:
  - Vuex
  - Promise
categories:
  - Vue
top_img: 'https://unpkg.com/nan-picture/img/wp10.jpg'
cover: 'https://unpkg.com/nan-picture/img/wp10.jpg'
abbrlink: 19dc
date: 2020-11-16 00:47:48
updated: 2020-11-16 00:47:56
---

# Promise

Promise 对象代表了未来将要发生的事件，用来传递**异步操作**的消息。

在CSDN找了一篇比较不错的博客：[ES6 Promise用法小结](https://blog.csdn.net/qq_34645412/article/details/81170576?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

补充：缩写

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>

<script>
  // wrapped into
  // 网络请求: aaa -> 自己处理(10行)
  // 处理: aaa111 -> 自己处理(10行)
  // 处理: aaa111222 -> 自己处理
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('aaa')
    }, 1000)
  }).then(res => {
    console.log(res, '第一层的10行处理代码');
    return new Promise((resolve, reject) => {
      // resolve(res + '111')
      reject('err')
    })
  }).then(res => {
    console.log(res, '第二层的10行处理代码');
    return new Promise(resolve => {
      resolve(res + '222')
    })
  }).then(res => {
    console.log(res, '第三层的10行处理代码');
  }).catch(err => {
    console.log(err);
  })

  // new Promise(resolve => resolve(结果))简写
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('aaa')
    }, 1000)
  }).then(res => {
    console.log(res, '第一层的10行处理代码');
    return Promise.resolve(res + '111')
  }).then(res => {
    console.log(res, '第二层的10行处理代码');
    return Promise.resolve(res + '222')
  }).then(res => {
    console.log(res, '第三层的10行处理代码');
  })

  // 省略掉Promise.resolve
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('aaa')
    }, 1000)
  }).then(res => {
    console.log(res, '第一层的10行处理代码');
    return res + '111'
  }).then(res => {
    console.log(res, '第二层的10行处理代码');
    return res + '222'
  }).then(res => {
    console.log(res, '第三层的10行处理代码');
  })

  // Promise.reject的缩写
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('aaa')
    }, 1000)
  }).then(res => {
    console.log(res, '第一层的10行处理代码');

    // return Promise.reject('error message')
    throw 'error message'
  }).then(res => {
    console.log(res, '第二层的10行处理代码');
    return Promise.resolve(res + '222')
  }).then(res => {
    console.log(res, '第三层的10行处理代码');
  }).catch(err => {
    console.log(err);
  })
</script>
</body>
</html>
```





# 认识Vuex

## Vuex是做什么的?

**Vuex 是什么？**

官方解释：Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式。**

它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

Vuex 也集成到 Vue 的官方调试工具 **devtools extension**，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

---

**状态管理到底是什么？**

其实，你可以简单的将其看成把需要多个组件共享的变量全部存储在一个对象里面。

然后，将这个对象放在顶层的Vue实例中，让其他组件可以使用，共享这个对象中的所有变量属性。

并且保证对象里面所有的属性做到**响应式**，Vuex就是为了提供这样一个在多个组件间共享状态的插件。

---

**有什么状态是需要我们在多个组件间共享的呢？**

如果你做过大型开发应用，你一定遇到过多个状态在多个界面间的共享问题。

比如用户的登录状态、用户名称、头像、地理位置信息等等。

比如商品的收藏、购物车中的物品等等。

这些状态信息，我们都可以放在统一的地方，对它进行保存和管理，而且它们还是响应式的。





## 单界面的状态管理

![](https://unpkg.com/nan-picture/blog/20201115061526.png)

- State：我们的状态。（姑且可以当做就是data中的属性）
- View：视图层，可以针对State的变化显示不同的信息。
- Actions：主要是用户的各种操作：点击、输入等，会导致状态的改变。

>类似我们计数器案例中（https://nanzx.top/posts/a5b0/）
>
>count需要某种方式被记录下来，也就是我们的State。
>count目前的值需要被显示在界面中，也就是我们的View。
>界面发生某些操作时（我们这里是用户的点击），需要去更新状态，也就是我们的Actions。



## 多界面的状态管理

- Vue已经帮我们做好了单个界面的状态管理，但是如果是多个界面呢？
  - 多个视图都依赖同一个状态（一个状态改了，多个界面需要进行更新）
  - 不同界面的Actions都想修改同一个状态（Home.vue需要修改，Profile.vue也需要修改这个状态）
- 也就是说对于某些状态(状态1/状态2/状态3)来说只属于我们某一个视图，但是也有一些状态(状态a/状态b/状态c)属于多个视图共同想要维护的。
  - 状态1/状态2/状态3你放在自己的房间中，你自己管理自己用，没问题。
  - 但是状态a/状态b/状态c我们希望交给一个大管家来统一帮助我们管理！！！
  - 没错，Vuex就是为我们提供这个大管家的工具。
- 全局单例模式（大管家）
  - 我们现在要做的就是将共享的状态抽取出来，交给我们的大管家统一进行管理。
  - 之后，你们每个视图按照我规定好的规定进行访问和修改等操作。
  - 这就是Vuex背后的基本思想。



## Vuex状态管理图例

![](https://unpkg.com/nan-picture/blog/20201115063220.png)



# Vuex的基本使用

我们可以在使用Vue CLI创建项目时安装vuex或者通过命令安装：`npm install vuex --save`

 我们通过vuex实现一下之前简单的案例：计数器。

## （一）创建store实例

在src/store/index.js中配置如下：

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {},
  getters: {},
  mutations: {},
  actions: {},
  modules: {}
})
```

## （二）挂载到Vue实例中

在src/main.js中配置如下：(这样在其他Vue组件中，我们就可以通过this.$store的方式，获取到这个store对象了)

```javascript
import Vue from 'vue'
import App from './App.vue'
import store from './store'

Vue.config.productionTip = false

new Vue({
  store,
  render: h => h(App)
}).$mount('#app')
```

## （三）配置state和mutations

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    count: 1000
  },
  getters: {},
  mutations: {
    add(state) {
      state.count++;
    },
    subtract(state) {
      state.count--;
    }
  },
  actions: {},
  modules: {}
})
```

## （四）使用vuex

在src/App.vue中配置如下：

```html
<template>
  <div id="app">
    <h2>{{ this.$store.state.count }}</h2>
    <button @click="addition">+</button>
    <button @click="substraction">-</button>
  </div>
</template>

<script>
export default {
  name: "App",
  methods: {
    addition() {
      this.$store.commit('add')
    },
    substraction() {
      this.$store.commit('subtract')
    }
  }
}
</script>

<style>

</style>

```

## （五）小结

1. 提取出一个公共的store对象，用于保存在多个组件中共享的状态
2. 将store对象放置在new Vue对象中，这样可以保证在所有的组件中都可以使用到
3. 在其他组件中使用store对象中保存的状态即可
   - 通过this.$store.state.属性的方式来访问状态
   - 通过this.$store.commit('mutation中方法')来修改状态

> **注意事项：**
> 我们通过提交mutation的方式改变state里的状态属性，而非直接改变store.state.count的值。
> 这是因为Vuex可以通过提交mutation的方式更明确的追踪状态的变化。



# State

通过`{{ this.$store.state.info }}`获取state。

Vuex提出使用单一状态树，英文名称是Single Source of Truth，也可以翻译成单一数据源。

用一个生活中的例子做一个简单的类比：

- 我们知道，在国内我们有很多的信息需要被记录，比如上学时的个人档案，工作后的社保记录，公积金记录，结婚后的婚姻信息，以及其他相关的户口、医疗、文凭、房产记录等等（还有很多信息）。
- 这些信息被分散在很多地方进行管理，有一天你需要办某个业务时(比如入户某个城市)，你会发现你需要到各个对应的工作地点去打印、盖章各种资料信息，最后到一个地方提交证明你的信息无误。
- 这种保存信息的方案，不仅仅低效，而且不方便管理，以及日后的维护也是一个庞大的工作(需要大量的各个部门的人力来维护，当然国家目前已经在完善我们的这个系统了)。

这个和我们在应用开发中比较类似：

- 如果你的状态信息是保存到多个Store对象中的，那么之后的管理和维护等等都会变得特别困难。
- 所以Vuex也使用了单一状态树来管理应用层级的全部状态。
- 单一状态树能够让我们最直接地找到某个状态的片段，而且在之后的维护和调试过程中，也可以非常方便地管理和维护。



# Getters

通过`{{ this.$store.getters.moreAgeStu }}`获取getters。传参在后面加括号传递。

有时候我们需要从store中获取一些state派生后的状态，可以通过getters（可以认为是 store 的计算属性，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。），比如下面的Store中：

- 获取所有年龄大于24岁的学生列表：`more24Stu(state)`

- 如果我们已经有了一个获取所有年龄大于24岁的学生列表的getters, 那么获取年龄大于24岁的学生个数可以这样来写：`more24stuLen(state, getters) `

- getters**默认是不能传递参数的**, 如果希望传递参数, 那么只能让getters本身返回另一个函数：

  `moreAgeStu(state) {return function (age) {`

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    students: [
      {id: 1, name: 'nan', age: 14},
      {id: 2, name: 'james', age: 23},
      {id: 3, name: 'kobe', age: 24},
      {id: 4, name: 'curry', age: 30},
    ]
  },
  getters: {
    more24Stu(state) {
      return state.students.filter(s => s.age > 24)
    },
    more24stuLen(state, getters) {
      return getters.more24Stu.length
    },
    moreAgeStu(state) {
      return function (age) {
        return state.students.filter(s => s.age > age)
      }
    }
  },
  mutations: {},
  actions: {},
  modules: {}
})

```



# Mutation

## 状态更新

Vuex的store状态的更新**唯一**方式：提交Mutation。

Mutation主要包括两部分：①字符串的事件类型（type）②一个回调函数（handler），该回调函数的第一个参数就是state。

- mutations的定义方式：

```javascript
  mutations: {
    add(state) {
      state.count++;
    }
  }
```

- 通过mutations更新：

```javascript
  addition() {
    this.$store.commit('add')
  }
```

## 传递参数

在通过mutation更新数据时，我们可能希望携带一些额外的参数，这些参数被称为是mutation的载荷(Payload)。

Mutation中的代码：

```javascript
  mutations: {
    add(state, n) {
      state.count += n;
    }
  }

  addition() {
    this.$store.commit('add',2)
  }
```

如果我们有多个参数需要传递，通常会以对象的形式传递，也就是payload是一个对象，然后再从对象中取出相关的信息。

## 提交风格

上面的通过commit进行提交是一种普通的方式，Vue还提供了另外一种风格，它是一个包含type属性的对象。

```js
  addition() {
     this.$store.commit({
 		type: 'increment',
 		amount: 10
     })
  }
```

当使用对象风格的提交方式，整个对象都作为载荷传给 mutations 函数，因此 handler 保持不变：

```js
    mutations: {
      increment (state, payload) {
        state.count += payload.amount
      }
    }
```

## 响应规则

Vuex的store中的state是**响应式**的，当state中的数据发生改变时，Vue组件会自动更新。

这就要求我们必须遵守一些Vuex对应的规则：提前在store中**初始化**好所需的属性。

当给state中的对象添加新属性时，使用下面的方式：

1. 使用Vue.set(obj, 'newProp', 123)
2. 用新对象给旧对象重新赋值：`state.info={...state.info,'height':payload.height}`

我们来看一个例子：

```js
export default new Vuex.Store({
  state: {
    info: {
      name: 'nan',
      age: 14
    }
  },
  mutations: {
    updateInformation: function (state) {
      state.info.age = 23//可以直接响应

      // state.info['address'] = '广州'//不可以直接响应

      // Vue.set(state.info, 'address', '广州')//可以直接响应

      // delete state.info.age//不可以直接响应

      // Vue.delete(state.info, 'age')//可以直接响应
    }
  }
})
```



## 常量类型

我们来考虑下面的问题：

- 在mutations中，我们定义了很多事件类型(也就是其中的方法名称)。
- 当我们的项目增大时，Vuex管理的状态越来越多，需要更新状态的情况越来越多，那么意味着Mutation中的方法越来越多。
- 方法过多，使用者需要花费大量的经历去记住这些方法，甚至是多个文件间来回切换查看方法名称，甚至如果不是复制的时候，可能还会出现写错的情况。

如何避免上述的问题呢？

- 在各种Flux实现中，一种很常见的方案就是使用**常量**替代Mutation事件的类型。
- 我们可以将这些常量放在一个单独的文件中，方便管理以及让整个app所有的事件类型一目了然。

具体怎么做呢？

- 我们可以创建一个文件：mutation-types.js，并且在其中定义我们的常量。
- 定义常量时，我们可以使用ES2015中的风格，使用一个常量来作为函数的名称。

```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```

```js
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

用不用常量取决于你——在需要多人协作的大型项目中，这会很有帮助。但如果你不喜欢，你完全可以不这样做。



## 同步函数

通常情况下，Vuex要求我们Mutation中的方法**必须是同步方法**。

主要的原因是当我们使用devtools时，devtools可以帮助我们捕捉mutations的快照。

但是如果是异步操作，那么devtools将不能很好的追踪这个操作什么时候会被完成。

比如我们之前的代码，当执行更新时，devtools中会有如下信息：

![](https://unpkg.com/nan-picture/blog/20201115214308.png)

但是，如果Vuex中的代码，我们使用了异步函数：

![](https://unpkg.com/nan-picture/blog/20201115214502.png)

你会发现state中的info数据一直没有被改变，因为它无法被追踪到。所以通常情况下，不要在mutations中进行异步的操作。



# Action

我们强调，不要再Mutation中进行异步操作。

但是某些情况，我们确实希望在Vuex中进行一些异步操作，比如网络请求必然是异步的。这个时候怎么处理呢？

Action类似于Mutation，但是是用来代替Mutation进行异步操作的。

Action的基本使用代码如下：

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      setTimeout(() => {
      	commit('increment')
      }, 1000)
    }
  }
})
```

context是一个和store对象具有相同方法和属性的对象。

也就是说，我们可以通过context去进行commit相关的操作，也可以获取context.state等。

但是注意，这里它们**并不是同一个对象**，我们后面学习Modules的时候再具体说。

---

在Vue组件中，如果我们调用action中的方法，那么就需要使用**dispatch**。同样的，也是支持传递payload的。

![](https://unpkg.com/nan-picture/blog/20201115215703.png)

---

在Action中，我们可以将异步操作放在一个Promise中，并且在成功或者失败后，调用对应的resolve或reject。

![](https://unpkg.com/nan-picture/blog/20201115222745.png)



#  Module

## 认识Module

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。

为了解决以上问题，Vuex 允许我们将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutations、actions、getters、甚至是嵌套子模块——从上至下进行同样方式的分割：

```js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a.属性 // -> moduleA 的状态
store.state.b.属性 // -> moduleB 的状态
mutations、actions、getters的调用方法照旧
```



## 局部状态

对于模块内部的 mutations 和 getters，接收的第一个参数是**模块的局部状态对象**。

```js
const moduleA = {
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      // 这里的 `state` 对象是模块的局部状态
      state.count++
    }
  },

  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  }
}
```

**注意：**虽然我们的doubleCount和increment都是定义在对象内部的，但是在调用的时候，依然是通过this.$store来直接调用的。

---

同样，对于模块内部的 action，局部状态通过 `context.state` 暴露出来，**根节点状态**则为 `context.rootState`。

而`{ state, commit, rootState }`其实就是该模块的context对象的解构（ES6），也可直接用context：

![](https://unpkg.com/nan-picture/blog/20201116003657.png)

![](https://unpkg.com/nan-picture/blog/20201116004418.png)

数组也可以解构：

![](https://unpkg.com/nan-picture/blog/20201116091310.png)

```js
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
```

对于模块内部的 getter，**根节点状态**会作为第三个参数暴露出来：

```js
const moduleA = {
  // ...
  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
}
```



# 项目结构

对于大型应用，我们会希望把 Vuex 相关代码分割到模块中。下面是项目结构示例：

```bash
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```

