---
title: Druid数据源的使用
tags:
  - Druid
categories:
  - SpringBoot
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp4.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp4.jpg'
abbrlink: 13be
date: 2021-01-10 00:38:40
updated: 2021-01-10 00:38:51
---



目前市场上最主流的数据源，是阿里巴巴计算平台事业部出品，为**监控**而生的数据库连接池。

>官网地址：https://github.com/alibaba/druid
>
>中文文档：https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98

整合Druid的两种方式：

- 导入相关依赖进行自定义配置
- 使用starter场景启动器自动配置

# 自定义方式

## 导入依赖

```xml
        <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.4</version>
        </dependency>
```

## 基本配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai&characterEncoding=UTF-8
```

## 配置示例【[常见问题](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)】

```java
package top.nanzx.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.sql.SQLException;

@Configuration
public class DataSourceConfig {

    // @ConfigurationProperties可以自动将配置文件中的字段映射到对象的属性中
    @ConfigurationProperties(prefix = "spring.data")
    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();
        //StatFilter（别名是stat），用于统计监控信息；WallFilter，它是基于SQL语义分析来实现防御SQL注入攻击的。
        druidDataSource.addFilters("stat,wall");
        return druidDataSource;
    }

    /**
     * 配置Druid的内置监控页面，用于展示Druid的统计信息。
     */
    @Bean
    public ServletRegistrationBean statViewServlet(){
        StatViewServlet statViewServlet = new StatViewServlet();
        ServletRegistrationBean<StatViewServlet> servletRegistrationBean = new ServletRegistrationBean<>(statViewServlet,"/druid/*");
        //配置监控页面访问密码
        servletRegistrationBean.addInitParameter("loginUsername","admin");
        servletRegistrationBean.addInitParameter("loginPassword","123456");
        //不允许清空统计数据
        servletRegistrationBean.addInitParameter("resetEnable","false");
        //IP地址访问控制（deny优先于allow）
        servletRegistrationBean.addInitParameter("allow","128.242.127.1/24,127.0.0.1");
        servletRegistrationBean.addInitParameter("deny","128.242.127.4");
        return servletRegistrationBean;
    }

    /**
     * WebStatFilter用于采集web-jdbc关联监控的数据。
     * @return
     */
    @Bean
    public FilterRegistrationBean webStatFilter(){
        WebStatFilter webStatFilter = new WebStatFilter();
        FilterRegistrationBean<WebStatFilter> filterRegistrationBean = new FilterRegistrationBean<>(webStatFilter);
        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        return filterRegistrationBean;
    }
}
```

## 访问Druid监控页

http://localhost:8080/druid/index.html

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20210110004042.png)

---

# 使用官方starter方式【推荐】

## 导入依赖

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.17</version>
</dependency>
```

## 分析自动配置

- 扩展配置项 **spring.datasource.druid**
- DruidSpringAopConfiguration.**class**,  监控SpringBean的；配置项：**spring.datasource.druid.aop-patterns**
- DruidStatViewServletConfiguration.**class**, 监控页的配置：**spring.datasource.druid.stat-view-servlet；默认开启**
-  DruidWebStatFilterConfiguration.**class**, web监控配置；**spring.datasource.druid.web-stat-filter；默认开启**
- DruidFilterConfiguration.**class**}) 所有Druid自己filter的配置

```java
    private static final String FILTER_STAT_PREFIX = "spring.datasource.druid.filter.stat";
    private static final String FILTER_CONFIG_PREFIX = "spring.datasource.druid.filter.config";
    private static final String FILTER_ENCODING_PREFIX = "spring.datasource.druid.filter.encoding";
    private static final String FILTER_SLF4J_PREFIX = "spring.datasource.druid.filter.slf4j";
    private static final String FILTER_LOG4J_PREFIX = "spring.datasource.druid.filter.log4j";
    private static final String FILTER_LOG4J2_PREFIX = "spring.datasource.druid.filter.log4j2";
    private static final String FILTER_COMMONS_LOG_PREFIX = "spring.datasource.druid.filter.commons-log";
    private static final String FILTER_WALL_PREFIX = "spring.datasource.druid.filter.wall";
```

## 配置示例

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai&characterEncoding=UTF-8

    druid:
      aop-patterns: top.nanzx.*  #监控SpringBean
      filters: stat,wall     # 底层开启功能，stat（sql监控），wall（防火墙）

      stat-view-servlet:   # 配置监控页功能
        enabled: true
        login-username: admin
        login-password: 123456
        resetEnable: false

      web-stat-filter:  # 监控web
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      filter:
        stat:    # 对上面filters里面的stat的详细配置
          slow-sql-millis: 1000
          logSlowSql: true
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false
```

[SpringBoot配置示例](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)

[配置项列表](https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8)

