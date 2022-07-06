---
title: Mybatis问题解决
tags:
  - null
categories:
  - Mybatis
top_img: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp2.jpg'
cover: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp2.jpg'
abbrlink: d428
date: 2020-12-26 22:45:30
updated: 2021-2-21 23:18:59
---

# There is no getter for property

前两天工作中遇到这个异常，一开始还以为是实体类漏写set/get方法，经排查不是。

**异常**大概如下：

```java
org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'user' in 'class top.nanzx.entity.User'
```

dao层代码大概如下：

```java
@Select("SELECT * FROM users WHERE id = #{user.id}")
User getUser(User user)
```

**解决：**

```java
@Select("SELECT * FROM users WHERE id = #{user.id}")
User getUser(@Param("user")User user)
```

当时有点迷惑平时写demo根本不用加**@Param注解**，回来测试后得出如下结论（id是参数user的一个属性）：

- 如果SQL语句中使用`#{user.id}`时，参数必须加@Param注解
- 如果SQL语句中使用`#{id}`时，参数可以不加@Param注解
- 如果参数有多个，则参数需加@Param注解

---



# 驼峰命名配置开启后没生效

```properties
#开启驼峰命名转换
mybatis.configuration.map-underscore-to-camel-case=true
```

**异常：**

```java
List<User> users = userService.getAllUsers();
system.out.println(users);
```

可以看到控制台显示mybatis查询成功，有打印出SQL语句和查询结果，但是users这个集合是null。

于是我猜测是没有成功将查询结果封装进对象里，但是我已经配置好了自动转换驼峰命名。

**解决：**

经过百度，发现可能是因为项目配置了多数据源，mybatis不知道配置哪个数据源，而自定义的sqlSessionFactory默认不会加载mybatis配置。

所以我们需要手动将mybatis配置注册到sqlSessionFactory中：

```java
    @Bean
    @ConfigurationProperties(prefix = "mybatis.configuration")
    public org.apache.ibatis.session.Configuration globalConfiguration(){
        return new org.apache.ibatis.session.Configuration();
    }

    @Bean(name = "igmSqlSessionFactory")
    public SqlSessionFactory igmSqlSessionFactory(@Qualifier("igmDataSource") DataSource igmDataSource,org.apache.ibatis.session.Configuration configuration) throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(igmDataSource);
        sessionFactory.setConfiguration(configuration);
        return sessionFactory.getObject();
    }
```

这是官网关于MyBatis-Spring-Boot-Starte的一段翻译：

>MyBatis-Spring-Boot-Starter将：
>
>- 自动检测现有的 `DataSource`
>- 将`DataSource`作为输入使用并通过`SqlSessionFactoryBean`创建和注册一个`SqlSessionFactory`实例
>- 将创建并注册一个从`SqlSessionFactory`中获取的`SqlSessionTemplate`实例
>- 自动扫描您的映射器，将它们链接到`SqlSessionTemplate`并将它们注册到Spring上下文中，以便可以将它们注入到您的bean中
>



# 控制台不打印SQL语句及参数问题

application.yml中配置：

```yaml
logging:
  level:
    top.nanzx.dao: debug
```





# 非法参数异常： argument type mismatch

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20210127232544.png)

我的Dao层：

```java
    List<ShowCourse> showCourse();
```

ShowCourse（使用了Lombok插件）：

```java
@Data
@AllArgsConstructor
public class ShowCourse {

    private Course course;

    private String type;//推荐、热门、最新
}
```

解决：多加一个@NoArgsConstructor



# 基于注解的多表查询

[CSDN MyBatis学习总结（十）---基于注解的多表查询（一对一，一对多，多对多）](https://blog.csdn.net/qq_40348465/article/details/84718602?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.not_use_machine_learn_pai&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.not_use_machine_learn_pai)



# order by注入问题

原Dao语句如下：

```java
    @Select("SELECT * FROM note WHERE no = #{no} ORDER BY #{prop} #{order}")
    List<Note> getNotes(@Param("no")String no, @Param("prop")String prop, @Param("order")String order);
```

这时调用sql语句会报如下异常：

```java
com.alibaba.druid.sql.parser.ParserException: syntax error, error in :'RDER BY ? ?
 ) tmp_count', expect RPAREN, actual QUES pos 68, line 2, column 44, token QUES
```

查了一下才知道：

**（1）使用#运算符，Mybatis会将传入的对象当成一个字符串，在进行变量替换时会加上引号，所以order by语句，替换后就变成了下面的样子**

```sql
SELECT * FROM note ORDER BY 'no' 'desc';
```

虽然不会报错，但也**不能正确排序**。

**（2）使用$运算符，Mybatis不会进行预编译，直接把值传进去，无法防止sql注入，当我们需要传字段的名称时，可以考虑使用$符号，但在后台需要进行数据校验，才能在一定程度上防止sql注入。**



# @Insert返回主键

```java
    @Insert("INSERT INTO course(course_name,img_url,teacher_no,create_time,intro,notice) " +
            "VALUES (#{course.courseName},#{course.imgUrl},#{course.teacher.no},#{course.date}," +
            "#{course.intro},#{course.notice})")
    @Options(useGeneratedKeys=true, keyProperty="course.id", keyColumn="id")
    Integer createCourse(@Param("course") Course course);
```

配置`@Options(useGeneratedKeys=true, keyProperty="对象.属性",keyColumn="数据库对应字段")` 

这个的作用是设置是否使用 JDBC 的`getGenereatedKeys()`方法获取主键并赋值到keyProperty设置的对象的属性中。

在插入后，使用对象的主键属性的getXXId()方法获取主键值。