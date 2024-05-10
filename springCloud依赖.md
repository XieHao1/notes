```xml
<!--spring boot 2.6.4-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.6.4</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>

<dependency>
     <groupId>org.projectlombok</groupId>
     <artifactId>lombok</artifactId>
     <optional>true</optional>
</dependency>

<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-test</artifactId>
     <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!--web监控依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



# springCloud使用2021.0.1版本

```xml
<!-- 2021.0.x aka Jubilee -->
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>2021.0.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```



# Ereueka

```xml
<!-- 2021.0.1 springCloud版本 eureka-->
<!--eureka-server-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   <version>3.1.1</version>
</dependency>

<!-- 2021.0.1 springCloud版本 eureka-client -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  <version>3.1.1</version>
</dependency>
```



# zookeeper

```xml
<!-- zookeeper依赖,和自己安装的zookeeper版本保持一致-->
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
  <version>3.5.7</version>
</dependency>

<!-- SpringBoot整合zookeeper客户端 -->
<!-- springCloud 2021.0.1 版本使用zookeeper3.1.1版本-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <version>3.1.1</version>
    <!--先排除自带的zookeeper版本 防止与自己的版本起冲突起冲突-->
     <exclusions>
        <exclusion>
           <groupId>org.apache.zookeeper</groupId>
           <artifactId>zookeeper</artifactId>
        </exclusion>
     </exclusions>
</dependency>
```



# Consul

```xml
<!--  springCloud 2021.0.1 版本使用consul 3.1.0 版本-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```



# Ribbon

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```



# LoadBancer

```xml
<!--Spring Cloud 2021.0.1版本 loadbalancer 3.1.1-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    <version>3.1.1</version>
</dependency>
```



# OpenFegin

```xml
<!-- springCloud 2021.0.1版本 --- openfeign 3.1.1-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>3.1.1</version>
</dependency>
```



# hystrix

```xml
<!-- hystrix -->
<dependency>
     <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
      <version>2.2.10.RELEASE</version>
</dependency>

<!-- SpringCloud 整合hystrix 可视化页面 -->
<!-- 整合依赖中包含springBoot和SpringCloud的启动依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```



# Gateway

```xml
<!-- springCloud 2021.0.1 版本使用Gateway3.1.1 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>3.1.1</version>
</dependency>
```



# Config

```xml
<!-- springCloud 2021.0.1 版本使用 config-server3.1.1 版本-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
    <version>3.1.1</version>
</dependency>
        
<!-- springCloud 2021.0.1 版本使用 config-client  3.1.1 版本 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>3.1.1</version>
</dependency>
```



# bus

```xml
<!--  消息总线RabbitMQ支持- -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>3.1.0</version>
</dependency>
```



# Stream

```xml
<!-- springCloud stream使用rabbitMQ为中间件 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    <version>3.2.2</version>
</dependency>
```



# sleuth

```xml
<!-- springCloud 2021.0.1版本 包含了sleuth+zipkin  -->
<!--使用该依赖无法在网页上显示，原因未知-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-sleuth-zipkin</artifactId>
   <version>3.1.1</version>
</dependency> 
       
<!--使用该依赖可以正常在客户端显示-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-zipkin</artifactId>
   <version>2.2.8.RELEASE</version>
</dependency>
```

