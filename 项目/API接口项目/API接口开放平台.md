# API接口开放平台

## 1.项目说明

一个提供API接口调用的平台，用户可以注册登录，开通接口调用权限。用户可以使用接口，并且每次调用会进行统计。管理员可以发布接口、下线接口、接入接口，以及可视化接口的调用情况、数据。

1.   防止攻击（安全性)
2.   不能随便调用(限制、开通)
3.   统计调用次数
4.   计费
5.   流量保护
6.   API接入

![image-20230508151052452](img.assest\image-20230508151052452.png)

## 2.技术选型

**前端：**

-   Ant Design Pro 

-   React
-   Ant Design Procomponents
-   Umi

-   Umi Request (Axios的封装)

**后端：**

-   Java Spring Boot
-   Spring Boot Starter (SDK开发)
-   Dubbo
-   Nacos
-   Spring Cloud Gateway (网关、限流、日志实现)



## 3.API签名认证

为开发者分配`AccessKey`（开发者标识，确保唯一）和`SecretKey`（用于接口加密，确保不易被穷举，生成算法不易被猜测）。

**怎么实现**

-   **accessKey** 	调用的标识（复杂，无序，无规律）
-   **secretKey**	 密钥 （复杂，无序，无规律）	

类似**用户名**和**密码**，区别：accessKey、secretKey是**无状态**的

千万不能把密钥直接在服务器间进行传递，有可能被拦截

> 在用户表中添加**`accessKey`**和 **`secretKey`**

```mysql
-- 用户表
create table if not exists user
(
    id            int 									 auto_increment comment 'id' primary key,
    username      varchar(256)                           null comment '用户昵称',
    user_account  varchar(256)                           not null comment '账号',
    user_avatar   varchar(1024)                          null comment '用户头像',
    gender        tinyint                                null comment '性别',
    user_role     varchar(256) default 'user'            not null comment '用户角色：user / admin',
    user_password varchar(512)                           not null comment '密码',
    access_key    varchar(512)                           not null comment 'accessKey',
    secret_key    varchar(515)                           not null comment 'secretKey',
    create_time   datetime     default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time   datetime     default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    is_delete     tinyint      default 0                 not null comment '是否删除',
    constraint uni_userAccount unique (user_account)
) comment '用户';
```



### 3.1 保证用户的合法性

客户端：在发送请求时带上`ACCOUNT_KEY`

```java
    private final String ACCOUNT_KEY = "afasdfgnssgs";

    //客户端
    public void get() {
        HashMap<String, String> headers = new HashMap<>();
        headers.put("accountKey", ACCOUNT_KEY);
        HttpResponse response =
        HttpRequest.get("http://localhost:8888/test/get").addHeaders(headers).form("name", "16541").execute();
        String body = response.body();
        System.out.println(body);
    }
```

服务端对拿到`ACCOUNT_KEY`去数据库中查询`SECRET_KEY`



### 3.2 防止请求参数被篡改

一般是根据密钥，生成**签名sign**

**签名的做法**

假如 ，我们有用户参数，我们用密钥与他拼接，用签名算法得到一个不可解密的值

**用户参数 	+	密钥	=>	签名生成算法（MD5,HMac,Sha1) 	=>	不可解密的值**

> 客户端

```java
    private final String ACCOUNT_KEY = "afasdfgnssgs";

    //客户端
    public void get() {
        HashMap<String, String> headers = new HashMap<>();
        headers.put("accountKey", ACCOUNT_KEY);
        //加密
        //使用用户id+accountKey进行加密
        String sign = DigestUtil.md5Hex("1" + ACCOUNT_KEY);
        Map<String, Object> data = new HashMap<>();
        data.put("name", "15494561");
        data.put("sign", sign);
        HttpResponse response =
                HttpRequest.get("http://localhost:8888/test/get").addHeaders(headers).form(data).execute();
        String body = response.body();
        System.out.println(body);
    }
```

> 服务端

```java
@GetMapping("/get")
public String getNameGet(String name, String sign, HttpServletRequest request) {
    //获取accountKey
    String accountKey = request.getHeader("accountKey");
    //TODO 从数据库中获取accountKey和SecretKey进行比较判断是否合法
    //验证sign
    String verifySign = DigestUtil.md5Hex("1" + accountKey);
    if (!sign.equals(verifySign)) {
        throw new RuntimeException("签名验证错误");
    }
    System.out.println(name);
    return "GET:myName is " + name;
}
```

请求携带参数AccessKey和Sign，只有拥有合法的身份AccessKey和正确的签名Sign才能放行。这样就解决了身份验证和参数篡改问题，即使请求参数被劫持，由于获取不到SecretKey（仅作本地加密使用，不参与网络传输），无法伪造合法的请求。



### 3.3 防止重攻击

虽然解决了请求参数被篡改的隐患，但是还存在着重复使用请求参数伪造二次请求的隐患。

**timestamp+nonce方案**

`nonce`指唯一的随机字符串，用来标识每个被签名的请求。通过为每个请求提供一个唯一的标识符，服务器能够防止请求被多次使用（记录所有用过的nonce以阻止它们被二次使用）。

然而，对服务器来说永久存储所有接收到的nonce的代价是非常大的。可以使用timestamp来优化nonce的存储。

假设允许客户端和服务端最多能存在15分钟的时间差，同时追踪记录在服务端的nonce集合。当有新的请求进入时，首先检查携带的timestamp是否在15分钟内，如超出时间范围，则拒绝，然后查询携带的nonce，如存在已有集合，则拒绝。否则，记录该nonce，并删除集合内时间戳大于15分钟的nonce（可以使用redis的expire，新增nonce的同时设置它的超时失效时间为15分钟）。

> 客户端

```java
    private final String ACCOUNT_KEY = "afasdfgnssgs";

    //客户端
    public void get() {
        HashMap<String, String> headers = new HashMap<>();
        headers.put("accountKey", ACCOUNT_KEY);
        //添加唯一随机字符串+时间防止请求重放攻击
        String nonce = RandomUtil.randomString(5);
        //获取当前时间戳--规定时间在30秒
        long timestamp = System.currentTimeMillis() / 30000;
        //加密
        //使用用户id+accountKey+时间戳+nonce 进行加密
        String sign = DigestUtil.md5Hex("1" + ACCOUNT_KEY + timestamp + timestamp + nonce);
        Map<String, Object> data = new HashMap<>();
        data.put("name", "15494561");
        data.put("sign", sign);
        data.put("nance", nonce);
        data.put("timestamp", timestamp);
        HttpResponse response =
                HttpRequest.get("http://localhost:8888/test/get").addHeaders(headers).form(data).execute();
        String body = response.body();
        System.out.println(body);
    }
```

> 服务端

```java
    @GetMapping("/get")
    public String getNameGet(String name, String sign, String nonce, long timestamp, HttpServletRequest request) {
        //获取accountKey
        String accountKey = request.getHeader("accountKey");
        //TODO 从数据库中获取accountKey和SecretKey进行比较判断是否合法
        //验证sign
        String verifySign = DigestUtil.md5Hex("1" + accountKey + timestamp + nonce);
        if (!sign.equals(verifySign)) {
            throw new RuntimeException("签名验证错误");
        }
        //TODO 从redis中判断nonce是否存在，如果存在,则拒绝;如果发送时间超过规定时间(5分钟),则拒绝
        //五分钟内的请求有效
        if (System.currentTimeMillis() - timestamp > 5 * 60 * 1000) {
            return "请求过期";
        }
        //TODO 将nance存入到Redis中，并且设置过期时间为5分钟，使用Set进行存储
        System.out.println(name);
        return "GET:myName is " + name;
    }
```



### 3.4 总结

综上所属

**传递的参数**

1.  accessKey
2.  sign （由accessKey(或者使用用户请求参数body等)、secretKey加密而来）
3.  nonce随机数
4.  timestamp
5.  body（用户请求参数 可要可不要）

**API签名认证是一个很灵活的设计，具体要有哪些参数，尽量服务端调用，参数名如何要根据场景来。**



## 4.开发签名认证SDK（starter）

```
理想情况：开发者只需要关心调用哪些接口、传递哪些参数。就跟调用自己写的代码一样简单。
```

> 开发starter的好处：开发者引入之后，可以直接在application.yml中写配置，自动创建客户端

### 4.1 创建项目

- 创建一个client-sdk的springboot项目 

- 勾选lombok、`Spring Configuration Processor`（作用：自动生成配置的代码提示）

- 然后处理pom.xml <build></build>这个<font color='red'>一定需要删除</font>因为这个是maven的构建项目成可运行jar包。现在是制作starter依赖包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.4</version>
        <relativePath/>
    </parent>
    
    <groupId>com.xh</groupId>
    <artifactId>client-sdk</artifactId>
    <version>0.0.1</version>
    <name>client-sdk</name>
    <description>client-sdk</description>
    
    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
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
</project>
```



### 4.2 删除主启动类

### 4.3 编写配置类

```java
package com.xh.clientSdk;

import com.xh.clientSdk.client.MyClient;
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties("xh.api.client")
@Data
@ComponentScan
public class XhClientConfig {

    private String accountKey;

    @Bean
    public MyClient myClient() {
        return new MyClient(accountKey);
    }
}
```



### 4.4 编写签名类MyClient

```java
package com.xh.clientSdk.client;

import cn.hutool.core.util.RandomUtil;
import cn.hutool.crypto.digest.DigestUtil;
import cn.hutool.http.HttpRequest;
import cn.hutool.http.HttpResponse;
import cn.hutool.http.HttpUtil;

import java.util.HashMap;
import java.util.Map;

public class MyClient {

    private final String ACCOUNT_KEY;

    public MyClient(String accountKey) {
        ACCOUNT_KEY = accountKey;
    }

    //客户端
    public void get() {
        HashMap<String, String> headers = new HashMap<>();
        headers.put("accountKey", ACCOUNT_KEY);
        //添加唯一随机字符串+时间防止请求重放攻击
        String nonce = RandomUtil.randomString(5);
        //获取当前时间戳
        long timestamp = System.currentTimeMillis();
        //加密
        //使用用户id+accountKey+时间戳 进行加密
        String sign = DigestUtil.md5Hex("1" + ACCOUNT_KEY + timestamp);
        Map<String, Object> data = new HashMap<>();
        data.put("name", "15494561");
        data.put("sign", sign);
        data.put("nonce", nonce);
        data.put("timestamp", timestamp);
        HttpResponse response =
                HttpRequest.get("http://localhost:8888/test/get").addHeaders(headers).form(data).execute();
        String body = response.body();
        System.out.println(body);
    }
}
```



### 4.5 创建spring.factories

新建resources/META-INF/spring.factories并指定

```properties
#spring boot stater
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.xh.clientSdk.XhClientConfig
```

![image-20230509212953830](img.assest\image-20230509212953830.png)



### 4.6 使用maven发布starter

**双击Maven lifecycle下的install或者命令行mvn install**



删除测试或者禁止maven测试

![image-20230509213257121](img.assest\image-20230509213257121.png)



### 4.7 引入starter

```xml
        <dependency>
            <groupId>com.xh</groupId>
            <artifactId>client-sdk</artifactId>
            <version>0.0.1</version>
        </dependency>
```

![image-20230509213733524](img.assest\image-20230509213733524.png)



### 4.8 测试

```java
@SpringBootTest
public class SDKTest {

    @Resource
    private MyClient myClient;

    @Test
    public void sdkTest() {
        myClient.get();
    }
}
```



## 5.在线调用

这里我们其实有两种方案

-   走后端调用
-   直接请求模拟接口

![image-20230510093633639](img.assest\image-20230510093633639.png)

这里用第一种流方案，更安全更规范。模拟接口的地址就不用暴露出来

大概流程如下

1.  前端将用户输入的请求参数和要测试的接口 id发给平台后端

2.  调用前校验

3.  平台后端去调用模拟接口



## 6.接口调用次数统计

**需求**

1. 用户每次调用接口成功，次数+1
2. 给用户分配或者用户自主申请调用次数

**业务流程**

1. 用户调用接口（之前已完成）
2. 修改数据库，调用次数+1

> 问题

如果每个接口的方法都写调用次数+1，是不是比较麻烦？

致命问题：接口开发者需要自己去添加统计代码

就想到可以使用**AOP、网关**

![image-20230510102852058](img.assest\image-20230510102852058.png)

**AOP切面的优点**：独立于接口，在每个接口调用后统计次数+1
**AOP切面的缺点**：只存在于单个项目中，如果每个团队都要开发自己的模拟接口，那么都要写一个切面



## 7.网关

> 作用

1. 路由
2. 负载均衡
3. 统一鉴权
4. 统一处理跨域
5. 统一业务处理（缓存）
6. 访问控制
7. 发布控制
8. 流量染色
9. 统一接口保护
   1. 限制请求
   2. 信息脱敏
   3. 降级（熔断）
   4. 限流 学习令牌桶算法，学习露桶算法，学习一下RedislimitHandler
   5. 超时时间
   6. 重试（业务保护）
10. 统一日志
11. 统一文档

> 网关的分类

**网关的分类**

-   **全局网关（接入层网关）**作用是负载均衡、请求日志等，不和业务逻辑绑定
-   **业务网关（微服务网关）**会有一些业务逻辑，作用是将请求转发到不同的业务/项目/接口/服务



**实现**

1. **Nginx** （全局网关），**Kong网关**（API网关），  **编程成本相对较高**
2. **Spring Cloud Gateway**（取代了Zuul）性能高 可以用java代码来写逻辑  适于学习



> 使用网关来进行API签名认证

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: http://localhost:8888
          predicates:
            - Path=/test/**
          filters:
            - AddRequestHeader=X-Request-red, blue
```



```java
@Slf4j
@Component
public class GateWayFilter implements GlobalFilter {

    private final long EXPIRE_1_MIN = 1000 * 1;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        ServerHttpResponse response = exchange.getResponse();
        log.info("请求唯一标识：{}", request.getId());
        log.info("请求目标路径：{}", request.getPath().value());
        log.info("请求方法：{}", request.getMethodValue());
        log.info("请求参数：{}", request.getQueryParams());
        log.info("请求来源地址：{}", request.getLocalAddress());

        MultiValueMap<String, String> queryParams = request.getQueryParams();
        log.info(String.valueOf(queryParams));
        String accountKey = queryParams.getFirst("accountKey");
        // TODO 远程调用获取用户的ak和sk，判断是否相等
        if (!"123456789".equals(accountKey)) {
            log.warn("用户验证错误");
            return this.responseError(response);
        }
        String nonce = queryParams.getFirst("nonce");
        String timestamp = queryParams.getFirst("timestamp");
        // TODO 远程调用查看nonce在redis中是否存在
        if (Objects.isNull(timestamp) || timestamp.length() == 0 || System.currentTimeMillis() - Long.parseLong(timestamp) > EXPIRE_1_MIN) {
            log.warn("请求时间超时");
            return this.responseError(response);
        }
        // TODO 生成sign和传递过来的参数进行匹对
        String sign = queryParams.getFirst("sign");
        //生成sign
        StringBuilder stringBuilder = new StringBuilder();
        queryParams.forEach((k, v) -> {
            stringBuilder.append(v);
        });
        String sign0 = DigestUtil.md5Hex(stringBuilder.toString());
        if (sign0.equals(sign)) {
            log.warn("签名错误");
            return this.responseError(response);
        }
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            HttpStatus responseStatus = exchange.getResponse().getStatusCode();
            // TODO 请求成功和失败的处理
            if (responseStatus != null && responseStatus.is2xxSuccessful()) {
                // 转发成功处理逻辑
                System.out.println(responseStatus.value());
                System.out.println("Request forwarded successfully. Status code: " + responseStatus.value());
                // TODO 远程调用接口次数+1
            } else {
                // 转发失败处理逻辑
                System.out.println("Request forwarding failed. Status code: " + responseStatus.value());
            }
        }));
    }

    public Mono<Void> responseError(ServerHttpResponse response) {
        response.setRawStatusCode(5000);
        return response.setComplete();
    }
}

```



## 8.RPC远程调用

使用dubbo+zookeeper进行远程调用

> 添加依赖

```xml
        <!-- dubbo -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper-curator5</artifactId>
            <version>3.2.0</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-reload4j</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

> yml

```yml
dubbo:
  application:
    name: api-backstage
  protocol:
    name: dubbo
    port: -1
  registry:
    address: zookeeper://localhost:2181
```

> 在消费端和服务端端启动类上添加@EnableDubbo注解

```java
@SpringBootApplication
@EnableDubbo
public class ApiBackStageApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiBackStageApplication.class);
    }
}
```

>编写远程调用接口

```java
public interface UserSk {
    /**
     * 获取用户的sk
     *
     * @param accountKey 用户ak
     * @param userId     用户id
     * @return 如果存在sk, 返回true
     */
    Boolean getUserSk(String accountKey, String userId);

    /**
     * 接口调用次数+1
     *
     * @param userId          用户id
     * @param interfaceInfoId 接口信息id
     * @return 调用的次数
     */
    int addInterfaceNum(String userId, String interfaceInfoId);
}

```

> 在服务提供端实现接口，并且添加@DubboService注解

```java
//暴露服务
@DubboService
public class UserSkImpl implements UserSk {

    private int count;

    @Override
    public Boolean getUserSk(String accountKey, String userId) {
        //TODO 在数据库中查询用户的sk
        System.out.println("用户的accountKey:" + accountKey + "---用户id:" + userId);
        if ("123456789".equals(accountKey)) {
            return Boolean.TRUE;
        }
        return Boolean.FALSE;
    }

    @Override
    public int addInterfaceNum(String userId, String interfaceInfoId) {
        System.out.println("接口次数+1,次数:" + (++count));
        return count;
    }
}
```

> 在客户端进行调用，并且添加@DubboReference注解

```java
    @DubboReference
    private UserSk userSk;
```

