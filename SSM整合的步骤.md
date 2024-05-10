1.新建一个maven web项目

2.加入依赖：

SpringMVC，Spring，mybatis，jsckson，mysql的驱动，druid连接池，jsp，servlet，aspectj，事务，Spring和mybatis整合

3.创建Controller，service，dao，domain类的包

4.写Spring，SpringMVC，Mybatis的配置文件，数据库的属性的配置文件

Spring的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--资源配置文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--数据源-->
    <bean id="myDataSource" class="com.alibaba.druid.pool.DruidDataSource"
          init-method="init" destroy-method="close">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="maxActive" value="${jdbc.max}"/>
    </bean>

    <!--SqlSessionFactory对象-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="myDataSource"/>
        <property name="configLocation" value="classpath:mybatis.xml"/>

        <!--  mapper文件和dao接口不在一个目录下使用  -->
        <property name="mapperLocations" value="classpath:com/xh/mapper/*.xml" />

    </bean>

    <!--声明Dao对象-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="basePackage" value="com.xh.dao"/><!--Dao接口所在包-->
    </bean>

    <!--声明自定义的Service-->
    <context:component-scan base-package="com.xh.service"/>

    <!-- 事务的配置：注解的配置，AspectJ的配置 -->

</beans>
```

SpringMVC的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--  组件扫描器  -->
    <context:component-scan base-package="com.xh.controller"/>
    <!--  视图解析器 -->
    <bean class = "org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--  声明前缀   -->
        <property name="prefix" value="/WEB-INF/view/"/>
        <!--  声明后缀  -->
        <property name="suffix" value ="*.jsp"/>
    </bean>
    <!-- 声明注解驱动,带有MVC的  -->
    <mvc:annotation-driven/>

    <!-- 声明静态资源的访问,web.xml文件中使用/的方式 -->
    <!-- <mvc:resources mapping="/static/**" location="/static/"/>-->
</beans>
```

Mybatis的配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!-- 设置mybatis输出日志  -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <!-- 设置别名  -->
<!--    <typeAliases>-->
<!--        <package name="实体类的包名"/>-->
<!--    </typeAliases>-->

    <mappers>
        <package name="com.xh.mapper"/>
    </mappers>

</configuration>
```

5.写web.xml文件

1).注册DispatcherServlet，创建SpringMVC的容器对象，接收请求

2).注册Spring的监听器：ContextLoaderListener，创建Spring的容器的对象，创建service，dao的对象

3).注册字符集过滤器，解决POST请求乱码的问题

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 声明中央调度器  -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>

    <!-- 声明监听器  -->
    <context-param>
        <!-- contextConfigLocation表示配置文件的位置-->
        <param-name>contextConfigLocation</param-name>
        <!--  自定义配置文件的位置 -->
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <listener>
        <!-- 配置监听器接口
            默认会在WEB-INF中读取applicationContext.xml文件-->
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

    <!--在web.xml-->
    <!-- 声明过滤器 解决post请求中乱码的问题   -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!-- 设置项目中使用的字符编码    -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <!-- 强制请求对象（HttpServletRequest）使用encoding编码的值    -->
        <init-param>
            <param-name>forceRequestEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
        <!--  强制响应对象（HttpServletResponse）使用encoding编码的值   -->
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

6.写代码和页面