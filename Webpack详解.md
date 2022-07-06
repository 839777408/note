---
title: Webpack详解
tags:
  - Webpack
categories:
  - Vue
top_img: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp7.jpg'
cover: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp7.jpg'
abbrlink: a5d2
date: 2020-11-5 15:16:07
updated: 2020-11-7 15:23:31
---



# 认识Webpack

> Webpack 是一个前端资源加载/打包工具。它将根据模块的依赖关系进行静态分析，然后将这些**模块**按照指定的规则**打包**生成对应的静态资源。

## 概念解释

**模块化：**https://nanzx.top/posts/e220/

在ES6之前，我们要想进行模块化开发，就必须借助于其他的工具让我们可以进行模块化开发。并且在通过模块化开发完成了项目后，还需要处理模块间的各种依赖，将其进行整合打包。

而webpack其中一个核心就是让我们可以进行模块化开发，并且会帮助我们处理模块间的依赖关系。
而且不仅仅是JavaScript文件，我们的CSS、图片、json文件等等在webpack中都可以被当做模块来使用，这就是webpack中模块化的概念。

**打包：**

就是将webpack中的各种资源模块进行打包合并成一个或多个包(Bundle)。
并且在打包的过程中，还可以对资源进行处理，比如压缩图片，将scss转成css，将ES6语法转成ES5语法，将TypeScript转成JavaScript等等操作。

## 和grunt/gulp的对比

grunt/gulp的核心是**Task**:

- 我们可以配置一系列的task，并且定义task要处理的事务（例如ES6、ts转化，图片压缩，scss转成css）
- 之后让grunt/gulp来依次执行这些task，而且让整个流程自动化。
- 所以grunt/gulp也被称为**前端自动化任务管理工具**。

我们来看一个gulp的task，这个task就是将src下面的所有js文件转成ES5的语法，并且最终输出到dist文件夹中。

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201104185928.png)



什么时候用grunt/gulp呢？

> 如果工程模块依赖非常简单，甚至是没有用到模块化的概念。只需要进行简单的合并、压缩，就使用grunt/gulp即可。但是如果整个项目使用了模块化管理，而且相互依赖非常强，我们就可以使用更加强大的webpack了。

grunt/gulp和webpack有什么不同呢？

> grunt/gulp更加强调的是前端流程的自动化，模块化不是它的核心。
> webpack更加强调模块化开发管理，而文件压缩合并、预处理等功能，是它附带的功能。



# webpack起步

## webpack安装

安装webpack首先需要安装Node.js，Node.js自带了软件包管理工具npm。

- 查看自己的node版本：`node -v`

- 使用淘宝镜像的命令(国内使用npm下载安装有点慢)：

  `npm install -g cnpm --registry=https://registry.npm.taobao.org`

- 或者把npm的下载源改为淘宝源：

  `npm config set registry https://registry.npm.taobao.org`

- 全局安装webpack（这里先指定版本号3.6.0，因为vue cli2依赖该版本，-g表示全局安装）

  `cnpm install webpack@3.6.0 -g`

- 局部安装webpack（--save-dev是开发时依赖）

  `cnpm install webpack@3.6.0 --save-dev`

> 为什么全局安装后，还需要局部安装呢？
>
> - 在终端直接执行webpack命令，使用的全局安装的webpack。
>- 当在package.json中定义了scripts时，其中包含了webpack命令，那么使用的是局部webpack。
> - 局部安装可以让每个项目拥有独立的包，不受全局包的影响，方便项目的移动、复制、打包等，**保证不同版本包之间的相互依赖**，这些优点是全局安装难以做到的。

- 使用npm卸载依赖也很慢，所以我们也可以使用cnpm卸载：

  `cnpm uninstall webpack@3.6.0 -g`

- 这时会出现”up to date in ...“，这是npm和cnpm的全局模块地址不同造成的。

  - 先获取npm全局模块地址 `npm config get prefix`
  - 再设置cnpm全局模块地址 `cnpm config set prefix <npm全局模块地址>`
  - 然后就可以通过cnpm卸载了



## 准备工作

文件和文件夹解析：

- dist文件夹：用于存放之后打包的文件
- src文件夹：用于存放我们写的源文件
  - mathUtils.js：定义了一些数学工具函数，可以在其他地方引用并且使用。
  - info.js：定义了一些变量。
  - main.js：项目的入口文件。

- index.html：浏览器打开展示的首页html
- package.json：通过`npm init`生成的，npm包管理的文件

---

mathUtils.js如下：

```javascript
function add(num1, num2) {
  return num1 + num2
}

function mul(num1, num2) {
  return num1 * num2
}

module.exports = {
  add,
  mul
}
```

info.js如下：

```javascript
export const name = 'nan';
export const age = 18;
export const height = 1.88;
```

main.js如下：

```javascript
// 1.使用commonjs的模块化规范
const {add, mul} = require('./mathUtils.js');
console.log(add(20, 30));
console.log(mul(20, 30));

// 2.使用ES6的模块化的规范
import {name, age, height} from "./info";
console.log(name);
console.log(age);
console.log(height)
```



## js文件的打包

现在的js文件中使用了模块化的方式进行开发，它们不可以直接使用。

因为如果直接在index.html引入这三个js文件，浏览器并不识别其中的模块化代码。另外，在真实项目中当有许多这样的js文件时，我们一个个引用非常麻烦，并且后期非常不方便对它们进行管理。

我们应该使用webpack工具，对多个js文件进行打包。
我们知道，webpack就是一个模块化的打包工具，所以它支持我们代码中写模块化，可以对模块化的代码进行处理。另外，如果在处理完所有模块之间的关系后，将多个js打包到一个js文件中，引入时就变得非常方便了。

在当前项目的控制台使用webpack的打包指令即可：`webpack src/main.js dist/bundle.js`



## 使用打包后的文件

打包后会在dist文件下，生成一个bundle.js文件，这个文件是webpack处理了项目的直接文件依赖后生成的一个js文件，我们只需要将这个js文件在index.html中引入即可。

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201104222230.png)



# webpack配置

## 入口和出口

如果每次使用webpack的命令都需要写上入口和出口作为参数，就非常麻烦。
我们可以创建一个webpack.config.js文件，将这两个参数写到配置中，在运行时直接读取。

path是node_modules文件夹中的一个webpack自带模块，__dirname是获取当前配置文件的绝对路径。

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201104223404.png)



## 局部安装webpack

- 目前，我们使用的webpack是全局的webpack，如果我们想使用局部来打包呢？
  - 因为一个项目往往依赖特定的webpack版本，全局的版本可能跟这个项目的webpack版本不一致，导出打包容易出现问题。
  - 所以通常一个项目都有自己局部的webpack。
- 第一步，项目中需要安装自己局部的webpack
  - 这里我们将局部安装webpack的3.6.0版本
  - Vue CLI3中已经升级到webpack4，但是它将配置文件隐藏了起来，所以查看起来不是很方便。
  - 安装命令：`cnpm install webpack@3.6.0 --save-dev`
- 第二步，通过`node_modules/.bin/webpack`启动webpack打包



## package.json中定义启动

但是每次执行都敲这么一长串很不方便，所以我们可以在package.json的**scripts中**定义自己的执行脚本。

```json
{
  "name": "first",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^3.6.0"
  }
}
```

package.json中的scripts的脚本在执行时，会按照一定的顺序寻找命令对应的位置。

- 首先，会寻找本地的node_modules/.bin路径中对应的命令。
- 如果没有找到，会去全局的环境变量中寻找。

如何执行我们的build指令呢？`npm run build`



# loader的使用

## 什么是loader

loader是webpack中一个非常核心的概念。

在我们之前的实例中，我们主要用webpack来处理我们写的js代码，并且webpack会自动处理js之间相关的依赖。

但是，在开发中我们不仅仅有基本的js代码处理，我们也需要加载css、图片，也包括一些高级的将ES6转成ES5代码，将TypeScript转成ES5代码，将scss、less转成css，将.jsx、.vue文件转成js文件等等。

对于webpack本身的能力来说，对于这些转化是不支持的。但是给webpack扩展对应的loader就可以了。

loader使用过程：

1. 通过npm安装需要使用的loader
2. 在webpack.config.js中的modules关键字下进行配置

`注意：此篇文章中所有安装的依赖需版本一致，否则容易出现错误！`

本次webpack详解的最终package.json如下：

```json
{
  "name": "first",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "dev": "webpack-dev-server --open"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.3",
    "babel-loader": "^7.1.5",
    "babel-preset-es2015": "^6.24.1",
    "css-loader": "^2.0.2",
    "file-loader": "^3.0.1",
    "html-webpack-plugin": "^3.2.0",
    "less": "^3.9.0",
    "less-loader": "^4.1.0",
    "style-loader": "^0.23.1",
    "uglifyjs-webpack-plugin": "^1.1.1",
    "url-loader": "^1.1.2",
    "vue-loader": "^13.0.0",
    "vue-template-compiler": "^2.6.12",
    "webpack": "^3.6.0",
    "webpack-dev-server": "^2.9.1"
  },
  "dependencies": {
    "vue": "^2.6.12"
  }
}
```



## css文件处理

项目开发过程中，我们必然需要添加很多的样式，而样式我们往往写到一个单独的文件中。

在src目录中，创建一个css文件，其中创建一个normal.css文件。

```css
body {
   background-color: pink;
}
```

我们也可以重新组织文件的目录结构，将零散的js文件放在一个js文件夹中。

但是，这个时候normal.css中的样式不会生效，因为我们压根就没有引用它。

webpack也不可能找到它，因为我们只有一个入口，webpack会从入口开始查找其他依赖的文件。

所以我们需要在入口main.js中引用这个样式：

```javascript
// 1.使用commonjs的模块化规范
const {add, mul} = require('./js/mathUtils.js');
console.log(add(20, 30));
console.log(mul(20, 30));

// 2.使用ES6的模块化的规范
import {name, age, height} from "./js/info";

console.log(name);
console.log(age);
console.log(height)

require("./css/normal.css")
```

这时候我们执行打包命令会发现报错，这个error提示我们需要有一个合适的loader去处理这个文件类型（也就是css）。

在webpack的官方网站中，我们可以找到如下关于样式的loader使用方法：

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201105102307.png)



webpack.config.js中的test是使用正则表达式匹配后缀为css的文件，use是使用了style-loader和css-loader。

需注意，如果我们只配置了css-loader，运行index.html，你会发现样式并没有生效。

原因是css-loader只负责加载css文件，但是并不负责将css具体样式嵌入到Dom中，这个工作是由style-loader来完成的。

**注意：**style-loader需要放在css-loader的前面。因为webpack在读取使用的loader的过程中，是按照从右向左的顺序读取的。需先加载再嵌入。

安装命令及对应版本号：

`cnpm install css-loader@2.0.2 --save-dev`

``cnpm install style-loader@0.23.1 --save-dev ``

最后执行`npm run build`就可以发现样式加载成功了。



## less文件处理

创建一个special.less文件并放入css文件夹中。

```less
@fontColor: orange;
@fontSize: 50px;

body{
  font-size: @fontSize;
  color: @fontColor;
}
```

在入口main.js中引用这个样式：

```javascript
// 1.使用commonjs的模块化规范
const {add, mul} = require('./js/mathUtils.js');
console.log(add(20, 30));
console.log(mul(20, 30));

// 2.使用ES6的模块化的规范
import {name, age, height} from "./js/info";

console.log(name);
console.log(age);
console.log(height)

require("./css/normal.css")
require("./css/special.less")
```

打包报错，继续在官网中查找，我们会找到 less-loader 相关的使用说明。

- 首先，还是需要安装对应的loader：`cnpm install less-loader@4.1.0 less@3.9.0 --save-dev`

- 注意：我们这里还安装了less，因为webpack会使用less对less文件进行编译。

- 其次，修改对应的配置文件web.config.js，添加一个rules选项，用于处理.less文件。

```javascript
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "bundle.js",
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.less$/,
        use: [{
          loader: "style-loader" // creates style nodes from JS strings
        }, {
          loader: "css-loader" // translates CSS into CommonJS
        }, {
          loader: "less-loader" // compiles Less to CSS
        }]
      },
    ]
  }
}
```



## 图片文件处理  

首先，我们在项目中加入两张图片：一张较小的图片test01.jpg(小于8kb)，一张较大的图片test02.jpeg(大于8kb)

待会儿我们会针对这两张图片进行不同的处理。

我们先考虑在css样式中引用图片的情况，所以更改了normal.css中的样式：

```css
body {
   /*background-color: pink;*/
  background: url("../img/test01.jpg");
}
```

如果我们现在直接打包还是会报错。

图片处理，我们使用url-loader来处理，依然先安装url-loader：

`cnpm install url-loader@1.1.2 --save-dev`

修改webpack.config.js配置文件：

```javascript
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "bundle.js",
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.less$/,
        use: [{
          loader: "style-loader" // creates style nodes from JS strings
        }, {
          loader: "css-loader" // translates CSS into CommonJS
        }, {
          loader: "less-loader" // compiles Less to CSS
        }]
      },
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
            }
          }
        ]
      },
    ]
  }
}
```

再次打包，运行index.html，就会发现我们的背景图片显示出来了。

仔细观察，背景图是通过base64显示的，这也是limit属性的作用，当图片小于8kb时，对图片进行base64编码：

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201105110001.png)



那么问题来了，如果大于8kb呢？我们将background的图片改成test02.jpg，打包后报错了。

这次因为是大于8kb的图片，会通过file-loader进行处理，但是我们的项目中并没有file-loader。

所以，我们需要安装file-loader：`cnpm install file-loader@3.0.1 --save-dev `

再次打包，就会发现dist文件夹下多了一个图片文件，并以32位哈希值命名。

但是，真实开发中，我们可能对打包的图片名字有一定的要求。比如：将所有的图片放在一个文件夹中，跟上图片原来的名称，同时也要防止重复。

所以，我们可以在options中添加上如下选项：

- img：文件要打包到的文件夹
- name：获取图片原来的名字，放在该位置
- hash:8：为了防止图片名称冲突，依然使用hash，但是我们只保留8位
- ext：使用图片原来的扩展名

```javascript
	{
        test: /\.(png|jpg|gif|jpeg)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              name: 'img/[name].[hash:8].[ext]'
            }
          }
        ]
      },
```

但是我们发现图片并没有显示出来，这是因为图片使用的路径不正确：

默认情况下，webpack会将生成的路径直接返回给使用者

但是，我们整个程序是打包在dist文件夹下的，所以这里我们需要在路径下再添加一个dist/

```javascript
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "bundle.js",
    publicPath: 'dist/'
  }
}
```



## ES6语法处理

如果你仔细阅读webpack打包的js文件，发现写的ES6语法并没有转成ES5，那么就意味着可能一些对ES6还不支持的浏览器没有办法很好的运行我们的代码。

如果希望将ES6的语法转成ES5，那么在webpack中，我们直接使用babel对应的loader就可以了。

`cnpm install --save-dev babel-loader@7 babel-core babel-preset-es2015`

配置webpack.config.js文件:

```javascript
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['es2015']
          }
        }
      }
```

重新打包，查看bundle.js文件，发现其中的内容变成了ES5的语法。



# webpack配置Vue

## 引入Vue

我们希望在项目中使用Vuejs，那么必然需要对其有依赖，所以需要先进行安装：`cnpm install vue --save`

注意：因为我们后续是在实际项目中也会使用vue的，所以并不是开发时依赖。

按照我们之前学习的方式来开始使用Vue了：

main.js文件：

```javascript
// 1.使用commonjs的模块化规范
const {add, mul} = require('./js/mathUtils.js');
console.log(add(20, 30));
console.log(mul(20, 30));

// 2.使用ES6的模块化的规范
import {name, age, height} from "./js/info";

console.log(name);
console.log(age);
console.log(height)

require("./css/normal.css")
require("./css/special.less")

import Vue from "vue";
// import cpn from "./vue/cpn"
import cpn from "./vue/cpn.vue";

new Vue({
  el: "#app",
  data: {
    message: 'come on'
  }
})
```

index.html文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
<div id="app">
	{{message}}
</div>
<script src="dist/bundle.js"></script>
</body>
</html>
```



## 运行出错

修改完成后，重新打包没有出现错误，但是运行程序，没有出现想要的效果，而且浏览器中有报错：

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20201105114424.png)

这里是因为涉及到Vue的不同版本构建，runtime-only（不可以有任何template）和runtime-compiler的区别。

而`<div id="app">{{message}}</div>`就是Vue实例的template模板，所以我们需添加alias配置：

```javascript
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "bundle.js",
    publicPath: 'dist/'
  },
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.esm.js'
    }
  }
}
```



## el和template区别

正常运行之后，我们来考虑另外一个问题：

- 如果我们希望将data中的数据显示在界面中，就必须修改index.html
- 如果我们后面自定义了组件，也必须修改index.html来使用组件
- 但是html模板在之后的开发中，我并不希望手动的来频繁修改，是否可以做到呢？

定义template属性：

- 在前面的Vue实例中，我们定义了el属性，用于和index.html中的#app进行绑定，让Vue实例之后可以管理它其中的内容
- 这里，我们可以将div元素中的`{{message}}`内容删掉，只保留一个基本的id为div的元素
- 但是如果我依然希望在其中显示`{{message}}`的内容，我们可以再定义一个template属性，代码如下：

```javascript
new Vue({
  el: "#app",
  template: '<div id="app">{{message}}</div>',
  data: {
    message: 'nan'
  }
})
```

重新打包运行程序，显示一样的结果和HTML代码结构。

那么，el和template模板的关系是什么呢？

- 我们知道el用于指定Vue要管理的DOM，可以帮助解析其中的指令、事件监听等等。
- 而如果Vue实例中同时指定了template，那么template模板的内容会替换掉挂载的对应el的模板。
- 这样做之后我们就不需要在以后的开发中再次操作index.html，只需要在template中写入对应的标签即可。



## Vue组件化开发引入

```javascript
const cpn = {
  template: `
    <div>
    <h2>{{ message }}</h2>
    <button @click="btnClick">按钮</button>
    <h2>{{ name }}</h2>
    </div>
  `,
  data() {
    return {
      message: "Hello Webpack",
      name: "nan"
    }
  },
  methods: {
    btnClick() {
    }
  }
}

new Vue({
  el: "#app",
  template: '<cpn/>',
  components: {
    cpn
  }
})
```

也可以将组件抽取到一个js文件中，main.js再导入：

```javascript
export default {
  template: `
    <div>
    <h2>{{ message }}</h2>
    <button @click="btnClick">按钮</button>
    <h2>{{ name }}</h2>
    </div>
  `,
  data() {
    return {
      message: "Hello Webpack",
      name: "nan"
    }
  },
  methods: {
    btnClick() {
    }
  }
}
```



## .vue文件封装处理

但是一个组件以一个js对象的形式进行组织和使用的时候是非常不方便的：

- 编写template模块非常的麻烦
- 如果有样式的话，我们写在哪里比较合适呢？

现在，我们以一种全新的方式来组织一个vue的组件，也就是用.vue文件封装处理。

```javascript
<template>
  <div>
    <h2 class="title">{{ message }}</h2>
    <button @click="btnClick">按钮</button>
    <h2>{{ name }}</h2>
    <App/>
  </div>
</template>

<script>
import App from "./App"

export default {
  name: "cpn",
  components: {App},
  data() {
    return {
      message: "Hello Webpack",
      name: "nan"
    }
  },
  methods: {
    btnClick() {
    }
  }
}
</script>

<style scoped>
 .title{
   color: green;
 }
</style>
```

但是，这种特殊的文件以及特殊的格式不可以被正确的加载，必须有vue-loader以及vue-template-compiler帮助我们处理。
安装vue-loader和vue-template-compiler：

`cnpm install vue-loader vue-template-compiler --save-dev`

修改webpack.config.js的配置文件：

```javascript
	{
        test: /\.vue$/,
        use: ['vue-loader']
      }
```

---

main.js引入组件：

```javascript
import cpn from "./vue/cpn.vue";
```

注意：需要加上后缀`.vue`，否则会报错。

如果要自动匹配文件，省略后缀的话可以在webpack.config.js添加如下配置：

```javascript
resolve: {
  extensions: ['.js','.css','.vue'],
}
```



# plugin的使用

## 认识plugin

plugin是插件的意思，通常是用于对某个现有的架构进行扩展。

webpack中的插件，就是对webpack现有功能的各种扩展，比如打包优化，文件压缩等等。

loader和plugin区别：

- loader主要用于转换某些类型的模块，它是一个转换器。
- plugin是插件，它是对webpack本身的扩展，是一个扩展器。

plugin的使用过程：

1. 通过npm安装需要使用的plugins(某些webpack已经内置的插件不需要安装)
2. 在webpack.config.js中的plugins中配置插件。
   

## 添加版权的plugin

为打包的文件添加版权声明，该插件名字叫BannerPlugin，属于webpack自带的插件。

按照下面的方式来修改webpack.config.js的文件：

```javascript
const path = require('path');
const webpack = require('webpack')

module.exports = {
  ...
  plugins: [
      new webpack.BannerPlugin("最终版权归阿楠所有"),
  ]
}
```

重新打包程序，查看bundle.js文件的头部，可以看到第一行有我们的注释，也就是版权说明。



## 打包html的plugin

目前，我们的index.html文件是存放在项目的根目录下的。

我们知道，在真实发布项目时，发布的是dist文件夹中的内容，但是dist文件夹中如果没有index.html文件，那么打包的js等文件也就没有意义了。所以，我们需要将index.html文件打包到dist文件夹中，这个时候就可以使用HtmlWebpackPlugin插件。

HtmlWebpackPlugin插件可以为我们做这些事情：

- 自动生成一个index.html文件(可以指定模板来生成)
- 将打包的js文件，自动通过script标签插入到body中

安装HtmlWebpackPlugin插件：`cnpm install html-webpack-plugin@3.2.0 --save-dev`

使用插件，修改webpack.config.js文件中plugins部分的内容如下：

```javascript
const path = require('path');
const webpack = require('webpack')
const htmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  ...
  plugins: [
      new webpack.BannerPlugin("最终版权归阿楠所有"),
      new htmlWebpackPlugin({
        template: 'index.html'
      }),
  ]
}
```

这里的template表示根据什么模板来生成index.html，我们根据根目录下的index.html来生成。

另外，我们需要删除之前图片文件处理中在output中添加的publicPath属性，否则插入的script标签中的src可能会有问题。现在html在dist目录中，自然能用dist里面的image了。



## js压缩的plugin

在项目发布之前，我们必然需要对js等文件进行压缩处理。这里，我们就对打包的js文件进行压缩。

我们使用一个第三方的插件uglifyjs-webpack-plugin，并且版本号指定1.1.1，和CLI2保持一致：
`cnpm install uglifyjs-webpack-plugin@1.1.1 --save-dev`

修改webpack.config.js文件，使用插件：

```javascript
const path = require('path');
const webpack = require('webpack')
const htmlWebpackPlugin = require('html-webpack-plugin')
const uglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  ...
  plugins: [
      new webpack.BannerPlugin("最终版权归阿楠所有"),
      new htmlWebpackPlugin({
        template: 'index.html'
      }),
      new uglifyjsWebpackPlugin()
  ]
}
```


查看打包后的bunlde.js文件，是已经被压缩过了。



# 本地服务器

webpack提供了一个可选的本地开发服务器，这个本地服务器基于node.js搭建，内部使用express框架，可以实现我们想要的让浏览器自动刷新显示我们修改后的结果。

不过它是一个单独的模块，在webpack中使用之前需要先安装它：

`cnpm install --save-dev webpack-dev-server@2.9.1`

devserver也是作为webpack中的一个选项，选项本身可以设置如下属性：

- contentBase：为哪一个文件夹提供本地服务，默认是根文件夹，我们这里要填写./dist
- port：端口号
- inline：页面实时刷新
- historyApiFallback：在SPA页面中，依赖HTML5的history模式

webpack.config.js文件配置修改如下：

```javascript
module.exports = {
  ...
  devServer: {
    contentBase: './dist',
    inline: true
  }
}
```

我们可以在package.json里再配置另外一个scripts，--open参数表示直接打开浏览器：

```json
{
  ...
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "dev": "webpack-dev-server --open"
  },
  ...
}
```

重新打包后可以通过`npm run dev`来实时刷新我们的修改结果。



# webpack的配置分离

我们package.json里，开发时依赖和运行时依赖都有区分开来。

那我们的webpack.config.js也应如此，像我们的本地服务器只是帮助我们开发时用的，项目发布时不需要；js压缩是项目发布时需要，但开发时不需要，所以我们应进行抽离：公共配置文件，开发时配置文件，项目发布时配置文件。

合并公共配置文件和其他配置文件是通过webpack-merge这个插件进行合并。

`cnpm install webpack-merge@4.1.5 --save-dev` 

---

Base.config.js是我们开发和项目发布时都需要的配置：

```javascript
const path = require('path');
const webpack = require('webpack')
const htmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: "bundle.js",
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.less$/,
        use: [{
          loader: "style-loader" // creates style nodes from JS strings
        }, {
          loader: "css-loader" // translates CSS into CommonJS
        }, {
          loader: "less-loader" // compiles Less to CSS
        }]
      },
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 13000,
              name: 'img/[name].[hash:8].[ext]'
            }
          }
        ]
      },
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['es2015']
          }
        }
      },
      {
        test: /\.vue$/,
        use: ['vue-loader']
      }
    ]
  },
  resolve: {
    extensions: ['.js','.css','.vue'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js'
    }
  },
  plugins: [
    new webpack.BannerPlugin("最终版权归阿楠所有"),
    new htmlWebpackPlugin({
      template: 'index.html'
    }),
  ],
}
```

---

prod.config.js是我们项目发布时需要的，开发时不需要的：

```javascript
const uglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')
const webMerge = require('webpack-merge')
const baseConfig = require('./base.config')

module.exports = webMerge(baseConfig,{
  plugins: [
    new uglifyjsWebpackPlugin()
  ],
})
```

---

dev.config.js是我们项目发布时不需要的，开发时需要的：

```javascript
const webMerge = require('webpack-merge')
const baseConfig = require('./base.config')

module.exports = webMerge(baseConfig,{
  devServer: {
    contentBase: './dist',
    inline: true
  }
})
```

---

删除原本的webpack.config.js文件，在package.json中指定我们运行命令时的配置文件：

```json
{
  ...
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --config ./build/prod.config.js",
    "dev": "webpack-dev-server --open --config ./build/dev.config.js"
  },
  ...
}

```

同时应注意由于配置文件的位置改变，所以我们应改变出口的位置：  

`path: path.resolve(__dirname, '../dist'),`

