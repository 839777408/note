---
title: 关于MBG和Mybatis-Plus的自动生成
categories:
  - Mybatis
tags:
  - Mybatis
  - Mybatis-Plus
top_img: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp6.jpg'
cover: 'https://npm.elemecdn.com/nan-picture@1.0.0/img/wp6.jpg'
abbrlink: 6e4f
date: 2021-01-27 23:45:21
updated: 2021-01-27 23:45:33
---

# MBG

[CSDN MyBatis Generator 详解](https://liuzh.blog.csdn.net/article/details/42102297)

[MyBatis Generator官方文档](http://mybatis.org/generator/index.html)

**具体使用：**

Pom.xml文件引入相关依赖：

```xml
<plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.4.0</version>
                <configuration>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                    <configurationFile>src/main/resources/Mybatis-GeneratorConfig.xml</configurationFile>
                </configuration>
                <executions>
                    <execution>
                        <id>Generate MyBatis Artifacts</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.4.0</version>
                    </dependency>
                </dependencies>
            </plugin>
```

pom中配置<configurationFile>的路径下的MBG配置文件（根据需求修改）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--配置要链接的数据库的数据源-->
    <classPathEntry
            location="D:\jdbc驱动\mysql-connector-java-8.0.15\mysql-connector-java-8.0.15/mysql-connector-java-8.0.15.jar"/>
    <!---Mybatis上下文-->
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <commentGenerator>
            <!--去除注释-->
            <property name="suppressAllComments" value="false"/>
            <!--注释中去除日期注释-->
            <property name="suppressDate" value="true"/>
            <!--注释中添加数据库字段备注注释-->
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>

        <!--配置数据库的链接信息-->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/course_center"
                        userId="root"
                        password="123456">
            <!--MySQL 8.x 需要指定服务器的时区-->
            <property name="serverTimezone" value="Asia/Shanghai"/>
            <!--MySQL 不支持 schema 或者 catalog 所以需要添加这个-->
            <!--参考 : http://www.mybatis.org/generator/usage/mysql.html-->
            <property name="nullCatalogMeansCurrent" value="true"/>
            <!-- MySQL8默认启用 SSL ,不关闭会有警告-->
            <property name="useSSL" value="false"/>
        </jdbcConnection>

        <!--为 true时把JDBC DECIMAL 和 NUMERIC 类型解析为java.math.BigDecimal-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!--实体类生成信息-->
        <javaModelGenerator targetPackage="top.nanzx.entity" targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false"/>
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- targetProject:mapper映射文件生成的位置 -->
<!--        <sqlMapGenerator targetPackage="top.nanzx.mapper" targetProject="src\main\resources">-->
<!--            &lt;!&ndash; enableSubPackages:是否让schema作为包的后缀 &ndash;&gt;-->
<!--            <property name="enableSubPackages" value="false"/>-->
<!--            <property name="trimStrings" value="true"/>-->
<!--        </sqlMapGenerator>-->

        <!-- targetPackage：mapper接口生成的位置 -->
<!--        <javaClientGenerator type="XMLMAPPER" targetPackage="top.nanzx.dao" targetProject="src\main\java">-->
<!--            &lt;!&ndash; enableSubPackages:是否让schema作为包的后缀 &ndash;&gt;-->
<!--            <property name="enableSubPackages" value="false"/>-->
<!--        </javaClientGenerator>-->

        <!--要生成的表结构，根据数据库的表追加-->
        <table tableName="student" domainObjectName="Student"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
    </context>

</generatorConfiguration>
```

最后在IDEA的右边栏中选中Maven，执行相应插件：

![](https://npm.elemecdn.com/nan-picture@1.0.0/blog/20210127234326.png)

---



# MyBatis-Plus的代码生成器

注意版本差异，具体查看[官方文档](https://mp.baomidou.com/config/generator-config.html#%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE)：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.2.0</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.2.0</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
</dependency>
```

```java
package top.nanzx.blog;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class CodeGenerator {
    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("nan");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        gc.setServiceName("%sService");
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/blog?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=UTC");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(null);
        pc.setParent("top.nanzx.blog");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/"
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });

        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}
```

---