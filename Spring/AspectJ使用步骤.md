1.新建maven的项目

2.加入依赖（spring-context，spring-aspects）

3.创建目标类：接口和他的实现类，或者普通类

- ​	 有接口--JDK的动态代理
- ​	没有接口--Cglib动态代理
- ​	有接口也可以使用CGLIB代理

```xml
<!--proxy-target-class="true"告诉框架使用CGLIB代理的方式   -->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

4.创建切面类：普通类，

​	1.在类的上面加入@Aspect

​	2.在类中定义方法，方法就是切面要执行的功能代码，在方法的上面加入aspectj的通知注解，例如：@Before，需要指定切入点表达式execution（）

5.创建Spring的配置文件：声明对象，把对象交给容器统一管理：声明对象可以使用注解或者XML文件配置bean

​	1.声明目标对象

​	2.声明切面类对象

​	3.声明aspectj框架中的自动代理生成器标签

​     自动代理生成类：用来完成代理对象的自动创建功能

```xml
<aop:aspectj-autoproxy/>
```

6.创建测试类，从Spring容器中获取目标对象（实际上为代理对象）。通过代理执行方法，实现AOP的功能增强