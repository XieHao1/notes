@Transactional注解增加事务：适合中小项目使用

Spring给业务方法加事务：在业务方法执行之前，先开启事务，在业务方法之后提交或回滚事务，使用AOP的环绕通知

1.声明事务管理器对象：

```xml
<!-- 使用Spring的事务处理   -->
<bean id = "transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<!-- 连接的数据库  -->
    <property name="dataSource" ref="myDataSource"/>
</bean>
```

2.开启事务注解驱动，告诉sping，使用注解的方式管理事务

```xml
<!--  开启注事务注解驱动,告诉spring使用注解管理事务，创建代理对象 -->
<!-- transaction-manager：事务管理器的id  -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

3.在业务的public方法上加入@Transactional注解

