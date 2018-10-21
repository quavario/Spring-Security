# Spring Security系列教程--快速入门

> 前言 
>
> > 因为最近工作中要用到Spring Security ,  之前只接触过Shiro,  花几天学习了Spring Security,  一些心得和大家分享,  欢迎转载,  请注明出处     ----赵群     

#### Spring Security简介

Spring Security是一个功能强大且可高度自定义的身份验证和访问控制框架。 Spring Security是一个专注于为Java应用程序提供身份验证和授权的框架。

> 翻译自:https://spring.io/projects/spring-security

这是Spring官方网站对Spring Security的简单概述,   和Shiro类似,  Spring Security也是一个权限认证框架.

-----



### 快速入门小DEMO

简单了解Spring Security,  现在写个小DEMO,   难点在于配置文件

#### 开发工具

- Intellij Idea
- Maven



#### 开发步骤

1. 创建Maven项目
2. 引入依赖
3. 配置web.xml
4. 权限配置文件
5. 启动tomcat

##### 创建Maven项目

这里打包方式为war包

```xml
<packaging>war</packaging>
<groupId>com.spring</groupId>
<artifactId>security-demo1</artifactId>
<version>1.0-SNAPSHOT</version>
```

##### 引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <packaging>war</packaging>
    <groupId>com.spring</groupId>
    <artifactId>security-demo1</artifactId>
    <version>1.0-SNAPSHOT</version>

 <properties>
    <spring.version>4.2.4.RELEASE</spring.version>
 </properties>
    <dependencies>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--spring security web-->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
            <version>4.1.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
            <version>4.1.0.RELEASE</version>
        </dependency>
        <!--servlet api-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- java编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.2</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <!-- 指定端口 -->
                    <port>9001</port>
                    <!-- 请求路径 -->
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

##### 配置web.xml

除了需要配置spring的监听器,还要配置一个`filter`,   这个`filter` 的`filter-name` **必须为`springSecurityFilterChain`** ,因为这里配置的filter---`DelegatingFilterProxy`  其实是一个代理filter,实际起拦截作用的是`springSecurityFilterChain` 

Spring Security的配置文件为`spring-security.xml`(文件名任意,也可以是其他的),  在resource下创建,  接下来我们就会对其进行配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-security.xml</param-value>
    </context-param>
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
</web-app>
```

##### 配置spring-security.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
             xmlns:beans="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">

	<!--对页面的拦截规则-->
    <!--use-expressions : 是否使用spel表达式,默认为true-->
    <http use-expressions="false">
        <!--拦截的url-->
        <!--
        /* : 不拦截子目录
        /**: 拦截子目录
        access : 当前用户必须有ROLE_USER角色才可以访问根目录及子目录
        -->
        <intercept-url pattern="/*" access="ROLE_USER"/>
        
        <!--开启表单登录功能-->
        <form-login/>
    </http>

    <!--认证管理器-->
    <authentication-manager>

        <!--认证提供者-->
        <authentication-provider>
            <user-service>
                <!--配置当前用户的名称-->
                <!--
                name: 用户名
                password : 密码
                authorities : 角色
                -->
                <user name="admin" authorities="ROLE_USER" password="123456"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>

</beans:beans>
```

这是一个基本的配置文档,  我们来看下他的基本结构

- `http` :  用于配置页面的拦截规则,  `use-expressions` 表示是否使用`SpEl`表达式 

  - `intercept-url` : 配置拦截url  
    - `pattern` : 拦截的目录,  配置`/*` 拦截根目录,   不拦截子目录, `/**` 可以拦截根目录和子目录  
    - `access` : 表示拥有什么权限才能访问我们在`pattern` 上配置的目录中的资源
  - `form-login` : 开启表单登录功能,  配置后spring为默认为我们提供登录界面

> 注1: `<form-login/>` 有很多属性,  为了演示我没有配置,  如果没有配置任何属性, spring会默认提供一个登录页面,  当未登录访问资源时,  就会跳转到这个页面.  实际开发中会配置其他属性,  我们仅作演示用
  
> 注2:  只有配置http标签的`use-expressions="false"`  关闭spEl表达式,  在`intercept-url` 中才能配置`access="ROLE_USER"` ,  否则就要使用spEL表达式方式配置`access="hasRole('ROLE_USER')`

- `authentication-manager` :  认证管理器

  - `authentication-provider` : 认证提供者

    - `user-service ` : 可以创建用户和赋予权限

      - `user` : 通过该标签可以创建用户并赋予权限,  `name`/`password`为登录名密码,  通过`authorities`属性可以为该角色赋予权限,  配置为`ROLE_USER` 我们刚才创建的角色,  这样配置后`admin` 用户就有了`ROLE_USER` 角色,  这样就可以访问`/* ` 下的资源

        > 为了方便演示,  直接将用户名和密码和角色数据写在了配置文件中,  实际工作开发中一般不用这种方法,  而是将用户信息和角色写在数据库中.   

##### 启动tomact

运行之前,我们在在`webapp`目录下创建一个html页面 `index.html` ,  我们会针对这个页面进行权限校验. 

```html
<!--index.html-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
欢迎进入Spring Security!!!
</body>
</html>
```



因为我们配置了拦截`/*` 目录,  所以我们要访问`index.html` 就要先进行权限验证.  选择`tomcat7:run` 启动tomcat

![1539580766363png](http://188.131.133.113/upload/8e249c2f6f4b47e6abe41f334b124b19_1539580766363.png) 



启动成功后,  打开`localhost:9001` ,   之前没用`Spring Security` 之前我们可以直接访问`index.html` . 现在我们打开主页会进入下面这个页面,  这就是我之前提到的spring security默认提供的登录界面

![1539581279092png](http://188.131.133.113/upload/7099ab6a0a47426b870590792298a33e_1539581279092.png) 



输入我们之前配置的账号密码

```html
User : admin
Password : 123456
```

通过权限校验,  登录成功了,   现在大家知道怎么使用Spring Security了吗,   有时间我会在写一篇进阶教程!

![1539581401624png](http://188.131.133.113/upload/0ca062836ec1418188ec494567adff45_1539581401624.png) 

