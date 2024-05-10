使用AspectJ配置事务：适合大型项目

实现步骤：都是在配置文件中实现的

**1.加入AspectJ依赖**

**2.声明事务管理器对象**

```xml
<!-- 使用Spring的事务处理   -->
<bean id = "transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<!-- 连接的数据库  -->
    <property name="dataSource" ref="myDataSource"/>
</bean>
```

**3.声明方法需要的事务的类型(配置方法的事务属性【隔离级别，传播行为，超时】)**

```xml
<!-- 声明业务方法它的事务属性【隔离级别,传播行为,超时时间】  -->
<!-- id:自定义名称,来表示tx:advice 和 </tx:advice> 直接的配置内容 -->
<!-- transaction-manager:事务管理器对象的id  -->
<tx:advice id="myAdvice" transaction-manager="transactionManager">
    <!--attributes:配置事务的属性 -->
    <tx:attributes>
       <!-- 给具体的方法配置事务属性，method可以有多个，分别给不同的方法设置事务  -->
        <!--  name:方法的名称 1):完整的方法名称，不带有包和类
                            2):方法名可以使用通配符 *,表示任意字符
               propagation:传播行为
               isolation:隔离级别
               rollback-for:指定的异常名，全限定类型名
          -->
        <tx:method name="buy" propagation="REQUIRED" isolation="DEFAULT"
                   rollback-for="com.xh.exection.CommodtiyExecption"/>
        <!--  使用通配符，指定很多的方法   -->
        <tx:method name="add*" propagation="NOT_SUPPORTED"/>
        <tx:method name="modify*"/>
        <tx:method name="remove*"/>
    </tx:attributes>
</tx:advice>
```

**4.配置AOP，指定哪些类要实现代理**

```xml
<!--  配置AOP  -->
<aop:config>
    <!--  配置切入点表达式：指定哪些包中类 要使用事务
        id:切入点表达式的名称,唯一值
        expression：切入点表达式，指定哪些类要使用事务，aspectJ会创建代理对象
    -->
    <aop:pointcut id="servicePt" expression="execution(* *..service*..*.*(..))"/>
    <!--  配置增强器：关联advice和pointcut  -->
    <!--  advice-ref: tx:advice中的id，关联advice
          pointcut-ref：aop:pointcut中的id 关联pointcut  -->
    <aop:advisor advice-ref="myAdvice" pointcut-ref="servicePt"/>
</aop:config>
```

