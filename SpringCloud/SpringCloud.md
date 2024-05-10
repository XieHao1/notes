# SpringCloud

[TOC]



# 一.springCloud技术栈

![image-20220507134256822](img.assets\image-20220507134256822.png)

# 二.springBoot和springCloud的依赖关系

1.从springCloud官网查看springboot和SpringCloud的依赖关系

![image-20220507133548014](img.assets\image-20220507133548014.png)

2.Spring Boot 与 Spring Cloud 兼容性查看

版本接口：`https://start.spring.io/actuator/info`

![image-20220507135156235](img.assets\image-20220507135156235.png)

# 三.springCloud组件使用

![cloud升级](img.assets\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2OTAzMjYx,size_16,color_FFFFFF,t_70)



# 四.微服务架构编码构建

## 1.创建父工程

**编程风格：约定 > 配置 > 编码**

创建微服务cloud整体聚合父工程Project步骤

### 1.1.New Project

<img src="img.assets\image-20220507141515062.png" alt="image-20220507141515062" style="zoom: 50%;" />

### 1.2.设置字符编码

<img src="img.assets\image-20220507141949810.png" alt="image-20220507141949810" style="zoom: 50%;" />

### 1.3.注解生效激活

<img src="img.assets\image-20220507142342650.png" alt="image-20220507142342650" style="zoom:50%;" />

### 1.4.创建父工程

父工程创建完成执行`mvn : install`将父工程发布到仓库方便子工程继承。

```xml
<groupId>com.xh</groupId>
  <artifactId>springCloud2022</artifactId>
  <version>1.0.0</version>
  <!-- 1.父工程的 packaging 标签的文本内容必须设置为pom-->
  <!-- 2.删除src -->
  <packaging>pom</packaging>

  <!--统一管理jar包版本-->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <junit.version>4.12</junit.version>
    <log4j.version>1.2.17</log4j.version>
    <lombok.version>1.16.18</lombok.version>
    <mysql.version>5.1.30</mysql.version>
    <druid.version>1.2.8</druid.version>
    <mybaitsPlus.version>3.4.3.4</mybaitsPlus.version>
  </properties>
  
  <!--子模块继承之后，提供作用：锁定版本+子module不用写groupId和version-->
  <dependencyManagement>
    <dependencies>
      
      <!--spring boot 2.6.4-->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.6.4</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      
      <!-- 2021.0.x aka Jubilee -->
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>2021.0.1</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>

      <!--spring cloud alibaba-->
      <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <version>2.1.0.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>

      <!--mysql-->
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
        <scope>runtime</scope>
      </dependency>

      <!-- druid-->
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>${druid.version}</version>
      </dependency>

      <!--mybatis-plus起步依赖-->
      <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>${mybaitsPlus.version}</version>
      </dependency>

      <!--junit-->
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
      </dependency>
      <!--log4j-->
      <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
      </dependency>
    </dependencies>

  </dependencyManagement>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <fork>true</fork>
          <addResources>true</addResources>
        </configuration>
      </plugin>
    </plugins>
  </build>

```

#### 1.4.1 DependencyManagement和Dependencies

Maven使用`dependencyManagement`元素来提供了一种管理依赖版本号的方式。

通常会在一个组织或者项目的最顶层的父POM中看到`dependencyManagement`元素。

使用pom.xml中的`dependencyManagement`元素能让所有在子项目中引用个依赖而不用显式的列出版本量。

Maven会沿着父子层次向上走，直到找到一个拥有`dependencyManagement`元素的项目，然后它就会使用这个
`dependencyManagement`元素中指定的版本号。

注意 ：

- `dependencyManagement`里只是声明依赖，**并不实现引入**，因此**子项目需要显示的声明需要用的依赖**。
- 如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项,并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom。
- 如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。



#### 1.4.2 maven跳过单元测试

IDEA右侧旁的Maven插件有`Toggle ' Skip Tests' Mode`按钮，这样maven可以跳过单元测试

<img src="img.assets\image-20220507150114832.png" alt="image-20220507150114832" style="zoom:50%;" />

## 2.支付模块构建

创建微服务模块套路：

1. 建Module
2. 改POM
3. 写YML
4. 主启动
5. 业务类

![image-20220507151344884](img.assets\image-20220507151344884.png)

### 2.1.编写公共接口

将实体类，vo类，第三方工具类等添加到公共接口中

<img src="img.assets\image-20220507181934273.png" alt="image-20220507181934273" style="zoom: 67%;" />

### 2.2.创建服务提供者

创建cloud-provider-payment8001微服务提供者支付Module模块：

#### 2.2.1.建名为cloud-provider-payment8001的Maven工程

pom文件：

```xml
<parent>
        <artifactId>springCloud2022</artifactId>
        <groupId>com.xh</groupId>
        <version>1.0.0</version>
</parent>
```

父工程pom文件：

```xml
<!-- 子module-->
<modules>
  <module>cloud-provider-payment8001</module>
</modules>
```

#### 2.2.2.改POM

```xml
<dependencies>

         <dependency>
        <!--  自己的公共接口地址   -->
            <groupId>com.xh</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
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

</dependencies>
```

#### 2.2.3.写YML

```yml
#微服务端口号
server:
  port: 8001
#微服务名称
spring:
  application:
    name: cloud-payment-service

  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root

mybatis-plus:
  mapper-locations: classpath:/mapper/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

```

#### 2.2.4.主启动

```java
@SpringBootApplication
public class Payment8001 {
    public static void main(String[] args) {
        SpringApplication.run(Payment8001.class);
    }
}
```

#### 2.2.5.业务类



### 3.创建消费者

创建名为cloud-consumer-order90的maven工程。



#### 3.1.RestTemplate模板

[RestTemplate Java Doc](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)

RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集。

- 使用restTemplate访问restful接口非常的简单粗暴无脑。
- `(url, requestMap, ResponseBean.class)`这三个参数分别代表。
- REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。



```
getForObject() / getForEntity() - GET请求方法

getForObject()：返回对象为响应体中数据转化成的对象，基本上可以理解为Json。

getForEntity()：返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等。

postForObject() / postForEntity() - POST请求方法
```



配置类：

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    //开启本地负载均衡调用
    @LoadBalance
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```



#### 3.2.消费者调用服务

```java
@RestController
@Slf4j
@RequestMapping("/consumer")
public class OrderController {

    public static final String PAYMENT_URL = "http://localhost:8001";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/payment/get/{id}")
    public ResultJSON<Payment> getPayment(@PathVariable("id") Long id) {
        log.info("id:{}",id);
        return restTemplate.getForObject(PAYMENT_URL+"/payment/getPayment/"+id,ResultJSON.class);
    }

    @GetMapping("/payment/create")
    public ResultJSON<Payment> create(Payment payment){
        log.info("id:{},serial:{}",payment.getId(),payment.getSerial());
        return restTemplate.postForObject(PAYMENT_URL+"/payment/create",payment,ResultJSON.class);
    }
}

```



# 五.服务注册与发现

## 1.Eureka

**服务治理**

Spring Cloud封装了Netflix 公司开发的Eureka模块来实现服务治理

在传统的RPC远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。



### 1.1 Eureka 原理说明

<img src="img.assets\14570c4b7c4dd8653be6211da2675e45.png" alt="img" style="zoom:80%;" />

服务注册：将服务信息注册进注册中心

服务发现：从注册中心上获取服务信息

**实质：存key服务命取value调用地址**



1先启动eureka注主册中心

2启动服务提供者payment支付服务

3支付服务启动后会把自身信息（比如服务地址以别名方式注进eureka）

4消费者order服务在需要调用接口时，使用服务别名去注册中心获取实际的RPC远程调用地址

5消去者导调用地址后，底屋实际是利用HttpClient技术实现远程调用

6消费者获得导服务地址后会缓存在本地jvm内存中，默认每间隔30秒更新—次服务调用地址



**服务注册与发现**

Eureka采用了CS的设计架构，Eureka Sever作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。

在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息比如服务地址通讯地址等**以别名(微服务的名称)方式注册到注册中心上**。另一方(消费者服务提供者)，以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用,框架核心设计思想:在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何RPC远程框架中，都会有一个注册中心存放服务地址相关信息(接口地址)

**注册中心只保存服务提供的地址和监听服务的上下线，不负责调用**

![image-20220514144017225](img.assets\image-20220514144017225.png)



**Eureka包含两个组件:Eureka Server和Eureka Client**

**Eureka Server**提供服务注册服务

各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

**EurekaClient**通过注册中心进行访问

它是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒)



### 1.2 单机版eureka

#### 1.2.1 声明服务的注册中心

1.创建名为cloud-eureka-server7001的Maven工程

2.pom文件

```xml
<!-- 2021.0.1 springCloud版本 eureka-->
<!--eureka-server-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   <version>3.1.1</version>
</dependency>
```

3.写yml

```yml
server:
  port: 7001
    
#erueak配置
eureka:
  instance:
    hostname: locathost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

4.启动类

```java
@SpringBootApplication
//声明该模块为eureka的服务注册中心
@EnableEurekaServer
public class CloudEurekaServer {

    public static void main(String[] args) {
        SpringApplication.run(CloudEurekaServer.class);
    }
}
```

5.测试运行`EurekaMain7001`，浏览器输入`http://localhost:7001/`回车，会查看到Spring Eureka服务主页。

![image-20220514151006927](img.assets\image-20220514151006927.png)

#### 1.2.2 将服务注册到注册中心

##### 1.2.2.1 支付微服务8001入驻进EurekaServer

添加spring-cloud-starter-netflix-eureka-client依赖

```xml
<!-- 2021.0.1 springCloud版本 eureka-client -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  <version>3.1.1</version>
</dependency>
```

yml

```yml
eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

启动类:

```java
@SpringBootApplication
//声明eureka客户端
@EnableEurekaClient
public class Payment8001 {

    public static void main(String[] args) {
        SpringApplication.run(Payment8001.class);
    }
}
```

在eureka客户端中查看相关信息 **该注册的服务名称与spring-application-name 相同**

- 浏览器输入 - http://localhost:7001/ 主页内的**Instances currently registered with Eureka**会显示cloud-provider-payment8001的配置文件application.yml设置的应用名`cloud-payment-service`

![image-20220514152637690](img.assets\image-20220514152637690.png)

eureka的自我保护机制

![image-20220514152954380](img.assets\image-20220514152954380.png)

紧急情况！EUREKA可能错误地声称实例在没有启动的情况下启动了。续订小于阈值，因此实例不会为了安全而过期。



##### 1.2.2.2 将微服务90入驻进EurekaServer

![image-20220514153542280](img.assets\image-20220514153542280.png)



### 1.2 集群版 erueka

微服务RPC远程服务调用最核心的是什么：
**高可用，**试想你的注册中心只有一个only one，万一它出故障了，会导致整个为服务环境不可用。

**互相注册，相互守望**

![image-20220514160019677](img.assets\image-20220514160019677.png)

#### 1.2.1 Eureka集群环境构建

创建cloud-eureka-server7002工程

- 找到C:\Windows\System32\drivers\etc路径下的hosts文件，修改映射配置添加进hosts文件

- ```
  127.0.0.1 eureka7001.com
  127.0.0.1 eureka7002.com
  ```

修改cloud-eureka-server7001配置文件

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
    #集群指向其它eureka 互相注册，相互守望
      defaultZone: http://eureka7002.com:7002/eureka/
    #单机就是7001自己
      #defaultZone: http://eureka7001.com:7001/eureka/
```

修改cloud-eureka-server7002配置文件

```yml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
    #集群指向其它eureka
      defaultZone: http://eureka7001.com:7001/eureka/
```

分别启动和eureka客户端变化

![image-20220514162532191](img.assets\image-20220514162532191.png)

#### 1.2.2 订单支付两微服务注册进Eureka集群

- 将支付服务8001微服务，订单服务90微服务发布到上面2台Eureka集群配置中

将它们的配置文件的eureka.client.service-url.defaultZone进行修改

```yml
eureka:
  client:
    #表示是否将自己注册进Eureka-server默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    #    service-url:
    #      defaultZone: http://localhost:7001/eureka
    #集群,eureka之间会相互复制消息，配置一个也可以
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
```

测试

1. 先要启动EurekaServer，7001/7002服务
2. 再要启动服务提供者provider，8001
3. 再要启动消费者，80

![image-20220514183848896](img.assets\image-20220514183848896.png)



#### 1.2.3 支付8001微服务集群配置

新建cloud-provider-payment8002

![image-20220514190152830](img.assets\image-20220514190152830.png)



#### 1.2.4 consumer90调用服务修改

cloud-consumer-order90订单服务访问地址不能写死，而是**采用微服务的别名作为地址**

```java
 public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
```

若直接访问，则会报错

```java
java.net.UnknownHostException: CLOUD-PAYMENT-SERVICE
```

RestTemplate返送请求时不知道应该将请求发送到该微服务别名下的哪个端口中，**因此需要设置负载均衡**

**使用@LoadBalanced注解赋予RestTemplate负载均衡的能力,默认方式为轮询 **

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```



#### 1.2.5 actuator微服务信息完善



**eureka.instance下hostname，instance-id，prefer-ip-address的作用及区别**

`eureka.instance`下的`hostname`即主机名不配置的话默认为电脑名，

`instanceID`不配置的话默认值为主机名+服务名+端口，

`prefer-ip-address`表示猜测主机名(hostname)为ip形式，不配置的话默认为false



##### 1.2.5.1 机名称：服务名称修改

（也就是将IP地址，换成可读性高的名字）

在一般情况下，web和actuator要一起导入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

在yml文件中添加：

**instance和 client平级**

```yml
eureka:
  client:
    #表示是否将自己注册进Eureka-server默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    #    service-url:
    #      defaultZone: http://localhost:7001/eureka
    #集群,eureka之间会相互复制消息，配置一个也可以
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
    #添加此处
  instance:
    instance-id: payment8002
```

![image-20220514193440911](img.assets\image-20220514193440911.png)



##### 1.2.5.2 访问信息有IP信息提示

（就是将鼠标指针移至payment8001，payment8002名下，会有IP地址提示）

```yml
eureka:
  ...
  instance:
    instance-id: payment8001 
    prefer-ip-address: true #添加此处
```



### 1.3 服务发现Discovery

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息

- 修改cloud-provider-payment8001的Controller

```java
@Resource
private DiscoveryClient discoveryClient;
    
@GetMapping("/discovery")
    public Object discovery(){
        //获取服务列表信息
        List<String> services = discoveryClient.getServices();
        for (String msg : services){
            log.info("service:{}",msg);
        }
        //通过微服务的名称获取
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance serviceInstance:instances){
            log.info(serviceInstance.getServiceId()+"\t"+serviceInstance.getHost()
                     +"\t"+serviceInstance.getPort()+"\t"+serviceInstance.getUri());
        }
        return this.discoveryClient;
    }
```

在主启动类上加上**@EnableDiscoveryClient**注解

```java
@SpringBootApplication
//声明eureka客户端
@EnableEurekaClient
//开启服务发现Discovery
@EnableDiscoveryClient
public class Payment8001 {

    public static void main(String[] args) {
        SpringApplication.run(Payment8001.class);
    }
}
```

访问后浏览器显示:

```
{"services":["cloud-payment-service","cloud-consumner-order"],"order":0}
```

日志结果

```
service:cloud-payment-service
service:cloud-consumner-order
CLOUD-PAYMENT-SERVICE	localhost	8002	http://localhost:8002
CLOUD-PAYMENT-SERVICE	192.168.153.1	8001	http://192.168.153.1:8001
```



### 1.4 Eureka自我保护理论知识

如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式:

EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY’RE NOT. RENEWALS ARE LESSER THANTHRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUSTTO BE SAFE



#### 1.4.1 导致原因

**某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存。属于CAP里面的AP分支。**



#### 1.4.2 为什么会产生Eureka自我保护机制?

为了EurekaClient可以正常运行，防止与EurekaServer网络不通情况下，EurekaServer不会立刻将EurekaClient服务剔除



#### 1.4.3 什么是自我保护模式?

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例**(默认90秒)**。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时(可能发生了网络分区故障)，那么这个节点就会进入自我保护模式。



**在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例**。

它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。



#### 1.4.4 禁止自我保护

在**server**端使用`eureka.server.enable-self-preservation = false`可以禁用自我保护模式

```yml
eureka:
  ...
  server:
    #关闭自我保护机制，保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

关闭效果：

spring-eureka主页会显示出一句：

**THE SELF PRESERVATION MODE IS TURNED OFF. THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.**



- 生产者客户端eureakeClient端8001

默认：

```properties
#心跳检测与续约时间
#开发时没置小些，保证服务关闭后注册中心能即使剔除服务
#Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
eureka.instance.lease-renewal-interval-in-seconds=30
#Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
eureka.instance.lease-expiration-duration-in-seconds=90
```

```yml
eureka:
  ...
  instance:
    instance-id: payment8001
    prefer-ip-address: true
    #心跳检测与续约时间
    #开发时没置小些，保证服务关闭后注册中心能即使剔除服务
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

## 

## 2.Zookeeper

zookeeper是一个分布式协调工具，可以实现注册中心功能

ZooKeeper的服务节点是**临时节点**,

```xml
<!-- zookeeper依赖,和自己安装的zookeeper版本保持一致-->
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
  <version>3.5.7</version>
</dependency>
```

```xml
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



### 2.1 声明服务提供者

新建名为cloud-provider-zookeeper-payment8004的Maven工程

1.pom文件

```xml
		<!-- zookeeper依赖-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.7</version>
            <!--zookeeper和lombok都有slf4j,排除这个slf4j-log4j12-->
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- SpringBoot整合zookeeper客户端 -->
        <!-- springCloud 2021.0.1 版本使用zookeeper3.1.1版本-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper版本 防止与自己的版本起冲突起冲突-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

2.yml文件

```yml
server:
  port: 8804

#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 121.41.112.246:2181
```

3.主启动类

```java
@SpringBootApplication
//该注解用于向使用consul或者zookeeper作为注册中心时注册服务
@EnableDiscoveryClient
public class Payment8004 {
    public static void main(String[] args) {
        SpringApplication.run(Payment8004.class);
    }
}
```



### 2.2 声明服务消费者

pom文件

```xml
  		<!-- zookeeper依赖-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.7</version>
            <!--排除这个slf4j-log4j12-->
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- SpringBoot整合zookeeper客户端 -->
        <!-- springCloud 2021.0.1 版本使用zookeeper3.1.1版本-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper版本 防止与自己的版本起冲突起冲突-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

2.yml文件

```yml
#微服务端口号
server:
  port: 90
#微服务名称
spring:
  application:
    name: cloud-consumner-order
  cloud:
    zookeeper:
      connect-string: 121.41.112.246:2181
```

3.主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Consumer90 {

    public static void main(String[] args) {
        SpringApplication.run(Consumer90.class);
    }
}
```

restTemplate模板配置类

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}

```

4.调用接口

```java
@RestController
@Slf4j
public class OrderZKController {

    public static final String INVOKE_URL = "http://cloud-provider-payment";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/zk")
    public String paymentInfo() {
        return restTemplate.getForObject(INVOKE_URL+"/payment/zk",String.class);
    }
}
```



## 3.Consul

Consul是一套开源的分布式服务发现和配置管理系统，由HashiCorp 公司用Go语言开发。

提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。

它具有很多优点。包括：基于raft协议，比较简洁；支持健康检查，同时支持HTTP和DNS协议支持跨数据中心的WAN集群提供图形界面跨平台，支持Linux、Mac、Windows。

[Spring Cloud Consul 中文文档 参考手册 中文版](https://www.springcloud.cc/spring-cloud-consul.html)



- Spring Cloud Consul features:
  - 服务发现：实例可以用Consul代理端注册，客户端可以发现使用Spring管理bean实例。
  - 支持Ribbon，利用Spring Cloud Netflix实现客户端负载均衡。
  - 支持Zuul，利用Spring Cloud Netflix实现动态路由和过滤。
  - 分布式配置：利用Consul的Key/Value存储。
  - 控制总线：使用Consul事件处理分布式控制事件



### 3.1 consul安装

CentOS 安装

```shell
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install consul
```

启动：若要使用可视化页面，开放8500端口,**页面地址:ip:8500**

```shell
consul agent -dev -client=0.0.0.0 #后台启动
```

consul命令:

```shell
consul leave #关闭节点
consul reload #重新加载配置文件
consul catalog services #查询所有注册的服务
```

consul可视化页面

![image-20220516102620224](img.assets\image-20220516102620224.png)



### 3.2 声明服务提供者

新建名为cloud-provider-cousul-payment8006的Maven工程

```xml
<!--  springCloud 2021.0.1 版本使用consul 3.1.0 版本-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

yml文件

```yml
server:
  port: 8006

spring:
  application:
    name: cloud-provider-payment
  #consul注册中心地址
  cloud:
    consul:
      host: 121.41.112.246
      port: 8500
      discovery:
        #微服务ip地址
        hostname: 127.0.0.1
        #获取的默认服务名称，实例ID和端口分别为${spring.application.name}，Spring上下文ID和${server.port}。
        service-name: ${spring.application.name}
        #首选ip地址
        prefer-ip-address: true
        heartbeat:
          enabled: true
```

主启动类:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Payment8006 {
    public static void main(String[] args) {
        SpringApplication.run(Payment8006.class);
    }
}
```

可以在客户端中查看刚刚注册的微服务

![image-20220516110313622](img.assets\image-20220516110313622.png)



### 3.3 声明服务消费者

新建Module消费服务cloud-consumer-consul-order90

yml:

```yml
server:
  port: 90

spring:
  application:
    name: cloud-consumner-order
  cloud:
    consul:
      host: 121.41.112.246
      port: 8500
      discovery:
        hostname: 127.0.0.1
        service-name: ${spring.application.name}
        prefer-ip-address: true
        heartbeat:
          enabled: true
```

配置restTemplate模板

```java
@Configuration
public class RestTemplate {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

在客户端查看:

![image-20220516111952934](img.assets\image-20220516111952934.png)



## 4.三个注册中心异同点

| 组件名    | 语言 | CAP  | 服务监控检查 | Spring Cloud集成 | 对外暴露接口 |
| --------- | ---- | ---- | ------------ | ---------------- | ------------ |
| Eureka    | Java | AP   | 可配支持     | 已集成           | 可配支持     |
| Consul    | Go   | CP   | 支持         | 已集成           | 支持         |
| Zookeeper | Java | CP   | 支持         | 已集成           | 支持         |

CAP：

- C：Consistency (强一致性)
- A：Availability (可用性)
- P：Partition tolerance （分区容错性)

![img](img.assets\b41e0791c9652955dd3a2bc9d2d60983.png)

**最多只能同时较好的满足两个**。

CAP理论的核心是：**一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求**。

因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类:

- CA - 单点集群，满足—致性，可用性的系统，通常在可扩展性上不太强大。
- CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。
- AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。



# 六.服务调用

## 1.Ribbon

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡的工具**。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。在配置文件中列出**Load Balancer(简称LB)**后面所有的机器，Ribbon会自动的帮助你基于某种规则(如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

**负载均衡 + RestTemplate调用**,默认为轮询算法

**Ribbon在springCloud 2021.0.1 版本中替换为Spring Cloud LoadBalacer。**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

### 1.1 LB负载均衡

将用户的请求平摊的分配到多个服务上，从而达到系统的HA (高可用)。

常见的负载均衡有软件Nginx，LVS，硬件F5等。

**Ribbon本地负载均衡客户端   VS   Nginx服务端负载均衡区别**

Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是**由服务端实现**的。
Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后**缓存到JVM本地**，从而在本地实现RPC远程服务调用技术。

#### 1.1.1 集中式LB

即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx)，由该设施负责把访问请求通过某种策略转发至服务的提供方;

#### 1.1.2 进程内LB

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

**Ribbon属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。



### 1.2 Ribbon的负载均衡和Rest调用

Ribbon其实就是一个软负载均衡的客户端组件，它可以和其他所需请求的客户端结合使用，和Eureka结合只是其中的一个实例。

Ribbon在工作时分成两步：

- 第一步先选择EurekaServer ,它优先选择在同一个区域内负载较少的server。
- 第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。

Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。

![img](img.assets\145b915e56a85383b3ad40f0bb2256e0.png)



### 1.3 Ribbon默认自带的负载规则

lRule：根据特定算法中从服务列表中选取一个要访问的服务

```java
public interface IRule{
    public Server choose(Object key);  
    public void setLoadBalancer(ILoadBalancer lb);   
    public ILoadBalancer getLoadBalancer();    
}
```

![img](img.assets\87243c00c0aaea211819c0d8fc97e445.png)

- RoundRobinRule 轮询
- RandomRule 随机
- RetryRule 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试。获取可用的服务
- WeightedResponseTimeRule 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- BestAvailableRule 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- AvailabilityFilteringRule 先过滤掉故障实例，再选择并发较小的实例
- ZoneAvoidanceRule **默认规则**,复合判断server所在区域的性能和server的可用性选择服务器



### 1.4 Ribbon负载规则替换

这个自定义配置类不能放在 `@ComponentScan`所扫描的当前包下以及子包下，

否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。

**(也就是说不要将Ribbon配置类与主启动类同包**）



新建package - com.xh.myIRule

```java
@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule(){
        return new RandomRule();
    }
}
```

主启动类添加**@RibbonClient**

```java
@SpringBootApplication
@EnableEurekaClient
//name 要负载均衡的提供者服务名称，configuration 自定义的负载均衡算法
//name 最好和RestTemplate中URL保持一致
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MySelfRule.class)
public class Order90
{
    public static void main( String[] args ){
        SpringApplication.run(Order90.class, args);
    }
}
```



### 1.5 Ribbon默认负载轮询算法原理

**默认负载轮训算法: rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标，每次服务重启动后rest接口计数从1开始**。

```
List<Servicelnstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
```

如:

- List [0] instances = 127.0.0.1:8002
- List [1] instances = 127.0.0.1:8001

8001+ 8002组合成为集群，它们共计2台机器，集群总数为2，按照轮询算法原理：

- 当总请求数为1时:1%2=1对应下标位置为1，则获得服务地址为127.0.0.1:8001
- 当总请求数位2时:2%2=О对应下标位置为0，则获得服务地址为127.0.0.1:8002
- 当总请求数位3时:3%2=1对应下标位置为1，则获得服务地址为127.0.0.1:8001
- 当总请求数位4时:4%2=О对应下标位置为0，则获得服务地址为127.0.0.1:8002
- 如此类推…



## 2.LoadBalancer

Spring Cloud LoadBalancer是Spring Cloud官方自己提供的客户端负载均衡器, 用来替代Ribbon。

```xml
<!--Spring Cloud 2021.0.1版本 loadbalancer 3.1.1-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    <version>3.1.1</version>
</dependency>
```

在Eureka**3.0**版本中

`spring-cloud-starter-netflix-eureka-client`自带了`spring-cloud-starter-loadbalacer`引用。

在springCloud2021.0.1版本中consul和zookeeper的依赖中都自带了`spring-cloud-starter-loadbalacer`依赖

![image-20220516131227101](img.assets\image-20220516131227101.png)

![image-20220516145659631](img.assets\image-20220516145659631.png)

Spring官方提供了两种负载均衡的客户端：
**RestTemplate**
RestTemplate是Spring提供的用于访问Rest服务的客户端，RestTemplate提供了多种便捷访问远程Http服务的方法，能够大大提高客户端的编写效率。默认情况下，RestTemplate默认依赖jdk的HTTP连接工具。
**WebClient**
WebClient是从Spring WebFlux 5.0版本开始提供的一个非阻塞的基于响应式编程的进行Http请求的客户端工具。它的响应式编程的基于Reactor的。WebClient中提供了标准Http请求方式对应的get、post、put、delete等方法，可以用来发起相应的请求



### 2.1 基本使用

```java
@Configuration
public class MyRestTemplate {
    @Bean
    @LoadBalanced
    //添加负载均衡注解，当调用服务时，应用程序和通过${spring.application.name} 去服务注册中心寻找对应服务和接口
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```



### 2.2 切换负载均衡策略

要实现负载均衡策略的切换，需要做如下两个步骤：
1、添加负载均衡配置类，@Configuration注解可加可不加

```java
@Configuration
public class LoadBalanceConfig {
    @Bean
    //切换为随机负载均衡（官方写法）
    public ReactorLoadBalancer<ServiceInstance> reactorLoadBalancer(Environment environment,
                                                                    LoadBalancerClientFactory loadBalancerClientFactory){
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        return new RandomLoadBalancer(loadBalancerClientFactory
                .getLazyProvider(name, ServiceInstanceListSupplier.class), name);
    }
}
```

2.在我们**注入restTemplate的地方或者主启动类上**使用注解@LoadBalancerClients 或者@LoadBalancerClient注解进行配置

```java
@Configuration
//在这里配置我们自定义的LoadBalancer策略 如果想自己扩展算法 需要实现ReactorServiceInstanceLoadBalancer接口
//@LoadBalancerClients(defaultConfiguration = {name = "CLOUD-PAYMENT-SERVICE", configuration = LoadBalanceConfig.class})
//注意这里的name属性需要和Rest中的服务提供者名字一致 此时Rest中是大写
@LoadBalancerClient(name = "CLOUD-PAYMENT-SERVICE",configuration = LoadBalanceConfig.class)
public class MyRestTemplate {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```



## 3.OpenFeign

Feign是一个声明式WebService客户端。使用Feign能让编写Web Service客户端更加简单。它的使用方法是**定义一个服务接口然后在上面添加注解**。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

[Spring Cloud OpenFeign 官网](https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/#spring-cloud-openfeign)

### 3.1 Feign的作用

Feign旨在使编写Java Http客户端变得更容易。

前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，**往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。**所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，**我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，**即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。



**Feign集成了Ribbon**,（高版本没有集成）

利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，**通过feign只需要定义服务绑定接口且以声明式的方法**，优雅而简单的实现了服务调用。



### 3.2 Feign和OpenFeign区别

**Feign**是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是:使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

**OpenFeign**是Spring Cloud在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的**@Feignclient**可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。

**客户端通过动态代理去调用服务端**

```xml
<!-- springCloud 2021.0.1版本 --- openfeign 3.1.1-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>3.1.1</version>
</dependency>
```



### 3.3 OpenFeign服务调用

接口+注解：微服务调用接口 + @FeignClient

**controller接收到请求  ----  PaymentFeignService实现动态代理 -----  将调用指定微服务的指定接口**

**fegin使用在消费者端**

新建cloud-consumer-feign-order90

pom

```xml
	 <!-- springCloud 2021.0.1版本 - openfeign 3.1.1-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

      <!-- 2021.0.1 springCloud版本 eureka-client -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           <version>3.1.1</version>
       </dependency>
```

yml

```yml
server:
  port: 90

eureka:
  client:
    #客户端不用注册进服务中心
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
```

启动类：**@EnableFeignClients注解**

```java
@SpringBootApplication
//开启服务发现
@EnableDiscoveryClient
//开启feign服务调用
@EnableFeignClients
public class OpenFeignConsumer {
    public static void main(String[] args) {
        SpringApplication.run(OpenFeignConsumer.class);
    }
}

```

**业务逻辑接口+@FeignClient配置调用provider服务**

```java
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
//指定微服务的名称
public interface PaymentFeignService {
    //动态代理请求的路径
    @GetMapping("payment/getPayment/{id}")
    ResultJSON<Payment> getPaymentById(@PathVariable("id") Long id);
}
```

controller:

```java
@RestController
@RequestMapping("/consumer")
public class ConsumerController {
    @Resource
    private PaymentFeignService paymentFeignService;
    @GetMapping("/payment/get/{id}")
    public ResultJSON<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getPaymentById(id);
    }
}
```



### 3.4 OpenFeign超时控制

**OpenFeign默认等待1秒钟，超过后报错  ---3.0版本以后默认为1分钟**

使用配置属性来配置所有 @FeignClient，则可以使用默认 feign 名称创建配置属性。

yml

```yml
#openFegin 3.0.1 版本设置
feign:
  client:
    config:
      default:
        #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
        connectTimeout: 5000
        #指的是建立连接后从服务器读取到可用资源所用的时间
        readTimeout: 5000
```

官网给出的相关配置:

```yml
feign:
  client:
    config:
      #给指定名字的fegin进行配置，若要全局配置，则省略
      feignName:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
        errorDecoder: com.example.SimpleErrorDecoder
        retryer: com.example.SimpleRetryer
        requestInterceptors:
          - com.example.FooRequestInterceptor
          - com.example.BarRequestInterceptor
        decode404: false
        encoder: com.example.SimpleEncoder
        decoder: com.example.SimpleDecoder
        contract: com.example.SimpleContract
```



### 3.5 OpenFeign日志增强

**日志打印功能**

Feign提供了日志打印功能，我们可以通过配置来调整日恙级别，从而了解Feign 中 Http请求的细节。

**日志级别**

- NONE：默认的，不显示任何日志;
- BASIC：仅记录请求方法、URL、响应状态码及执行时间;
- HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息;
- FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

Feign.logging 只响应 DEBUG 级别

```yml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    # 可以使用通配符
    com.xh.springcloud.service.PaymentFeignService: debug
```

#### 3.5.1 使用配置类

```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

#### 3.5.2  在yml中直接配置

```yml
#openFegin 3.0.1 版本设置
feign:
  client:
    config:
      default:
        #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
        connectTimeout: 5000
        #指的是建立连接后从服务器读取到可用资源所用的时间
        readTimeout: 5000
        #设置日志的级别
        loggerLevel: FULL

logging:
  level:
    com.xh.springcloud.service.PaymentFeignService: debug
```

### 3.6 OpenFeign多参数问题

SpringCloud Feign报错：Method has too many Body parameters 原因:

使用Feign时，**如果发送的是get请求，那么需要在请求参数前加上@RequestParam注解修饰，**Controller里面可以不加该注解修饰 ，@RequestParam可以修饰多个，@RequestParam是用来修饰参数，不能用来修饰整个对象。

**注意：@RequestParam `Content-Type` 为 `application/x-www-form-urlencoded` 而这种是默认的**



#### 3.6.1 POST方式

错误写法

```java
public int save(@RequestBody final User u, @RequestBody final School s);
```

feign中你可以有多个@RequestParam，**但只能有不超过一个@RequestBody**，@RequestBody用来修饰对象，但是既有@RequestBody也有@RequestParam，那么参数就要放在请求的url中，@RequestBody修饰的就要放在提交对象中。

**注意 用来处理@RequestBody `Content-Type` 为 `application/json`、`application/xml`编码的内容**



# 七.服务降级

## 1.Hystrix

Hystrix ([gitHub地址](https://github.com/Netflix/Hystrix))是一个延迟和容错库，旨在隔离对远程系统、服务和第三方库的访问点，停止级联故障并在故障不可避免的复杂分布式系统中实现弹性。

在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性**。

"断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（**类似熔断保险丝**)，**向调用方返回一个符合预期的、可处理的备选响应（FallBack)**，而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

**Hystrix已经停止更新进入维护,官网推荐使用[resilience4j](https://github.com/resilience4j/resilience4j)，但是国内使用Sentinel较多**

```xml
        <!-- hystrix -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.10.RELEASE</version>
        </dependency>
```

**Hystrix处理请求使用tomcat中的线程池**



### 1.1 服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的**“扇出”**。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.

对于高流量的应用来说，单一的后避依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。



### 1.2 Hystrix作用

​    1.延迟和容错,停止级联故障。回退和优雅降级。失败快，恢复快。 使用断路器进行线程和信号量隔离。 

​	2.实时操作,实时监控和配置更改。观察服务和财产变化在整个车队中传播时立即生效。 在几秒钟内收到警报、做出决定、影响变化并看到结果。

​	3.并发,并行执行。并发感知请求缓存。通过请求折叠自动批处理。



### 1.3 Hystrix的服务降级-熔断-限流

#### 1.3.1 服务降级 (fallback)

服务降级是指当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心业务正常运作或高效运作。说白了，就是尽可能的把系统资源让给优先级高的服务。

服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示**(Hystrix的服务降级一般用在客户端)**

**哪些情况会出发降级**

- 程序运行导常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级



#### 1.3.2 服务熔断（break）

服务达到最大访问量了，直接拒绝请求(**类比保险丝**),限制后续的服务访问,然后调用服务降级的方法并返回友好提示。然后根据一定的规则慢慢试着恢复服务。

服务降级–>进而熔断–>恢复调用链路



#### 1.3.3 服务限流（flowlimit）

 限流的目的是通过对并发访问/请求进行限速或者一个时间窗口内的的请求进行限速来保护系统，一旦达到限制速率则可以拒绝服务（定向到错误页或告知资源没有了）、排队或等待（比如秒杀、评论、下单）、降级（返回兜底数据或默认数据，如商品详情页库存默认有货）。



### 1.4 服务降级

#### 1.4.1 Hystrix支付微服务构建

新建cloud-provider-hygtrix-payment8001

```xml
        <!-- hystrix -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.10.RELEASE</version>
        </dependency>
        
        <!-- 2021.0.1 springCloud版本 eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>3.1.1</version>
        </dependency>
```

yml:使用eureka单机版 

```yml
server:
  port: 8001

spring:
  application:
    name: cloud-provider-hystrix-payment

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```



#### 1.4.2 JMeter高并发压测测试

添加线程组

![image-20220517214441896](img.assets\image-20220517214441896.png)

设置线程组：使其可以发送200*100个请求，保存

![image-20220517214630761](img.assets\image-20220517214630761.png)

发送HTTP请求

![image-20220517214929582](img.assets\image-20220517214929582.png)

![image-20220517215309270](img.assets\image-20220517215309270.png)



**tomcat的默认的工作线程数被打满了，没有多余的线程来分解压力和处理。导致服务被拖慢**



#### 1.4.3 订单微服务调用支付服务

新建 cloud-consumer-feign-hystrix-order90

pom

```xml
		 <!-- hystrix -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.10.RELEASE</version>
        </dependency>

        <!-- springCloud 2021.0.1版本 - openfeign 3.1.1-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <!-- 2021.0.1 springCloud版本 eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>3.1.1</version>
        </dependency>
```

yml

```yml
server:
  port: 90

eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/

#openFegin 3.0.1 版本设置
feign:
  client:
    config:
      default:
        #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
        connectTimeout: 5000
        #指的是建立连接后从服务器读取到可用资源所用的时间
        readTimeout: 5000
```

feign

```java
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT")
public interface FeignService {

    @GetMapping("/payment/hystrix/ok/{id}")
    String  paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```



2W个线程压8001

消费者80被拖慢

原因：8001同一层次的其它接口服务被困死，因为tomcat线程池里面的工作线程已经被挤占完毕。

正因为有上述故障或不佳表现才有我们的降级/容错/限流等技术诞生。



#### 1.4.4 降级容错解决的维度要求

超时导致服务器变慢(转圈) - 超时不再等待

出错(宕机或程序运行出错) - 出错要有兜底

解决：

- 对方服务(8001)超时了，调用者(80)不能一直卡死等待，必须有服务降级。
- 对方服务(8001)down机了，调用者(80)不能一直卡死等待，必须有服务降级。
- 对方服务(8001)OK，调用者(80)自己出故障或有自我要求(自己的等待时间小于服务提供者)，自己处理降级。



#### 1.4.5 Hystrix之服务降级provider侧fallback

##### 1.4.5.1 @HystrixCommand注解

业务类启用 - @HystrixCommand报异常后如何处理

—旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法

**@HystrixProperty注解中的相关属性配置可以再HystrixPropertiesManager类中找到**

**@HystrixCommand中属性的默认值可以在HystrixCommandProperties类中查询**

配置详情信息[Hystri配置参数说明](https://blog.csdn.net/tongtong_use/article/details/78611225?ops_request_misc=%7B%22request%5Fid%22%3A%22165288851716781667867990%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=165288851716781667867990&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-2-78611225-null-null.142^v10^pc_search_result_control_group,157^v4^new_style&utm_term=sleepWindowInMilliseconds&spm=1018.2226.3001.4187)

| execution.isolation.thread.timeoutInMilliseconds | 超时时间 | 默认值：1000,在THREAD模式下，达到超时时间，可以中断在SEMAPHORE模式下，会等待执行完成后，再去判断是否超时设置标准：有retry，99meantime+avg meantime没有retry，99.5meantime |
| :----------------------------------------------- | -------- | ------------------------------------------------------------ |

```java
	@Override
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler"/*指定善后的方法*/,commandProperties = {
            //指定该方法的用时为3秒钟，若超过三秒或者出现异常，则直接调用善后的方法
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")
    })
    public String paymentTimeout(Integer id) {
        // int age = 10/0;
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程名称:" + Thread.currentThread().getName() + "   paymentInfoOK,id: " + id +"   睡眠3秒";
    }

    public String paymentInfo_TimeOutHandler(Integer id){
        //Hystrix会自动将方法的参数传递,所以在写善后方法时主要要带上方法的参数
        return "线程名称:" + Thread.currentThread().getName() +"系统繁忙或者出现异常错误,请稍后再试";
    }
```

##### 1.4.5.2 @EnableHystrix 注解

**主启动类激活**

添加新注解@EnableHystrix

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
//开启Hystrix注解
public class Payment8001 {
    public static void main(String[] args) {
        SpringApplication.run(Payment8001.class);
    }
}
```

##### 1.4.5.3 测试结果

```
线程名称:HystrixTimer-1系统繁忙或者出现异常错误,请稍后再试 ----超时
线程名称:hystrix-PaymentServiceImpl-1系统繁忙或者出现异常错误,请稍后再试 ----报错
```

**由此可见，在使用Hystrix做服务降级时，将启动Hystrix的线程，而不是在tomcat的线程中进行**



#### 1.4.6 Hystrix之服务降级consumer侧fallback

Hystrix的服务降级一般用在客户端

**因为客户端是由openFeign做远程RPC调用，所以需要在yml文件中开启hystrix配置**

```yml
#openFegin 3.0.1 版本设置
feign:
  client:
    config:
      default:
        #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
        connectTimeout: 5000
        #指的是建立连接后从服务器读取到可用资源所用的时间
        readTimeout: 5000
  #开启hystrix配置
  circuitbreaker:
    enabled: true
```

```java
	@GetMapping("/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
    })
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        //int age = 10/0;
        return feignService.paymentInfo_TimeOut(id);
    }
    //善后方法
    public String paymentTimeOutFallbackMethod(Integer id){
        return "我是消费者90,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }
```



#### 1.4.7 Hystrix之全局服务降级DefaultProperties

除了个别重要核心业务有专属，其它普通的可以通过**@DefaultProperties(defaultFallback = “”)**统一跳转到统一处理结果页面

通用的和独享的各自分开，避免了代码膨胀，合理减少了代码量

```java
//使用默认的降级方法
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1000")
})
public class PaymentController {
    
    @HystrixCommand
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        //int age = 10/0;
        return feignService.paymentInfo_TimeOut(id);
    }
    
    //特定的善后方法，必须带上形参
    public String paymentTimeOutFallbackMethod(Integer id){
        return "我是消费者90,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }

    //全局fallback方法,若没有指定，则使用默认的方法来实现降级
    //全局默认的降级方法中不能有形参
    public String payment_Global_FallbackMethod(){
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }
}
```



#### 1.4.8 Hystrix之通配服务降级FeignFallback

**服务降级，客户端去调用服务端，碰上服务端宕机或关闭**

本次案例服务降级处理是在客户端90实现完成的，与服务端8001没有关系，只需要为Feign客户端定义的接口添加一个服务降级处理的实现类即可实现解耦

1.**在FeginService接口的@FeignClient注解中添加fallback属性指定降级类**

fallback属性：用于指定FeignClient回退类(容错处理类), **回退类必须实现@FeignClient标记的接口,需配置为一个有效的spring bean**

- **当调用远程接口失败或者超时（不能控制客户端的超时，异常）**, 会调用对应的回退类的逻辑
- 使用hystrix做降级, 需要配置: feign.hystrix.enabled=true（**无效，出现500状态码**）
- 使用circuitbreaker做降级, 需要配置: **feign.circuitbreaker.enabled=true（成功**）

```java
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",
        //指定服务降级的类FeignServiceImpl
        fallback = FeignServiceImpl.class)
public interface FeignService {

    @GetMapping("/payment/hystrix/ok/{id}")
    String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```

**2.该降级类要去实现FeginService接口**，并且配置为一个有效的spring bean

```java
@Component
//一定要加上@Component注解将该类注入spring容器中，否则fallback无法找到该类
public class FeignServiceImpl implements FeignService {
    @Override
    public String paymentInfo_OK(Integer id) {
        //在这里进行服务降级
        return "-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_TimeOut ,o(╥﹏╥)o";
    }
}
```

**注意：**

**1.通配服务降级只能处理服务端，无法处理客户端的错误。全局服务降级都可以处理**

**2.全局fallback的优先级 > 通配服务降级**

**3.若同时使用通配服务降级和全局服务降级，服务器宕机时候，调用通配服务降级中的方法**



### 1.5 服务熔断

**熔断机制概述**

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。**当检测到该节点微服务调用响应正常后，恢复调用链路**。

在Spring Cloud框架里，熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，**当失败的调用到一定阈值，缺省是10秒内20次请求中10次调用失败**，就会启动熔断机制。**熔断机制的注解是@HystrixCommand。**



**注意：**

**1.服务调用失败会触发降级，而降级会调用fallback方法**

**2.但是无论如何降级的流程一定会先调用正常的方法再调用fallback方法**

**3.假如单位时间内调用失败的次数过多，也就是降级的次数过多，则会触发熔断**

**4.熔断之后就会跳过正常方法直接调用fallback方法**

**5.所谓”熔断后服务不可用“就是因为跳过的正常方法直接执行fallback方法**



![img](img.assets\84d60234d01c4b7e9cae515066eb711b.png)

#### 1.5.1 熔断类型

- 熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间)，当打开时长达到所设时钟则进入半熔断状态。
- 熔断关闭：熔断关闭不会对服务进行熔断。
- 熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断。



#### 1.5.2 Hystrix之服务熔断案例

修改cloud-provider-hystrix-payment8001

**@HystrixProperty注解中的相关属性配置可以再HystrixPropertiesManager类中找到**

**@HystrixCommand中属性的默认值可以在HystrixCommandProperties类中查询**

配置详情信息[Hystri配置参数说明](https://blog.csdn.net/tongtong_use/article/details/78611225?ops_request_misc=%7B%22request%5Fid%22%3A%22165288851716781667867990%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=165288851716781667867990&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-2-78611225-null-null.142^v10^pc_search_result_control_group,157^v4^new_style&utm_term=sleepWindowInMilliseconds&spm=1018.2226.3001.4187)

| **参数**                                 | **描述**                                               | **默认值**                             |
| ---------------------------------------- | ------------------------------------------------------ | -------------------------------------- |
| circuitBreaker.enabled                   | 确定断路器是否用于跟踪运行状况和短路请求（如果跳闸）。 | 默认值为true                           |
| circuitBreaker.requestVolumeThreshold    | 熔断触发的最小个数/10s                                 | 默认值：20                             |
| circuitBreaker.sleepWindowInMilliseconds | 熔断多少秒后去尝试请求                                 | 默认值：5000                           |
| circuitBreaker.errorThresholdPercentage  | 失败率达到多少百分比后熔断                             | 默认值：50  主要根据依赖重要性进行调整 |

**circuitBreaker.requestVolumeThreshold**
默认值20.意思是至少有20个请求才进行errorThresholdPercentage错误百分比计算。比如一段时间（10s）内有19个请求全部失败了。错误百分比是100%，但熔断器不会打开，因为requestVolumeThreshold的值是20. 这个参数非常重要，

**circuitBreaker.sleepWindowInMilliseconds**
//过多长时间，熔断器再次检测是否开启，默认为5000，即5s钟,**在这5s内，该服务直接调用fallback方法，不会调用正常方法**，**在5s结束后，允许一次请求，若请求成功，则断开熔断器，若请求失败，则继续开启熔断器，重复以上过程。**

**circuitBreaker.errorThresholdPercentage**
//设定错误百分比，默认值50%，例如一段时间（10s）内有100个请求，其中有55个超时或者异常返回了，那么这段时间内的错误百分比是55%，大于了默认值50%，这种情况下触发熔断器打开。

**按照以上配置的熔断器如下：**
每当在10秒20个请求中，有50%失败时，熔断器就会打开，此时再调用此服务，将会直接返回失败，不再调远程服务。直到5s钟之后，重新检测该触发条件，判断是否把熔断器关闭，或者继续打开

![image-20210430192847690](img.assets\63b402640acc0455b097e9a6713843a4.png)

```java
//=====服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
        	// 请求次数至少要有10次请求才会计算错误百分比
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
        	// 时间窗口期 过多长时间，熔断器再次检测是否开启
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),
        	// 失败率达到多少后跳闸
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if(id < 0) {
            throw new RuntimeException("******id 不能负数");
        }
        String serialNumber = IdUtil.simpleUUID();

        return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
    }
    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
    }
```

#### 1.5.3 断路器开启或者关闭的条件

- 到达以下阀值，断路器将会开启：
  - 当满足一定的阀值的时候（默认10秒内超过20个请求次数)
  - 当失败率达到一定的时候（默认10秒内超过50%的请求失败)
- 当开启的时候，所有请求都不会进行转发
- 一段时间之后（默认是5秒)，这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。

**断路器打开之后**

1：再有请求调用的时候，**将不会调用主逻辑，而是直接调用降级fallback**。通过断路器，实现了自动地发现错误并将降级逻辑切换为主逻辑，减少响应延迟的效果。

2：原来的主逻辑要如何恢复

对于这一问题，hystrix实现了自动恢复功能。

当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑，当休眠时间窗到期，断路器将进入半开状态，**允许一次请求到原来的主逻辑上**，如果此次请求正常返回，那么断路器将继续闭合，主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时。



### 1.6 服务限流

在高并发访问下，由于系统资源有限，必须对访问量进行控制。

 Hystrix默认使用线程隔离模式，可以通过**线程数+队列大小**进行限流，通过以下参数进行控制：

**限流后的请求直接降级处理**

```java
@HystrixCommand(fallbackMethod = "fallback",
                threadPoolKey = "goods",
                //threadPoolKey 是线程池唯一标识, hystrix 会使用该标识来计数，看线程占用是否超过了，超过了就会直接降级该次调用；
                threadPoolProperties = {
                    // #并发执行的最大线程数，默认10
                    //最多同时处理的请求数
                    @HystrixProperty(name = "coreSize", value = "2"),
                    //#BlockingQueue的最大队列数，默认值-1
                    //请求的线程数
                	@HystrixProperty(name = "maxQueueSize", value = "1")
                })
```

yml配置：

```yml
hystrix:
  threadpool:
    default:
      coreSize: 200 #并发执行的最大线程数，默认10
      maxQueueSize: 1000 #BlockingQueue的最大队列数，默认值-1
      queueSizeRejectionThreshold: 800 #即使maxQueueSize没有达到，达到queueSizeRejectionThreshold该值后，请求也会被拒绝，默认值5
```



### 1.7 Hysrtix 的所有配置

**以下配置都可以在yml中做全局配置**

```yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 5000 # 设置hystrix的超时时间为5000ms
```



```java
@HystrixCommand(fallbackMethod = "fallbackMethod", 
                groupKey = "strGroupCommand", 
                commandKey = "strCommand", 
                threadPoolKey = "strThreadPool",
                
           commandProperties = {
           // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
           @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
           // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
           @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
           // 配置命令执行的超时时间
           @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "1000"),
           // 是否启用超时时间
           @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
           // 执行超时的时候是否中断
           @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),         
           // 执行被取消的时候是否中断
           @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
               
           // 允许回调方法执行的最大并发数
           @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
           // 服务降级是否启用，是否执行回调函数
           @HystrixProperty(name = "fallback.enabled", value = "true"),
               
           // 是否启用断路器
           @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
           // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
           @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
               
           // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过 circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50, 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
           @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
           // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，如果成功就设置为 "关闭" 状态。
           @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
           // 断路器强制打开
           @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
           // 断路器强制关闭
           @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
               
           // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
           @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),          
           // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
          // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
          @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
          // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。
          @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
          // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
          @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
          // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
          @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
          // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
          // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
          // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
          @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
                    
          // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。
          @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
               
          // 是否开启请求缓存
          @HystrixProperty(name = "requestCache.enabled", value = "true"),
          // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
          @HystrixProperty(name = "requestLog.enabled", value = "true"),
          },
                
       threadPoolProperties = {
           // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
           @HystrixProperty(name = "coreSize", value = "10"),
           // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，否则将使用 LinkedBlockingQueue 实现的队列。
           @HystrixProperty(name = "maxQueueSize", value = "-1"),
           // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
           // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
           @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
            }
)
```



### 1.8 Hystrix工作流程总结



![img](img.assets\hystrix-command-flow-chart.png)

1. 创建HystrixCommand （用在依赖的服务返回单个操作结果的时候）或HystrixObserableCommand（用在依赖的服务返回多个操作结果的时候）对象。
2. 命令执行。
3. 其中 HystrixCommand实现了下面前两种执行方式
   1. execute()：同步执行，从依赖的服务返回一个单一的结果对象或是在发生错误的时候抛出异常。
   2. queue()：异步执行，直接返回一个Future对象，其中包含了服务执行结束时要返回的单一结果对象。
4. 而 HystrixObservableCommand实现了后两种执行方式：
   1. obseve()：返回Observable对象，它代表了操作的多个统
      果，它是一个Hot Observable （不论“事件源”是否有“订阅者”，都会在创建后对事件进行发布，所以对于Hot Observable的每一个“订阅者”都有可能是从“事件源”的中途开始的，并可能只是看到了整个操作的局部过程）。
   2.  toObservable()：同样会返回Observable对象，也代表了操作的多个结果，但它返回的是一个Cold Observable（没有“订间者”的时候并不会发布事件，而是进行等待，直到有“订阅者"之后才发布事件，所以对于Cold Observable 的订阅者，它可以保证从一开始看到整个操作的全部过程）。
5. 若当前命令的请求缓存功能是被启用的，并且该命令缓存命中，那么缓存的结果会立即以Observable对象的形式返回。
6. 检查断路器是否为打开状态。如果断路器是打开的，那么Hystrix不会执行命令，而是转接到fallback处理逻辑(第8步)；如果断路器是关闭的，检查是否有可用资源来执行命令(第5步)。
7. 线程池/请求队列信号量是否占满。如果命令依赖服务的专有线程地和请求队列，或者信号量（不使用线程的时候）已经被占满，那么Hystrix也不会执行命令，而是转接到fallback处理理辑(第8步) 。
8. Hystrix会根据我们编写的方法来决定采取什么样的方式去请求依赖服务。
   1. HystrixCommand.run()：返回一个单一的结果，或者抛出异常。
   2. HystrixObservableCommand.construct()：返回一个Observable对象来发射多个结果，或通过onError发送错误通知。
9. Hystix会将“成功”、“失败”、“拒绝”、“超时” 等信息报告给断路器，而断路器会维护一组计数器来统计这些数据。断路器会使用这些统计数据来决定是否要将断路器打开，来对某个依赖服务的请求进行"熔断/短路"。
10. 当命令执行失败的时候，Hystix会进入fallback尝试回退处理，我们通常也称波操作为“服务降级”。而能够引起服务降级处理的情况有下面几种：
    1. 第4步∶当前命令处于“熔断/短路”状态，断断器是打开的时候。
    2. 第5步∶当前命令的钱程池、请求队列或者信号量被占满的时候。
    3. 第6步∶HystrixObsevableCommand.construct()或HytrixCommand.run()抛出异常的时候。
11. 当Hystrix命令执行成功之后，它会将处理结果直接返回或是以Observable的形式返回。

**tips**：如果我们没有为命令实现降级逻辑或者在降级处理逻辑中抛出了异常，Hystrix依然会运回一个Obsevable对象，但是它不会发射任结果数惯，而是通过onError方法通知命令立即中断请求，并通过onError方法将引起命令失败的异常发送给调用者。



### 1.9 Hystrix图形化Dashboard搭建

**概述**

除了隔离依赖服务的调用以外，Hystrix还提供了准**实时的调用监控(Hystrix Dashboard)**，Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。

Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。Spring Cloud也提供了Hystrix Dashboard的整合，对监控内容转化成可视化界面。

```xml
<!--hystrix 可视化页面 -->
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-metrics-event-stream</artifactId>
    <version>1.5.18</version>
</dependency>
```

```xml
<!-- SpringCloud 整合hystrix 可视化页面 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

**整合依赖中包含springBoot和SpringCloud的启动依赖**

![image-20220519105455443](img.assets\image-20220519105455443.png)

#### 1.9.1 Dashboard搭建

新建cloud-consumer-hystrix-dashboard9001

pom：

```xml
        <!-- SpringCloud 整合hystrix 可视化页面 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.2.10.RELEASE</version>
        </dependency>

		<!-- 监控信息依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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

```

yml:

```yml
server:
  port: 9001
  
hystrix:
  dashboard:
    proxy-stream-allow-list: "localhost"
```

主启动类

```java
@SpringBootApplication
//开启Hystrix可视化界面注解
@EnableHystrixDashboard
public class DashBoard9001 {
    public static void main(String[] args) {
        SpringApplication.run(DashBoard9001.class);
    }
}
```

页面:

![image-20220519110100357](img.assets\image-20220519110100357.png)

#### 1.9.2 监控配置

修改cloud-provider-hystrix-payment8001

注意：新版本Hystrix需要在主启动类Payment8001或者yml中指定监控路径

**yml:推荐使用该方式指定**

```yml
management:
  endpoints:
    web:
      exposure:
        # "*" 监视所有
        include: hystrix.stream
```

主启动类Payment8001：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
//开启Hystrix注解
public class Payment8001 {
    public static void main(String[] args) {
        SpringApplication.run(Payment8001.class);
    }

    /**
     *此配置是为了服务监控而配置，与服务容错本身无关，springCloud升级后的坑
     *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
     *只要在自己的项目里配置上下面的servlet就可以了
     *否则，Unable to connect to Command Metric Stream 404
     */
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```



在Dashboard界面中输入要监控的服务：http://localhost:8001/hystrix.stream

![image-20220519111545060](img.assets\image-20220519111545060.png)

发送请求之后查看页面变化

![image-20220519111951532](img.assets\image-20220519111951532.png)

#### 1.9.3 页面参数说明

- 7色

![img](img.assets\6740b2a462751db0ce8f2813f740c5b5.png)

- 1圈

实心圆：共有两种含义。它通过颜色的变化代表了实例的健康程度，它的健康度从绿色<黄色<橙色<红色递减。

该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，**流量越大该实心圆就越大**。所以通过该实心圆的展示，就可以在大量的实例中快速的发现故障实例和高压力实例。

- 1线

曲线：用来记录2分钟内流量的相对变化，可以通过它来观察到流量的上升和下降趋势。

![img](img.assets\8a8c682ab027e313e4d9af9e4bd96206.png)

![img](img.assets\7fe0003d738028e6e20a3bf8f802cd2d.png)

# 八.服务网关

## 1.Gateway

![img](img.assets\54b61d819aa1630bc61732de340b55b4.png)

Gateway[Spring Cloud Gateway](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/)是在Spring生态系统之上构建的API网关服务，基于Spring 5，Spring Boot 2和Project Reactor等技术。

Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能，例如:熔断、限流、重试等。

SpringCloud Gateway作为Spring Cloud 生态系统中的网关，目标是替代Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，**SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty**。

Spring Cloud Gateway的目标提供统一的路由方式且**基于 Filter链**的方式提供了网关基本的功能，例如:安全，监控/指标，和限流。

```xml
<!-- springCloud 2021.0.1 版本使用Gateway3.1.1 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>3.1.1</version>
</dependency>
```

![image-20220519131725953](img.assets\image-20220519131725953.png)



**作用**

- 方向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控等



**微服务架构中网关的位置**

![img](img.assets\5877d4b9035ead9cd2d037609dceb442.png)

### 1.1 Gateway 特性

1. 基于Spring Framework 5，Project Reactor和Spring Boot 2.0进行构建；
2. 动态路由：能够匹配任何请求属性；
3. 可以对路由指定Predicate (断言)和Filter(过滤器)；
4. 集成Hystrix的断路器功能；
5. 集成Spring Cloud 服务发现功能；
6. 易于编写的Predicate (断言)和Filter (过滤器)；
7. 请求限流功能；
8. 支持路径重写。



### 1.2 Gateway与Zuul的区别

在SpringCloud Finchley正式版之前，Spring Cloud推荐的网关是Netflix提供的Zuul。

1. Zuul 1.x，是一个基于阻塞I/O的API Gateway。
2. Zuul 1.x基于Servlet 2.5使用阻塞架构它不支持任何长连接(如WebSocket)Zuul的设计模式和Nginx较像，每次I/О操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是Nginx用C++实现，Zuul用Java实现，而JVM本身会有第一次加载较慢的情况，使得Zuul的性能相对较差。
3. Zuul 2.x理念更先进，想基于Netty非阻塞和支持长连接，但SpringCloud目前还没有整合。Zuul .x的性能较Zuul 1.x有较大提升。在性能方面，根据官方提供的基准测试,Spring Cloud Gateway的RPS(每秒请求数)是Zuul的1.6倍。
4. Spring Cloud Gateway建立在Spring Framework 5、Project Reactor和Spring Boot2之上，使用非阻塞API。
5. Spring Cloud Gateway还支持WebSocket，并且与Spring紧密集成拥有更好的开发体验



**Zuul1.x模型**

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Serviet IO处理模型。

Servlet的生命周期？servlet由servlet container进行生命周期管理。

- container启动时构造servlet对象并调用servlet init()进行初始化；
- container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用service)；
- container关闭时调用servlet destory()销毁servlet。

![img](img.assets\b71ecbfb29c939615c988123a0704306.png)

上述模式的**缺点**：

Servlet是一个简单的网络IO模型，当请求进入Servlet container时，Servlet container就会为其绑定一个线程，在**并发不高的场景下**这种模型是适用的。但是一旦高并发(如使用Jmeter压测)，线程数量就会上涨，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间。在一些简单业务场景下，不希望为每个request分配一个线程，只需要1个或几个线程就能应对极大并发的请求，这种业务场景下servlet模型没有优势。

所以Zuul 1.X是基于servlet之上的一个阻塞式处理模型，即Spring实现了处理所有request请求的一个servlet (DispatcherServlet)并由该servlet阻塞式处理处理。所以SpringCloud Zuul无法摆脱servlet模型的弊端。



### 1.3 GateWay非阻塞异步模型

**SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty**。

传统的Web框架，比如说: Struts2，SpringMVC等都是基于Servlet APl与Servlet容器基础之上运行的。

但是在Servlet3.1之后有了异步非阻塞的支持。而**WebFlux是一个典型非阻塞异步的框架**，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程(Spring 5必须让你使用Java 8)。

Spring WebFlux是Spring 5.0 引入的新的响应式框架，区别于Spring MVC，它不需要依赖Servlet APl，它是完全异步非阻塞的，并且基于Reactor来实现响应式流规范。



### 1.4 Gateway工作流程

#### 1.4.1 **三大核心概念**

1. Route(路由) - 路由是构建网关的基本模块,它由ID,目标URI,**一系列的断言和过滤器组成**,如断言为true则匹配该路由；
2. Predicate(断言) - 参考的是Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数),如果请求与断言相匹配则进行路由；
3. Filter(过滤) - 指的是Spring框架中GatewayFilter的实例,使用过滤器,可以在请求被路由前或者之后对请求进行修改。

![img](img.assets\70da1eecc951a338588356ee2db3fa1f.png)

web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。

predicate就是我们的匹配条件；而fliter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了



#### 1.4.2 工作模式

![Spring Cloud Gateway Diagram](img.assets\spring_cloud_gateway_diagram.png)

客户端向Spring Cloud Gateway发出请求。然后在Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到GatewayWeb Handler。

Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前(“pre”)或之后(“post"）执行业务逻辑。

Filter在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

**核心逻辑**：路由转发 + 执行过滤器链。



### 1.5 Gateway9527搭建

新建Module - cloud-gateway-gateway9527

pom:**gateway使用WebFlux框架，不需要引入web依赖**

```xml
		<!-- springCloud 2021.0.1 版本使用Gateway3.1.1 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!-- 2021.0.1 springCloud版本 eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>3.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
            <!--  自己的公共接口地址   -->
            <groupId>com.xh</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0.0</version>
        </dependency>
```

yml:

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
#############################新增网关配置###########################
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          #uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/hystrix/ok/**     # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001           #匹配后提供服务的路由地址
          #uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**             # 断言，路径相匹配的进行路由
####################################################################

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

主启动类：

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class Gateway9527 {
    public static void main(String[] args) {
        SpringApplication.run(Gateway9527.class);
    }
}
```



测试：

添加网关前：http://localhost:8001/payment/hystrix/ok/1

添加网关后：http://localhost:9527/payment/hystrix/ok/1

两者访问成功，返回相同结果



### 1.6 Gateway配置路由的两种方式

#### 1.6.1 使用yml配置

**Path中的P要大写**

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
#############################新增网关配置###########################
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/hystrix/ok/**     # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001           #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**             # 断言，路径相匹配的进行路由
####################################################################

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```



#### 1.6.2  注入RouteLocator的Bean

官网案例：

```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```

通过9527网关访问到外网的百度新闻网址  http://news.baidu.com/guonei

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        //ID:path_route_xh
        //path:/guonei
        //uri: http://news.baidu.com
        //将http://localhost:9527/guonei 映射到http://news.baidu.com/guonei
        routes.route("path_route_xh",
                r -> r.path("/guonei").uri("http://news.baidu.com/guonei"));
        return routes.build();
    }
}
```

链式写法：

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        return routes.route("path_route_xh1",
                r -> r.path("/guonei").uri("http://news.baidu.com/guonei"))
                     .route("path_route_xh2", 
                r -> r.path("/guonei").uri("http://news.baidu.com/guonei"))
                .build();
    }
}

```



### 1.7 GateWay配置动态路由

默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上**微服务名为路径**创建**动态路由进行转发，从而实现动态路由的功能**（不写死一个地址）。

```yml
spring:
  application:
    name: cloud-gateway
#############################新增网关配置###########################
  cloud:
    gateway:
      #不写discovery也可以实现动态路由？
      discovery:
        locator:
          enabled: true # 开启注册中心默认动态创建路由的过程。利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-provider-hystrix-payment  #匹配后提供服务的路由地址
          #lb是指路由的一种通信协议，它实现了负载均衡通信功能
          predicates:
            - Path=/payment/hystrix/*/**      # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-provider-hystrix-payment   #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**             # 断言，路径相匹配的进行路由
####################################################################
```



### 1.8 路由的三种实现方式

路由方式有三种

 1）ws://host:port（websocket方式） (全双工通信，前端实时获取后端信息): uri: ws://localhost:9000)

2）http://host:port（HTTP方式）   uri: http://localhost:80/

3）lb://微服务名  uri: lb://CLOUD-PAYMENT-SERVICE  **使用微服务名必须配置负载均衡，否则无法调用服务**



### 1.9 GateWay常用的Predicate

[官方文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gateway-request-predicates-factories)

启动网关9527,在启动日志中我们可以看到一下字段：

![image-20220520145953421](img.assets\image-20220520145953421.png)



Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。

Spring Cloud Gateway包括许多内置的Route Predicate工厂。所有这些Predicate都与HTTP请求的不同属性匹配。多个RoutePredicate工厂可以进行组合。

Spring Cloud Gateway创建Route 对象时，使用RoutePredicateFactory 创建 Predicate对象，Predicate 对象可以赋值给Route。Spring Cloud Gateway包含许多内置的Route Predicate Factories。
所有这些谓词都匹配HTTP请求的不同属性。多种谓词工厂可以组合，并通过逻辑and。



#### 1.9.1 请求的时间设置

The **After** Route Predicate Factory             在请求的时间之后

The **Before** Route Predicate Factory            在请求的时间之前

The **Between** Route Predicate Factory           在请求的时间之间

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        # 这个时间后才能起效
        - After=2022-05-20T15:51:37.485+08:00[Asia/Shanghai]
       #- Before=2022-05-20T15:51:37.485+08:00[Asia/Shanghai]
       #- Between=2022-05-20T15:51:37.485+08:00[Asia/Shanghai], 2022-05-22T15:51:37.485+08:00[Asia/Shanghai]
```

**使用java8新特性获取带有时区的时间**

```java
public class T2
{
    public static void main(String[] args)
    {
        ZonedDateTime zbj = ZonedDateTime.now(); // 默认时区
        System.out.println(zbj);
       //2022-05-20T15:51:37.485+08:00[Asia/Shanghai]
    }
}
```



#### 1.9.2 Cookie设置

**The Cookie Route Predicate Factory**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

Cookie Route Predicate 有两个参数，**cookie  name和cookie value的正则表达式**。

路由规则会通过获取对应的cookie name 值和正则表达式去匹配，匹配上就执行路由



使用**curl命令**发送带有cookie的请求

```cmd
# 该命令相当于发get请求，且没带cookie
curl http://localhost:9527/payment/lb

# 带cookie的
curl http://localhost:9527/payment/lb --cookie "chocolate=chip"
```



#### 1.9.3 请求头设置

**The Header Route Predicate Factory**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

Header Route Predicate 有两个参数，**属性名称和属性值的正则表达式**。

```cmd
# 带指定请求头的参数的CURL命令
curl http://localhost:9527/payment/lb -H "X-Request-Id:123"
```



#### 1.9.4 主机配置

**The Host Route Predicate Factory**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```



#### 1.9.5  请求方式配置

The Path Route Predicate Factory

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```



#### 1.9.6 请求地址配置

The Path Route Predicate Factory

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

如果请求路径是，则此路由匹配，例如：/red/1 或 /red/blue 或 /blue/green。



#### 1.9.7 请求参数配置

The Query Route Predicate Factory

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree. #请求的参数名为red，请求的值为geee***
```



### 1.10 GateWay的Filter

[官方文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gatewayfilter-factories)

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。Spring Cloud Gateway内置了多种路由过滤器，他们都由GatewayFilter的工厂类来产生。

Spring Cloud Gateway的Filter:

- 生命周期：
  - pre
  - post
- 种类（具体看官方文档）：
  - GatewayFilter - 有31种
  - GlobalFilter - 有10种

比如：

The AddRequestHeader GatewayFilter Factory  请求头配置

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment} #过滤器工程会在匹配的请求头上加上一对请求头，名称为X-Request-Red=Blue-**
```

其余的请查询官网



#### 1.10.1 自定义全局GlobalFilter

类似于springMVC中的拦截器



两个主要接口介绍：

1. **GlobalFilter**
2. Ordered

能干什么：

1. 全局日志记录
2. 统一网关鉴权
3. …



GateWay9527项目添加MyLogGateWayFilter类：

```java
@Component
@Slf4j
public class MyLogGatewayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("***********come in MyLogGateWayFilter:  "+ LocalDateTime.now());
        String uName = exchange.getRequest().getQueryParams().getFirst("uName");
        if(uName == null) {
            log.info("*******用户名为null，非法用户，o(╥﹏╥)o");
            //设置返回数据 设置状态码信息
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            //返回信息
            return exchange.getResponse().setComplete();
        }
        //放行
        return chain.filter(exchange);
    }

    @Override
    //设置过滤的级别
    //也可以使用order注解来实现
    public int getOrder() {
        return 0;
    }
}

```



# 九.服务配置

## 1.Config

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以**一套集中式的、动态的配置管理**设施是必不可少的。

SpringCloud提供了ConfigServer来解决这个问题。

[官网](https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/)

![img](img.assets\d5462e3b8c3a063561f5f8fc7fde327e.png)

SpringCloud Config为微服务架构中的微服务提供**集中化的外部配置支持**，配置服务器为**各个不同微服务**应用的所有环境提供了一个**中心化的外部配置。**



**Config Server**

```xml
  <!-- springCloud 2021.0.1 版本使用 config-server3.1.1 版本-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
            <version>3.1.1</version>
        </dependency>
```

**Cilent**

```xml
<!-- springCloud 2021.0.1 版本使用 config  3.1.1 版本 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>3.1.1</version>
</dependency>
```



SpringCloud Config分为**服务端**和**客户端**两部分。

- 服务端也称为**分布式配置中心，它是一个独立的微服务应用**，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。
- 客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

**能干嘛**

- 集中管理配置文件
- 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
- 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
- 将配置信息以REST接口的形式暴露 - post/crul访问刷新即可…

**与GitHub整合配置**

由于SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件)，但最推荐的还是Git，而且使用的是http/https访问的形式。



### 1.1 Config配置总控中心搭建

#### 1.1.1 从github上修改

1.在GitHub上新建一个名为springcloud-config的新Repository。

2.获得刚新建的git地址 - `git@github.com:abc/springcloud-config.git`

3.本地硬盘目录上新建git仓库并clone。

- 工作目录为D:\SpringCloud2022
- `git clone git@github.com:abc/springcloud-config.git`

4.修改后上传合并 `git add .`，`git commit -m "sth"`,`git push origin main`



github文件:

​	**以下文件名称请严格遵循官网的配置:{application}-{profile}**

- config-dev.yml

```yml
config:
  info: "master branch,springcloud-config/config-dev.yml version=7"
```

- config-prod.yml

```yml
config:
  info: "master branch,springcloud-config/config-prod.yml version=1"
```

- config-test.yml

```yml
config:
  info: "master branch,springcloud-config/config-test.yml version=1" 
```



#### 1.1.2 新搭建

新建Module模块cloud-config-center-3344，它即为Cloud的配置中心模块CloudConfig Center

pom:

```xml
		<!-- springCloud 2021.0.1 版本使用config3.1.1 版本-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <!-- 2021.0.1 springCloud版本 eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>3.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
```

yml:

```yml
server:
  port: 3344
  
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          #跳过SSL认证
          skip-ssl-validation: true
          #连接github仓库
          uri: https://github.com/XieHao1/springcloud-config.git
          # 搜索目录
          search-paths:
            - springcloud-config
      # 读取分支
      label: main
      
eureka:
  client:
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

主启动类添加 **@EnableConfigServer**

```java
@SpringBootApplication
@EnableEurekaClient
//开启服务配置config注解
@EnableConfigServer
public class Config3344 {
    public static void main(String[] args) {
        SpringApplication.run(Config3344.class);
    }
}
```

windows下修改hosts文件，增加映射

```txt
127.0.0.1 config-3344.com
```

 

测试通过Config微服务是否可以从GitHub上获取配置内容

http://config-3344.com:3344/main/config-dev.yml

```
config:
  info: master branch,springcloud-config/config-dev.yml version=7
```



### 1.2 配置读取规则

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

**1./{label}/{application}-{profile}.yml（推荐）**

- main分支
  - http://config-3344.com:3344/main/config-dev.yml
  - http://config-3344.com:3344/main/config-test.yml
  - http://config-3344.com:3344/main/config-prod.yml
- dev分支
  - http://config-3344.com:3344/dev/config-dev.yml
  - http://config-3344.com:3344/dev/config-test.yml
  - http://config-3344.com:3344/dev/config-prod.yml



2./{application}-{profile}.yml

- http://config-3344.com:3344/config-dev.yml
- http://config-3344.com:3344/config-test.yml
- http://config-3344.com:3344/config-prod.yml
- http://config-3344.com:3344/config-xxxx.yml(不存在的配置)



3./{application}/{profile}[/{label}]

- http://config-3344.com:3344/config/test/main
- http://config-3344.com:3344/config/test/dev
- http://config-3344.com:3344/config/dev/main



重要配置细节总结

- /{name}-{profiles}.yml
- /{label}-{name}-{profiles}.yml
- label：分支(branch)
- name：服务名
- profiles：环境(dev/test/prod)



### 1.3 Config客户端配置与测试

**新建cloud-config-client-3355**

pom:

```xml
        <!-- springCloud 2021.0.1 版本使用 bootstrap 3.1.1 版本 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
            <version>3.1.1</version>
        </dependency>

		<!-- springCloud 2021.0.1 版本使用 config  3.1.1 版本 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>3.1.1</version>
        </dependency>

        <!-- 2021.0.1 springCloud版本 eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>3.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
```



#### 1.3.1 **bootstrap.yml**

applicaiton.yml是用户级的资源配置项

bootstrap.yml**是系统级的，优先级更加高**

pring Cloud会创建一个Bootstrap Context，作为Spring应用的Application Context的父上下文。

初始化的时候，BootstrapContext负责**从外部源加载配置属性并解析配置**。这两个上下文共享一个从外部获取的Environment。

Bootstrap属性有高优先级，**默认情况下，它们不会被本地配置覆盖**。Bootstrap context和Application Context有着不同的约定，所以新增了一个bootstrap.yml文件，保证Bootstrap Context和Application Context配置的分离。

要将Client模块下的application.yml文件改为bootstrap.yml,这是很关键的，因为**bootstrap.yml是比application.yml先加载的**。bootstrap.yml优先级高于application.yml。



**springCloud 2021.0.1版本使用bootstarp.yml文件需要引入bootstrap依赖**

```xml
		<!-- 使用bootstrap.yml 需要添加 bootstrap 依赖-->
        <!-- springCloud 2021.0.1 版本使用 bootstrap 3.1.1 版本 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
            <version>3.1.1</version>
        </dependency>
```



```yml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #config客户端配置
    config:
      label: main #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名
      #会自动添加-
      #上述3个综合：main分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/main/config-dev.yml
      uri:
        - http://localhost:3344 #配置中心地址

eureka:
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```



主启动类：

```java
@SpringBootApplication
@EnableEurekaClient
public class ConfigClient3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient3355.class);
    }
}
```



获取总服务配置3344 的配置信息

```java
@RestController
public class ConfigClientController {

    @Value("${config.info}")
    //读取3344的配置信息
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

- http://localhost:3355/configlnfo

```
master branch,springcloud-config/config-dev.yml version=7
```

**成功实现了客户端3355访问SpringCloud Config3344通过GitHub获取配置信息**



#### 1.3.2 **分布式配置的动态刷新问题**

若github上的配置文件内容进行改变，3344ConfigServer配置中心立刻响应，**3355ConfigClient客户端没有任何响应,读取本地缓存,**需要重启或者重新加载。



### 1.4 Config手动动态刷新

修改3355模块

POM引入actuator监控

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

修改YML，添加暴露监控端口配置：

```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

Controller添加**@RefreshScope**

```java
@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

发送Post请求刷新3355

```cmd
curl -X POST "http://localhost:3355/actuator/refresh"
```

成功实现了客户端3355刷新到最新配置内容，避免了服务重启



# 十.服务总线

## 1.bus

Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了Java的事件处理机制和消息中间件的功能。**Spring Clud Bus目前支持RabbitMQ和Kafka。**

![img](img.assets\458fd679c01274ca84f785e1f75c1336.png)

```xml
       <!--  消息总线RabbitMQ支持- -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
            <version>3.1.0</version>
        </dependency>
```



**能干嘛**

Spring Cloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道。

**Spring Cloud Bus 配合Spring Cloud Config 使用可以实现配置的动态刷新**



**为何被称为总线**

什么是总线

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个**共用的消息主题**(topic)，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为**消息总线**。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。

基本原理

ConfigClient实例都监听MQ中同一个**topic(默认是Spring Cloud Bus)**。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置。

![img](img.assets\26c6ced30935219d4717814a446eb67a.png)

### 1.1 Bus动态刷新全局广播的设计思想和选型

必须先具备良好的RabbitMQ环境

新建cloud-config-client-3366

**设计思想:**

1.利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置

![img](img.assets\3a0975f4bac7393fe406821531e9daef.png)

2.利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，而刷新所有客户端的配置

![img](img.assets\e2809f728b8eb3e776883e4f905b8712.png)

**方式二的架构显然更加适合**，方式一不适合的原因如下：

- 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责。
- 破坏了微服务各节点的对等性。
- 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改。



### 1.2 Bus动态刷新全局广播配置实现

**给cloud-config-center-3344配置中心服务端添加消息总线支持**

```xml
       <!--  消息总线RabbitMQ支持- -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
            <version>3.1.0</version>
        </dependency>
```

yml:

```yml
spring:
#rabbitMQ配置
 rabbitmq:
   host: 124.221.229.229
   port: 5672
   username: root
   password: root

#暴露bus刷新配置的端点<--------------------------
management:
  endpoints: #暴露bus刷新配置的端点
    web:
      exposure:
        include: 'bus-refresh'
```

**给cloud-config-client-3355客户端和cloud-config-client-3366客户端添加消息总线支持**

yml:

```yml
spring:
#rabbitMQ配置
 rabbitmq:
   host: 124.221.229.229
   port: 5672
   username: root
   password: root

# 暴露监控端点
management:
  endpoints: 
    web:
      exposure:
        include: '*'
```

在controller上加上**@RefreshScope**注解

```java
@RestController
@RefreshScope
public class ConfigClientController {}
```



在RabbitMQ客户端中查看

exchange：

![image-20220520214227525](img.assets\image-20220520214227525.png)

queue：

![image-20220520213714502](img.assets\image-20220520213714502.png)

测试：

- 修改Github上配置文件内容，增加版本号

- 发送POST请求
  - curl -X POST "http://localhost:3344/actuator/busrefresh"
  - **—次发送，处处生效**



### 1.3 Bus动态刷新定点通知

不想全部通知，只想定点通知,**指定具体某一个实例生效而不是全部**

- 公式：http://localhost:3344/actuator/busrefresh/{destination}
- /bus/refresh请求不再发送到具体的服务实例上，而是发给config server通过destination参数类指定需要更新配置的服务或实例
- {destination}：**config名称:端口 ** ,名称默认为config-client和config-servet,**可以使用通配符**



测试：

- 刷新运行在3355端口上的config-client（配置文件中设定的应用名称）为例，只通知3355，不通知3366

- `curl -X POST "http://localhost:3344/actuator/busrefresh/config-client:3355`



### 1.4 总结

![img](img.assets\ccd5fcc8293edec24d7e889e189d0bfe.png)

# 十一.消息驱动 Stream

Cloud Stream是屏蔽底层消息中间件的差异，降低切换成本，统一消息的**编程模型**。

官方定义Spring Cloud Stream是一个构建消息驱动微服务的框架。

应用程序通过**inputs或者 outputs** 来与Spring Cloud Stream中binder对象交互。

通过我们配置来**binding(绑定)**，而Spring Cloud Stream 的**binder对象负责与消息中间件交互**。所以，我们只需要搞清楚如何与Spring Cloud Stream交互就可以方便使用消息驱动的方式。

通过使用**Spring Integration来连接消息代理中间件以实现消息事件驱动。**
Spring Cloud Stream为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布-订阅、消费组、分区的三个核心概念。

目前仅支持RabbitMQ、 Kafka。

官网[Spring Cloud Stream](https://spring.io/projects/spring-cloud-stream)



```xml
        <!-- springCloud stream使用rabbitMQ为中间件 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
            <version>3.2.2</version>
        </dependency>
```



## 1. Stream的设计思想

标准的MQ：

![img](img.assets\dd57e502418ecdae99f29991abe8bb02.png)

- 生产者/消费者之间靠**消息**媒介传递信息内容
- 消息必须走特定的通道 - 消息通道 Message Channel
- 消息通道里的消息如何被消费呢，谁负责收发处理 
  	- 消息通道MessageChannel的子接口SubscribableChannel，由MessageHandler消息处理器所订阅。



在没有绑定器这个概念的情况下，我们的SpringBoot应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性通过定义绑定器作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。通过向应用程序暴露统一的Channel通道，使得应用程序不需要再考虑各种不同的消息中间件实现。

**通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离**。



## 2. Binder

**消息生产者(Output…Binding) 、消息接收者(Input…Binding)**

![img](img.assets\96256569e677453570b55209c26e0b8c.png)

**Stream中的消息通信方式遵循了发布-订阅模式**

Topic主题进行广播

- 在RabbitMQ就是Exchange
- 在Kakfa中就是Topic



## 3.Stream编码常用注解

**Spring Cloud Stream标准流程套路**

![img](img.assets\077a3b34aec6eed91a7019a9d5ca4e3c.png)

![img](img.assets\1ca02dd31581d92a7a610bcd137f6848.png)

- Binder - 很方便的连接中间件，屏蔽差异。
- Channel - 通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置。
- Source和Sink - 简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入。

**编码API和常用注解**

| 组成            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件，目前只支持RabbitMQ和Kafka                            |
| Binder          | Binder是应用与消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应于Kafka的topic,RabbitMQ的exchange)，这些都可以通过配置文件来实现 |
| @Input          | 注解标识输入通道，通过该输乎通道接收到的消息进入应用程序     |
| @Output         | 注解标识输出通道，发布的消息将通过该通道离开应用程序         |
| @StreamListener | 监听队列，用于消费者的队列的消息接收                         |
| @EnableBinding  | 指信道channel和exchange绑定在一起                            |



## 4.实例使用

工程中新建三个子模块

- cloud-stream-rabbitmq-provider8801，作为生产者进行发消息模块
- cloud-stream-rabbitmq-consumer8802，作为消息接收模块
- cloud-stream-rabbitmq-consumer8803，作为消息接收模块

### 4.1 Stream消息驱动之生产者

pom

```xml
        <!-- springCloud stream使用rabbitMQ为中间间 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>3.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
```

yml

```yml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  rabbitmq:
    host: 124.221.229.229
    port: 5672
    username: root
    password: root
  cloud:
    stream:
      binders:  # 在此处配置要绑定的rabbitmq的服务信息；
        #defaultRabbit 是RabbitMQ的一个代理实例名称，名字随便。要与下面的binder对应
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
      bindings: # 服务的整合处理
        #绑定的通道名称
        output: # 这个名字是一个通道的名称
          #在RabbitMQ中是给交换机起名字
          destination: studyExchange # 表示要使用的Exchange名称定义
          contentType: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:  # 客户端进行Eureka注册的配置
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
    lease-expiration-duration-in-seconds: 5  # 如果现在超过了5秒的间隔（默认是90秒)
    lease-renewal-interval-in-seconds: 2  # 设置心跳的时间间隔（默认是30秒）
```

主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class StreamProvider8801 {

    public static void main(String[] args) {
        SpringApplication.run(StreamProvider8801.class);
    }
}
```

发送消息的接口：

```java
//消息的接口
public interface IMessageProvider {
    String send();
}
```

实现类:

```java
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;

@EnableBinding(Source.class) //该注解已经过时
//生产放使用Source
//定义消息的推送管道
@Slf4j
public class IMessageProviderImpl implements IMessageProvider {

    @Resource
    //output为yml文件中bindings的通道名字
    private MessageChannel output;//消息发送管道

    @Override
    public String send() {
        String uuid = UUID.randomUUID().toString().replaceAll("-", "");
        //发送消息  messaging.support.MessageBuilder
        output.send(MessageBuilder.withPayload(uuid).build());
        log.info("uuid:{}",uuid);
        return uuid;
    }
}
```

**Source接口**

```java
public interface Source {
	//Name of the output channel.
	String OUTPUT = "output";
	//@return output channel
	@Output(Source.OUTPUT)
	MessageChannel output();
}
```

controller:

```java
@RestController
public class SendMessageController {

    @Resource
    private IMessageProvider iMessageProvider;

    @GetMapping("/send")
    public String sendMassage(){
       return iMessageProvider.send();
    }
}
```

RabbitMQ可视化界面:

![image-20220521213837477](img.assets\image-20220521213837477.png)



### 4.2  Stream消息驱动之消费者

pom文件和生产者相同

yml:**将bindings下方的output改为input**,其余只用修改服务名和端口号就行 

```yml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  rabbitmq:
    host: 124.221.229.229
    port: 5672
    username: root
    password: root
  cloud:
    stream:
      binders:  # 在此处配置要绑定的rabbitmq的服务信息；
        #defaultRabbit 是RabbitMQ的一个代理实例名称，名字随便。要与下面的binder对应
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
      bindings: # 服务的整合处理
        #绑定的通道名称
        input: # 这个名字是一个通道的名称
          #在RabbitMQ中是给交换机起名字
          destination: studyExchange # 表示要使用的Exchange名称定义
          contentType: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:  # 客户端进行Eureka注册的配置
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: send-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
    lease-expiration-duration-in-seconds: 5  # 如果现在超过了5秒的间隔（默认是90秒)
    lease-renewal-interval-in-seconds: 2  # 设置心跳的时间间
```

监听消息:

```java
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;

@Component
@EnableBinding(Sink.class)
@Slf4j
public class ReceiveMessageListener{

    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
      log.info("消费者:{},接收到的消息:{}",serverPort,message.getPayload());  
    }
}
```

RabbitMQ客户端中绑定了两个队列:

![image-20220521223206730](img.assets\image-20220521223206730.png)





### 4.3 Stream之消息重复消费 

由下图绑定的队列可知，交换机的类型为topic，路由key为#，可以匹配任何的单词，**就相当于广播 （fanout）类型**,会将消息发送到所有绑定的队列上。

![image-20220521230043792](img.assets\image-20220521230043792.png)

**使用Stream的分组机制可以解决重复消费的问题**



### 4.4 Stream分组

微服务应用放置于同一个group中，就能够保证消息只会被其中一个应用消费一次。

**不同的组**是可以重复消费的，**同一个组**内会发生竞争关系，只有其中一个可以消费。

**(若队列的名称相同，就会发生竞争关系(不是轮询)，能者多劳，若队列的名称不同，则重复消费)**



在消费者的8802和8803的yml文件中添加分组:

```yml
bindings: # 服务的整合处理
        #绑定的通道名称
        input: # 这个名字是一个通道的名称
          #在RabbitMQ中是给交换机起名字
          destination: studyExchange # 表示要使用的Exchange名称定义
          contentType: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: groupA #添加组名
```

分组之后的RabbitMQ界面显示:

![image-20220521233614510](img.assets\image-20220521233614510.png)

### 4.5 Stream之消息持久化

**配置了分组的消息队列，将会自动变为持久队列**

![image-20220521234150081](img.assets\image-20220521234150081.png)

**若没有配置分组，则每次启动队列的名称都是随机的，无法持久划**



### 4.6 Stream 3.1版本后写法

@EnableBinding源码中明确声明

该注解在从3.1版本开始被弃用，推荐我们使用函数编程的方式

**注意bingdings 集合中的key由   通道名-out/in-数字     组成**

**生产者**

yml:

```yml
      bindings: # 服务的整合处理
        #绑定的通道名称
        myChannel-out-0: # 这个名字是一个通道的名称
          #在RabbitMQ中是给交换机起名字
          destination: studyExchange # 表示要使用的Exchange名称定义
          contentType: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
```

**使用StreamBridge发送消息**

```java
import org.springframework.cloud.stream.function.StreamBridge;
import org.springframework.messaging.support.MessageBuilder;

@Slf4j
@Component
public class IMessageProviderImpl implements IMessageProvider {

    @Resource
    private StreamBridge streamBridge;

    @Override
    public String send() {
        String uuid = UUID.randomUUID().toString().replaceAll("-", "");
        //StreamBridge的send方法第一个参数是binding的名字，第二个参数是想要发送的消息
        streamBridge.send("myChannel-out-0",MessageBuilder.withPayload(uuid).build());
        log.info("uuid:{}",uuid);
        return uuid;
    }
}
```

**消费者**

yml:

```yml
      bindings: # 服务的整合处理
        #绑定的通道名称
        myChannel-in-0: # 这个名字是一个通道的名称
          #在RabbitMQ中是给交换机起名字
          destination: studyExchange # 表示要使用的Exchange名称定义
          contentType: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: groupA #添加组命
```

**使用函数式接口来消费数据，注意方法的名称要和绑定的通道名称相同**

```java
@Component
@Slf4j
public class ReceiveMessageListener{

    @Value("${server.port}")
    private String serverPort;
    
    @Bean
    //方法的名字要与myChannel-in-0的前缀相同
    public Consumer<String> myChannel(){
        return message -> log.info("消费者:{},接收到的消息:{}",serverPort,message);
    }
}
```



# 十二.分布式请求链路追踪 Sleuth

## 1.Sleuth概述

在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

[github地址](https://github.com/spring-cloud/spring-cloud-sleuth)

- Spring Cloud Sleuth提供了一套完整的服务跟踪的解决方案
- 在分布式系统中提供追踪解决方案并且兼容支持了zipkin

![img](img.assets\ca541262b26f809a0c25014feaa069d7.png)

## 2. Sleuth之zipkin搭建安装

- [下载地址](https://repo1.maven.org/maven2/io/zipkin/zipkin-server/)
- [官网](https://zipkin.io/)

- SpringCloud从F版起已不需要自己构建Zipkin Server了，只需调用jar包即可



在liunx上安装，并且启动

```shell
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar &
```

启动图标：

![image-20220522005019835](img.assets\image-20220522005019835.png)

可视化页面：

![image-20220522005810618](img.assets\image-20220522005810618.png)

完整的调用链路

表示一请求链路，一条链路通过Trace ld唯一标识，Span标识发起的请求信息，各span通过parent id关联起来

**就是一个请求在每个地方所有的时间**

![img](img.assets\webp)

—条链路通过Trace ld唯一标识，Span标识发起的请求信息，各span通过parent id关联起来。

![img](img.assets\f75fcfd2146df03428b9c8c53d13c1f1.png)

整个链路的依赖关系如下：

![img](img.assets\c1d19c5e9724578ee9c8668903685fa4.png)

- Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标识
- span：表示调用链路来源，通俗的理解span就是一次请求信息



## 3. Sleuth链路监控展现

使用该依赖无法在网页上显示，原因未知

```xml
		<!-- springCloud 2021.0.1版本 包含了sleuth+zipkin  -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-sleuth-zipkin</artifactId>
            <version>3.1.1</version>
        </dependency>
```

使用该依赖可以正常在客户端显示

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
            <version>2.2.8.RELEASE</version>
        </dependency>
```

![image-20220522011114683](img.assets\image-20220522011114683.png)



### 3.1 服务提供者

修改cloud-provider-payment8001

pom

```xml
 <!-- 包含了sleuth+zipkin  -->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-zipkin</artifactId>
   <version>2.2.8.RELEASE</version>
</dependency>
```

yml:

```yml
spring:
  application:
    name: cloud-payment-service

  zipkin:
    enabled: true
    #监控的的可视化地址
    base-url: http://121.41.112.246:9411/
    # 不注册成一个服务
    discovery-client-enabled: false
    sebder:
      #  数据传输方式，web 表示以 HTTP 报文的形式向服务端发送数据，还有kafka 、ACTIVEMQ 等
      type: web
    locator:
      discovery:
        enabled: true
  sleuth:
    web:
      client:
        # 是否启用 webClient
        enable: true
    sampler:
      #采样率值介于 0 到 1 之间，1 则表示全部采集
      probability: 1

```



### 3.2 服务消费者

修改cloud-consumer-order90

配置和提供者相同



客户端显示结果:

![image-20220522021640558](img.assets\image-20220522021640558.png)

