---
title: Vue基础语法
tags:
  - Vue
categories:
  - Vue
top_img: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp4.jpg'
cover: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp4.jpg'
abbrlink: '4159'
date: 2020-10-25 14:18:36
updated: 2020-10-25 14:18:40
---

# 插值操作

## Mustache

Mustache语法(也就是双大括号)。Mustache 标签将会被替代为对应数据对象上的值。

无论何时，绑定的数据对象上值发生了改变，插值处的内容都会更新，数据是响应式的。

Mustache: 胡子/胡须

```html
<div id="app">
    <h2>{{message}}</h2>
    <h2>{{message}},阿楠</h2>
    <h2>{{firstName + lastName}}</h2>
    <h2>{{firstName + ' ' + lastName}}</h2>
    <h2>{{firstName}} {{lastName}}</h2>
    <h2>{{count * 2}}</h2>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '下午好',
            firstName: 'Lebron',
            lastName: 'James',
            count: 100
        }
    })
</script>
```



## v-once

该指令后面不需要跟任何表达式(比如之前的v-for后面是跟表达式的)。

该指令表示元素和组件只渲染一次，不会随着数据的改变而改变。(留心这会影响到该节点上的其它数据绑定)

```html
<div id="app">
    <h2>{{message}}</h2>
    <h2 v-once>{{message}}</h2>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '下午好，阿楠'
        }
    })
</script>
```

页面展示如下：

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20201013152521.png)



## v-html

某些情况下，我们从服务器请求到的数据本身就是一个HTML代码，如果我们直接通过Mustache语法来输出，会将HTML代码也一起输出。

但是我们可能希望的是按照HTML格式进行解析，并且显示对应的内容，我们可以使用 v-html 指令。

该指令后面往往会跟上一个string类型，会将string的html解析出来并且进行渲染。

```html
<div id="app">
    <h2>{{url}}</h2>
    <h2 v-html="url"></h2>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            url: '<a href="https://nanzx.top">阿楠的博客</a>'
        }
    })
</script>
```

页面展示如下：

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20201013152543.png)



## v-text

v-text作用和Mustache比较相似：都是用于将数据显示在界面中

v-text通常情况下，接受一个string类型

```html
<div id="app">
    <h2>{{message}}，詹姆斯FMVP</h2>
    <h2 v-text="message">，詹姆斯FMVP</h2>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '湖人总冠军'
        }
    })
</script>
```

页面展示如下：

湖人总冠军，詹姆斯FMVP

湖人总冠军



## v-pre

v-pre用于跳过这个元素和它子元素的编译过程，用于显示原本的Mustache语法。

第一个h2元素中的内容会被编译解析出来对应的内容

第二个h2元素中会直接显示{{message}}

```html
<div id="app">
    <h2>{{message}}</h2>
    <h2 v-pre>{{message}}</h2>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '湖人总冠军'
        }
    })
</script>
```

页面展示如下：

湖人总冠军

`{{message}}`



## v-cloak

在某些情况下，我们浏览器可能会直接显然出未编译的Mustache标签。

v-cloak 这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 `[v-cloak] { display: none }` 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。

cloak: 斗篷

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        [v-cloak] {
            display: none;
        }
    </style>
</head>
<body>
<div id="app" v-cloak>
    <h2>{{message}}</h2>
</div>
<script src="../js/vue.js"></script>
<script>
    // vue解析之前，div中有一个属性v-cloak，解析后就没有了
    setTimeout(function () {
        const app = new Vue({
            el: '#app',
            data: {
                message: '湖人总冠军'
            }
        })
    }, 1000)
</script>
</body>
</html>
```



# 绑定属性

## v-bind

插值操作是将值插入到我们模板的**内容**当中。

但是，除了内容需要动态来决定外，某些**属性**我们也希望动态来绑定。

比如动态绑定a元素的href属性，动态绑定img元素的src属性，我们都可以使用 v-bind 指令。

v-bind有一个对应的语法糖，也就是**简写方式**，在开发中，我们通常会使用语法糖的形式，因为这样更加简洁。

语法糖： : (属性前加一个冒号)

```html
<div id="app">
    <img v-bind:src="imgUrl">
    <a v-bind:href="aHref">Vue官网</a>
    <!--语法糖格式如下：-->
    <img :src="imgUrl">
    <a :href="aHref">Vue官网</a>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            imgUrl: 'https://cn.vuejs.org/images/logo.png',
            aHref: 'https://cn.vuejs.org/v2/api/#%E6%8C%87%E4%BB%A4'
        }
    })
</script>
```



## v-bind绑定class

很多时候，我们希望动态的来切换class，比如：

- 当数据为某个状态时，字体显示红色。

- 当数据另一个状态时，字体显示黑色。

**绑定class有两种方式：①对象语法   ②数组语法**

---

绑定方式：对象语法

对象语法的含义是：class后面跟的是一个对象。

用法一：直接通过{}绑定一个类
`<h2 :class="{'active': isActive}">Hello World</h2>`

用法二：也可以通过判断，传入多个值
`<h2 :class="{'active': isActive, 'line': isLine}">Hello World</h2>`

用法三：和普通的类同时存在，并不冲突
注：如果isActive和isLine都为true，那么会有title/active/line三个类
`<h2 class="title" :class="{'active': isActive, 'line': isLine}">Hello World</h2>`

用法四：如果过于复杂，可以放在一个methods或者computed中
注：classes是一个计算属性
`<h2 class="title" :class="classes">Hello World</h2>`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .active {
            color: red;
        }
    </style>
</head>
<body>
<div id="app">
    <h2 class="title" :class="{active: isActive,line: isLine}">{{message}}</h2>
    <h2 class="title" :class="getClasses()">{{message}}</h2>
    <button v-on:click="btnClick">按钮</button>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '下午好，阿楠',
            isActive: true,
            isLine: false
        },
        methods:{
            btnClick: function (){
                this.isActive = !this.isActive
            },
            getClasses: function () {
                return {active: this.isActive,line: this.isLine}
            }
        }
    })
</script>
</body>
</html>
```

---

绑定方式：数组语法

数组语法的含义是：class后面跟的是一个数组。

用法一：直接通过[]绑定一个数组
`<h2 :class="['active']">Hello World</h2>`

用法二：也可以传入多个值
`<h2 :class=“[‘active’, 'line']">Hello World</h2>`

用法三：和普通的类同时存在，并不冲突
注：会有title/active/line三个类
`<h2 class="title" :class=“[‘active’, 'line']">Hello World</h2>`

用法四：如果过于复杂，可以放在一个methods或者computed中
注：classes是一个计算属性
`<h2 class="title" :class="classes">Hello World</h2>`

```html
<div id="app">
    <!-- 加引号是字符串 -->
    <h2 class="title" :class="['active','line']">{{message}}</h2>
    <!--没加引号是属性-->
    <h2 class="title" :class="[active,line]">{{message}}</h2>
    <h2 class="title" :class="getClasses()">{{message}}</h2>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '下午好，阿楠',
            active: 'aaa',
            line: 'bbb'
        },
        methods: {
            getClasses: function () {
                return [this.active, this.line]
            }
        }
    })
</script>
```



## v-bind绑定style

我们可以利用v-bind:style来绑定一些CSS内联样式。

在写CSS属性名的时候，比如font-size

- 我们可以使用驼峰式 (camelCase)  fontSize 

- 或短横线分隔 (kebab-case，记得用单引号括起来) ‘font-size’

**绑定style有两种方式：①对象语法   ②数组语法**

---

绑定方式一：对象语法

`:style="{color: currentColor, fontSize: fontSize + 'px'}"`

style后面跟的是一个对象类型

对象的key是CSS属性名称

对象的value是具体赋的值，值可以来自于data中的属性

```html
<div id="app">
    <!--<h2 :style="{key(属性名): value(属性值)}">{{message}}</h2>-->

    <!--'50px'必须加上单引号, 否则是当做一个变量去解析-->
    <!--<h2 :style="{fontSize: '50px'}">{{message}}</h2>-->

    <!--finalSize当成一个变量使用-->
    <!--<h2 :style="{fontSize: finalSize}">{{message}}</h2>-->
    <h2 :style="{fontSize: finalSize + 'px', backgroundColor: finalColor}">{{message}}</h2>
    <h2 :style="getStyles()">{{message}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '你好啊',
            finalSize: 100,
            finalColor: 'red',
        },
        methods: {
            getStyles: function () {
                return {fontSize: this.finalSize + 'px', backgroundColor: this.finalColor}
            }
        }
    })
</script>
```

---

绑定方式二：数组语法

`<div v-bind:style="[baseStyles, overridingStyles]"></div>`

style后面跟的是一个数组类型，多个值以逗号分割即可

```java
<div id="app">
    <h2 :style="[baseStyle, baseStyle1]">{{message}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '你好啊',
            baseStyle: {backgroundColor: 'red'},
            baseStyle1: {fontSize: '100px'},
        }
    })
</script>
```



# 计算属性

## computed选项

在模板中可以直接通过插值语法显示一些data中的数据。

但是在某些情况，我们可能需要对数据进行一些转化后再显示，或者需要将多个数据结合起来进行显示。

比如我们有firstName和lastName两个变量，需要显示完整的名称。

但是如果多个地方都需要显示完整的名称，我们就需要写多个`{{firstName}} {{lastName}}`

我们可以将上面的代码换成计算属性，计算属性是写在实例的computed选项中的

```html
<div id="app">
    <h2>{{firstName + ' ' + lastName}}</h2>
    <h2>{{firstName}} {{lastName}}</h2>

    <h2>{{getFullName()}}</h2>

    <h2>{{fullName}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            firstName: 'Lebron',
            lastName: 'James'
        },
        // computed: 计算属性()
        computed: {
            fullName: function () {
                return this.firstName + ' ' + this.lastName
            }
        },
        methods: {
            getFullName() {
                return this.firstName + ' ' + this.lastName
            }
        }
    })
</script>
```



## 计算属性的复杂操作

```html
<div id="app">
    <h2>总价格: {{totalPrice}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            books: [
                {id: 110, name: 'Unix编程艺术', price: 119},
                {id: 111, name: '代码大全', price: 105},
                {id: 112, name: '深入理解计算机原理', price: 98},
                {id: 113, name: '现代操作系统', price: 87},
            ]
        },
        computed: {
            totalPrice: function () {
                let result = 0
                for (let i=0; i < this.books.length; i++) {
                    result += this.books[i].price
                }
                return result

                // for (let i in this.books) {
                //   this.books[i]
                // }
                //
                // for (let book of this.books) {
                //
                // }
            }
        }
    })
</script>
```



## 计算属性的setter和getter

**每个计算属性都包含一个getter和一个setter。**

在上面的例子中，我们只是使用getter来读取。

在某些情况下，你也可以提供一个setter方法（不常用）。

```html
<div id="app">
    <h2>{{fullName}}</h2>
</div>
<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            firstName: 'Kobe',
            lastName: 'Bryant'
        },
        computed: {
            // fullName是一个对象，所以调用时后面不用加括号
            // 计算属性一般是没有set方法, 只读属性.
            fullName: {
                set: function(newValue) {
                    const names = newValue.split(' ');
                    this.firstName = names[0];
                    this.lastName = names[1];
                },
                get: function () {
                    return this.firstName + ' ' + this.lastName
                }
            },
            // 简写方式：
            // fullName: function () {
            //   return this.firstName + ' ' + this.lastName
            // }
        }
    })
</script>
```



## 计算属性的缓存

methods和computed看起来都可以实现我们的功能，

那么为什么还要多一个计算属性这个东西呢？

原因：**计算属性会进行缓存，如果多次使用时，计算属性只会调用一次。**

```html
<div id="app">
    <!--1.直接拼接: 语法过于繁琐-->
    <h2>{{firstName}} {{lastName}}</h2>

    <!--2.通过定义methods-->
    <!--<h2>{{getFullName()}}</h2>-->
    <!--<h2>{{getFullName()}}</h2>-->
    <!--<h2>{{getFullName()}}</h2>-->
    <!--<h2>{{getFullName()}}</h2>-->

    <!--3.通过computed-->
    <h2>{{fullName}}</h2>
    <h2>{{fullName}}</h2>
    <h2>{{fullName}}</h2>
    <h2>{{fullName}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            firstName: 'Kobe',
            lastName: 'Bryant'
        },
        methods: {
            getFullName: function () {
                console.log('getFullName');
                return this.firstName + ' ' + this.lastName
            }
        },
        computed: {
            fullName: function () {
                console.log('fullName');
                return this.firstName + ' ' + this.lastName
            }
        }
    })
</script>
```

当使用`{{getFullName()}}`时可以看到控制台打印了四次，而使用`{{fullName}}`时控制台只打印了一次。



# 事件监听

## v-on

在前端开发中，我们需要经常和用于交互。这个时候，我们就必须监听用户发生的时间，比如点击、拖拽、键盘事件等等。在Vue中如何监听事件呢？使用v-on指令。

v-on也有对应的语法糖：v-on:click可以写成@click

```html
<div id="app">
    <h2>{{counter}}</h2>
    <!--<button v-on:click="counter++">+</button>-->
    <!--<button v-on:click="counter--;">-</button>-->

    <!--<button v-on:click="increment">+</button>-->
    <!--<button v-on:click="decrement">-</button>-->

    <button @click="increment">+</button>
    <button @click="decrement">-</button>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            counter: 0
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
</script>
```



## v-on参数问题

当通过methods中定义方法，以供@click调用时，需要注意参数问题：

```html
<div id="app">
    <!--1.事件调用的方法没有参数，带不带括号都可以-->
    <button @click="btn1Click()">按钮1</button>
    <button @click="btn1Click">按钮1</button>

    <!--2.在事件定义时，如果函数需要参数,但是没有传入, 那么函数的形参为undefined
    写方法时省略了小括号, 但是方法本身是需要一个参数的, 这时Vue会默认将浏览器生产的event事件对象作为参数传入到方法-->
    <button @click="btn2Click(123)">按钮2</button>
    <button @click="btn2Click()">按钮2</button>
    <button @click="btn2Click">按钮2</button>

    <!--3.方法定义时, 我们需要event对象, 同时又需要其他参数-->
    <!-- 在调用方式, 如何手动的获取到浏览器参数的event对象: $event-->
    <button @click="btn3Click(abc, $event)">按钮3</button>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '你好啊',
            abc: 123
        },
        methods: {
            btn1Click() {
                console.log("btn1Click");
            },
            btn2Click(event) {
                console.log('--------', event);
            },
            btn3Click(abc, event) {
                console.log('++++++++', abc, event);
            }
        }
    })
</script>
```

各个按钮依次点击效果如下：

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20201018232510.png)



## v-on的修饰符

- `.stop` - 调用 `event.stopPropagation()`。
- `.prevent` - 调用 `event.preventDefault()`。
- `.capture` - 添加事件侦听器时使用 capture 模式。
- `.self` - 只当事件是从侦听器绑定的元素本身触发时才触发回调。
- `.{keyCode | keyAlias}` - 只当事件是从特定键触发时才触发回调。
- `.native` - 监听组件根元素的原生事件。
- `.once` - 只触发一次回调。
- `.left` - (2.2.0) 只当点击鼠标左键时触发。
- `.right` - (2.2.0) 只当点击鼠标右键时触发。
- `.middle` - (2.2.0) 只当点击鼠标中键时触发。
- `.passive` - (2.3.0) 以 `{ passive: true }` 模式添加侦听器

```html
<div id="app">
    <!--1. .stop修饰符的使用-->
    <!--停止冒泡行为，点击按钮时只调用btnClick-->
    <div @click="divClick">
        aaaaaaa
        <button @click.stop="btnClick">按钮</button>
    </div>

    <!--2. .prevent修饰符的使用-->
    <!--阻止默认行为，手动提交form表单的数据-->
    <br>
    <form action="baidu">
        <input type="submit" value="提交" @click.prevent="submitClick">
    </form>

    <!--3. .监听某个键盘的键帽-->
    <!--监听回车键的弹起-->
    <input type="text" @keyup.enter="keyUp">

    <!--4. .once修饰符的使用-->
    <!--只触发一次回调-->
    <button @click.once="btn2Click">按钮2</button>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '你好啊'
        },
        methods: {
            btnClick() {
                console.log("btnClick");
            },
            divClick() {
                console.log("divClick");
            },
            submitClick() {
                console.log('submitClick');
            },
            keyUp() {
                console.log('keyUp');
            },
            btn2Click() {
                console.log('btn2Click');
            }
        }
    })
</script>
```



# 条件判断

## v-if、v-else-if、v-else

这三个指令与JavaScript的条件语句if、else if、else类似。

Vue的条件指令可以根据表达式的值在DOM中渲染或销毁元素或组件。

v-if的原理：v-if后面的条件为false时，对应的元素以及其子元素不会渲染。也就是根本没有不会有对应的标签出现在DOM中。

```html
<div id="app">
    <h2 v-if="score>=90">优秀</h2>
    <h2 v-else-if="score>=80">良好</h2>
    <h2 v-else-if="score>=60">及格</h2>
    <h2 v-else>不及格</h2>

    <h1>{{result}}</h1>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: "#app',
        data: {
            score: 99
        },
        computed: {
            result() {
                let showMessage = '';
                if (this.score >= 90) {
                    showMessage = '优秀'
                } else if (this.score >= 80) {
                    showMessage = '良好'
                } else if (this.score >= 60) {
                    showMessage = '及格'
                } else {
                    showMessage = '不及格'
                }
                return showMessage
            }
        }
    })
</script>
```



## 条件渲染案例

用户再登录时，可以切换使用用户账号登录还是邮箱地址登录。

```html
<div id="app">
  <span v-if="isUser">
    <label for="username">用户账号</label>
    <input type="text" id="username" placeholder="用户账号">
  </span>
  <span v-else>
    <label for="email">用户邮箱</label>
    <input type="text" id="email" placeholder="用户邮箱">
  </span>
  <button @click="isUser = !isUser">切换类型</button>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      isUser: true
    }
  })
</script>
```

for属性可以直接点击label标签，然后直接在对应id相同的文本框进行输入。



## 案例小问题

**问题：**

如果我们在有输入内容的情况下，切换了类型，我们会发现文字依然显示之前的输入的内容。

但是按道理讲，我们应该切换到另外一个input元素中了。在另一个input元素中，我们并没有输入内容。

**问题解答：**

这是因为Vue在进行DOM渲染时，出于性能考虑，会尽可能的复用已经存在的元素，而不是重新创建新的元素。

在上面的案例中，Vue内部会发现原来的input元素不再使用，直接作为else中的input来使用了。

**解决方案：**

如果我们不希望Vue出现类似重复利用的问题，可以给对应的input添加key，并且我们需要保证key的不同。

```html
<div id="app">
  <span v-if="isUser">
    <label for="username">用户账号</label>
    <input type="text" id="username" placeholder="用户账号" key="username">
  </span>
  <span v-else>
    <label for="email">用户邮箱</label>
    <input type="text" id="email" placeholder="用户邮箱" key="email">
  </span>
  <button @click="isUser = !isUser">切换类型</button>
</div>
```



## v-show

v-show的用法和v-if非常相似，也用于决定一个元素是否渲染。

v-if当条件为false时，压根不会有对应的元素在DOM中。

v-show当条件为false时，仅仅是将元素的display属性设置为none而已。

开发中如何选择呢？

- 当需要在显示与隐藏之间切片很频繁时，使用v-show

- 当只有一次切换时，通常使用v-if

```html
<div id="app">
  <!--v-if: 当条件为false时, 包含v-if指令的元素, 根本就不会存在dom中-->
  <h2 v-if="isShow" id="aaa">{{message}}</h2>

  <!--v-show: 当条件为false时, v-show只是给我们的元素添加一个行内样式: display: none-->
  <h2 v-show="isShow" id="bbb">{{message}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      isShow: true
    }
  })
</script>
```



# 循环遍历

## v-for遍历数组

当我们有一组数据需要进行渲染时，我们就可以使用v-for来完成。v-for的语法类似于JavaScript中的for循环。

格式为：item in items的形式。其中 `items` 是源数据数组，而 `item` 则是被迭代的数组元素的**别名**。

我们来看一个简单的案例：

- 如果在遍历的过程中不需要使用索引值
  语法格式：v-for="movie in movies"
  依次从movies中取出movie，并且在元素的内容中，我们可以通过Mustache语法来使用movie
- 如果在遍历的过程中，我们需要拿到元素在数组中的索引值
  语法格式：v-for=(item, index) in items
  其中的index就代表了取出的item在原数组的索引值。

```html
<div id="app">
  <!--1.在遍历的过程中,没有使用索引值(下标值)-->
  <ul>
    <li v-for="item in names">{{item}}</li>
  </ul>

  <!--2.在遍历的过程中, 获取索引值-->
  <ul>
    <li v-for="(item, index) in names">
      {{index+1}}.{{item}}
    </li>
  </ul>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      names: ['jordan', 'kobe', 'james', 'curry']
    }
  })
</script>
```



## v-for遍历对象

v-for可以用于遍历对象：比如某个对象中存储着个人信息，我们希望以列表的形式显示出来。

```html
<div id="app">
  <!--1.在遍历对象的过程中, 如果只是获取一个值, 那么获取到的是value-->
  <ul>
    <li v-for="item in info">{{item}}</li>
  </ul>


  <!--2.获取key和value 格式: (value, key) -->
  <ul>
    <li v-for="(value, key) in info">{{key}}:{{value}}</li>
  </ul>


  <!--3.获取key和value和index 格式: (value, key, index) -->
  <ul>
    <li v-for="(value, key, index) in info">{{index}}.{{key}}:{{value}}</li>
  </ul>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      info: {
        name: 'nan',
        age: 23,
        height: 1.78
      }
    }
  })
</script>
```

`注意：格式(value, key, index)是有序的。`



## 维护状态-key

当 Vue 正在更新使用 `v-for` 渲染的元素列表时，它默认使用“就地更新”的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染。

>A B C D E
>
>我们希望可以在B和C之间加一个F，Diff算法默认执行起来是这样的：即把C更新成F，D更新成C，E更新成D，最后再插入E。是不是很没有效率？

为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，你需要为每项提供一个**唯一** `key` attribute：

```html
<div v-for="item in items" v-bind:key="item.id">
  <!-- 内容 -->
</div>
```

**建议尽可能在使用 `v-for` 时提供 `key` attribute**，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升。

---

key 是 Vue 识别节点的一个通用机制，`key` 并不仅与 `v-for` 特别关联。

不要使用对象或数组之类的非基本类型值作为 `v-for` 的 `key`。请用字符串或数值类型的值。

`key` 的特殊 attribute 主要用在 Vue 的虚拟 DOM 算法，在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法。而使用 key 时，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。

有相同父元素的子元素必须有**独特的 key**。重复的 key 会造成渲染错误。它可以用于强制替换元素/组件而不是重复使用它。

**key的作用主要是为了高效的更新虚拟DOM。**



## 检测数组更新

因为Vue是响应式的，所以当数据发生变化时，Vue会自动检测数据变化，视图会发生对应的更新。

Vue中包含了一组观察数组编译的方法，使用它们改变数组也会触发视图的更新。

- push()：在数组最后面添加元素
- pop()：删除数组中的最后一个元素
- shift()：删除数组中的第一个元素
- unshift()：在数组最前面添加元素
- splice(index,howmany,item1,.....,itemX)：删除元素/插入元素/替换元素
  - index：规定添加/删除项目的起始位置，使用负数可从数组结尾处规定位置
  - 删除元素：第二个参数传入你要删除几个元素(如果没有传,就删除 index 后面所有的元素)
  - 替换元素：第二个参数, 表示我们要替换几个元素, 后面参数是用于替换前面的元素
  - 插入元素：第二个参数, 传入0, 并且后面跟上要插入的元素
- sort()：对数组中的元素进行排序
- reverse()：颠倒数组中元素的顺序

```html
<div id="app">
    <ul>
        <li v-for="item in letters">{{item}}</li>
    </ul>
    <button @click="btnClick">按钮</button>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            letters: ['c', 'b', 'a', 'd']
        },
        methods: {
            btnClick() {
                // 1.push方法：在数组最后面添加元素
                // this.letters.push('aaa')
                // this.letters.push('aaaa', 'bbbb', 'cccc')

                // 2.pop(): 删除数组中的最后一个元素
                // this.letters.pop();

                // 3.shift(): 删除数组中的第一个元素
                // this.letters.shift();

                // 4.unshift(): 在数组最前面添加元素
                // this.letters.unshift('aaa')
                // this.letters.unshift('aaa', 'bbb', 'ccc')

                // 5.splice作用: 删除元素/插入元素/替换元素
                // this.letters.splice(0)//删除从index为0开始往后的元素
                // this.letters.splice(0,2)//删除从index为0开始往后的2个元素
                // this.letters.splice(0, 3, 'm', 'n', 'l', 'x')//替换从index为0的元素开始往后的3个元素
                // this.letters.splice(1, 0, 'x', 'y', 'z')//在index为1的元素后面追加元素

                // 6.sort()：对数组中的元素进行排序
                // this.letters.sort()

                // 7.reverse()：颠倒数组中元素的顺序
                // this.letters.reverse()

                // 注意: 通过索引值修改数组中的元素并不会响应式地更新视图
                // this.letters[0] = 'bbbbbb';

                // 可通过Vue的set(要修改的对象, 索引值, 修改后的值)响应式地更新视图
                Vue.set(this.letters, 0, 'bbbbbb')
            }
        }
    })
</script>
```



# 阶段案例（书籍购物车）

## index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
<div id="app">
    <div v-if="books.length != 0">
        <table>
            <thead>
            <tr>
                <th></th>
                <th>书籍名称</th>
                <th>出版日期</th>
                <th>价格</th>
                <th>购买数量</th>
                <th>操作</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(item,index) in books" v-if="item">
                <td>{{item.id}}</td>
                <td>{{item.name}}</td>
                <td>{{item.date}}</td>
                <td>{{item.price | showPrice}}</td>
                <td>
                    <button @click="decrement(index)" :disabled=" item.count === 1">-</button>
                    {{item.count}}
                    <button @click="increment(index)">+</button>
                </td>
                <td>
                    <button @click="remove(index)">移除</button>
                </td>
            </tr>
            </tbody>
        </table>
        <h2>总价格为：{{totalPrice | showPrice}}</h2>
    </div>
    <div v-else>购物车为空</div>
</div>
<script src="../js/vue.js"></script>
<script src="main.js"></script>
</body>
</html>
```

## main.js

```javascript
const app = new Vue({
    el: '#app',
    data: {
        books: [
            {
                id: 1,
                name: '《算法导论》',
                date: '2006-9',
                price: 85.00,
                count: 1
            },
            {
                id: 2,
                name: '《UNIX编程艺术》',
                date: '2006-2',
                price: 59.00,
                count: 1
            },
            {
                id: 3,
                name: '《编程珠玑》',
                date: '2008-10',
                price: 39.00,
                count: 1
            },
            {
                id: 4,
                name: '《代码大全》',
                date: '2006-3',
                price: 128.00,
                count: 1
            },
        ]
    },
    filters: {
        showPrice(price) {
            return '¥' + price.toFixed(2)
        }
    },
    computed: {
        totalPrice() {
            let totalPrice = 0
            for (let i = 0; i < this.books.length; i++) {
                totalPrice += this.books[i].count * this.books[i].price
            }
            return totalPrice
        }
    },
    methods: {
        increment(index) {
            this.books[index].count++
        },
        decrement(index) {
            this.books[index].count--
        },
        remove(index) {
            this.books.splice(index, 1)
        }
    }
})
```

## style.css

```css
table {
    border: 1px solid #e9e9e9;
    border-collapse: collapse;
    border-spacing: 0;
}

th, td {
    padding: 8px 16px;
    border: 1px solid #e9e9e9;
    text-align: left;
}

th {
    background-color: #f7f7f7;
    color: #5c6b77;
    font-weight: 600;
}
```



# JavaScript高阶函数的使用

编程范式: 命令式编程/声明式编程

编程范式: 面向对象编程(第一公民:对象)/函数式编程(第一公民:函数)

**filter/map/reduce**

filter中的回调函数有一个要求: 必须返回一个boolean值

- true: 当返回true时, 函数内部会自动将这次回调的n加入到新的数组中

- false: 当返回false时, 函数内部会过滤掉这次的n

---

传统方式：

```javascript
const nums = [10, 20, 111, 222, 444, 40, 50]

// 1.需求: 取出所有小于100的数字
let newNums = []
for (let n of nums) {
  if (n < 100) {
    newNums.push(n)
  }
}
console.log(newNums);

// 2.需求:将所有小于100的数字进行转化: 全部*2
let new2Nums = []
for (let n of newNums) {
  new2Nums.push(n * 2)
}
console.log(new2Nums);


// 3.需求:将所有new2Nums数字相加,得到最终的记过
let total = 0
for (let n of new2Nums) {
  total += n
}
console.log(total);
```

使用高阶函数的方式：

```javascript
const nums = [10, 20, 111, 222, 444, 40, 50]

//1.filter函数的使用
// 10, 20, 40, 50
let newNums = nums.filter(function (n) {
  return n < 100
})
console.log(newNums);

// 2.map函数的使用
// 20, 40, 80, 100
let new2Nums = newNums.map(function (n) { 
  return n * 2
})
console.log(new2Nums);

// 3.reduce函数的使用
// reduce作用对数组中所有的内容进行汇总
let total = new2Nums.reduce(function (preValue, n) {
  return preValue + n
}, 0)
console.log(total);
// 第一次: preValue=0 n=20，函数的第二个参数就是为preValue赋初始值
// 第二次: preValue=20 n=40
// 第二次: preValue=60 n=80
// 第二次: preValue=140 n=100
// 240
```

高阶函数更简洁的方式：

```javascript
const nums = [10, 20, 111, 222, 444, 40, 50]

let total = nums.filter(function (n) {
    return n < 100
}).map(function (n) {
    return n * 2
}).reduce(function (prevValue, n) {
    return prevValue + n
}, 0)
console.log(total);

let total = nums.filter(n => n < 100).map(n => n * 2).reduce((pre, n) => pre + n);
console.log(total);
```



# v-model

## 表单绑定v-model

表单控件在实际开发中是非常常见的。特别是对于用户信息的提交，需要大量的表单。

Vue中使用v-model指令来实现表单元素和数据的**双向绑定**。

```html
<div id="app">
  <input type="text" v-model="message">
  {{message}}
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    }
  })
</script>
```

案例的解析：

当我们在输入框输入内容时，因为input中的v-model绑定了message，所以会实时将输入的内容传递给message，message发生改变。

当message发生改变时，因为上面我们使用Mustache语法，将message的值插入到DOM中，所以DOM会发生相应的改变。所以，通过v-model实现了双向的绑定。

当然，我们也可以将v-model用于textarea元素



## v-model原理

v-model其实是一个语法糖，它的背后本质上是包含两个操作：

1. v-bind绑定一个value属性
2. v-on指令给当前元素绑定input事件
   也就是说下面的代码：等同于下面的代码：

```html
<input type="text" v-model="message">
等同于
<input type="text" v-bind:value="message" v-on:input="message = $event.target.value">
```



## v-model：radio

```html
<div id="app">
    <label for="male">
        <input type="radio" id="male" value="男" v-model="sex">男
    </label>
    <label for="female">
        <input type="radio" id="female" value="女" v-model="sex">女
    </label>
    <h2>您选择的性别是: {{sex}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            sex: '男'
        }
    })
</script>
```



## v-model：checkbox

复选框分为两种情况：单个勾选框和多个勾选框

单个勾选框：v-model即为布尔值。此时input的value并不影响v-model的值。

```html
<div id="app">
    <label for="protocol">
        <input type="checkbox" id="protocol" v-model="isAgree">同意协议
    </label>
    <h2>您选择的是: {{isAgree}}</h2>
    <button :disabled="!isAgree">下一步</button>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
           isAgree: false
        }
    })
</script>
```

多个复选框：当是多个复选框时，因为可以选中多个，所以对应的data中属性是一个数组。每选中某一个，就会将input的value添加到数组中。

```html
<div id="app">
    <input type="checkbox" value="篮球" v-model="hobbies">篮球
    <input type="checkbox" value="足球" v-model="hobbies">足球
    <input type="checkbox" value="乒乓球" v-model="hobbies">乒乓球
    <input type="checkbox" value="羽毛球" v-model="hobbies">羽毛球
    <h2>您的爱好是: {{hobbies}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            hobbies: []
        }
    })
</script>
```



## v-model：select

和checkbox一样，select也分单选和多选两种情况。

单选：只能选中一个值，v-model绑定的是一个值。当我们选中option中的一个时，会将它对应的value赋值到mySelect中

多选：可以选中多个值，v-model绑定的是一个数组。当选中多个值时，就会将选中的option对应的value添加到数组mySelects中。

```html
<div id="app">
  <!--1.选择一个-->
  <select name="abc" v-model="fruit">
    <option value="苹果">苹果</option>
    <option value="香蕉">香蕉</option>
    <option value="榴莲">榴莲</option>
    <option value="葡萄">葡萄</option>
  </select>
  <h2>您选择的水果是: {{fruit}}</h2>

  <!--2.选择多个-->
  <select name="abc" v-model="fruits" multiple>
    <option value="苹果">苹果</option>
    <option value="香蕉">香蕉</option>
    <option value="榴莲">榴莲</option>
    <option value="葡萄">葡萄</option>
  </select>
  <h2>您选择的水果是: {{fruits}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      fruit: '香蕉',
      fruits: []
    }
  })
</script>
```



## 值绑定

官方文档仔细阅读之后，发现很简单，就是动态的给value赋值而已。

我们前面的value中的值，可以回头去看一下，都是在定义input的时候直接给定的。

但是真实开发中，这些input的值可能是从网络获取或定义在data中的。所以我们可以通过v-bind:value动态的给value绑定值。

其实会用v-bind，就会值绑定的应用了。

```html
<div id="app">
    <label v-for="item in originHobbies" :for="item">
        <input type="checkbox" :value="item" :id="item" v-model="hobbies">{{item}}
    </label>
    <h2>您的爱好是: {{hobbies}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            hobbies: [], 
            originHobbies: ['篮球', '足球', '乒乓球', '羽毛球', '台球', '高尔夫球']
        }
    })
</script>
```



## 修饰符

- lazy修饰符：
  - 默认情况下，v-model是在input事件中同步输入框的数据的。
  - 也就是说，一旦有数据发生改变对应的data中的数据就会自动发生改变。
  - lazy修饰符可以让数据在失去焦点（点击输入框的外面部分）或者回车时才会更新：
- number修饰符：
  - 默认情况下，在输入框中无论我们输入的是字母还是数字（type="number"也只有第一次输入是数字类型），都会被当做字符串类型进行处理。
  - 但是如果我们希望处理的是数字类型，那么最好直接将内容当做数字处理。
  - number修饰符可以让在输入框中输入的内容自动转成数字类型：
- trim修饰符：
  - 如果输入的内容首尾有很多空格，通常我们希望将其去除
  - trim修饰符可以过滤内容左右两边的空格

```html
<div id="app">
  <!--1.修饰符: lazy-->
  <input type="text" v-model.lazy="message">
  <h2>{{message}}</h2>

  <!--2.修饰符: number-->
  <input type="number" v-model.number="age">
  <h2>{{age}}-{{typeof age}}</h2>

  <!--3.修饰符: trim-->
  <input type="text" v-model.trim="name">
  <h2>您输入的名字:{{name}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      age: 0,
      name: ''
    }
  })
</script>
```

