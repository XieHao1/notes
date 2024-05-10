Spring实现的步骤：

1.创建maven项目

2.加入maven依赖（Spring-context依赖 ioc核心依赖）

3.创建类（接口和它的实现类，也可以为普通类）

4.创建Spring需要使用的配置文件，声明类的信息，这些类由Spring创建和管理

```java
<!--
     一个bean标签声明一个对象
     告诉Spring创建对象
     声明bean，就是告诉Spring要创建某个类的对象
     id：对象的自定义名称，唯一值。Spring通过这个名称找到对象
     class：类的全限定名称(类的，不是接口，因为Spring是反射机制创建对象，必须使用类)

     Spring完成 SomeService someService = new SomeServiceImpl()
     Spring是把创建好的对象放入Map中，Spring框架有一个map存放对象
     SpringMap.pui(id的值,对象)
  -->
<bean id = "someService" class="com.xieHao.service.impl.SomeServiceImpl"/>
```

5.测试Spring创建的类

​	1.指定Spring配置文件的名称

​	2.创建表示Spring容器的对象 ApplicationContext

​	ApplicationContext（接口）表示Spring容器,通过容器获取对象

​	ClassPathXmlApplicationContext 表示从类路径中加载Spring的配置文件

​	3.从容器中获取某个对象，强转为接口类型，调用对象的方法

​	getBean(配置文件中bean的ID值)

```java
@Test
public void test1(){
    //使用Spring容器创建对象
    //1.指定Spring配置文件的名称
    String config = "beans.xml";
    //2.创建表示Spring容器的对象 ApplicationContext
    //ApplicationContext表示Spring容器,通过容器获取对象
    //ClassPathXmlApplicationContext 表示从类路径中加载Spring的配置文件
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    //3.从容器中获取某个对象，强转为接口类型，调用对象的方法
    //getBean(配置文件中bean的ID值)
    SomeService someService = (SomeService) applicationContext.getBean("someService");
    //4.使用方法
```

