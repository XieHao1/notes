```xml
<!--  springbootJAVA工程起步依赖    -->
<dependency>
      <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
</dependency>

<!-- SpringbootWeb项目起步依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Springboot框架测试起步依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!--springboot-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-aop</artifactId>
</dependency>

<!--springboot集成thymeleaf起步依赖-->
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
 </dependency>

<!--引入jwt-->
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>java-jwt</artifactId>
  <version>3.4.0</version>
</dependency>

<!-- 添加mybatis整合springboot框架的起步依赖 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
 <!--mybatis-plus起步依赖-->
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>3.4.3.4</version>
</dependency>

<!-- springboot集成druid起步依赖  -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.6</version>
</dependency>

<!-- Springboot-redis依赖 -->
<dependency>
 <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--logback日志-->
<dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.20</version>
      <scope>provided</scope>
</dependency>

 <!-- springboot热部署 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <version>2.5.6</version>
</dependency>

 <!--rabbitMQ起步依赖-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<!--rabbitMQ测试依赖   -->
<dependency>
  <groupId>org.springframework.amqp</groupId>
  <artifactId>spring-rabbit-test</artifactId>
  <scope>test</scope>
</dependency>

<!-- swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!-- swagger2可视化界面 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>

<!-- spring -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

```xml
<!--引入Springboot内嵌Tomcat对jsp的解析依赖，不添加解析不了 -->
<!--springboot默认使用的前端引擎为thymeleaf-->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>

<!-- Springboot集成jsp编译的路径是springboot规定好的位置
     META-INF/resources-->
 <resources>
     <resource>
         <!--源文件夹 -->
        <directory>src/main/webapp</directory>
          <!-- 指定编译到META-INF/resources  -->
          <targetPath>META-INF/resources</targetPath>
                <!-- 指定源文件夹中的哪个资源要进行编译-->
      <includes>
         <include>*.*</include>
       </includes>
     </resource>
</resources>

#配置视图解析器
spring.mvc.view.prefix=/
spring.mvc.view.suffix=.jsp
```

```xml
<!-- 使用servlet，要添加的依赖   -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.0</version>
</dependency>
```

