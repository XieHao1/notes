# SpringCloudAibaba

**为什么会出现SpringCloud alibaba**

Spring Cloud Netflix项目进入维护模式

![cloud升级](img.assets\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2OTAzMjYx,size_16,color_FFFFFF,t_70)



[官网](https://github.com/alibaba/spring-cloud-alibaba/blob/2.2.x/README-zh.md)

[学习地址](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html)

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。



引入依赖

- 2021.x 版本适用于 Spring Boot 2.6.x

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2021.0.1.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



**主要功能**

- **服务限流降级**：默认支持 WebServlet、WebFlux、OpenFeign、RestTemplate、Spring Cloud Gateway、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。
- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。



组件

- **[Sentinel](https://github.com/alibaba/Sentinel)**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **[Nacos](https://github.com/alibaba/Nacos)**：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **[RocketMQ](https://rocketmq.apache.org/)**：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- **[Dubbo](https://github.com/apache/dubbo)**：Apache Dubbo™ 是一款高性能 Java RPC 框架。
- **[Seata](https://github.com/seata/seata)**：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
- **[Alibaba Cloud OSS](https://www.aliyun.com/product/oss)**: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **[Alibaba Cloud SchedulerX](https://help.aliyun.com/document_detail/43136.html)**: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
- **[Alibaba Cloud SMS](https://www.aliyun.com/product/sms)**: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。



# 一.Nacos



- 一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- Nacos: Dynamic Naming and Configuration Service
- Nacos就是注册中心＋配置中心的组合 -> **Nacos = Eureka+Config+Bus**
- [首页](https://nacos.io/zh-cn/index.html)
- [文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)



**能干嘛**

- 替代Eureka做服务注册中心
- 替代Config做服务配置中心



## 1.各中注册中心比较

| 服务注册与发现框架 | CAP模型         | 控制台管理 | 社区活跃度      |
| ------------------ | --------------- | ---------- | --------------- |
| Eureka             | AP              | 支持       | 低(2.x版本闭源) |
| Zookeeper          | CP              | 不支持     | 中              |
| consul             | CP              | 支持       | 高              |
| Nacos              | AP,CP(可以切换) | 支持       | 高              |



## 2.nacos可视化页面

http://Ip:8848/nacos/

![image-20220522135814168](img.assets\image-20220522135814168.png)



## 3.nacos服务注册中心

在bin目录下：

```shell
sh startup.sh -m standalone #启动单机版

# 关闭nacos
sh shutdown.sh
```



```xml
<!-- nacos服务注册中心 版本要好springclousaibaba的版本保持一致 -->
<dependency>
   <groupId>com.alibaba.cloud</groupId>
   <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   <version>2021.0.1.0</version>
</dependency>
```



### 3.1 服务提供者注册

新建Module - cloudalibaba-provider-payment9001

[官方学习手册](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_service_registrationdiscovery_nacos_discovery)

pom:

```xml
<!-- nacos服务注册中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848

management:
  endpoints:
    web:
      exposure:
        include:
          - '*'
```

主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosPayment9001 {
    public static void main(String[] args) {
        SpringApplication.run(NacosPayment9001.class);
    }
}
```

controller:

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/payment/nacos/{id}")
    public String payment(@PathVariable("id") Integer id){
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }
}
```

启动项目后可以在nacos可视化页面中查看服务信息

![image-20220522142605333](img.assets\image-20220522142605333.png)



重复上面过程新建新建Module - cloudalibaba-provider-payment9002

![image-20220522143609712](img.assets\image-20220522143609712.png)



### 3.2 服务消费者注册

新建Module - cloudalibaba-consumer-nacos-order83

创建方式提供者基本相同,可以不用暴露服务端点

pom:

**nacos2021.0.1.0 的依赖中已经移出了Rabbon依赖，需要自己手动添加或者使用loadbalancer**

```xml
<!-- nacos服务注册中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

 <!--Spring Cloud 2021.0.1版本 loadbalancer 3.1.1-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

yml:

```yml
server:
  port: 83

spring:
  application:
    name: nacos-payment-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848
```

主启动类:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosOrder83 {
    public static void main(String[] args) {
        SpringApplication.run(NacosOrder83.class);
    }
}
```

RestTemplate模板配置类

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

controller：

```java
@RestController
@Slf4j
public class OrderController {

    private static final String URL_PREFIX = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {
        return restTemplate.getForObject(URL_PREFIX+"/payment/nacos/"+id,String.class);
    }
}
```



### 3.3 服务中心的对比

**Nacos全景图**

![image-20220522150515632](img.assets\image-20220522150515632.png)

**Nacos和CAP**

Nacos与其他注册中心特性对比

![Nacos与其他注册中心特性对比](img.assets\62d5a8566a2dc588a5ed52346049a054.png)

**Nacos服务发现实例模型**

![Nacos服务发现实例模型](img.assets\6578e36df056a995a39034045c36fc40.png)



#### 3.3.1 **Nacos支持AP和CP模式的切换**

**C是所有节点在同一时间看到的数据是一致的;而A的定义是所有的请求都会收到响应。**

A:可用性,C:强一致性,P:多备份性

—般来说，如果**不需要存储服务级别的信息且服务实例是通过nacos-client注册**，并能够保持心跳上报，那么就可以选择AP模式。

当前**主流的服务如Spring cloud和Dubbo服务，都适用于AP模式**，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

如果需**要在服务级别编辑或者存储配置信息，那么CP是必须**，**K8S服务和DNS服务则适用于CP模式**。CP模式下则支持注册持久化实例，此时则是以Raft协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。



切换命令：

```shell
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP
```



## 4.nacos服务配置

代替config做服务配置

```xml
<!-- nacos服务配置 版本要好springcloudAibaba的版本保持一致 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>2021.0.1.0</version>
</dependency>
```



### 4.1 在Nacos中添加配置信息

#### 4.4.1 配置规则

Nacos中的dataid的组成格式及与SpringBoot配置文件中的匹配规则

1. 在 `bootstrap.properties` 中配置 Nacos server 的地址和应用名

```properties
spring.cloud.nacos.config.server-addr=127.0.0.1:8848

spring.application.name=example
```

说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```cmd
${prefix}-${spring-profile.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

2.通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新：



#### 4.4.2 在Nacos客户端中添加配置信息

**建议使用火狐浏览器，ege浏览器无法编辑**

![image-20220522162420185](img.assets\image-20220522162420185.png)



![image-20220522184110545](img.assets\image-20220522184110545.png)



发布:

![image-20220522184157296](img.assets\image-20220522184157296.png)

#### 4.4.3 获取配置信息

新建 module cloudalibaba-config-nacos-client3377

pom:

```xml
       	<!-- 使用bootstrap.yml 需要添加 bootstrap 依赖-->
        <!-- springCloud 2021.0.1 版本使用 bootstrap 3.1.1 版本 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
            <version>3.1.1</version>
        </dependency>

		<!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        
        <!--nacos-discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础配置-->
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

Nacos同springcloud-config一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。

**springboot中配置文件的加载是存在优先级顺序的，bootstrap优先级高于application**



bootstrap.yml:

```yml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848 #Nacos服务注册中心地址
      config:
        server-addr: 121.41.112.246:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
#        group: DEV_GROUP
#        namespace: 7d8f0f5a-6a53-4785-9686-dd460158e5d4

#${prefix}-${spring-profile.active}.${spring.cloud.file-extension} 读取公式 :
#	nacos-config-client-dev.yaml

#prefix 默认为 `spring.application.name` 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。

#`spring.profiles.active` 即为当前环境对应的 profile，spring.profiles.active=dev
#当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**

#`file-exetension` 为配置内容的数据格式
#可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。
#目前只支持 `properties` 和 `yaml` 类型。
```

application.yml:

```yml
spring:
  profiles:
    active: dev
```

![img](img.assets\b3bffc4a646b30f9bf64fc649bf26f7d.png)

controller:

```java
@RestController
@RefreshScope //支持Nacos的动态刷新功能。
public class ConfigClientController
{
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

#### 4.4.4  自带动态刷新

修改nacos中的配置，再次发送请求，可以发现获得修改之后的值。



## 5.Nacos之命名空间分组和DataID三者关系

如何保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件,一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境…那怎么对这些微服务配置进行管理？

Nacos的图形化管理界面：

![image-20220522195654479](img.assets\image-20220522195654479.png)



### 5.1 命名空间，分组和DataID关系

这三者类似Java里面的package名和类名，最外层**的namespace是可以用于区分部署环境的，Group和DatalD逻辑上区分两个目标对象。**

**优先级:namespace > Group > DataId**

**namespace区分开发环境，group区分地区集群**

![img](img.assets\60712abd615dd86ac6c119bf132a28d6.png)

默认情况：Namespace=public(预留空间)，Group=DEFAULT_GROUP，默认Cluster是DEFAULT

- 比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。

- Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组里面去

- Service就是微服务:一个Service可以包含多个Cluster (集群)，Nacos默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分。

  比方说为了容灾，将Service微服务分别部署在了杭州机房和广州机房，这时就可以给杭州机房的Service微服务起一个集群名称(HZ) ，给广州机房的Service微服务起一个集群名称(GZ)，还可以尽量让同一个机房的微服务互相调用，以提升性能。

- 最后是Instance，就是微服务的实例。



### 5.2 DataID配置

指定**spring.profiles.active**和**配置文件的DatalD**来使不同环境下读取不同的配置

默认空间+默认分组+新建dev和test两个DatalD

![image-20220522201504092](img.assets\image-20220522201504092.png)

通过**spring.profiles.active属性**就能进行多环境下配置文件的读取



### 5.3 Group分组方案

通过Group实现环境区分 - 新建Group

![image-20220523115931145](img.assets\image-20220523115931145.png)

在bootstrap.yml文件中`spring.cloud.nacos.config.group`指定读取的组:

```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848 #Nacos服务注册中心地址
      config:
        server-addr: 121.41.112.246:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP #指定组
```



### 5.4 Namespace空间

命名空间用于隔离不同租户的配置。组和数据 ID 在不同的命名空间中可以相同。命名空间的典型场景是**不同环境的配置隔离**，例如开发/测试环境和生产环境（配置和服务等）的隔离。 

如果 ${spring.cloud.nacos.config.namespace} 中没有指定命名空间，则使用 Nacos 的“public”命名空间。

新建dev的Namespace

![image-20220523120706002](img.assets\image-20220523120706002.png)

我们可以在配置列表中切换命名空间来创建配置信息

![image-20220523121033767](img.assets\image-20220523121033767.png)

在bootstrap.yml文件中指定命名空间--`spring.cloud.nacos.config.namespace`，如果不配置，则为默认public命名空间。

```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848 #Nacos服务注册中心地址
      config:
        server-addr: 121.41.112.246:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP #指定组
        namespace: ea60fcb4-0d46-4364-80c5-f218773caec7
```



## 6.Nacos持久化切换配置

Nacos默认**自带的是嵌入式数据库derby**

derby到mysql切换配置步骤：

1. usr\local\nacos\conf录下找到nacos-mysql.sql文件，执行脚本。---将sql语句粘贴到本地的mysql中
2. usr\local\nacos\conf目录下找到application.properties，添加以下配置（按需修改对应值）。

```properties
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=root
```

3. 重启Nacos,新建一个配置后，可以 在config_info表中查询到相关配置

![image-20220523140939675](img.assets\image-20220523140939675.png)





## 7.nacos集群

Nacos支持三种部署模式

- 单机模式-用于测试和单机试用。
- 集群模式-用于生产环境，确保高可用。
- 多集群模式-用于多数据中心场景。



默认Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题的。为了解决这个问题，**Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySQL的存储**。





![deployDnsVipMode.jpg](img.assets\deployDnsVipMode.jpg)

细化：**要3个以及3个以上的nacos才能构成集群**

![img](img.assets\681c3dc16a69f197896cbff482f2298e.png)

### 7.1 集群配置cluster.conf

梳理出3台nacos集器的不同服务端口号，设置3个端口：3333  4444  5555

拷贝cluster.conf.example(在conf目录下):

```shell
cp cluster.conf.example cluster.conf
```

![image-20220523143103416](img.assets\image-20220523143103416.png)

修改cluster.conf的内容:

**注意**，这个IP不能写127.0.0.1，必须是Linux命令`hostname -i`能够识别的IP

![image-20220523143438531](img.assets\image-20220523143438531.png)

```txt
172.28.133.176:8848
172.28.133.176:8849
172.28.133.176:8850
```

### 7.2 修改端口号

进入 conf 文件夹下的 `application.properties` ，将端口号修改

![image-20220523145713977](img.assets\image-20220523145713977.png)





### 7.3 **编辑Nacos的启动脚本startup.sh**

/nacos/bin目录下有startup.sh,**先将脚本进行备份**

```shell
cp startup.sh startup.sh.bk
```

修改内存大小，否则可能内存不够

![image-20220523145917316](img.assets\image-20220523145917316.png)



### 7.4 复制 Nacos

修改完成后，将 Nacos 复制两份

切换到 Nacos 的安装目录，我这里是在 `/usr/local`

首先输入 `sudo mv nacos nacos8848` 将原来的 Nacos 重命名为 `nacos8848` ，root 用户不需要输入 `sudo`

![image-20220523150414589](img.assets\image-20220523150414589.png)

执行 `sudo cp -r nacos8848 nacos8849` 复制出一份 8849
执行 `sudo cp -r nacos8848 nacos8850` 复制出一份 8850

![image-20220523151047622](img.assets\image-20220523151047622.png)

### 7.5  修改复制Nacos的端口号

### 7.6 分别启动 Nacos 服务

```shell
sh shartup.sh #启动集群环境
```



若在三台服务器上分别部署，**则只需要修改cluster.conf端口和数据库连接地址即可**

![image-20220524114225080](img.assets\image-20220524114225080.png)



### 7.7 Nginx的配置

修改nginx的配置文件 - nginx.conf

```c++
   #负载均衡
	upstream cluster{
        server 192.168.153.136:8848 ;
        server 192.168.153.137:8848 ;
        server 192.168.153.138:8848 ;
    }

    server {
         # listen       80;
         # 监听的端口设置
        listen 1111;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
           # root   html;
           # index  index.html index.htm;
           # 配置代理地址
           proxy_pass http://cluster ;
        }
```

按照指定配置文件启动:bin目录下:

```
./nginx -c /usr/local/nginx/conf/nginx.conf
```

### 7.8 启动测试

输入http://nginx地址:1111/nacos

http://192.168.153.139:1111/nacos 成功访问

yaml文件修改

```yaml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        #使用nginx做代理实现nacos集群
        server-addr: 192.168.153.139:1111 #Nacos服务注册中心地址
      config:
        server-addr: 192.168.153.139:1111 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
```

测试成功

![img](img.assets\42ff7ef670012437b046f099192d7484.png)

# 二.Sentinel

[官网](https://github.com/alibaba/Sentinel/wiki/介绍)

[下载](https://github.com/alibaba/Sentinel/releases)

[学习文档](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel)

```xml
        <!-- springCloudAibaba 2021.0.1.0版本-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
            <version>2021.0.1.0</version>
        </dependency>
```

Sentinel: 分布式系统的流量防卫兵

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 分为两个部分：

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

Sentinel 的主要特性:

![Sentinel-features-overview](img.assets\50505538-2c484880-0aaf-11e9-9ffc-cbaaef20be2b.png)



- 运行命令
  - 前提
    - Java 8 环境
    - 8080端口不能被占用
  - 命令
    - `java -jar sentinel-dashboard-1.8.4.jar`
- 访问Sentinel管理界面
  - localhost:8080
  - 登录账号密码均为sentinel
  
  

关闭tomcat服务

```shell
service tomcat stop
```

也可以修改启动的端口

```shell
java -Dserver.port=8080 #用于指定 Sentinel 控制台端口
```



界面：

![image-20220524133415992](img.assets\image-20220524133415992.png)

![image-20220524133443499](img.assets\image-20220524133443499.png)



## 1.Sentinel初始化监控

**启动Nacos8848成功**

**新建工程 - cloudalibaba-sentinel-service8401**

**尽量保持监控和客户端在同一个ip下运行，否则无法准确监控**

pom:

```xml
        <!-- springCloudAibaba 2021.0.1.0版本 sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        
        <!-- nacos服务注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848
    #sentinel配置
    sentinel:
      transport:
        #配置Sentinel dashboard地址,最好和监视处于同一个ip
        dashboard: 127.0.0.1:8080
        #sentinel 后台监控的端口 sentinel会启动一个http server端口和dashoard进行通信
        #默认8719端口，假如端口被占用，就自动从8719端口进行+1扫描，直到找到未被占用的端口
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include:
          - '*'
```

主启动类:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Sentinel8401 {
    public static void main(String[] args) {
        SpringApplication.run(Sentinel8401.class);
    }
}
```

controller:

```java
@RestController
@Slf4j
public class FlowLimitController {
    
    @GetMapping("/testA")
    public String testA() {
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB() {
        log.info(Thread.currentThread().getName()+"\t"+"...testB");
        return "------testB";
    }
}
```

启动

- 刚启动，空空如也，啥都没有

- Sentinel采用的懒加载说明
  - 执行一次访问即可
    - http://localhost:8401/testA
    - http://localhost:8401/testB
  - 效果 - sentinel8080正在监控微服务8401

![image-20220524140401608](img.assets\image-20220524140401608.png)



## 2.流控规则

Sentinel**默认只标记Controller中的方法为资源**，如果要标记其它方法，需要利用@SentinelResource注解

![image-20220524143924045](img.assets\image-20220524143924045.png)

- 资源名：唯一名称，默认请求路径。
- 针对来源：Sentinel可以针对调用者进行限流，**填写微服务名**，默认default（不区分来源）。
- 阈值类型/单机阈值：
  - QPS(每秒钟的请求数量)︰当调用该API的QPS达到阈值的时候，进行限流。
  - 线程数：当调用该API的线程数达到阈值的时候，进行限流。
- 是否集群：不需要集群。
- 流控模式：
  - 直接：API达到限流条件时，直接限流。
  - 关联：当关联的资源达到阈值时，就限流自己。
  - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流)【API级别的针对来源】。
- 流控效果：
  - 快速失败：直接失败，抛异常。
  - Warm up：根据Code Factor（冷加载因子，默认3）的值，从阈值/codeFactor，经过预热时长，才达到设置的QPS阈值。
  - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效。



### 2.1 QPS 直接失败

**直接 -> 快速失败（系统默认）**

设置testA的流控

![image-20220524144509163](img.assets\image-20220524144509163.png)

测试结果：快速发送testA请求，被Sentinel限流，返回默认的结果

```
Blocked by Sentinel (flow limiting)
```



### 2.2  线程数直接失败

线程数：当调用该API的线程数达到阈值的时候，进行限流。

![image-20220524145341940](img.assets\image-20220524145341940.png)

测试结果，使用两个线程进行访问，第二个线程被Sentinel限流，返回默认的结果

```
Blocked by Sentinel (flow limiting)
```



### 2.3 关联

- 当自己关联的资源达到阈值时，就限流自己
- 当与A关联的资源B达到阀值后，就限流A自己（B惹事，A挂了）

**当付款业务访问达到阈值，就限流下单业务**

![image-20220524150346312](img.assets\image-20220524150346312.png)

使用JMeter发送testB请求2W次

在这期间访问testA

```
Blocked by Sentinel (flow limiting)
```

### 2.4 链路

只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流)【API级别的针对来源】

**针对从指定链路访问到本资源的请求做统计，判断是否超过阈值。**

从入口/A来访问/testA请求，若超过阈值，就对testA进行限流

![image-20220524151922440](img.assets\image-20220524151922440.png)



Sentinel**默认只标记Controller中的方法为资源**，如果要标记其它方法，需要利用@SentinelResource注解，示例：

```java
@SentinelResource("goods")
public void queryGoods(){
    System.err.println("查询商品");
}
```

Sentinel默认会将Controller方法做context整合，导致链路模式的流控失效，需要修改application.yml，添加配置：

```yml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8080 # sentinel控制台地址
      web-context-unify: false # 关闭context整合
```



### 2.5 预热  Warm Up

Warm Up（`RuleConstant.CONTROL_BEHAVIOR_WARM_UP`）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。

**默认coldFactor为3**，即请求QPS 从 **threshold(阈值) / 3开始**，经预热时长逐渐升至设定的QPS阈值。

[限流 冷启动 · alibaba/Sentinel Wiki (github.com)](https://github.com/alibaba/Sentinel/wiki/限流---冷启动)

这个场景主要用于启动需要额外开销的场景，例如建立数据库连接等。

**应用场景**

如：秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是把为了保护系统，可慢慢的把流量放进来,慢慢的把阀值增长到设置的阀值。



当QPS达到 10/3=3  后**（请求数超过3时开始计时）**，经过10秒的预热时长升至设定的QPS阈值。在3秒中之前，只能处理3个请求，在预热后才能处理10个。

![image-20220524153027266](img.assets\image-20220524153027266.png)



### 2.6  排队等待

匀速排队（`RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。

[流量控制 匀速排队模式 · alibaba/Sentinel Wiki (github.com)](https://github.com/alibaba/Sentinel/wiki/流量控制-匀速排队模式)

当请求到来的时候，如果当前请求距离上个通过的请求通过的**时间间隔不小于预设值**，则让当前请求通过；否则，计算当前请求的预期通过时间，如果该请求的预期通过时间小于规则预设的 timeout 时间，则该请求会等待直到预设时间到来通过（排队等待处理）；若预期的通过时间超出最大排队时长，则直接拒接这个请求。

![img](img.assets\79f93ab9f5dc11b05bbed9b793ef7c20.png)

这种方式主要用于处**理间隔性突发的流量，例如消息队列**。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

注意：匀速排队模式暂时不支持 QPS > 1000 的场景。



匀速排队，让请求以均匀的速度通过，阀值类型必须设成QPS，否则无效。

设置：/testA每秒处理 10次请求，超过的话就排队等待，等待的超时时间为20000毫秒。

![image-20220524175834542](img.assets\image-20220524175834542.png)



## 3.熔断规则

Sentinel 提供以下几种熔断策略：

- 调用比例 (`SLOW_REQUEST_RATIO`)：选择以慢调用比例作为阈值，需要设置允许的慢调用 **RT（即最大的响应时间）**，请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过**熔断时长后熔断器会进入探测恢复状态**（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。**（超时降级）**
- 异常比例 (`ERROR_RATIO`)：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。**(发生异常后降级,按比例)**
- 异常数 (`ERROR_COUNT`)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。**(发生异常后降级,按数量)**
- 在半开状态下，发生请求将会返回结果，无论对错

注意异常降级**仅针对业务异常**，对 Sentinel 限流降级本身的异常（`BlockException`）不生效。为了统计异常比例或异常数，需要通过 `Tracer.trace(ex)` 记录业务异常。

![image-20220524191431275](img.assets\image-20220524191431275.png)

|                    |                                                              |            |
| ------------------ | ------------------------------------------------------------ | ---------- |
| Field              | 说明                                                         | 默认值     |
| resource           | 资源名，即规则的作用对象                                     |            |
| grade              | 熔断策略，支持慢调用比例/异常比例/异常数策略                 | 慢调用比例 |
| count              | 慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值 |            |
| timeWindow         | 熔断时长，单位为 s                                           |            |
| minRequestAmount   | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入） | 5          |
| statIntervalMs     | 统计时长（单位为 ms），如 60*1000 代表分钟级（1.8.0 引入）   | 1000 ms    |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入）           |            |



### 3.1 RT

​	调用比例 (`SLOW_REQUEST_RATIO`)：选择以慢调用比例作为阈值，需要设置允许的慢调用 **RT（即最大的响应时间）**，请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过**熔断时长后熔断器会进入探测恢复状态**（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。**（超时熔断）**

-----------就是在统计时间中达到了最小请求数，且一些请求请求的时间大于最大的响应时间，就会发生服务降级，若数量大于了一定的比例，就会触发服务熔断。在熔断时间过去后，熔断器进入半开状态，允许一个请求访问，若通过，则关闭断路器，若没有通过，则继续熔断。



测试：设置请求的最大响应时间为200ms

![image-20220524195653302](img.assets\image-20220524195653302.png)

```java
@GetMapping("/testD")
    public String testD() {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info("testD 测试RT");
        return "------testD";
    }
```

使用Jmeter测试后

```
Blocked by Sentinel (flow limiting)
```



### 3.2 异常比例

异常比例 (`ERROR_RATIO`)：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。**(发生异常后熔断,按比例)**

![image-20220524201029435](img.assets\image-20220524201029435.png)

```java
@GetMapping("/testD")
public String testD() {
    int age = 10/0;
    log.info("testD 测试RT");
    return "------testD";
}
```

使用Jmeter测试后

```
Blocked by Sentinel (flow limiting)
```

### 3.3 异常数

- 异常数 (`ERROR_COUNT`)：当**单位统计时长**内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。**(发生异常后降级,按数量)**

![image-20220524202319069](img.assets\image-20220524202319069.png)

```java
@GetMapping("/testD")
public String testD() {
    int age = 10/0;
    log.info("testD 测试RT");
    return "------testD";
}
```

使用Jmeter测试后

```
Blocked by Sentinel (flow limiting)
```



## 4.热点规则

[热点参数限流 · alibaba/Sentinel Wiki (github.com)](https://github.com/alibaba/Sentinel/wiki/热点参数限流)

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计**传入参数中的热点参数**，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

源码：`com.alibaba.csp.sentinel.slots.block.BlockException`

![Sentinel Parameter Flow Control](E:\笔记\SpringCloud\img.assets\sentinel-hot-param-overview-1.png)

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。



![image-20220524203124737](img.assets\image-20220524203124737.png)

|                   |                                                              |          |
| ----------------- | ------------------------------------------------------------ | -------- |
| 属性              | 说明                                                         | 默认值   |
| resource          | 资源名，必填                                                 |          |
| count             | 限流阈值，必填                                               |          |
| grade             | 限流模式                                                     | QPS 模式 |
| durationInSec     | 统计窗口时间长度（单位为秒），1.6.0 版本开始支持             | 1s       |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持   | 快速失败 |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持 | 0ms      |
| paramIdx          | 热点参数的索引，必填，对应 `SphU.entry(xxx, args)` 中的参数索引位置 |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型和字符串类型** |          |
| clusterMode       | 是否是集群参数流控规则                                       | `false`  |
| clusterConfig     | 集群流控相关配置                                             |          |

### 4.1 基本配置

```java
	@GetMapping("/testHotKey")
 	//自定义降级配置，value可以自定义
	//在进行热点降级的时候，最好设置兜底方法，否则前台会直接出现错误页面
    @SentinelResource(value = "testHotKey",/*兜底方法*/ blockHandler = "deal_testHotKey")
    public String testHotkey(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2",required = false) String p2){
        return "------testHotKey";
    }

    /*兜底方法*/
    public String deal_testHotKey (String p1, String p2, BlockException exception) {
        return "------deal_testHotKey,o(╥﹏╥)o";  //sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
    }
```

![image-20220524212825556](img.assets\image-20220524212825556.png)

测试结果：

![image-20220524213040769](img.assets\image-20220524213040769.png)

### 4.2 参数列外项

- **我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样**
- 特例 - 假如当p1的值等于5时，它的阈值可以达到200



**前提条件** - 热点参数的注意点，参数必须是基本类型或者String

![image-20220524213956298](img.assets\image-20220524213956298.png)

测试结果：

![image-20220524214043251](img.assets\image-20220524214043251.png)

## 5.系统规则

[系统自适应限流 · alibaba/Sentinel Wiki (github.com)](https://github.com/alibaba/Sentinel/wiki/系统自适应限流)

Sentinel 系统自适应限流**从整体维度对应用入口流量进行控制**，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

![image-20220524214406839](img.assets\image-20220524214406839.png)

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

- **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
- **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。



## 6.@SentinelResource注解

**@SentinelResource 注解**

> 注意：注解方式埋点不支持 private 方法。

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

- `value`：资源名称，必需项（不能为空）
- `entryType`：entry 类型，可选项（默认为 `EntryType.OUT`）
- `blockHandler` / `blockHandlerClass`: `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。blockHandler 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，**参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 `BlockException`**。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，**注意对应的函数必需为 static 函数，否则无法解析。**
- `fallback/fallbackClass`：fallback 函数名称，可选项，用于在**抛出异常的时候提供 fallback 处理逻辑**。fallback 函数可以针对所有类型的异常（除了exceptionsToIgnore里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：
  - 返回值类型必须与原函数返回值类型一致；
  - **方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable`** 类型的参数用于接收对应的异常。
  - fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意**对应的函数必需为 static 函数**，否则无法解析。
- `defaultFallback`（since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了exceptionsToIgnore里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：
  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  - defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对**应的函数必需为 static 函数，否则无法解析。**
- `exceptionsToIgnore`（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。



**fallback管java程序出现的异常，blockHandler管服务相关配置错误**



Sentinel主要有三个核心Api：

1. SphU定义资源
2. Tracer定义统计
3. ContextUtil定义了上下文



###  6.1  按资源名称限流 + 后续处理

```java
	@GetMapping("/byResource")
	//value的值为资源名称
    @SentinelResource(value = "byResource", /*兜底方法*/ blockHandler = "handleException")
    public String byResource() {
        return "按资源名称限流测试OK";
    }

	//处理方法中的参数要和原方法保持一致，且处理方法中需要有BlockException参数
    public String handleException(BlockException exception) {
        return exception.getClass().getCanonicalName()+"\t 服务不可用";
    }

```

![image-20220525152320782](img.assets\image-20220525152320782.png)

对byResource进行配置，**没有/**

![image-20220525152405104](img.assets\image-20220525152405104.png)

测试结果:

```
com.alibaba.csp.sentinel.slots.block.flow.FlowException	 服务不可用
```



### 6.2  按Url地址限流 + 后续处理

**若使用URL地址限流，将不会执行blockHandler在中的方法,而是返回默认的结果**

```java
@GetMapping("/rateLimit/byUrl")
@SentinelResource(value = "byUrl",blockHandler = "handleException")
public String byUrl() {
    return "按URL限流测试OK";
}
```

通过URL进行限流:

![image-20220525152907585](img.assets\image-20220525152907585.png)

测试结果:

```
Blocked by Sentinel (flow limiting)
```



### 6.3 自定义限流处理逻辑

自定义限流处理类 - 创建CustomerBlockHandler类用于自定义限流处理逻辑

**统一限流处理类中的方法必须为静态方法**

**处理方法中的参数要和原方法保持一致，且处理方法中需要有BlockException参数**

```java
//统一限流处理类
public class CustomerBlockHandler {

    //统一限流处理类中的方法必须为静态方法
    public static String handlerException(BlockException exception) {
        return "按客戶自定义,global handlerException----1";
    }

    public static String handlerException2(BlockException exception) {
        return "按客戶自定义,global handlerException----1";
    }
}
```

```java
    @GetMapping("/rateLimit/customerBlockHandler")
    @SentinelResource(value = "customerBlockHandler",
            //去CustomerBlockHandler下找handlerException方法进行限流处理
            blockHandlerClass = CustomerBlockHandler.class,
            blockHandler = "handlerException")
    public String customerBlockHandler() {
        return "自定义限流测试";
    }
```

设置限流规则:

![image-20220525155013414](img.assets\image-20220525155013414.png)

测试结果：

```
按客戶自定义,global handlerException----1
```



## 7. 自定义服务熔断

### 7.1 服务提供者

新建cloudalibaba-provider-payment9003/9004

pom：

```xml
        <!-- nacos服务注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
  port: 9003

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848

management:
  endpoints:
    web:
      exposure:
        include:
          - '*'
```

主启动类：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Payment9003 {
    public static void main(String[] args) {
        SpringApplication.run(Payment9003.class);
    }
}
```

controller:

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    //模拟数据库
    public static HashMap<Long,String> hashMap = new HashMap<>();
    
    static {
        hashMap.put(1L,"28a8c1e3bc2742d8848569891fb42181");
        hashMap.put(2L,"bba8c1e3bc2742d8848569891ac32182");
        hashMap.put(3L,"6ua8c1e3bc2742d8848569891xt92183");
    }

    @GetMapping(value = "/paymentSQL/{id}")
    public String paymentSQL(@PathVariable("id") Long id) {
        return hashMap.get(id) +"端口号："+ serverPort;
    }
}
```



### 7.2 服务消费者

cloudalibaba-consumer-nacos-order84

pom:

```xml
       <!-- nacos服务注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <!--Spring Cloud 2021.0.1版本 loadbalancer 3.1.1-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>

        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
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
  port: 84

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719
```

主启动：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderNacos84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacos84.class);
    }
}
```

RestTemplatep配置

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

controller:

```java
@RestController
public class ConsumerController {

    private static final String URL_PREFIX = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;
    
    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback")//没有配置
    public String fallback(@PathVariable Long id) {
        String result = restTemplate.getForObject(URL_PREFIX + "/paymentSQL/"+id,String.class,id);
        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }
}
```



### 7.3 使用fallback

```java
	@RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback")
    public String fallback(@PathVariable Long id) {
        String result = restTemplate.getForObject(URL_PREFIX + "/paymentSQL/"+id,String.class,id);
        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }

    //处理异常的方法，必须和原方法有相同的返回值类型，参数列表，处理方法可以多加一个Throwable
    public String handlerFallback(Long id,Throwable e) {
        return "兜底异常handlerFallback,exception内容  "+e.getMessage();
    }
```

测试：

```
兜底异常handlerFallback,exception内容  IllegalArgumentException,非法参数异常....
```



### 7.4 使用blockHandler

**fallback管java程序出现的异常，blockHandler管服务相关配置错误**

```java
    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler")
    public String fallback(@PathVariable Long id) {
        String result = restTemplate.getForObject(URL_PREFIX + "/paymentSQL/"+id,String.class,id);
        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result == null || result.contains("null")) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }


    //处理异常的方法，必须和原方法有相同的返回值类型，参数列表，处理方法可以多加一个Throwable
    public String handlerFallback(Long id,Throwable e) {
        return id+"  兜底异常handlerFallback,exception内容  "+e.getMessage();
    }

    //处理流控的方法，必须和原方法有相同的返回值类型，参数列表，处理方法可以多加一个 BlockException
    public String blockHandler(Long id, BlockException e) {
        return id+"  不满足流控配置 blockHandler,exception内容  "+e.getMessage();
    }
```

新增异常熔断规则:

![image-20220525170838994](img.assets\image-20220525170838994.png)

使用Jmeter测试

![image-20220525172108248](img.assets\image-20220525172108248.png)



### 7.5  exceptionsToIgnore配置

`exceptionsToIgnore`（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

```java
@RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",
            //忽略空指针异常，既空指针异常保持不走fallback方法
            exceptionsToIgnore = NullPointerException.class)
    public String fallback(@PathVariable Long id) {
        String result = restTemplate.getForObject(URL_PREFIX + "/paymentSQL/"+id,String.class,id);
        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result == null || result.contains("null")) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }
```

测试结果:

![image-20220525172447813](img.assets\image-20220525172447813.png)

### 7.6 使用openFeign

**openFegin只能处理服务端的异常**

在服务消费者83中添加openFeign依赖

pom:

```xml
<!--SpringCloud openfeign -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

在yml设置openFeign的超时时间和激活对sentinel的支持

```yml
feign:
  client:
    config:
      default:
        #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
        connectTimeout: 5000
        #指的是建立连接后从服务器读取到可用资源所用的时间
        readTimeout: 5000
  #激活Sentinel对Feign的支持
  sentinel:
    enabled: true
  #开启openfeign服务降级
  circuitbreaker:
    enabled: true
```

在主配置类上开启@EnableFeignClients注解

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class OrderNacos84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacos84.class);
    }
}
```

带@Feignclient注解的业务接口，fallback = PaymentFallbackService.class

```java
@FeignClient(value = "nacos-payment-provider",fallback = PaymentServiceImpl.class)
public interface PaymentService {

    @GetMapping("/paymentSQL/{id}")
    String paymentSQL(@PathVariable("id") Long id);

}
```

降级类:

```java
@Component
@Slf4j
public class PaymentServiceImpl implements PaymentService {

    @Override
    public String paymentSQL(Long id) {
        log.info("openFeign服务降级");
        return null;
    }
}
```

调用:

```java
    @Resource private PaymentService paymentService;

	@GetMapping("/consumer/openFeign/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler")
    public String openFeign(@PathVariable("id") Long id){
        String result =paymentService.paymentSQL(id);
        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result == null || result.contains("null")) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }
```



## 8.Sentinel持久化规则

一旦我们重启应用，sentinel规则将消失，生产环境需要将配置规则进行持久化。

**原理：将限流配置规则持久化进Nacos保存**

```xml
<!-- Sentinel持久化依赖 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <version>1.8.3</version>
</dependency>
```



修改cloudalibaba-sentinel-service8401

在pom文件中增加

```xml
<!-- Sentinel持久化依赖 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

修改yml文件

```yml
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 121.41.112.246:8848
    #sentinel配置
    sentinel:
      transport:
        #配置Sentinel dashboard地址,最好和监视处于同一个ip
        dashboard: 127.0.0.1:8080
        #sentinel 后台监控的端口 sentinel会启动一个http server端口和dashoard进行通信
        #默认8719端口，假如端口被占用，就自动从8719端口进行+1扫描，直到找到未被占用的端口
        port: 8719
      #添加nacos数据源配置,将sentinel的配置写入nacos中
      datasource:
        ds1:
          nacos:
			username: nacos
            password: nacos
            serverAddr: 121.41.112.246:8848
            namespace: public
            dataId: ${spring.application.name}
            groupId: DEFAULT_GROUP
            dataType: json
            ruleType: flow
```

在nacos的配置列表中新增配置:**注意dataID，Group，namespace要和yml文件中保持一致**

![image-20220525182342915](img.assets\image-20220525182342915.png)

```json
[{
    "resource": "/rateLimit/byUrl",			 //资源名称
    "IimitApp": "default", 					 //来源应用
    "grade": 1, 							 //阈值类型，0表示线程数, 1表示QPS
    "count": 1,  							 //单机阈值
    "strategy": 0, 							 //流控模式，0表示直接，1表示关联，2表示链路
    "controlBehavior": 0, 					 //流控效果，0表示快速失败，1表示Warm Up，2表示排队等待
    "clusterMode": false 					 //是否集群
}]
```

测试后在sentinel控制台出现流控规则

![image-20220525183211564](img.assets\image-20220525183211564.png)



## 9. 三大服务降级框架比较

| -              | Sentinel                                                   | Hystrix                | resilience4j                     |
| -------------- | ---------------------------------------------------------- | ---------------------- | -------------------------------- |
| 隔离策略       | 信号量隔离（并发线程数限流）                               | 线程池隔商/信号量隔离  | 信号量隔离                       |
| 熔断降级策略   | 基于响应时间、异常比率、异常数                             | 基于异常比率           | 基于异常比率、响应时间           |
| 实时统计实现   | 滑动窗口（LeapArray）                                      | 滑动窗口（基于RxJava） | Ring Bit Buffer                  |
| 动态规则配置   | 支持多种数据源                                             | 支持多种数据源         | 有限支持                         |
| 扩展性         | 多个扩展点                                                 | 插件的形式             | 接口的形式                       |
| 基于注解的支持 | 支持                                                       | 支持                   | 支持                             |
| 限流           | 基于QPS，支持基于调用关系的限流                            | 有限的支持             | Rate Limiter                     |
| 流量整形       | 支持预热模式匀速器模式、预热排队模式                       | 不支持                 | 简单的Rate Limiter模式           |
| 系统自适应保护 | 支持                                                       | 不支持                 | 不支持                           |
| 控制台         | 提供开箱即用的控制台，可配置规则、查看秒级监控，机器发观等 | 简单的监控查看         | 不提供控制台，可对接其它监控系统 |



# 三.Seata分布式业务处理

单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用,分别使用三个独立的数据源，业务操作需要调用三个服务来完成。此时**每个服务内部的数据一致性由本地事务来保证， 但是全局的数据一致性问题没法保证**。

一句话：**一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式事务问题**。

分布式事务的产生：跨JVM进程，跨数据库实例

```xml
<!-- seata -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <version>2021.0.1.0</version>
</dependency>
```

![image-20220526215226941](img.assets\image-20220526215226941.png)

**若使用的版本于seata的版本不一致,请去掉该依赖，使用自己的seata版本**

```xml
        <!-- seata -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>seata-all</artifactId>
                    <groupId>io.seata</groupId>
                </exclusion>
            </exclusions>
            <version>2021.0.1.0</version>
        </dependency>
        
        <dependency>
            <groupId>io.seata</groupId>
            <artifactId>seata-all</artifactId>
            <version>1.4.2</version>
        </dependency>
```



## 1.分布式事务

分布式事务处理过程的一ID+三组件模型：

- Transaction ID XID 全局唯一的事务ID
- 三组件概念
  - TC (Transaction Coordinator) - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
  - TM (Transaction Manager) - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
  - RM (Resource Manager) - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

处理过程：

1. TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；
2. XID在微服务调用链路的上下文中传播；
3. RM向TC注册分支事务，将其纳入XID对应全局事务的管辖；
4. TM向TC发起针对XID的全局提交或回滚决议；
5. TC调度XID下管辖的全部分支事务完成提交或回滚请求。



![img](img.assets\2d2c6aa29c3158413f66d4ef8c1000dc.png)



## 2.Seata配置

[下载](https://github.com/seata/seata/releases)

### 2.1 在nacos配置seata

#### 2.1.1 新建命名空间

![image-20220526195549657](img.assets\image-20220526195549657.png)

#### 2.1.2 创建dataID，分组和配置文件

![image-20220526200542832](img.assets\image-20220526200542832.png)

使用数据库文件:

```properties
service.vgroupMapping.my_test_tx_group=default
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true
store.db.user=root
store.db.password=root
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
```

全部文件:

```properties
transport.type=TCP
transport.server=NIO
transport.heartbeat=true
transport.enableClientBatchSendRequest=true
transport.threadFactory.bossThreadPrefix=NettyBoss
transport.threadFactory.workerThreadPrefix=NettyServerNIOWorker
transport.threadFactory.serverExecutorThreadPrefix=NettyServerBizHandler
transport.threadFactory.shareBossWorker=false
transport.threadFactory.clientSelectorThreadPrefix=NettyClientSelector
transport.threadFactory.clientSelectorThreadSize=1
transport.threadFactory.clientWorkerThreadPrefix=NettyClientWorkerThread
transport.threadFactory.bossThreadSize=1
transport.threadFactory.workerThreadSize=default
transport.shutdown.wait=3
transport.serialization=seata
transport.compressor=none
# server
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
server.distributedLockExpireTime=10000
# store
#model改为db
store.mode=db
store.lock.mode=file
store.session.mode=file
# store.publicKey=""
store.file.dir=file_store/data
store.file.maxBranchSessionSize=16384
store.file.maxGlobalSessionSize=512
store.file.fileWriteBufferCacheSize=16384
store.file.flushDiskMode=async
store.file.sessionReloadReadSize=100

store.db.datasource=druid
store.db.dbType=mysql
#修改数据驱动，这里是mysql5，使用mysql8的话请修改
store.db.driverClassName=com.mysql.jdbc.Driver
# 改为上面创建的seata服务数据库
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true&rewriteBatchedStatements=true
# 改为自己的数据库用户名
store.db.user=root
# 改为自己的数据库密码
store.db.password=root
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.distributedLockTable=distributed_lock
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000

store.redis.mode=single
store.redis.single.host=127.0.0.1
store.redis.single.port=6379
store.redis.password="13883981813xH"
# store.redis.sentinel.masterName=""
# store.redis.sentinel.sentinelHosts=""
store.redis.maxConn=10
store.redis.minConn=1
store.redis.maxTotal=100
store.redis.database=0
store.redis.queryLimit=100
# log
log.exceptionRate=100
# metrics
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
# service
# 自己命名一个vgroupMapping
service.vgroupMapping.fsp_tx_group=default
service.default.grouplist=127.0.0.1:8091
service.enableDegrade=false
service.disableGlobalTransaction=false
# client
client.rm.asyncCommitBufferLimit=10000
client.rm.lock.retryInterval=10
client.rm.lock.retryTimes=30
client.rm.lock.retryPolicyBranchRollbackOnConflict=true
client.rm.reportRetryCount=5
client.rm.tableMetaCheckEnable=false
client.rm.tableMetaCheckerInterval=60000
client.rm.sqlParserType=druid
client.rm.reportSuccessEnable=false
client.rm.sagaBranchRegisterEnable=false
client.rm.tccActionInterceptorOrder=-2147482648
client.tm.commitRetryCount=5
client.tm.rollbackRetryCount=5
client.tm.defaultGlobalTransactionTimeout=60000
client.tm.degradeCheck=false
client.tm.degradeCheckAllowTimes=10
client.tm.degradeCheckPeriod=2000
client.tm.interceptorOrder=-2147482648
client.undo.dataValidation=true
client.undo.logSerialization=jackson
client.undo.onlyCareUpdateColumns=true
client.undo.logTable=undo_log
client.undo.compress.enable=true
client.undo.compress.type=zip
client.undo.compress.threshold=64k
```



### 2.2 修改registry.conf文件

解压后修改conf目录下的**registry.conf**文件，先对文件进行备份

注册方式有 file 、nacos 、eureka、redis、zk、consul、etcd3、sofa。

我们这次选择注册方式为nacos

- 修改注册类型为nacos
- 修改nacos的服务地址、命名空间、用户名和密码

```shell
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "SEATA_GROUP"
    #去nacos注册中心创建命名空间
    namespace = "bb38df6c-8a91-4648-adef-317f0cf5a73f"
    cluster = "default"
    username = "nacos"
    password = "nacos"
  	}
  	
  ---
 
  config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = "bb38df6c-8a91-4648-adef-317f0cf5a73f"
    group = "SEATA_GROUP"
    username = "nacos"
    password = "nacos"
    dataId = "seataServer.properties"
  	}
  }
}
```



### 2.3 修改file.conf文件

seata事物日志的存储方式：file、db、redis。file只适合单节点模式

这次我们先选择DB数据库模式

- 修改mode为DB模式
- 修改DB模块的配置：
  - mysql驱动，我的测试环境是5.0
  - mysql地址、用户名、密码
  - 注意：服务文件目录中含有mysql 5.X和8.X的两个数据库连接驱动。目录位置：seata-server-1.4.2\lib\jdbc。最好是把里面不使用的驱动移除或备份。

```shell
store {
  ## store mode: file、db、redis
  #修改数据库
  mode = "db"
  ## rsa decryption public key
  publicKey = ""
  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    ## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
    url = "jdbc:mysql://127.0.0.1:3306/seata?rewriteBatchedStatements=true"
    user = "root"
    password = "root"
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
}
```

### 2.4.创建seata数据库

```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`       CHAR(20) NOT NULL,
    `lock_value`     VARCHAR(20) NOT NULL,
    `expire`         BIGINT,
    primary key (`lock_key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('AsyncCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryRollbacking', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('TxTimeoutCheck', ' ', 0);
```

### 2.5 启动seata

在bin目录下seata-server.sh启动

```shell
sh seata-server.sh
```

![image-20220526202627390](img.assets\image-20220526202627390.png)

![image-20220526202739183](img.assets\image-20220526202739183.png)



## 3.Seata业务数据库准备

![img](img.assets\302377d33ddcd708e20b996bd9f2c7b8.png)

这里我们会创建三个服务，一个订单服务，一个库存服务，一个账户服务。

当用户下单时,会在订单服务中创建一个订单, 然后通过远程调用库存服务来扣减下单商品的库存，再通过远程调用账户服务来扣减用户账户里面的余额，最后在订单服务中修改订单状态为已完成。

该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题。

**一言蔽之**，下订单—>扣库存—>减账户(余额)。



### 3.1 创建业务数据库

- seata_ order：存储订单的数据库;
- seata_ storage：存储库存的数据库;
- seata_ account：存储账户信息的数据库。

```sql
CREATE DATABASE seata_order;
CREATE DATABASE seata_storage;
CREATE DATABASE seata_account;
```

按照上述3库分别建对应业务表

- seata_order库下建t_order表

```sql
CREATE TABLE t_order (
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `count` INT(11) DEFAULT NULL COMMENT '数量',
    `money` DECIMAL(11,0) DEFAULT NULL COMMENT '金额',
    `status` INT(1) DEFAULT NULL COMMENT '订单状态: 0:创建中; 1:已完结'
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

SELECT * FROM t_order;
```

- seata_storage库下建t_storage表

```sql
CREATE TABLE t_storage (
`id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
`product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
`total` INT(11) DEFAULT NULL COMMENT '总库存',
`used` INT(11) DEFAULT NULL COMMENT '已用库存',
`residue` INT(11) DEFAULT NULL COMMENT '剩余库存'
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO seata_storage.t_storage(`id`, `product_id`, `total`, `used`, `residue`)
VALUES ('1', '1', '100', '0','100');

SELECT * FROM t_storage;
```

- seata_account库下建t_account表

```sql
CREATE TABLE t_account(
	`id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT 'id',
	`user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
	`total` DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
	`used` DECIMAL(10,0) DEFAULT NULL COMMENT '已用余额',
	`residue` DECIMAL(10,0) DEFAULT '0' COMMENT '剩余可用额度'
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO seata_account.t_account(`id`, `user_id`, `total`, `used`, `residue`)
VALUES ('1', '1', '1000', '0', '1000');

SELECT * FROM t_account;
```

按照上述3库分别建对应的回滚日志表

- 订单-库存-账户3个库下**都需要建各自的回滚日志表**
- 建表SQL

```sql
-- the table to store seata xid data
-- 0.7.0+ add context
-- you must to init this sql for you business databese. the seata server not need it.
-- 此脚本必须初始化在你当前的业务数据库中，用于AT 模式XID记录。与server端无关（注：业务数据库）
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
drop table `undo_log`;
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```



## 4.Seata使用

下订单 -> 减库存 -> 扣余额 -> 改（订单）状态

新建module  seata-order-service200,seata- storage - service2002,seata- account- service2003

pom:

```xml
        <!--nacos-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <!-- seata -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
        </dependency>

        <!--feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <!--Spring Cloud 2021.0.1版本 loadbalancer 3.1.1-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>


        <!--web-actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!-- druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>

        <!--mybatis-plus起步依赖-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
```

yml:

```yml
server:
  port: 2001

spring:
  application:
    name: seata-order-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.153.136:3306/seata_order
    username: root
    password: root
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.153.136

#seata1.4.2配置
seata:
  #事务群组（可以每个应用独立取名，也可以使用相同的名字），
  #要与服务端nacos-config.txt中service.vgroup_mapping中存在,并且要保证多个群组情况下后缀名要保持一致-tx_group
  enabled: true
  enable-auto-data-source-proxy: true #是否开启数据源自动代理,默认为true
  tx-service-group: my_test_tx_group  #要与nacos配置文件中的vgroupMapping一致,在nacos配置中心查看
  registry: #registry根据seata服务端的registry配置
    type: nacos #默认为file
    nacos:
      application: seata-server #配置自己的seata服务
      server-addr: ${spring.cloud.nacos.discovery.server-addr} #根据自己的seata服务配置
      username: nacos #根据自己的seata服务配置
      password: nacos #根据自己的seata服务配置
      cluster: default # 配置自己的seata服务cluster, 默认为 default
      group: SEATA_GROUP #根据自己的seata服务配置
      namespace: 3e921bf9-183b-49d7-8f94-c959c11badb0 #改为自己的nacos的namespace,这里填写的是刚才创建seata命名空间的id
    config:
      type: nacos #默认file,如果使用file不配置下面的nacos,直接配置seata.service
      nacos:
        server-addr: ${spring.cloud.nacos.discovery.server-addr} #配置自己的nacos地址
        group: SEATA_GROUP #配置自己的dev
        username: nacos #配置自己的username
        password: nacos #配置自己的password
        dataId: seataServer.properties # #配置自己的dataId,由于搭建服务端时把客户端的配置也写在了seataServer.properties,所以这里用了和服务端一样的配置文件,实际客户端和服务端的配置文件分离出来更好
        namespace: 3e921bf9-183b-49d7-8f94-c959c11badb0 #改为自己的nacos的namespace,这里填写的是刚才创建seata命名空间的id


feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000

mybatis-plus:
  mapper-locations:
    - classpath:mapper/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  #设置表的前缀
  global-config:
    db-config:
      table-prefix: t_
```

主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
@MapperScan("com.xh.seata.dao")
//seata1.1版本后使用注解开启DataSourceProxy,
//之前版本需要手动写DataSourceProxy配置使seata管理数据库
@EnableAutoDataSourceProxy
public class Order2001 {
    public static void main(String[] args) {
        SpringApplication.run(Order2001.class);
    }
}
```

service

```java
	@Resource
    private OrderDao orderDao;

    @Resource
    private StorageFeignService storageFeignService;

    @Resource
    private AccountFeignService accountFeignService;

	@Override
    public void creat(Order order) {

        log.info("------->开始创建订单");
        orderDao.insert(order);

        log.info("------->订单微服务开始调用库存，做扣减");
        ResultJSON resultStorage = storageFeignService.deduct(order.getProductId(), order.getCount());
        if (!resultStorage.isSuccess()){
            log.info("减少库存失败");
            return;
        }
        log.info("支付微服务开始调用，开始减余额");
        ResultJSON resultAccount = accountFeignService.payDeduct(order.getUserId(),
                BigDecimal.valueOf(order.getMoney()));
        if(!resultAccount.isSuccess()){
            log.info("支付失败");
            return;
        }
        log.info("修改订单的状态");
        update(order.getUserId(),0);
        log.info("新建订单结束");
    }
```

AccountFeignService发起远程调用

```java
@FeignClient(value = "seata-account-service",path = "/account")
public interface AccountFeignService {

    /**
     * 调用支付接口减少余额
     * @param userId 用户的id
     * @param money 支付金额
     * @return json
     */
    @PostMapping("/deduct")
    //在fegin调用时一定要加上@RequestParam注解或者@RequestBody注解.@RequestBody注解只能有一个
    ResultJSON payDeduct(@RequestParam("userId") Long userId, @RequestParam("money") BigDecimal money);
}
```

 StorageFeignService  发起远程调用

```java
@FeignClient(value = "seata-storage-service",path = "/storage")
public interface StorageFeignService {

    /**
     * 调用减少库存的方法
     * @param productId 产品id
     * @param count 减少的数量
     * @return json
     */
    @PostMapping("/deduct")
    ResultJSON deduct(@RequestParam("productId") Long productId,@RequestParam("count") Integer count);
}
```

测试，结果正常



### 4.1 @GlobalTransactional注解

**在业务方法上添加@GlobalTransactional注解即可对全局事务进行回滚**

```json
 	//进行全局事务回滚
    @GlobalTransactional
    public void creat(Order order) {

        log.info("------->开始创建订单");
        order.setStatus(0);
        orderDao.insert(order);

        log.info("------->订单微服务开始调用库存，做扣减");
        ResultJSON resultStorage = storageFeignService.deduct(order.getProductId(), order.getCount());
        if (!resultStorage.isSuccess()){
            log.info("减少库存失败");
            return;
        }
        log.info("支付微服务开始调用，开始减余额");
        ResultJSON resultAccount = accountFeignService.payDeduct(order.getUserId(),
                BigDecimal.valueOf(order.getMoney()));
        if(!resultAccount.isSuccess()){
            log.info("支付失败");
            return;
        }
        log.info("修改订单的状态");
        update(order.getUserId(),0);
        log.info("新建订单结束");
    }

```



回滚日志：

![image-20220527113427650](img.assets\image-20220527113427650.png)



## 5. Seata事务原理

[官网说明](https://seata.io/zh-cn/docs/overview/what-is-seata.html)

![img](img.assets\2d2c6aa29c3158413f66d4ef8c1000dc.png)

- TM开启分布式事务(TM向TC注册全局事务记录) ;
- 按业务场景，编排数据库、服务等事务内资源(RM向TC汇报资源准备状态) ;
- TM结束分布式事务，事务一阶段结束(TM通知TC提交/回滚分布式事务) ;
- TC汇总事务信息，决定分布式事务是提交还是回滚；
- TC通知所有RM提交/回滚资源，事务二阶段结束。



### 5.1 AT模式

​	Seata的优势在于提供了多种事务模式，用户可以根据需要进行选择，Seata目前复合主键仅支持mysql，如果项目使用mysql数据库，AT模式基本能满足你的全部需求，seata配置好后，在接口方法入口处加上@GlobalTransactional注解，**Seata会自动完成一阶段SQL解析、执行、将日志记录插入到UNDO_log表，二阶段提交/回滚等操作**，无需自己代码实现，基本能实现对原本业务代码零侵入。



**前提**

- 基于支持本地 ACID 事务的关系型数据库。
- Java 应用，通过 JDBC 访问数据库。

**整体机制**

两阶段提交协议的演变：

- 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
- 二阶段：
  - 提交异步化，非常快速地完成。
  - 回滚通过一阶段的回滚日志进行反向补偿。



#### 5.1.1 一阶段加载

在一阶段，Seata会拦截“业务SQL” ---AOP

1. 解析SQL语义，找到“业务SQL" 要更新的业务数据，**在业务数据被更新前，将其保存成"before image”**
2. 执行“业务SQL" 更新业务数据，在业务数据更新之后,
3. 其保存成"after image”，最后**生成行锁**。

以上操作全部在一个数据库事务内完成, 这样保证了一阶段操作的原子性。

![img](img.assets\80a7bd6cacef78392b278af04d446562.png)

#### 5.1.2 二阶段提交

二阶段如果顺利提交的话，因为"业务SQL"在一阶段已经提交至数据库，所以Seata框架只需将一阶段**保存的快照数据和行锁删掉，完成数据清理即可。**

![img](img.assets\a16483118166481bd7f9d06f91a28146.png)

#### 5.1.3 二阶段回滚

二阶段如果是回滚的话，Seata 就需要回滚一阶段已经执行的 “业务SQL"，还原业务数据。

回滚方式便是**用"before image"还原业务数据**；但在还原前要**首先要校验脏写，对比“数据库当前业务数据”和"after image"**。

如果两份数据完全一致就说明没有脏写， 可以还原业务数据，如果不一致就说明有脏写, 出现脏写就需要转人工处理。

![img](img.assets\828b79e4c7679ce5f09069e551c2a717.png)



通过debug的方式观察表的数据:

**gload_table:表**：记录了发起全局事务的方法，xid等数据

xid由 ip:端口:事务id组成

![image-20220527121102726](img.assets\image-20220527121102726.png)

**branch_table表:**记录了全局事务TC管理的事务分支

![image-20220527121249503](img.assets\image-20220527121249503.png)

**lock_table表:**在保存after image之后生成的行锁

![image-20220527121509535](img.assets\image-20220527121509535.png)

**undo_log表:**保存了在sql语句执行前(before image)和执行后(after image)的数据，**用来进行提交，回滚**

![image-20220527121618309](img.assets\image-20220527121618309.png)

```json
{
	"branchId": 641789253,
	"undoItems": [{
		"afterImage": {
			"rows": [{
				"fields": [{
					"name": "id",
					"type": 4,
					"value": 1
				}, {
					"name": "name",
					"type": 12,
					"value": "GTS"
				}, {
					"name": "since",
					"type": 12,
					"value": "2014"
				}]
			}],
			"tableName": "product"
		},
		"beforeImage": {
			"rows": [{
				"fields": [{
					"name": "id",
					"type": 4,
					"value": 1
				}, {
					"name": "name",
					"type": 12,
					"value": "TXC"
				}, {
					"name": "since",
					"type": 12,
					"value": "2014"
				}]
			}],
			"tableName": "product"
		},
		"sqlType": "UPDATE"
	}],
	"xid": "xid:xxx"
}
```



![img](img.assets\21da4fdc4260008c3324574abc33f0ae.png)



## 6. 雪花算法

### 6.1 分布式全局唯一ID

**为什么需要分布式全局唯一ID以及分布式ID的业务需求？集群高并发情况下如何保证分布式唯一全局Id生成？**

在复杂分布式系统中，往往需要对大量的数据和消息进行唯一标识，如在美团点评的金融、支付、餐饮、酒店，猫眼电影等产品的系统中数据日渐增长，对数据分库分表后需要有一个唯一ID来标识一条数据或消息。特别一点的如订单、骑手、优惠券也都雷要有唯一ID做标识。此时一个能够生成全局唯一ID的系统是非常必要的。

**ID生成规则部分硬性要求**

- *全局唯一*：不能出现重复的ID号，既然是唯一标识，这是最基本的要求
- *趋势递增*：在MySQL的InnoDB引擎中使用的是聚集索引，由于多数RDBMS使用Btree的数据结构来存储索引数据，在主键的选择上面我们应该尽量使用有序的主键保证写入性能。
- *单调递增*：保证下一个ID一定大于上一个ID，例如事务版本号、IM增量消息、排序等特殊需求
- *信息安全*：如果ID是连续的，恶意用户的扒取工作就非常容易做了，直接按照顺序下载指定URL即可。如果是订单号就更危险了，竞对可以直接知道我们一天的单量。所以在一些应用场景下，需要ID无规则不规则，让竞争对手否好猜。
- *含时间戳*：这样就能够在开发中快速了解这个分布式id的生成时间。

**ID号生成系统的可用性要求**

- *高可用*：发一个获取分布式ID的请求，服务器就要保证99.999%的情况下给我创建一个唯一分布式ID。
- *低延迟*：发一个获取分布式ID的请求，服务器就要快，极速。
- *高QPS*：假如并发一口气10万个创建分布式ID请求同时杀过来，服务器要顶的住且一下子成功创建10万个分布式ID



### 6.2 UUID的缺点

UUID(Universally Unique ldentifer)的标准型式包含32个16进制数字，以连了号分为五段，形式为8-4-4-4-12的36个字符， 

示例：550e8400-e29b-41d4-a716-446655440000

性能非常高：本地生成，没有网络消耗

如果只是考虑唯一性，那就选用它吧

但是，**入数据库性能差**

**为什么无序的UUID会导致入库性能变差呢？**

1. 无序，无法预测他的生成顺序，不能生成递增有序的数字。首先分布式ID一般都会作为主键， 但是安装MySQL官方推荐主键要尽量越短越好，UUID每一个都很长，所以不是很推荐。
2. 主键，ID作为主键时在特定的环境会存在一些问题。比如做DB主键的场景下，UUID就非常不适用MySQL官方有明确的建议主键要尽量越短越好36个字符长度的UUID不符合要求。
3. 索引，既然分布式ID是主键，然后主键是包含索引的，然后MySQL的索引是通过B+树来实现的，每一次新的UUID数据的插入，为了查询的优化，都会对索引底层的B+树进行修改，因为UUID数据是无序的，所以**每一次UUID数据的插入都会对主键地械的B+树进行很大的修改**，这一点很不好。 插入完全无序，不但会导致一些中间节点产生分裂，也会白白创造出很多不饱和的节点，这样大大降低了数据库插入的性能。



### 6.3 数据库自增主键

在单机里面，数据库的自增ID机制的主要原理是：数据库**自增ID**和MySQL数据库的**replace into**实现的。

REPLACE INTO的含义是插入一条记录，如果表中唯一索引的值遇到冲突，则替换老数据。

这里的replace into跟inset功能类似，不同点在于**：replace into首先尝试插入数据列表中，如果发现表中已经有此行数据（根据主键或唯一索引判断）则先删除，再插入。否则直接插入新数据。**



### 6.4 基于Redis生成全局ID策略

因为Redis是单线的天生保证原子性，可以使用原子操作**INCR和INCRBY**来实现

注意：在Redis集群情况下，同样和MySQL一样需要设置不同的增长步长，同时key一定要设置有效期可以使用Redis集群来获取更高的吞吐量。



### 6.5 Twitter的分布式自增ID算法snowflake

Twitter的分布式雪花算法SnowFlake ，经测试snowflake 每秒能够产生26万个自增可排序的ID

1. Twitter的SnowFlake生成ID能够按照时间有序生成。
2. SnowFlake算法生成ID的结果是一个64bit大小的整数， 为一个Long型（转换成字符串后长度最多19）。
3. 分布式系统内不会产生ID碰撞（由datacenter和workerld作区分）并且效率较高。

分布式系统中，有一些需要使用全局唯一ID的场景， 生成ID的基本要求：

1. 在分布式的环境下必须全局且唯一 。
2. 一般都需要单调递增，因为一般唯一ID都会存到数据库，而**Innodb的特性就是将内容存储在主键索引树上的叶子节点而且是从左往右**，递增的，所以考
   虑到数据库性能，一般生成的ID也最好是单调递增。 为了防止ID冲突可以使用36位的UUID，但是UUID有一些缺点， 首先他相对比较长， 另外UUID一般是无序的。
3. 可能还会需要无规则，因为如果使用唯一ID作为订单号这种，为了不然别人知道一天的订单量是多少，就需要这个规则。



#### 6.5.1 结构

雪花算法的几个核心组成部分：

![img](img.assets\795b3d1fa01bbd15d8b7b85c2724bf42.png)

**1bit：**

不用，因为二进制中最高位是符号位，1表示负数，0表示正数。**生成的id一般都是用整数，所以最高位固定为0。**

**41bit - 时间戳，用来记录时间戳，毫秒级：**

- 41位可以表示2 41 − 1 2^{41}-1241−1个数字。
- 如果只用来表示正整数（计算机中正数包含0），可以表示的数值范围是：0至2 41 − 1 2^{41}-1241−1， **减1是因为可表示的数值范围是从0开始算的，而不是1**。
- 也就是说41位可以表示2 41 − 1 2^{41}-1241−1个毫秒的值，转化成单位年则是( 2 41 − 1 ) / ( 1000 ∗ 60 ∗ 60 ∗ 24 ∗ 365 ) = 69 (2^{41}-1)/ (1000 * 60 * 60 * 24 *365) = 69(241−1)/(1000∗60∗60∗24∗365)=69年(1970-**2039-09-07**)。

**10bit - 工作机器ID，用来记录工作机器ID：**

- 可以部署在2 10 = 1024 2^{10}= 1024210=1024个节点，包括5位DataCenterId和5位Workerld。
- 5位(bit) 可以表示的最大正整数是2 5 − 1 = 31 2^{5}-1=3125−1=31,即可以用0、1、2、3、…31这32个数字，来表示不同的DataCenterld或Workerld。

**12bit - 序列号，用来记录同毫秒内产生的不同id。**

- 12位(bit) 可以表示的最大正整数是2 12 − 1 = 4095 2^{12} - 1 = 4095212−1=4095， 即可以用0、1、2、 3、…4094这4095个数字，来表示同一机器同一时间截**(毫秒)内产生的4095个ID序号**。



SnowFlake可以保证：

- 所有生成的ID按时间趋势递增。
- 整个分布式系统内不会产生重复id（因为有DataCenterId和Workerld来做区分)



#### 6.5.2 雪花算法代码

```java

/**
 * Twitter_Snowflake
 * SnowFlake的结构如下(每部分用-分开):
 * 0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000
 * 1位标识，由于long基本类型在Java中是带符号的，最高位是符号位，正数是0，负数是1，所以id一般是正数，最高位是0
 * 41位时间戳(毫秒级)，注意，41位时间戳不是存储当前时间的时间戳，而是存储时间戳的差值（当前时间戳 - 开始时间戳)
 * 得到的值），这里的的开始时间戳，一般是我们的id生成器开始使用的时间，由我们程序来指定的（如下面程序SnowflakeIdWorker类的startTime属性）。41位的时间戳，可以使用69年，年T = (1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69
 * 10位的数据机器位，可以部署在1024个节点，包括5位datacenterId和5位workerId
 * 12位序列，毫秒内的计数，12位的计数顺序号支持每个节点每毫秒(同一机器，同一时间戳)产生4096个ID序号
 * 加起来刚好64位，为一个Long型。
 */
public class SnowflakeIdWorker {
    /** 开始时间戳 (1970-01-01) */
    private final long twepoch = 1420041600000L;

    /** 机器id所占的位数 */
    private final long workerIdBits = 5L;

    /** 数据标识id所占的位数 */
    private final long datacenterIdBits = 5L;

    /** 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数) */
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);

    /** 支持的最大数据标识id，结果是31 */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);

    /** 序列在id中占的位数 */
    private final long sequenceBits = 12L;

    /** 机器ID向左移12位 */
    private final long workerIdShift = sequenceBits;

    /** 数据标识id向左移17位(12+5) */
    private final long datacenterIdShift = sequenceBits + workerIdBits;

    /** 时间戳向左移22位(5+5+12) */
    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

    /** 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095) */
    private final long sequenceMask = -1L ^ (-1L << sequenceBits);

    /** 工作机器ID(0~31) */
    private long workerId;

    /** 数据中心ID(0~31) */
    private long datacenterId;

    /** 毫秒内序列(0~4095) */
    private long sequence = 0L;

    /** 上次生成ID的时间戳 */
    private long lastTimestamp = -1L;

    //==============================Constructors=====================================
    /**
     * 构造函数
     * @param workerId 工作ID (0~31)
     * @param datacenterId 数据中心ID (0~31)
     */
    public SnowflakeIdWorker(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }

    // ==============================Methods==========================================
    /**
     * 获得下一个ID (该方法是线程安全的)
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();

        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                    String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }

        //如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            //毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        //时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }

        //上次生成ID的时间戳
        lastTimestamp = timestamp;

        //移位并通过或运算拼到一起组成64位的ID
        return ((timestamp - twepoch) << timestampLeftShift) //
                | (datacenterId << datacenterIdShift) //
                | (workerId << workerIdShift) //
                | sequence;
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     * @param lastTimestamp 上次生成ID的时间戳
     * @return 当前时间戳
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    /**
     * 返回以毫秒为单位的当前时间
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }

    /** 测试 */
    public static void main(String[] args) {
        System.out.println("开始："+System.currentTimeMillis());
        SnowflakeIdWorker idWorker = new SnowflakeIdWorker(0, 0);
        for (int i = 0; i < 50; i++) {
            long id = idWorker.nextId();
            System.out.println(id);
//            System.out.println(Long.toBinaryString(id));
        }
        System.out.println("结束："+System.currentTimeMillis());
    }
}
```



#### 6.5.3 使用hutool工具包生成

```xml
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.1.0</version>
        </dependency>
```



### 6.6 优缺点

优点：

毫秒数在高位，自增序列在低位，整个ID都是趋势递增的。

不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的。

可以根据自身业务特性分配bit位，非常灵活。

缺点：

依赖机器时钟，如果机器时钟回拨，会导致重复ID生成。

在单机上是递增的，但是由于设计到分布式环境，每台机器上的时钟不可能完全同步，有时候会出现不是全局递增的情况。

（此缺点可以认为无所谓，一般分布式ID只要求趋势递增，并不会严格要求递增，90%的需求都只要求趋势递增）
