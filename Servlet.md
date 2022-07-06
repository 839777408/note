---
title: Servlet
tags:
  - Servlet
categories:
  - Java
  - JavaWeb
top_img: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp12.jpg'
cover: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp12.jpg'
abbrlink: d9a3
date: 2020-09-28 23:39:04
updated: 2020-09-28 23:39:11
---



# Servlet技术

## 什么是**Servlet** 

1. Servlet 是 JavaEE 规范之一。规范就是接口 

2. Servlet 就 JavaWeb 三大组件之一。三大组件分别是：Servlet 程序、Filter 过滤器、Listener 监听器。 

3. Servlet 是运行在服务器上的一个 java 程序，它可以接收客户端发送过来的请求，并响应数据给客户端。



## 手动实现 Servlet 程序 

- IDEA中创建项目

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200922132609.png)

- 编写一个类去实现 Servlet 接口并实现 service 方法，用来处理请求和响应数据

```java
package com.nanzx.servlet;

import javax.servlet.*;
import java.io.IOException;

public class HelloServlet implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    //service方法是专门用来处理请求和响应的
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("HelloServlet 被访问了");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```

- 到 web.xml 中去配置 servlet 程序的访问地址

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- servlet标签给Tomcat配置Servlet程序 -->
    <servlet>
        <!--servlet-name标签 Servlet程序起一个别名（一般是类名） -->
        <servlet-name>HelloServlet</servlet-name>
        <!--servlet-class是Servlet程序的全类名-->
        <servlet-class>com.nanzx.servlet.HelloServlet</servlet-class>
    </servlet>

    <!--servlet-mapping标签给servlet程序配置访问地址-->
    <servlet-mapping>
        <!--servlet-name标签的作用是告诉服务器，我当前配置的地址给哪个Servlet程序使用-->
        <servlet-name>HelloServlet</servlet-name>
        <!--
         url-pattern标签配置访问地址
            / 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径
            /hello 表示地址为：http://ip:port/工程路径/hello
        -->
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

工程路径：

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200922135054.png)

- 启动Tomcat Server服务，在浏览器中访问：http://localhost:8080/servlet/hello，可以看到控制台输出：HelloServlet 被访问了



## url 地址到 Servlet 程序的访问

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200922145404.png)



## Servlet 的生命周期 

1. 执行 Servlet 构造器方法 。是在**第一次访问**的时候创建 Servlet 程序会调用。 

2. 执行 init 初始化方法。是在**第一次访问**的时候创建 Servlet 程序会调用。 

3. 执行 service 方法 。每次访问都会调用。 

4. 执行 destroy 销毁方法 。在 web 工程停止的时候调用。



## **GET** **和** **POST** **请求的分发处理** 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="http://localhost:8080/servlet/hello" method="post">
    <input type="hidden" name="action" value="login" />
    <input type="hidden" name="username" value="root" />
    <input type="submit">
</form>
</body>
</html>
```

```java
package com.nanzx.servlet;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

public class HelloServlet implements Servlet {

    //service方法是专门用来处理请求和响应的
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("HelloServlet 被访问了");
        // 类型转换（因为它有 getMethod()方法） 
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        // 获取请求的方式 
        String method = httpServletRequest.getMethod();
        if ("GET".equals(method)) {
            doGet();
        } else if
        ("POST".equals(method)) {
            doPost();
        }
    }

    private void doPost() {
        System.out.println("post 请求");
    }

    private void doGet() {
        System.out.println("get 请求");
    }
}
```



## 通过继承HttpServlet实现Servlet程序

一般在实际项目开发中，都是使用继承 HttpServlet 类的方式去实现 Servlet 程序。 

1. 编写一个类去继承 HttpServlet 类 

2. 根据业务需要重写 doGet 或 doPost 方法 

3. 到 web.xml 中的配置 Servlet 程序的访问地址

```java
package com.nanzx.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet2的doGet方法");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet2的doPost方法");
    }
}
```



## 使用 IDEA 创建 Servlet 程序

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200922235424.png)

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200922235740.png)

`最后在web.xml中配置<servlet-mapping>`



## **Servlet** **类的继承体系** 

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200923140122.png)



# ServletConfig 类 

- ServletConfig 类是 Servlet 程序的**配置信息类**。 

- Servlet 程序和 ServletConfig 对象都是由 Tomcat 负责创建，我们负责使用。 

- Servlet 程序默认是第一次访问的时候创建，ServletConfig 是每个 Servlet 程序创建时，就创建一个对应的 ServletConfig 对象。

**ServletConfig** **类的三大作用** 

1. 可以获取 Servlet 程序的别名 servlet-name 的值 

2. 获取初始化参数 init-param 

3. 获取 ServletContext 对象

---

web.xml 中的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.nanzx.servlet.HelloServlet</servlet-class>
        <init-param>
            <param-name>user</param-name>
            <param-value>root</param-value>
        </init-param>
        <init-param>
            <param-name>password</param-name>
            <param-value>123456</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```

Servlet 中的代码：

```java
public class HelloServlet implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        //1、可以获取Servlet程序的别名servlet-name的值
        System.out.println("HelloServlet程序的别名是:" + servletConfig.getServletName());
        //2、获取初始化参数init-param
        System.out.println("初始化参数user的值是:" + servletConfig.getInitParameter("user"));
        System.out.println("初始化参数password的值是:" + servletConfig.getInitParameter("password"));
        //3、获取ServletContext对象
        System.out.println(servletConfig.getServletContext());
    }
}
```

运行结果：

HelloServlet程序的别名是:HelloServlet
初始化参数username的值是;root
初始化参数url的值是;123456
org.apache.catalina.core.ApplicationContextFacade@7709e74c

---

**注意：**

当**继承**重写 init(ServletConfig config) 方法的时候，记得调用 super.init(config);

调用 super.init(config) 的目的，主要是由于在父类（GenericServlet）中有一个ServletConfig实例变量，super.init(config) 就是给这个实例变量赋值。这样在后续的getServletContext()操作，才可以拿到ServletContext对象；否则就会出现java.lang.NullPointerException异常。

如果重写无参数的init()就不需要调用。

```java
public class HelloServlet2 extends HttpServlet {

    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
    }
}
```



# ServletContext 类

- ServletContext 是一个接口，它表示 Servlet 上下文对象 

- **一个 web 工程，只有一个 ServletContext 对象实例。**

- ServletContext 对象是一个域对象。 

- ServletContext 是在 web 工程部署启动的时候创建。在 web 工程停止的时候销毁。 

**域对象**是可以像 Map 一样存取数据的对象，叫域对象。 这里的域指的是存取数据的操作范围，**整个** web 工程。

|        | 存数据           | 取数据           | 删数据              |
| ------ | ---------------- | ---------------- | ------------------- |
| Map    | put（）          | get（）          | remove（）          |
| 域对象 | serAttribute（） | getAttribute（） | removeAttribute（） |

**ServletContext** **类的四个作用** 

1. 获取 web.xml 中配置的上下文参数 context-param 

2. 获取当前的工程路径，格式: /工程路径 

3. 获取工程部署后在服务器硬盘上的绝对路径 

4. 像 Map 一样存取数据

---

web.xml 中的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <context-param>
        <param-name>id</param-name>
        <param-value>context</param-value>
    </context-param>
    <context-param>
        <param-name>method</param-name>
        <param-value>test</param-value>
    </context-param>

    <servlet>
        <servlet-name>ContextServlet</servlet-name>
        <servlet-class>com.nanzx.servlet.ContextServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>ContextServlet</servlet-name>
        <url-pattern>/contextServlet</url-pattern>
    </servlet-mapping>

</web-app>
```

Servlet 中的代码：

```java
package com.nanzx.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class ContextServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletContext context = getServletContext();

        //1、获取web.xml中配置的上下文参数context-param
        String id = context.getInitParameter("id");
        System.out.println("context-param参数id的值是:" + id);
        System.out.println("context-param参数method的值是:" + context.getInitParameter("method"));

        //2、获取当前的工程路径，格式: /工程路径
        System.out.println("当前工程路径:" + context.getContextPath());

        //3、获取工程部署后在服务器硬盘上的绝对路径
        // 斜杠被服务器解析地址为:http://ip:port/工程名/  映射到IDEA代码的web目录
        System.out.println("工程部署的路径是:" + context.getRealPath("/"));
        System.out.println("工程下css目录的绝对路径是:" + context.getRealPath("/css"));
        System.out.println("工程下imgs目录1.jpg的绝对路径是:" + context.getRealPath("/imgs/1.jpg"));

        //4.像 Map 一样存取数据
        System.out.println("保存之前: Context1 获取 key1的值是:" + context.getAttribute("key1"));
        context.setAttribute("key1", "value1");
        System.out.println("Context1 中获取域数据key1的值是:" + context.getAttribute("key1"));
    }
}
```

运行结果：

context-param参数id的值是:context
context-param参数method的值是:test
当前工程路径:/servlet
工程部署的路径是:D:\javaProjects\JavaWeb\out\artifacts\JavaWeb_war_exploded\
工程下css目录的绝对路径是:D:\javaProjects\JavaWeb\out\artifacts\JavaWeb_war_exploded\css
工程下imgs目录1.jpg的绝对路径是:D:\javaProjects\JavaWeb\out\artifacts\JavaWeb_war_exploded\imgs\1.jpg
保存之前: Context1 获取 key1的值是:null
Context1 中获取域数据key1的值是:value1



# Http协议

http（超文本传输协议）是一个简单的请求-响应协议，它通常运行在TCP之上。它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应。请求和响应消息的头以ASCII码形式给出；而消息内容则具有一个类似MIME的格式。这个简单模型是早期Web成功的有功之臣，因为它使得开发和部署是那么的直截了当。

## GET 请求 

- 请求行 
  - 请求的方式												GET 
  - 请求的资源路径[+?+请求参数] 
  - 请求的协议的版本号								 HTTP/1.1 

- 请求头 
  - key : value 组成 ，不同的键值对，表示不同的含义。

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200926114823.png)

GET 请求有哪些： 

> 1. form 标签 method=get 
>
> 2. a 标签 
> 3. link 标签引入 css 
> 4. Script 标签引入 js 文件 
> 5. img 标签引入图片 
> 6. iframe 引入 html 页面 
> 7. 在浏览器地址栏中输入地址后敲回车 



## POST 请求 

- 请求行
  - 请求的方式												POST
  - 请求的资源路径[+?+请求参数] 
  - 请求的协议的版本号								 HTTP/1.1 

- 请求头 
  - key : value 组成 ，不同的键值对，表示不同的含义。

- **空行** 

- 请求体 ===>>> 就是发送给服务器的数据

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200926120009.png)

POST 请求有： form 标签 method=post



## **响应的** **HTTP** 协议格式

- 响应行 
  - 响应的协议和版本号 
  - 响应状态码 
  - 响应状态描述符 

- 响应头 
  - key : value ，不同的响应头，有其不同含义 

- **空行** 

- 响应体 ---->>> 就是回传给客户端的数据

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20200926120818.png)



## 常用的响应码说明 

200 表示请求成功 

302 表示请求重定向 

404 表示请求服务器已经收到了，但是你要的数据不存在（请求地址错误） 

500 表示服务器已经收到请求，但是服务器内部错误（代码错误） 



## MIME类型说明 

MIME 是 HTTP 协议中数据类型。 

MIME 的英文全称是"Multipurpose Internet Mail Extensions" 多功能 Internet 邮件扩充服务。MIME 类型的格式是“大类型/小类型”，并与某一种文件的扩展名相对应。 

常见的 MIME 类型： 

| 文件               | MIME 类型                                               |
| ------------------ | ------------------------------------------------------- |
| 超文本标记语言文本 | .html , .htm                  text/html                 |
| 普通文本           | .txt                                 text/plain         |
| RTF文本            | .rtf                                 application/rtf    |
| GIF图形            | .gif                                 image/gif          |
| JPEG图形           | .jpeg，.jpg                    image/jpeg               |
| au声音文件         | .au                                  audio/basic        |
| MIDI音乐文件       | .mid，.midi                   audio/midi，audio/x-midi  |
| RealAudio音乐文件  | .ra，.ram                       audio/x-pn-realaudio    |
| MPEG文件           | .mpg，.mpeg                video/mpeg                   |
| AVI文件            | .avi                                  video/x-msvideo   |
| GZIP文件           | .gz                                   application/x-zip |
| TAR文件            | .tar                                  application/x-tar |



# HttpServletRequest 类

## 作用

每次只要有请求进入Tomcat 服务器，Tomcat 服务器就会把请求过来的 HTTP 协议信息解析好封装到 Request 对象中，然后传递到 service 方法（doGet 和 doPost）中给我们使用。我们可以通过 HttpServletRequest 对象，获取到所有请求的信息。



## 常用方法

1. getRequestURI() ：获取请求的资源路径 

2. getRequestURL() ：获取请求的统一资源定位符（绝对路径） 

3. getRemoteHost() ：获取客户端的 ip 地址 

4. getHeader() ：获取请求头 

5. getParameter() ：获取请求的参数 

6. getParameterValues() ：获取请求的参数（多个值的时候使用） 
7. getMethod() ：获取请求的方式 GET 或 POST 

8. setAttribute(key, value)：设置域数据 

9. getAttribute(key)：获取域数据 
10. getRequestDispatcher() ：获取请求转发对象

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="http://localhost:8080/servlet/requestServlet" method="get">
    用户名：<input type="text" name="username"><br/>
    密码：<input type="password" name="password"><br/>
    兴趣爱好：<input type="checkbox" name="hobby" value="cpp">C++
    <input type="checkbox" name="hobby" value="java">Java
    <input type="checkbox" name="hobby" value="js">JavaScript<br/>
    <input type="submit">
</form>
</body>
</html>
```

```java
package com.nanzx.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Arrays;

public class RequestServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        System.out.println("URI => " + req.getRequestURI());
        System.out.println("URL => " + req.getRequestURL());
        System.out.println("客户端 ip 地址 => " + req.getRemoteHost());
        System.out.println("请求头 User-Agent ==>> " + req.getHeader("User-Agent"));
        System.out.println( "请求的方式 ==>> " + req.getMethod() );

        // 获取请求参数
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        String[] hobby = req.getParameterValues("hobby");
        System.out.println("用户名：" + username);
        System.out.println("密码：" + password);
        System.out.println("兴趣爱好：" + Arrays.asList(hobby));
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```

运行结果：

URI => /servlet/requestServlet
URL => http://localhost:8080/servlet/requestServlet
客户端 ip 地址 => 0:0:0:0:0:0:0:1
请求头 User-Agent ==>> Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36
请求的方式 ==>> GET
用户名：839777408@qq.com
密码：13486
兴趣爱好：[java]



## 解决乱码问题

当username接收的值为中文时，可能出现乱码。

**Get 请求的中文乱码解决：**

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String username = req.getParameter("username");

        username = new String(username.getBytes("iso-8859-1"), "UTF-8");

        System.out.println("用户名：" + username);
}
```

我的Tomcat是9.0.22版本，不知道是不是以前修改过配置文件，所以上述方法无效。

可以在Tomcat的conf文件夹里修改server.xml（添加上 URIEncoding="utf-8"）：

```xml
    <Connector URIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

---

**POST 请求的中文乱码解决: **

```java
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        String username = req.getParameter("username");
        System.out.println("用户名：" + username);
}
```

`注意： req.setCharacterEncoding("UTF-8");必须在所有请求获得参数前调用`



## 请求转发

客户浏览器发送http请求----》web服务器接受此请求--》调用内部的一个方法在容器内部完成请求处理和转发动作----》将目标资源发送给客户请求转发时，从发送第一次到最后一次请求的过程中，web容器只创建一次request和response对象，新的页面继续处理同一个请求。也可以理解为服务器将request对象在页面之间传递。

**例子：**

你先去了A局，A局看了以后，知道这个事情其实应该B局来管，但是他没有让你自己去找B局，而是让你坐一会儿，自己到后面办公室联系了B局的人，让他们办好后送了过来。

**特点：**

1. 浏览器地址栏没有变化
2. 他们是一次请求
3. 他们共享Request域中的数据
4. 可以转发到WEB-INF目录下，WEB-INF目录下的内容不能直接访问
5. 不可以访问工程以外的资源

```java
package com.nanzx.servlet;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class Servlet1 extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        System.out.println("在 Servlet1（柜台 1）中查看参数（材料）：" + username);
        request.setAttribute("key1","柜台 1 的章");
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/servlet2");
        requestDispatcher.forward(request,response);
    }
}
```

```java
package com.nanzx.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class Servlet2 extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        System.out.println("在 Servlet2（柜台 2）中查看参数（材料）：" + username);
        Object key1 = request.getAttribute("key1");
        System.out.println("柜台 1 是否有章：" + key1);
        System.out.println("Servlet2 处理自己的业务 ");
    }
}
```

访问：http://localhost:8080/servlet/servlet1?username=nan

可以看到后台输出：

在 Servlet1（柜台 1）中查看参数（材料）：nan
在 Servlet2（柜台 2）中查看参数（材料）：nan
柜台 1 是否有章：柜台 1 的章
Servlet2 处理自己的业务 



## base标签的作用

当我们点击a标签进行跳转的时候，浏览器地址栏中的地址是：http://localhost:8080/servlet/a/b/c.html

跳转回去的a标签路径是：../../index.html

所有相对路径在工作时候都会参照当前浏览器地址栏中的地址来进行跳转。

那么参照后得到的地址是：http://localhost:8080/servlet/index.html，是正确的跳转路径

---

当我们使用**请求转发**来进行跳转的时候，浏览器地址栏中的地址是：http://localhost:8080/servlet/forwardC

跳转回去的a标签路径是：../../index.html

那么参照后得到的地址是：http://localhost:8080/index.html，是错误的跳转路径

**base标签可以设置当前页面中所有相对路径工作时，参照哪个路径来进行跳转。**

```html
<!DOCTYPE html>
<html lang="zh_CN">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!--base标签设置页面相对路径工作时参照的地址，href 属性就是参考的地址值-->
    <base href="http://localhost:8080/07_servlet/a/b/">
</head>
<body>
    这是a下的b下的c.html页面<br/>
    <a href="../../index.html">跳回首页</a><br/>
</body>
</html>
```



## Web中的相对路径和绝对路径

在 javaWeb 中，路径分为相对路径和绝对路径两种： 

相对路径是： 

- .                         表示当前目录 

- ..                        表示上一级目录 

- 资源名              表示当前目录/资源名 

绝对路径是： http://ip:port/工程路径/资源路径 

> 在实际开发中，路径都使用绝对路径，而不简单的使用相对路径。 
>
> 要么绝对路径，要么base+相对路径



## web 中 / 斜杠的不同意义

在 web 中 / 斜杠 是一种**绝对路径**。 

1. / 斜杠 如果被浏览器解析，得到的地址是：http://ip:port/ 

   `<a href="/">斜杠</a>`

2. / 斜杠 如果被服务器解析，得到的地址是：http://ip:port/工程路径 

   `<url-pattern>/servlet1</url-pattern> `

   `servletContext.getRealPath("/");`

   `request.getRequestDispatcher("/"); `

**特殊情况：** `response.sendRedirect("/"); `把斜杠发送给浏览器解析，得到 http://ip:port/



# HttpServletResponse 类 

## 作用

HttpServletResponse 类和 HttpServletRequest 类一样。每次请求进来，Tomcat 服务器都会创建一个 Response 对象传 递给 Servlet 程序去使用。HttpServletRequest 表示请求过来的信息，HttpServletResponse 表示所有响应的信息， 我们如果需要设置返回给客户端的信息，都可以通过 HttpServletResponse 对象来进行设置。

## 两个输出流的说明

字节流：getOutputStream(); 常用于下载（传递二进制数据） 

字符流：getWriter(); 常用于回传字符串（常用） 

**两个流同时只能使用一个**

使用了字节流，就不能再使用字符流，反之亦然，否则就会报错。

## **如何往客户端回传数据** 

```java
package com.nanzx.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class ResponseServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        PrintWriter writer = response.getWriter();
        writer.write("阿楠博客");
    }
}
```



## 响应的乱码解决

解决响应中文乱码方案一（**不推荐使用**）：

```java
        // 设置服务器字符集为 UTF-8
        response.setCharacterEncoding("UTF-8");
        // 通过响应头，设置浏览器也使用 UTF-8 字符集
        response.setHeader("Content-Type", "text/html; charset=UTF-8");

        PrintWriter writer = response.getWriter();
        writer.write("阿楠博客");
```

解决响应中文乱码方案二（**推荐**）：

```java
        // 它会同时设置服务器和客户端都使用 UTF-8 字符集，还设置了响应头
        // 此方法一定要在获取流对象之前调用才有效
        response.setContentType("text/html; charset=UTF-8");

        PrintWriter writer = response.getWriter();
        writer.write("阿楠博客");
```



## 请求重定向

客户浏览器发送http请求----》web服务器接受后发送302状态码响应及对应新的location给客户浏览器--》客户浏览器发现是302响应，则**自动**再发送一个新的http请求，请求url是新的location地址----》服务器根据此请求寻找资源并发送给客户。

**例子：**

你先去了A局，A局的人说：“这个事情不归我们管，去B局”，然后你就从A局走了出来，自己乘车去了B局。

**特点：**

1. 浏览器地址栏会发生变化
2. 两次请求
3. 不共享Request域中数据
4. 不能访问WEB-INF下的资源
5. 可以访问工程外的资源

请求重定向的第一种方案： 

> // 设置响应状态码 302 ，表示重定向
>
> resp.setStatus(302); 
>
> // 设置响应头，说明 新的地址在哪里 
>
> resp.setHeader("Location", "http://localhost:8080"); 

请求重定向的第二种方案（推荐使用）： 

> resp.sendRedirect("http://localhost:8080");

