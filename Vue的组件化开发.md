---
title: Vue的组件化开发
tags:
  - 组件化
categories:
  - Vue
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp5.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp5.jpg'
abbrlink: '7098'
date: 2020-10-30 11:23:42
updated: 2020-10-30 11:23:48
---



# 认识组件化

## 什么是组件化

- 人面对复杂问题的处理方式：
  - 任何一个人处理信息的逻辑能力都是有限的，所以当面对一个非常复杂的问题时，我们不太可能一次性搞定它。但是我们人有一种天生的能力，就是将问题进行拆解。
  - 如果将一个复杂的问题拆分成很多个可以处理的小问题，再将其放在整体当中，你会发现大的问题也会迎刃而解。类似分治算法。
- 组件化也是类似的思想：
  - 如果我们将一个页面中所有的处理逻辑全部放在一起，处理起来就会变得非常复杂，而且不利于后续的管理以及扩展。
  - 但如果我们将一个页面拆分成一个个小的功能块，每个功能块完成属于自己这部分独立的功能，那么之后整个页面的管理和维护就变得非常容易了。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201025155124.png)

## Vue组件化思想

组件化是Vue.js中的重要思想。它提供了一种抽象，让我们可以开发出一个个独立可复用的小组件来构造我们的应用。

任何的应用都会被抽象成一颗**组件树**。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201025155348.png)

组件化思想的应用：

- 有了组件化的思想，我们在之后的开发中就要充分的利用它。
- 尽可能的将页面拆分成一个个小的、可复用的组件。
- 这样让我们的代码更加方便组织和管理，并且扩展性也更强。



# 组件化的实现

## 基本步骤

组件的使用分成三个步骤：

1. 创建组件构造器
2. 注册组件
3. 使用组件

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201025160121.png)

```html
<div id="app">
  <!--3.使用组件-->
  <my-cpn></my-cpn>
  <my-cpn></my-cpn>
  <my-cpn></my-cpn>
  <my-cpn></my-cpn>
</div>

<script src="../js/vue.js"></script>
<script>
  // 1.创建组件构造器对象
  const cpnC = Vue.extend({
    template: `
      <div>
        <h2>我是标题</h2>
        <p>我是内容, 哈哈哈哈</p>
        <p>我是内容, 呵呵呵呵</p>
      </div>`
  })

  // 2.注册组件
  Vue.component('my-cpn', cpnC)

  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    }
  })
</script>
```

和直接使用一个div看起来并没有什么区别。

但是我们可以设想，如果很多地方都要显示这样的信息，我们就可以直接使用`<my-cpn></my-cpn>`来完成。



## 步骤解析

（一）Vue.extend()：

- 调用Vue.extend()创建的是一个组件构造器。 
- 通常在创建组件构造器时，传入template代表我们自定义组件的模板。该模板就是在使用到组件的地方，要显示的HTML代码。
- 事实上，这种写法在Vue2.x的文档中几乎已经看不到了，它会直接使用下面我们会讲到的语法糖，但是在很多资料还是会提到这种方式，而且这种方式是学习后面方式的基础。

（二）Vue.component()：

- 调用Vue.component()是将刚才的组件构造器注册为一个组件，并且给它起一个组件的标签名称。
- 所以需要传递两个参数：1.注册组件的标签名 2.组件构造器

（三）**组件必须挂载在某个Vue实例下**，否则它不会生效。

​        如下图所示，使用了三次`<my-cpn></my-cpn>`，而第三次其实并没有生效：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201025160745.png)

# 全局组件和局部组件

当我们调用Vue.component()注册组件时，组件的注册是全局的，这意味着该组件可以在任意Vue实例下使用。

如果我们注册的组件是挂载在某个实例中, 那么就是一个局部组件，只能在该实例下使用。

```html
<div id="app">
  <cpn></cpn>
</div>

<div id="app2">
  <cpn></cpn>
</div>

<script src="../js/vue.js"></script>
<script>
  // 1.创建组件构造器
  const cpnC = Vue.extend({
    template: `
      <div>
        <h2>我是标题</h2>
        <p>我是内容,哈哈哈哈啊</p>
      </div>
    `
  })

  // 2.注册组件(全局组件, 意味着可以在多个Vue的实例下面使用)
  // Vue.component('cpn', cpnC)

  const app = new Vue({
    el: '#app',
    components: {
      // 局部组件
      // cpn表示使用组件时的标签名，cpnC表示组件构造器
      cpn: cpnC
    }
  })

  const app2 = new Vue({
    el: '#app2'
  })
</script>
```



# 父组件和子组件

在前面我们看到了组件树：组件和组件之间存在层级关系，而其中一种非常重要的关系就是父子组件的关系。
我们来看通过代码如何组成的这种层级关系：

```html
<div id="app">
  <cpn2></cpn2>
  <!--<cpn1></cpn1>使用不了，会报错，因为没有全局注册或在Vue实例中注册该组件-->
</div>

<script src="../js/vue.js"></script>
<script>
  // 1.创建第一个组件构造器(子组件)
  const cpnC1 = Vue.extend({
    template: `
      <div>
        <h2>我是标题1</h2>
        <p>我是内容, 哈哈哈哈</p>
      </div>
    `
  })


  // 2.创建第二个组件构造器(父组件)
  const cpnC2 = Vue.extend({
    template: `
      <div>
        <h2>我是标题2</h2>
        <p>我是内容, 呵呵呵呵</p>
        <cpn1></cpn1>
      </div>
    `,
    components: {
      cpn1: cpnC1
    }
  })

  // root组件
  const app = new Vue({
    el: '#app',
    components: {
      cpn2: cpnC2
    }
  })
</script>
```

父子组件错误用法：以子标签的形式在Vue实例中使用

因为当子组件注册到父组件的components时，Vue会编译好父组件的模块，该模板的内容已经决定了父组件将要渲染的HTML（相当于父组件中已经有了子组件中的内容了）

`<child-cpn></child-cpn>`是只能在父组件中被识别的。

类似这种用法，`<child-cpn></child-cpn>`是会被浏览器忽略的。



# 注册组件语法糖

在上面注册组件的方式，可能会有些繁琐。Vue为了简化这个过程，提供了注册的语法糖。

主要是省去了调用Vue.extend()的步骤，而是可以直接使用一个对象来代替。

```html
<div id="app">
    <cpn1></cpn1>
    <cpn2></cpn2>
</div>

<script src="../js/vue.js"></script>
<script>

  // 创建组件构造器
  // const cpn1 = Vue.extend()

  // 1.全局组件注册的语法糖
  Vue.component('cpn1', {
    template: `
      <div>
        <h2>我是标题1</h2>
        <p>我是内容, 哈哈哈哈</p>
      </div>
    `
  })

  // 2.注册局部组件的语法糖
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    components: {
      'cpn2': {
        template: `
          <div>
            <h2>我是标题2</h2>
            <p>我是内容, 呵呵呵</p>
          </div>
    `
      }
    }
  })
</script>
```



# 模板的分离写法

刚才我们通过语法糖简化了Vue组件的注册过程，另外还有一个地方的写法比较麻烦，就是template模块写法。

如果我们能将其中的HTML分离出来写，然后挂载到对应的组件上，必然结构会变得非常清晰。

Vue提供了两种方案来定义HTML模块内容：

- 使用`<script>`标签
- 使用`<template>`标签

```html
<div id="app">
    <cpn1></cpn1>
    <cpn2></cpn2>
</div>

<!--1.script标签, 注意:类型必须是text/x-template-->
<script type="text/x-template" id="cpn1">
    <div>
        <h2>我是标题</h2>
        <p>我是内容,哈哈哈</p>
    </div>
</script>

<!--2.template标签-->
<template id="cpn2">
    <div>
        <h2>我是标题</h2>
        <p>我是内容,呵呵呵</p>
    </div>
</template>

<script src="../js/vue.js"></script>
<script>
  Vue.component('cpn1', {
    template: id = "#cpn1"
  })
  Vue.component('cpn2', {
    template: id = "#cpn2"
  })

  const app = new Vue({
    el: '#app',
  })
</script>
```



# 组件的数据存放

组件去访问message，message定义在Vue的data中，我们发现最终并没有显示结果。所以**组件是不能【直接】访问Vue实例中的data数据**。

组件是一个单独功能模块的封装：这个模块有属于自己的HTML模板，也应该有属于自己的数据data，只是这个data属性**必须是一个函数**，而且这个函数返回一个对象，对象内部保存着数据。

```html
<div id="app">
  <cpn></cpn>
  <cpn></cpn>
  <cpn></cpn>
</div>

<template id="cpn">
  <div>
    <h2>{{title}}</h2>
    <p>我是内容,呵呵呵</p>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  
  Vue.component('cpn', {
    template: id = '#cpn',
    data() {
      return {
        title: 'abc'
      }
    }
  })

  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      // title: '我是标题'
    }
  })
</script>
```

---

为什么data在组件中必须是一个函数呢?

- 如果不是一个函数，Vue直接就会报错。
- 其次，原因是在于Vue让每个组件对象都返回一个**新的对象**，因为如果是同一个对象的话，组件在多次使用时会相互影响。

```html
<!--组件实例对象-->
<div id="app">
  <cpn></cpn>
  <cpn></cpn>
  <cpn></cpn>
</div>

<template id="cpn">
  <div>
    <h2>当前计数: {{counter}}</h2>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  // 1.注册组件
  const obj = {
    counter: 0
  }
  Vue.component('cpn', {
    template: id = '#cpn',
    // data() {
    //   return {
    //     counter: 0
    //   }
    // },
    data() {
      return obj
    },
    methods: {
      increment() {
        this.counter++
      },
      decrement() {
        this.counter--
      }
    }
  })

  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    }
  })
</script>
```

这里因为data返回同一个对象，所以在操作一个计数器的加减时，会影响到其他计数器的显示结果。



# 父子组件的通信

在上面我们提到子组件是不能直接引用父组件或者Vue实例的数据的。

但是在开发中，往往有一些数据确实需要从上层传递到下层：

> 比如在一个页面中，我们从服务器请求到了很多的数据，其中一部分数据，并非是我们整个页面的大组件来展示的，而是需要下面的子组件进行展示。这个时候，并不会让子组件再次发送一个网络请求，而是直接让大组件(父组件)将数据传递给小组件(子组件)。

如何进行父子组件间的通信呢？

- 父组件通过props向子组件传递数据

- 子组件通过事件向父组件发送消息

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201027105959.png)



## 父级向子级传递

在组件中，使用**选项props**来声明需要从父级接收到的数据。

props的值有两种方式：

- 字符串数组，数组中的字符串就是传递时的名称。
- 对象，对象**可以设置传递时的类型，也可以设置默认值**等。

```html
<div id="app">
  <cpn :cmessage="message" :cmovies="movies"></cpn>
</div>

<template id="cpn">
  <div>
    <ul>
      <li v-for="item in cmovies">{{item}}</li>
    </ul>
    <h2>{{cmessage}}</h2>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  // 父传子: props
  const cpn = {
    template: id = '#cpn',
    // props: ['cmovies', 'cmessage'],//字符串数组的方式
    props: {//对象的方式
      // 1.类型限制
      // cmovies: Array,
      // cmessage: String,

      // 2.提供一些默认值, 以及是否要求必传
      cmessage: {
        type: String,
        default: 'aaaaaaaa',
        required: true
      },
      // 类型是对象或者数组时, 默认值必须是一个函数
      cmovies: {
        type: Array,
        default() {
          return []
        }
      }
    },
    data() {
      return {}
    },
    methods: {}
  }

  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      movies: ['海王', '海贼王', '海尔兄弟']
    },
    components: {
      cpn
    }
  })
</script>
```

---

当需要对props进行类型等验证时，就需要对象写法了。

验证支持的数据类型：

- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol

当我们有自定义构造函数时，验证也支持自定义的类型

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201028171445.jpg)
![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201028165717.png)



## 子级向父级传递

props用于父组件向子组件传递数据，还有一种比较常见的是子组件传递数据或事件到父组件中。这个时候，我们需要使用自定义事件来完成。

我们之前学习的v-on不仅仅可以用于监听DOM事件，也可以用于监听组件间的自定义事件。

自定义事件的流程：

- 在子组件中，通过$emit()来触发事件。
- 在父组件中，通过v-on来监听子组件事件。

```html
<!--父组件模板-->
<div id="app">
  <cpn @item-click="cpnClick"></cpn>
</div>

<!--子组件模板-->
<template id="cpn">
  <div>
    <button v-for="item in categories"
            @click="btnClick(item)">
      {{item.name}}
    </button>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  const cpn = {
    template: id = '#cpn',
    data() {
      return {
        categories: [
          {id: 'aaa', name: '热门推荐'},
          {id: 'bbb', name: '手机数码'},
          {id: 'ccc', name: '家用家电'},
          {id: 'ddd', name: '电脑办公'},
        ]
      }
    },
    methods: {
      btnClick(item) {
        // 发射事件: 自定义事件
        this.$emit('item-click', item)
      }
    }
  }
  
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    components: {
      cpn
    },
    methods: {
      cpnClick(item) {
        console.log('cpnClick', item);
      }
    }
  })
</script>
```



# 父子组件的访问

## 父组件访问子组件的方式： $children

> 有时父组件需要访问获得子组件的数据和调用子组件的方法。

父组件访问子组件：使用$children或$refs （reference：引用）

我们先来看下$children的访问，this.$children是一个**数组类型，它包含所有子组件对象**。

```html
<div id="app">
  <cpn></cpn>
  <cpn></cpn>
  <button @click="btnClick">按钮</button>
</div>

<template id="cpn">
  <div>{{message}}</div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    methods: {
      btnClick() {
        console.log(this.$children)
        console.log(this.$children[0].message);
        this.$children[0].showMessage()
      }
    },
    components: {
      cpn: {
        template: id = '#cpn',
        data() {
          return {
            message: '我是子组件'
          }
        },
        methods: {
          showMessage() {
            console.log('showMessage')
          }
        }
      }
    }
  })
</script>
```

点击按钮后的运行结果：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201029005840.png)

## 父组件访问子组件的方式： $refs

$children的缺陷：

> 通过$children访问子组件时，是一个数组类型，访问其中的子组件必须通过索引值。
>
> 但是当子组件过多，我们需要拿到其中一个时，往往不能确定它的索引值，甚至其索引值还可能会发生变化。
>
> 有时候，我们想明确获取其中一个特定的组件，这个时候就可以使用$refs

$refs的使用：

- $refs和ref指令通常是一起使用的。
- 首先，我们通过ref给某一个子组件绑定一个特定的ID。
- 其次，通过this.$refs.ID就可以访问到该组件了。

```html
<div id="app">
  <cpn></cpn>
  <cpn></cpn>
  <cpn ref="abc"></cpn>
  <button @click="btnClick">按钮</button>
</div>

<template id="cpn">
  <div>{{message}}</div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {},
    methods: {
      btnClick() {
        console.log(this.$refs.abc.message)
        this.$refs.abc.showMessage()
      }
    },
    components: {
      cpn: {
        template: id = '#cpn',
        data() {
          return {
            message: '我是子组件'
          }
        },
        methods: {
          showMessage() {
            console.log('showMessage')
          }
        }
      }
    }
  })
</script>
```



## 子组件访问父组件或根组件

如果我们想在子组件中直接访问父组件，可以通过$parent

如果我们想在子组件中直接访问根组件，可以通过$root

注意事项：

> - 尽管在Vue开发中，我们允许通过$parent来访问父组件，但是在真实开发中尽量不要这样做。
> - 子组件应该尽量避免直接访问父组件的数据，因为这样耦合度太高了。
> - 如果我们将子组件放在另外一个组件之内，很可能该父组件没有对应的属性，往往会引起问题。
> - 另外，更不好做的是通过$parent直接修改父组件的状态，那么父组件中的状态将变得飘忽不定，很不利于我们的调试和维护。

```html
<div id="app">
  <cpn></cpn>
</div>

<template id="cpn">
  <div>
    <h2>我是cpn组件</h2>
    <ccpn></ccpn>
  </div>
</template>

<template id="ccpn">
  <div>
    <h2>我是cpn组件的子组件</h2>
    <button @click="btnClick">按钮</button>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    components: {
      cpn: {
        template: id = '#cpn',
        data() {
          return {
            name: '我是cpn组件的name'
          }
        },
        components: {
          ccpn: {
            template: id = '#ccpn',
            methods: {
              btnClick() {
                // 1.访问父组件$parent
                // console.log(this.$parent);
                // console.log(this.$parent.name);

                // 2.访问根组件$root
                console.log(this.$root);
                console.log(this.$root.message);
              }
            }
          }
        }
      }
    }
  })
</script>
```



# 插槽slot

## 为什么使用slot

slot翻译为插槽：

- 在生活中很多地方都有插槽，电脑的USB插槽，插板当中的电源插槽。
- 插槽的目的是让我们原来的设备具备更多的**扩展性**。
- 比如电脑的USB我们可以插入U盘、硬盘、手机、音响、键盘、鼠标等等。

组件的插槽：组件的插槽也是为了让我们封装的组件更加具有扩展性。让使用者可以决定组件内部的一些内容到底展示什么。

例子：移动网站中的导航栏

移动开发中，几乎每个页面都有导航栏。导航栏我们必然会封装成一个插件，比如nav-bar组件。一旦有了这个组件，我们就可以在多个页面中复用了。但是，每个页面的导航是一样的吗？No，以京东M站为例：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201029175544.png)

---

如何封装合适呢？**抽取共性，保留不同**。

最好的封装方式就是将共性抽取到组件中，将不同保留为插槽。

一旦我们预留了插槽，就可以让使用者根据自己的需求，决定插槽中插入什么内容。

是搜索框，还是文字，还是菜单。由调用者自己来决定。

这就是为什么我们要学习组件中的插槽slot的原因。



## slot基本使用

在子组件中使用特殊的元素`<slot>`就可以为子组件开启一个插槽。该插槽插入什么内容取决于父组件如何使用。

`<slot>`中的内容表示，如果没有在该组件中插入任何其他内容，就默认显示该内容，也称**后备内容**。

```html
<!--
1.插槽的基本使用 <slot></slot>
2.插槽的默认值 <slot>button</slot>
3.如果有多个值, 同时放入到组件进行替换时, 一起作为替换元素
-->

<div id="app">
  <cpn></cpn>

  <cpn><span>哈哈哈</span></cpn>

  <cpn><i>呵呵呵</i></cpn>

  <cpn>
    <i>呵呵呵</i>
    <div>我是div元素</div>
    <p>我是p元素</p>
  </cpn>

</div>


<template id="cpn">
  <div>
    <h2>我是组件</h2>
    <slot>
      <button>按钮</button>
    </slot>
  </div>
</template>
```

页面如下：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201029180534.png)

## 具名插槽

> 当子组件的功能复杂时，子组件的插槽可能并非是一个。
>
> 比如我们封装一个导航栏的子组件，可能就需要三个插槽，分别代表左边、中间、右边。
>
> 那么，外面在给插槽插入内容时，如何区分插入的是哪一个呢？
>
> 这个时候，我们就需要给插槽起一个名字，也就是使用具名插槽。

只要给slot元素一个name属性即可`<slot name='myslot'></slot>`

一个不带 `name` 的 `<slot>` 出口会带有隐含的名字“default”。

在向具名插槽提供内容的时候，我们可以在一个 `<template>` 元素上使用 `v-slot` 指令，并以 `v-slot` 的参数的形式提供其名称。

跟 `v-on` 和 `v-bind` 一样，`v-slot` 也有缩写，即把参数之前的所有内容 (`v-slot:`) 替换为字符 `#`。例如 `v-slot:header` 可以被重写为 `#header`

```html
<div id="app">
  <cpn>
    <!--自 2.6.0 起引入v-slot-->
    <template v-slot:center>
      <span>标题</span>
    </template>
  </cpn>
  <cpn>
    <template v-slot:default>
      <span>这是默认插槽的内容</span>
    </template>
  </cpn>
  <cpn>任何没有被包裹在带有 v-slot 的template标签中的内容都会被视为默认插槽的内容。</cpn>
  <cpn>
    <!--自 2.6.0 起被废弃的旧语法-->
    <button slot="left">返回</button>
  </cpn>
</div>


<template id="cpn">
  <div>
    <slot name="left"><span>左边</span></slot>
    <slot name="center"><span>中间</span></slot>
    <slot name="right"><span>右边</span></slot>
    <slot></slot>
  </div>
</template>
```

页面如下：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20201030094511.png)



## 编译作用域

```html
<div id="app">
  <cpn v-show="isShow"></cpn>
</div>

<template id="cpn">
  <div>
    <h2>我是子组件</h2>
    <p>我是内容, 哈哈哈</p>
    <button v-show="isShow">按钮</button>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      isShow: true
    },
    components: {
      cpn: {
        template: id = '#cpn',
        data() {
          return {
            isShow: false
          }
        }
      },
    }
  })
</script>
```

我们来考虑下面的代码是否最终是可以渲染出来的：
`<cpn v-show="isShow"></cpn>`中，我们使用了isShow属性，该属性既包含在组件中，也包含在Vue实例中。

答案：最终可以渲染出来，也就是使用的是Vue实例的属性。

官方给出了一条准则：**父组件模板的所有东西都会在父级作用域内编译；子组件模板的所有东西都会在子级作用域内编译。**

而我们在使用`<cpn v-show="isShow"></cpn>`的时候，整个组件的使用过程是相当于在父组件中出现的。那么它的作用域就是父组件，使用的属性也是属于父组件的属性。因此，isShow使用的是Vue实例中的属性，而不是子组件的属性。



## 作用域插槽

有时插槽内容能够访问子组件中才有的数据是很有用的，这时需要用到作用域插槽。

作用域插槽一般用于：内容在子组件，希望父组件告诉我们如何展示。

例如，一个带有如下模板的 `<cpn>` 组件：

```html
<template id="cpn">
  <div>
    <slot>
      {{user.lastName}}
    </slot>
  </div>
</template>
```

我们可能想换掉备用内容，用名而非姓（firstName：名；lastName：姓）来显示。如下：

```html
<cpn>
  {{ user.firstName }}
</cpn>
```

然而上述代码不会正常工作，因为只有 `<cpn>` 组件可以访问到 `user` ，而我们提供的内容是在父级渲染的（编译作用域）。

为了让 `user` 在父级的插槽内容中可用，我们可以将 `user` 作为 `<slot>` 元素的一个 attribute（任意命名，注意不与其他属性名冲突）绑定上去：

```html
<template id="cpn">
  <div>
    <slot :n="user">
      {{user.firstName}}
    </slot>
  </div>
</template>
```

绑定在 `<slot>` 元素上的 attribute 被称为**插槽 prop**，可以有多个。

现在在父级作用域中，我们可以使用带值的 `v-slot` 来定义我们提供的插槽 prop 的名字：

```html
<cpn>
    <template v-slot:default="slotProps">
        <span>{{slotProps.n.lastName}}</span>
    </template>
</cpn>
```

在这个例子中，我们选择将**包含所有插槽 prop 的对象**命名为 `slotProps`，但你也可以使用任意你喜欢的名字。

```html
<div id="app">
  <cpn></cpn>
  
  <!--推荐-->
  <cpn>
    <template v-slot:default="slotProps">
      <span>{{slotProps.n.lastName}}</span>
    </template>
  </cpn>

  <!--自 2.6.0 起被废弃的旧语法-->
  <cpn>
    <template slot-scope="slot">
      <span>{{slot.n.lastName}}</span>
    </template>
  </cpn>
</div>

<template id="cpn">
  <div>
    <slot :n="user">
      {{user.firstName}}
    </slot>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    components: {
      cpn: {
        template: id = '#cpn',
        data() {
          return {
            user: {
              firstName: "Lebron",
              lastName: "James"
            }
          }
        }
      }
    }
  })
</script>
```

