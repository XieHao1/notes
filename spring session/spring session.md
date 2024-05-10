# spring session



## 1.为什么要spring-session

​	在传统单机web应用中，一般使用tomcat/jetty等web容器时，用户的session都是由容器管理。浏览器使用cookie中记录sessionId，容器根据sessionId判断用户是否存在会话session。这里的限制是，**session存储在web容器中，被单台服务器容器管理。**

​		但是网站主键演变，分布式应用和集群是趋势（提高性能）。此时**用户的请求可能被负载分发至不同的服务器**，此时传统的web容器管理用户会话session的方式即行不通。

![image-20220225221154542](img.assats\image-20220225221154542.png)

## 2.如何解决session共享问题

​			Spring Session提供了一套创建和管理Servlet HttpSession的方案。Spring Session提供了集群Session（Clustered Sessions）功能，默认**采用外置的Redis来存储Session数据**，以此来解决Session共享的问题。

![image-20220225222330043](img.assats\image-20220225222330043.png)

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.session/spring-session -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
    <version>1.3.1.RELEASE</version>
</dependency>

```

```markdown
- Spring Session的特性：
- 1、提供用户session管理的API和实现；
- 2、提供HttpSession，以中立的方式取代web容器的session，比如tomcat中的session；
- 3、支持集群的session处理，不必绑定到具体的web容器去解决集群下的session共享问题；
```



## 3.springboot集成SpringSession

1.在构建springBoot项目时勾选Spring Session和 Spring Data Redis

![image-20220225224256506](img.assats\image-20220225224256506.png)

**pom文件**

```xml
<!-- springboot集成redis -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- springSession集成 redis -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

2.在application.propertites文件中配置相关信息

```properties
spring.redis.host=192.168.153.132
spring.redis.port=6379
server.port=8001

#设置springSession的生命周期为30分钟(默认)
#server.reactive.session.timeout=30m

#设置Cookie的存放路径为根路径用于实现同域名不同session共享
#server.servlet.session.cookie.path=/

#设置Cookie的存放，用于实现同根域名不同二级子域名的Session共享
#server.servlet.session.cookie.domain=myWeb.com
```

