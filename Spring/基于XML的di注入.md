1.简单的set注入

```xml
一个property只能给一个属性赋值  
<bean id = "" class = "">
​   <property name = "属性名字" value = “属性的值"/>
​ </bean>
   只调用set方法，有set方法就可以执行（找方法，不找属性），XML文件中规定，值必须放在" "之中
```

2.引用类型的set注入

```xml
一个property只能给一个属性赋值
<bean id = "" class = "">
	 <property name = "属性名字" ref= “bean的id(对象的名称)"/>
</bean>
需要创建两个bean标签
```

3.构造注入

```xml
使用标签 <constructor-arg>标签

一个<constructor-arg>标签表示构造方法的一个参数

<constructor-arg>标签的属性：

​     name:表示构造方法的形参名

​     index:表示构造方法的参数的位置，从0开始

​     value：构造方法的属性是简单类型的，使用value

​     ref：构造方法的形参是引用类型的，使用ref
```

4.引用类型自动注入byName

```xml
byName：按名称注入，java类引用类型的属性名和Spring容器中（配置文件）<bean>的id名称一样，且数据类型是一致的，这样容器中的bean，Spring能够赋值给引用类型
语法规则:
<bean id = "" class = ""  autowire = "byName">
    autowire = "byName"相当于 ref = "bean的id(对象的名称)"
简单类型属性赋值

</bean>
```

5.引用类型自动注入byType

```xml
byType：按属性注入，java类中引用类型的数据和Spring容器中（配置文件）的class属性是同源（一类）关系的，这样的bean可以赋值给引用类型

同源：
1.java类中引用数据类型和bean的class的值是一样的

2.java类引用类型的数据类型和bean的class是父子类关系

3.ava类引用类型的数据类型和bean的class是接口和实现类关系

语法规则:

<bean id = "" class = ""  autowire = "byType">

简单类型属性赋值

</bean>

在byType中，在XML文件中声明bean只能有一个符合条件的，多出一个是错误的（以上三种同源关系，只能存在一种）
```

