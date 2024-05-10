在监听器中创建容器，放入全局作用域对象ServlectContext中

监听器的作用：

1.创建容器对象	

2.把容器对象放入到ServlectContext中

监听器可以自己创建，也可以使用框架中提供好的ContextLoaderListener（加入spring-web依赖）

在web.xml中注册监听器

```xml
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
```

使用框架中的方法获取容器对象

```java
//使用框架中的方法，获取容器对象
ServletContext sc = getServletContext();
WebApplicationContext ctx = WebApplicationContextUtils.getRequiredWebApplicationContext(sc);
```

