1.新建一个web maven工程

2.加入依赖 （spring-webmvc，jsp，servlet）

spring-webmvc：会间接把spring的依赖都加入到项目

**3.在web.xml文件中注册Springmvc框架的核心对象DispatcherServlet**

- 1.需要在Tomcat服务器启动后，创建DisPatcherServlet对象的实例:

  使用<load-on-startup>标签

- 2.指定springmvc配置文件的路径

- 3.设置别名

  ```xml
  <!--
  在web.xml文件中注册Springmvc框架的核心对象DispatcherServlet
      需要在Tomcat服务器启动后，创建DisPatcherServlet对象的实例:
      因为DispatcherServlet在他的创建的过程中，会同时创建springmvc容器对象，读取springmvc的配置文件
      把这个配置文件中的对象都创建好，当用户发起请求时就可以直接使用对象了
  
      servlet在初始化会执行init()方法，DisPatcherServlet在init()中{
         //创建容器
         WebApplicationContext ctx = new ClassPathXmlApplicationContext("springmvc.xml")
         //把容器对象放入到ServletContext中
         getServletContext().setAttribute(key,ctx)
      }
    -->
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <!-- 自定义springmvc读取的配置文件的位置
             默认配置文件在/WEB-INF/springmvc-servlet.xml中
         -->
          <init-param>
              <!--  springmvc的配置文件的位置的属性-->
              <param-name>contextConfigLocation</param-name>
              <!-- 指定自定义文件的位置 -->
              <param-value>classpath:springmvc.xml</param-value>
          </init-param>
  
          <!--  在tomcat启动后，创建Servlet对象，值为正整数，值越小，tomcat创建对象的时间越早 -->
          <load-on-startup>1</load-on-startup>
  
      </servlet>
      <servlet-mapping>
          <servlet-name>springmvc</servlet-name>
          <!--  使用框架时，有两个方式可以用：
                 1.使用扩展名方式，语法： *.xxx  xxx是自定义的扩展名
                    常用的方式:*.do, *.action, *.mvc等
                    http://localhost:8080/myweb/some.do
                 2.使用"/"
            -->
          <url-pattern>*.do</url-pattern>
  </servlet-mapping>
  ```

4.创建一个发起请求的页面：index.jsp

**5.创建一个控制器类**

1).在类的上面加入@Controller注解，创建对象，并放入到SpringMVC容器中

2).在类的方法上加入@RequestMapeer注解

```java
@Controller
public class myController {
    // 处理用户提交的请求，springmvc中是使用方法来处理的
    // 方法是自定义的，可以有多种返回值，多种参数，方法名称自定义

    /*  准备使用doSome方法处理some.do请求
     * @RequestMapping:请求映射，作用是把一个请求地址和一个方法绑定在一起
     *                 一个请求指定一个方法处理
     *            属性：1.value 是一个String[，表示请求的uri地址（some.do）
     *                   value的值必须是唯一的，不能重复，在使用时，推荐地址以"/"开头
     *            位置：1.在方法的上面，最常用的
     *                 2.在类的上面
     * 使用@RequestMapping修饰的方法叫做处理器方法或者控制器方法，可以处理请求。类似Servlet中的doGet
     * 能处理请求的都是控制器（处理器）：myController能处理请求，叫做后端控制器
     */
    @RequestMapping(value = "/some.do")
    public ModelAndView doSome(){
        //处理some.do请求
        //返回值：ModeAndView 本次请求处理的数据,视图等
        //       Mode:数据，请求处理完成后，要显示给用户的数据
        //       View:视图,比如：jsp等
        ModelAndView mv = new ModelAndView();
        //添加数据,框架在请求的最后把数据放入到request作用域
        //相当于执行request.setAttribute()方法
        mv.addObject("msg","欢迎使用SpringMVC做web开发");
        mv.addObject("fun","执行的是doSome方法");
        //指定视图,指定视图的完整路径
        //框架对视图执行forward操作:request.DisRequestDisPatcher("/show.jsp").forward(..)
        mv.setViewName("/show.jsp");
        return mv;
    }
}
```

6.创建一个作为结果的jsp，显示请求的处理结果

**7.创建springMVC的配置文件（和spring的配置文件一样）**

1.声明组件扫描器，指定@Controller注解所在的包名

```xml
<context:component-scan base-package="com.xh.controller"/>
```

2.声明视图解析器。帮助处理视图

```xml
<!-- 声明SpringMVC框架中的视图解析器，帮助开发人员设置视图文件路径  -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!--  前缀：视图文件的路径 -->
    <property name = "prefix" value="/WEB-INF/view"/>
    <!-- 后缀：表示视图文件的扩展名   -->
    <property name="suffix" value=".jsp"/>
</bean>
```

```java
mv.setViewName("/show");
//当配置了视图解析器后，可以使用逻辑名称（文件名），指定视图
//框架会使用视图解析器+逻辑名称+后缀来组成完整路径，这里是字符串连接的操作
```

