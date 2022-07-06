---
title: Cookie和Session
tags:
  - Cookie
  - Session
categories:
  - Java
  - JavaWeb
top_img: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp1.jpg'
cover: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp1.jpg'
abbrlink: '9279'
date: 2020-10-04 21:27:18
updated: 2020-10-04 21:27:22
---

会话跟踪是Web程序中常用的技术，用来**跟踪用户的整个会话**。常用的会话跟踪技术是Cookie与Session。**Cookie通过在客户端记录信息确定用户身份**，**Session通过在服务器端记录信息确定用户身份**。



# Cookie

## 什么是Cookie

由于HTTP是一种无状态的协议，服务器单从网络连接上无从知道客户身份。怎么办呢？就**给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理**。

Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。

在Session出现之前，基本上所有的网站都采用Cookie来跟踪会话。

>**Cookie具有不可跨域名性**。例如贴吧不能访问微博的Cookie。
>
>由于浏览器每次请求服务器都会携带Cookie，因此Cookie内容不宜过多，否则影响速度。**Cookie的内容应该少而精。**



## Cookie常用属性

Cookie对象使用key-value属性对的形式保存用户状态，一个Cookie对象保存一个属性对，一个request或者response同时使用多个Cookie。

| 属 性 名       | 描  述                                                       |
| -------------- | ------------------------------------------------------------ |
| String name    | 该Cookie的名称。Cookie一旦创建，名称便不可更改               |
| Object value   | 该Cookie的值。如果值为Unicode字符（例如中文），需要为字符编码。如果值为二进制数据（例如图片），则需要使用BASE64编码 |
| int maxAge     | 该Cookie失效的时间，单位秒。如果为正数，则该Cookie在maxAge秒之后失效。如果为负数，该Cookie为临时Cookie，关闭浏览器即失效，浏览器也不会以任何形式保存该Cookie。如果为0，表示立即删除该Cookie。默认为–1 |
| boolean secure | 该Cookie是否仅被使用安全协议传输。安全协议。安全协议有HTTPS，SSL等，在网络上传输数据之前先将数据加密。默认为false |
| String path    | 该Cookie的使用路径。如果设置为“/sessionWeb/”，则只有contextPath为“/sessionWeb”的程序可以访问该Cookie。如果设置为“/”，则本域名下contextPath都可以访问该Cookie。 |
| String domain  | 可以访问该Cookie的域名。如果设置为“.google.com”，则所有以“google.com”结尾的域名都可以访问该Cookie。注意第一个字符必须为“.” |
| String comment | 该Cookie的用处说明。浏览器显示Cookie信息的时候显示该说明     |
| int version    | 该Cookie使用的版本号。0表示遵循Netscape的Cookie规范，1表示遵循W3C的RFC 2109规范 |



## 测试页面和BaseServlet

**测试页面：**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="pragma" content="no-cache" />
<meta http-equiv="cache-control" content="no-cache" />
<meta http-equiv="Expires" content="0" />
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Cookie</title>
	<base href="http://localhost:8080/cookie_session/">
<style type="text/css">

	ul li {
		list-style: none;
	}
	
</style>
</head>
<body>
	<iframe name="target" width="500" height="500" style="float: left;"></iframe>
	<div style="float: left;">
		<ul>
			<li><a href="cookieServlet?action=createCookie" target="target">Cookie的创建</a></li>
			<li><a href="cookieServlet?action=getCookie" target="target">Cookie的获取</a></li>
			<li><a href="cookieServlet?action=updateCookie" target="target">Cookie值的修改</a></li>
			<li>Cookie的存活周期</li>
			<li>
				<ul>
					<li><a href="cookieServlet?action=defaultLife" target="target">Cookie的默认存活时间（会话）</a></li>
					<li><a href="cookieServlet?action=deleteNow" target="target">Cookie立即删除</a></li>
					<li><a href="cookieServlet?action=life3600" target="target">Cookie存活3600秒（1小时）</a></li>
				</ul>
			</li>
			<li><a href="cookieServlet?action=testPath" target="target">Cookie的路径设置</a></li>
			<li><a href="login.jsp" target="target">Cookie的用户免登录练习</a></li>
		</ul>
	</div>
</body>
</html>
```

**BaseServlet程序：**

```java
package com.nanzx.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Method;

public abstract class BaseServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 解决post请求中文乱码问题
        // 一定要在获取请求参数之前调用才有效
        req.setCharacterEncoding("UTF-8");
        // 解决响应中文乱码问题
        resp.setContentType("text/html; charset=UTF-8");

        String action = req.getParameter("action");
        try {
            // 获取action业务鉴别字符串，获取相应的业务 方法反射对象
            Method method = this.getClass().getDeclaredMethod(action, HttpServletRequest.class, HttpServletResponse.class);
            // System.out.println(method);
            // 调用目标业务方法
            method.invoke(this, req, resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



## 创建和获取Cookie 

```java
public class CookieServlet extends BaseServlet {

    protected void createCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie1 = new Cookie("key1", "value1");
        resp.addCookie(cookie1);
        Cookie cookie2 = new Cookie("key2", "value2");
        resp.addCookie(cookie2);
        resp.getWriter().write("Cookie创建成功！");
    }

    protected void getCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie[] cookies = req.getCookies();
        for (Cookie cookie : cookies) {
            resp.getWriter().write("Cookie[" + cookie.getName() + "=" + cookie.getValue() + "] <br/>");
        }
    }
}
```



## 创建Cookie的工具类

```java
package com.nanzx.util;

import javax.servlet.http.Cookie;

public class CookieUtils {

    public static Cookie findCookie(String name, Cookie[] cookies) {
        if (name == null || cookies == null || cookies.length == 0) {
            return null;
        }
        for (Cookie cookie : cookies) {
            if (name.equals(cookie.getName())) {
                return cookie;
            }
        }
        return null;
    }
}
```



## 修改Cookie

方案一： 

1. 先创建一个要修改的同名（指的就是 key）的 Cookie 对象 。

2. 在构造器，同时赋于新的 Cookie 值。 

3. 调用 response.addCookie( Cookie );;通知客户端保存修改。

方案二： 

1. 先查找到需要修改的 Cookie 对象 。

2. 调用 setValue()方法赋于新的 Cookie 值。 

3. 调用 response.addCookie( Cookie );通知客户端保存修改。

```java
public class CookieServlet extends BaseServlet {

    protected void updateCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie1 = new Cookie("key1", "newValue1");
        resp.addCookie(cookie1);

        Cookie cookie2 = CookieUtils.findCookie("key2", req.getCookies());
        cookie2.setValue("newValue2");

        resp.addCookie(cookie2);
        resp.getWriter().write("Cookie修改成功！");
    }
}
```



## Cookie 生命周期控制

Cookie 的生命周期控制指的是如何管理 Cookie 什么时候被销毁（删除） 

setMaxAge() 

- 正数，表示在指定的秒数后过期 

- 负数，表示浏览器一关，Cookie 就会被删除（默认值是-1） 

- 零，表示马上删除 Cookie

```java
public class CookieServlet extends BaseServlet {

    protected void defaultLife(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie = new Cookie("defalutLife", "defaultLife");
        cookie.setMaxAge(-1);//设置存活时间，默认为-1
        resp.addCookie(cookie);
    }

    protected void deleteNow(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie = CookieUtils.findCookie("key2", req.getCookies());
        if (cookie != null) {
            cookie.setMaxAge(0); // 表示马上删除，不需要等待浏览器关闭
            resp.addCookie(cookie);
            resp.getWriter().write("key2的Cookie已经被删除");
        }
    }

    protected void life3600(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie = new Cookie("life3600", "life3600");
        cookie.setMaxAge(60 * 60); // 设置 Cookie 一小时之后被删除。
        resp.addCookie(cookie);
        resp.getWriter().write("已经创建了一个存活一小时的 Cookie");
    }
}
```

**注意：**

> 修改、删除Cookie时，新建的Cookie除value、maxAge之外的所有属性，例如name、path、domain等，都要与原Cookie完全一样。否则，浏览器将视为两个不同的Cookie不予覆盖，导致修改、删除失败。



## Cookie 有效路径 Path 的设置

Cookie 的 path 属性可以有效的过滤哪些 Cookie 可以发送给服务器，哪些不发。 

path 属性是通过请求的地址来进行有效的过滤。 

CookieA path = /工程路径 

CookieB path = /工程路径/abc 

请求地址如下： 

http://ip:port/工程路径/a.html 【CookieA 发送 ，CookieB 不发送 】

http://ip:port/工程路径/abc/a.html 【CookieA 发送 ，CookieB 发送】

```java
public class CookieServlet extends BaseServlet {

    protected void testPath(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie = new Cookie("path1", "path1");
        // getContextPath() ===>>>> 得到工程路径
        cookie.setPath( req.getContextPath() + "/abc" );// ===>>>> /工程路径/abc
        resp.addCookie(cookie);
        resp.getWriter().write("创建了一个带有 Path 路径的 Cookie");
    }
}
```



## Cookie 练习---免输入用户名登录 

**login.jsp页面：**

```jsp
<form action="http://localhost:8080/cookie_session/loginServlet" method="get">
    用户名：<input type="text" name="username" value="${cookie.username.value}"> <br>
    密码：<input type="password" name="password"> <br>
    <input type="submit" value="登录">
</form>
```

**LoginServlet程序：**

```java
package com.nanzx.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class LoginServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        if ("nan".equals(username) && "123456".equals(password)) {
            //登录 成功
            Cookie cookie = new Cookie("username", username);
            cookie.setMaxAge(60 * 60 * 24 * 7);//当前 Cookie 一周内有效
            response.addCookie(cookie);
            System.out.println("登录 成功");
        } else { // 登录 失败
            System.out.println("登录 失败");
        }
    }
}
```





# Session

## 什么是Session

Session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而Session保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。

如果说**Cookie机制是通过检查客户身上的“通行证”来确定客户身份的话，那么Session机制就是通过检查服务器上的“客户明细表”来确认客户身份。Session相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。**

> 为了获得更高的存取速度，服务器一般把Session放在内存里。每个用户都会有一个独立的Session，**彼此独立，互不可见**。如果Session内容过于复杂，当大量客户访问服务器时可能会导致内存溢出。因此，Session里的信息应该尽量精简。



## Session 常用方法

| 方 法 名                                          | 描  述                                                       |
| ------------------------------------------------- | ------------------------------------------------------------ |
| void setAttribute(String attribute, Object value) | 设置Session属性。value参数可以为任何Java Object。通常为Java Bean。value信息不宜过大 |
| String getAttribute(String attribute)             | 返回Session属性值                                            |
| Enumeration getAttributeNames()                   | 返回Session中存在的属性名                                    |
| void removeAttribute(String attribute)            | 移除Session属性                                              |
| String getId()                                    | 返回Session的ID。该ID由服务器自动创建，不会重复              |
| long getCreationTime()                            | 返回Session的创建日期。返回类型为long，常被转化为Date类型，例如：Date createTime = new Date(session.get CreationTime()) |
| long getLastAccessedTime()                        | 返回Session的最后活跃时间。返回类型为long                    |
| int getMaxInactiveInterval()                      | 返回Session的超时时间。单位为秒。超过该时间没有访问，服务器认为该Session失效 |
| void setMaxInactiveInterval(int second)           | 设置Session的超时时间。单位为秒                              |
| void putValue(String attribute, Object value)     | 不推荐的方法。已经被setAttribute(String attribute, Object Value)替代 |
| Object getValue(String attribute)                 | 不被推荐的方法。已经被getAttribute(String attr)替代          |
| boolean isNew()                                   | 返回该Session是否是新创建的                                  |
| void invalidate()                                 | 使该Session失效                                              |



## 测试页面和BaseServlet

**测试页面：**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="pragma" content="no-cache" />
<meta http-equiv="cache-control" content="no-cache" />
<meta http-equiv="Expires" content="0" />
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Session</title>
	<base href="http://localhost:8080/cookie_session/">
<style type="text/css">

	ul li {
		list-style: none;
	}
	
</style>
</head>
<body>
	<iframe name="target" width="500" height="500" style="float: left;"></iframe>
	<div style="float: left;">
		<ul>
			<li><a href="sessionServlet?action=createOrGetSession" target="target">Session的创建和获取（id号、是否为新创建）</a></li>
			<li><a href="sessionServlet?action=setAttribute" target="target">Session域数据的存储</a></li>
			<li><a href="sessionServlet?action=getAttribute" target="target">Session域数据的获取</a></li>
			<li>Session的存活</li>
			<li>
				<ul>
					<li><a href="sessionServlet?action=defaultLife" target="target">Session的默认超时及配置</a></li>
					<li><a href="sessionServlet?action=life3" target="target">Session3秒超时销毁</a></li>
					<li><a href="sessionServlet?action=deleteNow" target="target">Session马上销毁</a></li>
				</ul>
			</li>
			<li><a href="" target="target">浏览器和Session绑定的原理</a></li>
		</ul>
	</div>
</body>
</html>
```

**BaseServlet程序：**

```java
package com.nanzx.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Method;

public abstract class BaseServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 解决post请求中文乱码问题
        // 一定要在获取请求参数之前调用才有效
        req.setCharacterEncoding("UTF-8");
        // 解决响应中文乱码问题
        resp.setContentType("text/html; charset=UTF-8");

        String action = req.getParameter("action");
        try {
            // 获取action业务鉴别字符串，获取相应的业务 方法反射对象
            Method method = this.getClass().getDeclaredMethod(action, HttpServletRequest.class, HttpServletResponse.class);
            // System.out.println(method);
            // 调用目标业务方法
            method.invoke(this, req, resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



## 创建和获取Session

**Session在用户第一次访问服务器的时候自动创建**。需要注意只有访问JSP、Servlet等程序时才会创建Session，只访问HTML、IMAGE等静态资源并不会创建Session。如果尚未生成Session，也可以使request.getSession(true)强制生成Session。

创建和获取 Session，它们的 API 是一样的，都是request.getSession() ；

**第一次调用是**：创建 Session 会话 

**之后调用都是**：获取前面创建好的 Session 会话对象。 

isNew(); 判断Session是不是刚创建出来的（新的） 【true 表示刚创建 ，false 表示获取之前创建 】

getId() ;得到 Session 的会话 id 值，每个会话都有一个 ID 值。而且这个 ID 是**唯一**的。 

**getSession() 和 getSession(boolen isNew)区别：**

>getSession();相当于getSession(true);
>
>getSession(boolen isNew)
>
>参数为true时，若存在会话，则返回该会话，否则新建一个会话；
>
>参数为false时，如存在会话，则返回该会话，否则返回NULL；



```java
public class SessionServlet extends BaseServlet {

    protected void createOrGetSession(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 创建或获取Session会话对象
        HttpSession session = req.getSession();
        // 判断当前Session会话是否是新创建出来的
        boolean isNew = session.isNew();
        // 获取Session会话的唯一标识 id
        String id = session.getId();

        resp.getWriter().write("得到的Session，它的id是：" + id + " <br /> ");
        resp.getWriter().write("这个Session是否是新创建的：" + isNew + " <br /> ");
    }
}
```



## Session 域数据的存取 

```java
public class SessionServlet extends BaseServlet {

    protected void setAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.getSession().setAttribute("key1", "value1");
        resp.getWriter().write("已经往Session中保存了key1的数据");
    }

    protected void getAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Object attribute = req.getSession().getAttribute("key1");
        resp.getWriter().write("从Session中获取出key1的数据是：" + attribute);
    }
}
```



## Session 生命周期控制

- Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，并维护该Session。用户每访问服务器一次，**无论是否读写Session，服务器都认为该用户的Session“活跃（active）”了一次。**
- public void setMaxInactiveInterval(int interval) 
  - 设置 Session 的超时时间（**以秒为单位**），超过指定的时长，Session 就会被销毁。
  - 值为正数的时候，设定 Session 的超时时长。 
  - 负数表示永不超时（极少使用）。
  - **注意：setMaxInactiveInterval(3);3秒内session没有“活跃”一次才会销毁，否则会一直重新倒计时。**

- public int getMaxInactiveInterval()；获取 Session 的超时时间 

- public void invalidate() ；让当前 Session 会话马上超时无效。 

Session **默认的超时时间长为 30 分钟**。 

> 因为在 Tomcat 服务器的配置文件 web.xml中默认有以下的配置，它就表示配置了当前 Tomcat 服务器下所有 web 工程的所有 Session 超时配置默认时长为：30 分钟。 （**以分钟为单位**）
>
> `<session-config>`
>          `<session-timeout>30</session-timeout>`
> ` </session-config>`

如果说你希望你的 web 工程默认的 Session 的超时时长为其他时长。你可以在你自己的 web.xml 配置文件中做相同的配置。

```xml
    <!--表示当前 web 工程创建出来的所有 Session 默认是10分钟的超时时长-->
    <session-config>
        <session-timeout>10</session-timeout>
    </session-config>
```

```java
public class SessionServlet extends BaseServlet {

    protected void defaultLife(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 获取了Session的默认超时时长
        int maxInactiveInterval = req.getSession().getMaxInactiveInterval();
        resp.getWriter().write("Session的默认超时时长为：" + maxInactiveInterval + " 秒 ");
    }

    protected void life3(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 先获取Session对象
        HttpSession session = req.getSession();
        // 设置当前Session3秒后超时
        session.setMaxInactiveInterval(3);
        resp.getWriter().write("当前Session已经设置为3秒后超时");
    }

    protected void deleteNow(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 先获取Session对象
        HttpSession session = req.getSession();
        // 让Session会话马上超时
        session.invalidate();
        resp.getWriter().write("Session已经设置为超时（无效）");
    }
}
```



## 浏览器和 Session 之间关联的技术内幕

Session 技术底层其实是 **基于Cookie技术** 来实现的。服务器向客户端浏览器发送一个名为JSESSIONID的Cookie，它的值为该Session的id。该Cookie的maxAge属性一般为–1，表示仅当前浏览器内有效，并且各浏览器窗口间不共享，关闭浏览器就会失效。

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20220706214941.png)



注意：新开的浏览器窗口会生成新的Session，但子窗口除外。子窗口会共用父窗口的Session。例如，在链接上右击，在弹出的快捷菜单中选择“在新窗口中打开”时，子窗口便可以访问父窗口的Session。



参考博客：https://blog.csdn.net/fangaoxin/article/details/6952954