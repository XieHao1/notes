集成的步骤：

1.创建新的Maven项目

2.加入Maven依赖（Spring-context,MyBatis,mysql,spring事务，spring和mybatis集成依赖）

3.创建实体类

4.创建dao接口和mpper文件

5.创建mybatis的主配置文件

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

    <mappers>
        <package name="com.xh.mapper"/>
    </mappers>

</configuration>
```

6.创建Service接口和实现类，属性Dao

**7.创建spring的配置文件：声明mybatis的对象，交给Spring创建：**

**1）：数据源：**

```xml
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
```

**2）：SqlSessionFactory对象**

```xml
<!--SqlSessionFactory对象-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="myDataSource"/>
    <property name="configLocation" value="classpath:mybatis-congif.xml"/>

    <!--  mapper文件和dao接口不在一个目录下使用   -->
    <property name="mapperLocations" value="classpath:com/xh/mapper/*.xml" />
</bean>
```

**3）：声明Dao对象**

```xml
<!--声明Dao对象-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <property name="basePackage" value="com.xh.dao"/><!--Dao接口所在包-->
</bean>
```

**4）：声明自定义的Service**

```xml
<bean id = "loginImpl" class="com.xh.service.impl.LoginImpl">
    <property name="userDao" ref="userDao"/>
</bean>
```