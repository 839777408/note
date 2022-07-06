---
title: SpringBoot整合Vue的小案例
tags:
  - 跨域
categories:
  - Vue
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp12.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp12.jpg'
abbrlink: bccf
date: 2020-11-18 09:34:36
updated: 2020-12-7 12:00:28
---

众所周知现在开发都是前后端分离，后端选用Spring Boot，前端选用Vue技术。

由于刚学完Vue.js，所以想了解下现在主流的前后端分离是如何实现的，所以写了个小demo。

# 案例

需求：将数据库的goods表展示到前端页面。

所用技术：

- SpringBoot
- MyBatis
- Vue.js



![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201118090102.png)



# SpringBoot

## 目录结构

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201118090504.png)



## 基础项目配置

### Pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.nanzx</groupId>
    <artifactId>bootvue</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>bootvue</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### application.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/keshe?serverTimezone=UTC&characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis:
  configuration:
    map-underscore-to-camel-case: true

server:
  port: 8181
```



## 编写实体类

```java
package top.nanzx.bootvue.entity;

import lombok.Data;

@Data
public class Good {

    private Integer id;
    private String name;
    private String intro;

}
```



## 编写dao层

```java
package top.nanzx.bootvue.dao;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import top.nanzx.bootvue.entity.Good;

import java.util.List;

@Mapper
public interface GoodsMapper {

    @Select("select id, name, intro from goods")
    List<Good> getAllGoods();
}
```



## 编写controller层

```java
package top.nanzx.bootvue.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import top.nanzx.bootvue.dao.GoodsMapper;
import top.nanzx.bootvue.entity.Good;

import java.util.List;

@RestController
public class GoodsController {

    @Autowired
    private GoodsMapper goodsMapper;

    @GetMapping("/getAllGoods")
    public List<Good> getAllGoods() {
        return goodsMapper.getAllGoods();
    }
}
```



# Vue.js

## 项目结构

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201118091714.png)



## 封装网络请求模块

在network/request.js中：

```js
import axios from 'axios'

export function request(config) {
  // 创建axios的实例
  const instance = axios.create({
    timeout: 5000
  })
  // 发送真正的网络请求
  return instance(config)
}
```



## 编写视图

在views/Goods.vue中：

```html
<template>
  <div>
    <table>
      <thead>
      <tr>
        <th>Id</th>
        <th>商品名称</th>
        <th>商品描述</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="item in goods">
        <td>{{item.id}}</td>
        <td>{{item.name}}</td>
        <td>{{item.intro}}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
  import {request} from "../network/request";

  export default {
    name: "Goods",
    data() {
      return {
        goods: {}
      }
    },
    created() {
      const _this = this
      request({
        url: 'http://localhost:8181/getAllGoods'
      }).then(res => {
        _this.goods = res.data
      })
    }
  }
</script>

<style scoped>
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
</style>
```



## 配置路由映射

在router/index.js中：

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const routes = [
  {
    path: '/goods',
    component: () => import('../views/Goods')
  }
]

const router = new VueRouter({
  routes,
  mode: 'history'
})

export default router
```



## 配置App.vue

```html
<template>
  <div id="app">
    <router-link to="/goods">商品列表</router-link>
    <router-view></router-view>
  </div>
</template>

<style>
</style>
```



# 跨域问题

启动SpringBoot和Vue项目（`npm run serve`），访问http://localhost:8080/

点击“商品列表”，这时浏览器的控制台报错：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201117165902.png)

这是因为出现跨域问题，我们的vue项目部署在8080端口，而服务器端口是8181端口。

我们可以在后端解决跨域问题：

```java
package top.nanzx.bootvue.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CrosConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOriginPatterns("*")
                .allowedMethods("GET", "POST", "HEAD", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

这时重新点击“商品列表”，就可以正确接收后台传输的数据并进行展示。