# OAuth2



# 一.简介

​		Oauth2是一个关于授权的开放标准，该标准允许用户让第三方应用访问该用户在某一网站上存储的私密资源(如头像，照片，视频等)，并且在这个过程中无需将用户名和密码提供给第三方应用。通过**令牌(token)**可以实现这一功能，每一个令牌授权一个特定的网站，使得第三方应用可以使用该令牌在**限定时间、限定范围访问指定资源**。Oauth让用户可以授权第三方网站灵活访问它们储存在另外一台服务器上的特定信息，而非所有内容。

​		对于用户而言，我们在互联网中最常见的OAuth2就是第三方登录，如QQ授权，微信授权，GitHub授权登录等

​		

## 1.1 协议特点

- 简单：不管是 OAuth 服务提供者还是应用开发者，都很易于理解与使用。
- 安全：没有涉及到用户密钥等信息，更安全更灵活。
- 开放：任何服务提供商都可以实现 OAuth，任何软件开发商都可以使用 OAuth。



## 1.2 应用场景

- 原生 app 授权：app 登录请求后台接口，为了安全认证，所有请求都带 token 信息，需要登录验证、请求后台数据。
- 前后端分离单页面应用：前后端分离框架，前端请求后台数据，需要进行 OAuth 2.0 安全认证，比如使用 vue、react 或者 h5 开发的 app。
- 第三方应用授权登录，比如 QQ，微博，微信的授权登录。



# 二.OAuth2协议流程

[官方文档](https://www.rfc-editor.org/rfc/rfc6749)

## 2.1 四种角色

- **Resource owner**：资源所有者，也叫用户

> 能够授予对受保护资源的访问权限的实体。当资源所有者是一个人时，它被称为最终用户

- **Resource server**：资源服务器，服务提供商用来存储资源，以及处理对资源的请求的服务器

> 托管受保护资源的服务器，能够使用访问令牌接受和响应受保护资源请求

- **Client**：客户端，也叫第三方应用，通过获取用户的授权，继而访问用户在资源服务器上的资源

> 代表资源所有者并经其授权发出受保护资源请求的应用程序。“客户”一词确实 不暗示任何特定的实现特征（例如， 应用程序是否在服务器、桌面或其他 设备上执行）。

- **Authorization server**：授权服务器，服务提供商用来处理授权的服务器，物理上与资源服务器可以是同一台服务器

> 服务器在成功验证资源所有者并获得授权后向客户端颁发访问令牌。授权服务器和资源服务器之间的交互超出了本规范的范围。授权服务器 可以是与资源服务器相同的服务器，也可以是单独的实体。 单个授权服务器可以发布多个资源服务器接受的访问令牌。



## 2.2 两种实体

- HTTP service：服务提供商
- User Agent：用户代理，通常指浏览器



## 2.3 OAuth2授权过程

**大体总流程**:

```
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|  			  | (如github)
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               | github授权服务器
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    | github资源服务器
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

（A）用户打开客户端以后，客户端要求用户给予授权。

**（B）用户同意给予客户端授权。**

（C）客户端使用上一步获得的授权，向授权服务器申请令牌。

（D）授权服务器对客户端进行认证以后，确认无误，同意发放令牌。

（E）客户端使用令牌，向资源服务器申请获取资源。

（F）资源服务器确认令牌无误，同意向客户端开放资源。

上面六个步骤之中，**B是关键，即用户怎样才能给于客户端授权**。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。

![image-20220925140738613](.\img.assets\image-20220925140738613.png)



# 三. 授权模式



## 3.1 授权码模式

​		授权码授权(Authorization Code)类型用于同时获取访问权限 令牌和刷新令牌，并针对机密客户端进行了优化。由于这是一个基于重定向的流程，客户端必须能够与资源所有者的用户代理（通常是网络浏览器）并能够接收传入的请求（通过重定向)从授权服务器。



**应用场景:**

​		这种方式是**最常用的流程**，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏



- Third-party-application:第三方应用程序，简称"客户端(client)"
- Resource Owner:资源所有者,简称用户"user"
- User-Agent:用户代理，指浏览器
- Authorization Server:授权服务器，即服务端专门用来处理授权的服务器
- Resource Server:资源服务器，级服务器存放用户生成资源的服务器，它与资源服务器可以是同一台服务器，也可以是不同的服务器

```
  	 +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)
```

具体流程如下:

> (A):用户访问第三方应用，第三方应用通过浏览器导向授权服务器
>
> (B):用户选择是否给予客户端授权
>
> (C):假设客户授权，授权服务器将用户导向客户端事先指定的“重定向URL”(Redirecton URL),**同时附上一个授权码**
>
> (D): 客户端收到授权码，附上早先的“重定向URL”，向授权服务器申请令牌，这一步是在客户端的后台服务器上完成的，对用户不可见
>
> (E):授权服务器核对授权码和重定向URL，确认无误后，向客户端发送访问令牌（access_token）和更新令牌(refresh_token)



![image-20220925141904192](img.assets\image-20220925141904192.png)

> 1：用户访问页面
> 2：访问的页面将请求重定向到授权服务器
> 3：认证服务器向用户展示授权页面，等待用户授权
> 4：用户授权，**授权服务器生成一个code和带上client_id**发送给应用服务器然后，应用服务器拿到code，并用client_id去后台查询对应的client_secret
> 5：**将code、client_id、client_secret传给授权服务器换取access_token和refresh_token**
> 6：将access_token和refresh_token传给应用服务器
> 7：验证token，访问真正的资源页面



**时序图:**

![授权码模式时序图](img.assets\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQ09ESU5H5LiA5Zy656m6,size_20,color_FFFFFF,t_70,g_se,x_16)



### 请求核心参数

```http
https://github.com/login/oauth/authorize?
client_id=f252166dc176d078abac&
redirect_uri=https%3A%2F%2Fruby-china.org%2Faccount%2Fauth%2Fgithub%2Fcallback&
response_type=code&
scope=user%3Aemail&
state=e9cf8832bf630d94e33b712503c2dbf863a055a43667e989
```

| 字段          | 描述                                           |
| ------------- | ---------------------------------------------- |
| client_id     | 授权服务器注册应用后的唯一标识                 |
| response_type | 必须，固定值，在授权码中必须为code             |
| redirect_uri  | 必须，通过客户端重定向URL                      |
| scope         | 必须，令牌可以访问资源权限，read 只读，all读写 |
| state         | 可选，存在原样返回客户端，用来防止csrf跨站攻击 |



### 刷新token

![image-20220925154511174](img.assets\image-20220925154511174.png)



## 3.2 隐式授权模式

隐式授权模式(implicit grant type) 不通过第三方应用程序的服务器，直接在浏览器向授权服务器申请令牌，**跳过了“授权码”**。所有的步骤都在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。****

​	**应用场景：**

​	有些 Web 应用是纯前端应用，没有后端。这时必须将令牌储存在前端。RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）“隐藏式”（implicit）

```
     +----------+
     | Resource |
     |  Owner   |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier     +---------------+
     |         -+----(A)-- & Redirection URI --->|               |
     |  User-   |                                | Authorization |
     |  Agent  -|----(B)-- User authenticates -->|     Server    |
     |          |                                |               |
     |          |<---(C)--- Redirection URI ----<|               |
     |          |          with Access Token     +---------------+
     |          |            in Fragment
     |          |                                +---------------+
     |          |----(D)--- Redirection URI ---->|   Web-Hosted  |
     |          |          without Fragment      |     Client    |
     |          |                                |    Resource   |
     |     (F)  |<---(E)------- Script ---------<|               |
     |          |                                +---------------+
     +-|--------+
       |    |
      (A)  (G) Access Token
       |    |
       ^    v
     +---------+
     |         |
     |  Client |
     |         |
     +---------+
```

具体流程如下:

> (A):第三方应用将用导向服务器
>
> (B):用户决定是否给客户端授权
>
> (C):用户给予授权，授权服务器将用户导向客户端指定的“重定向URL”，并且**在URL的Hash部分包含了访问令牌**,#token
>
> (D):浏览器向资源服务器发送请求，其中不包含上一步收到的Hash值
>
> (E):资源服务器返回一个网页，其中**包含的代码可以获取Hash值中的令牌**
>
> (F):浏览器执行上一步获得的脚本，提取出令牌
>
> (G):浏览器将令牌发送个客户端

![隐式授权模式](.\img.assets\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oiR5bCx5piv5Liq6I-c6bifMjAyMQ==,size_9,color_FFFFFF,t_70,g_se,x_16)

> 1：用户访问页面时，重定向到认证服务器。
> 2：认证服务器给用户一个认证页面，等待用户授权。
> 3：用户授权，认证服务器想应用页面返回Token
> 4：验证Token，访问真正的资源页面



**时序图:**

![隐式授权模式时序图](img.assets\13456498456)



**核心参数:**

```http
https://wx.com/oauth/authorize?
client_id=CLIENT_ID&
redirect_uri=http://www.baidu.com&
response_type=token&
scope=read
```

| 字段          | 描述                                           |
| ------------- | ---------------------------------------------- |
| client_id     | 授权服务器注册应用后的唯一标识                 |
| response_type | 必须，固定值，在授权码中必须为token            |
| redirect_uri  | 必须，通过客户端重定向URL                      |
| scope         | 必须，令牌可以访问资源权限，read 只读，all读写 |
| state         | 可选，存在原样返回客户端，用来防止csrf跨站攻击 |



## 3.3 密码模式

​		Resource Owner Password Credentials Grant（密码模式)中，用户向客户端提供自己的用户名和密码，客户端使用这些信息，向“服务提供商”索要授权。

在这种模式中，**用户必须把自己的密码给客户端，但是客户端不得储存密码**。



**应用场景:**	

​		如果你高度信任某个应用,或者客户端时操作系统的一部分，或者由一个相同的公司出品，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）



```
   +----------+
     | Resource |
     |  Owner   |
     |          |
     +----------+
          v
          |    Resource Owner
         (A) Password Credentials
          |
          v
     +---------+                                  +---------------+
     |         |>--(B)---- Resource Owner ------->|               |
     |         |         Password Credentials     | Authorization |
     | Client  |                                  |     Server    |
     |         |<--(C)---- Access Token ---------<|               |
     |         |    (w/ Optional Refresh Token)   |               |
     +---------+                                  +---------------+
```

具体步骤如下:

> (A):用户向客户端提供用户名和密码
>
> (B):客户端将用户名和密码发送给授权服务器，向后者请求令牌
>
> (C):授权服务器确认无误后，向客户端提供访问令牌

![image-20220925152527995](img.assets\image-20220925152527995.png)



> 1：用户访问用页面时，输入第三方认证所需要的信息(QQ/微信账号密码)
> 2：应用页面那种这个信息去认证服务器授权
> 3：认证服务器授权通过，拿到token，访问真正的资源页面



**时序图**:

![密码模式时序图](img.assets\12321421421412)



请求核心参数:

```http
https://wx.com/token?
client_id=CLIENT_ID&
grent_type=password&
usernmae=USERNAME&
passwoed=PASSWORD
```



## 3.4 客户端凭证模式

​		客户端模式(Client Credentials Grant) 指客户端以自己的名义，而不是以用户名的名义向“服务提供商”进行授权，严格来说，客户端模式并不属于OAuth框架要解决的问题。在这种模式中，**用户直接向客户端注册，客户端以自己的名义要求“服务提供商”提供服务，其实不存在授权问题。**



**应用场景:**

​		适用于没有前端的命令行应用，即在命令行下请求令牌

```

     +---------+                                  +---------------+
     |         |                                  |               |
     |         |>--(A)- Client Authentication --->| Authorization |
     | Client  |                                  |     Server    |
     |         |<--(B)---- Access Token ---------<|               |
     |         |                                  |               |
     +---------+                                  +---------------+
```

> (A):客户端向授权服务器进行身份验证，并要求一个访问令牌
>
> (B):授权服务器确认无误后，向客户端提供访问令牌

![image-20220925153800135](img.assets\image-20220925153800135.png)

> 1：用户访问应用客户端
> 2：通过客户端定义的验证方法，拿到token，无需授权
> 3：访问资源服务器A
> 4：拿到一次token就可以畅通无阻的访问其他的资源页面



**时序图**:

![客户端模式时序图](img.assets\123123214)



请求核心参数:

```http
https://wx.com/token?
client_id=CLIENT_ID&
grent_type=client_credentials&
client_secret=CLIENT_SECRECT
```





# 四.OAuth2 标准接口

OAuth默认接口,端点--访问路径

- **/oauth/authorize**:授权端点
- **/oauth/token**：获取令牌端点
- /oauth/confirm_access:用户确认授权提交端点
- /oauth/error:授权服务器错误信息端点
- /oauth/check_token:用于资源服务器访问的令牌解析端点
- /oauth/token_key:提供公共密钥的端点，如果使用JWT令牌的话



# 五.github授权登录

## 5.1 创建OAuth应用

在https://github.com/settings中找到Developer settings

![image-20220925155707331](img.assets\image-20220925155707331.png)

**创建一个新的OAuth应用**

![image-20220925155807505](img.assets\image-20220925155807505.png)

**注册新的OAuth应用，输入网站的首页和授权回调地址**

![image-20220925160355720](img.assets\image-20220925160355720.png)

**获取Client_Id和Client_Secrets密钥**

![image-20220925160647565](img.assets\image-20220925160647565.png)



## 5.2 创建OAuth客户端

pom文件:

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

		<!-- oauth2-client 客户端依赖  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>	
```

properties文件:**指定刚才配置好的Client_Id和Client_Secrets密钥以及回调地址**

```properties
spring.security.oauth2.client.registration.github.client-id=a8eb1b2572f8a8fd41c6
spring.security.oauth2.client.registration.github.client-secret=b77ae7e5086a7d417f8c483226436a015960d82c
#一定要和github中设定的重定向URL保持一致
spring.security.oauth2.client.registration.github.redirect-uri=http://localhost:8080/login/oauth2/code/github
```

在security的配置类中指定登录方式为OAuth2

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                //指定认证方式为oauth2
                .oauth2Login()
                .and()
                .csrf().disable();
    }
}
```

controller

```java
@RestController
public class Hello {
    @GetMapping("hello")
    //无论是使用哪一种第三方授权，授权信息都放在DefaultOAuth2User中
    public DefaultOAuth2User hello() {
        final Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        return (DefaultOAuth2User) principal;
     }
}
```

测试:访问登录界面(/login)

![image-20220925162913430](img.assets\image-20220925162913430.png)

查看页面源码

```html
<a href="/oauth2/authorization/github">GitHub</a>
```

点击查看请求

```http
https://github.com/login/oauth/authorize? 								 //授权接口
response_type=code&  					   								 //认证类型为授权码模式
client_id=a8eb1b2572f8a8fd41c6&			   								 //客户端id
scope=read:user&						  								 //令牌方式资源权限，只读权限
state=3RRpx2e3OucnDJJD18fknTdVijxAjwI7Y2BO0rDJRNk%3D& 					 //防止csrf攻击
redirect_uri=http://localhost:8080/login/oauth2/code/github              //重定向地址
```



# 六.OAuth2授权原理

在springSecurity中，关于OAuth2的处理是交给以下的过滤器来处理的

| OAuth2AuthorizationRequestRedirectFilter | 处理 OAuth2 认证重定向 | NO   |
| ---------------------------------------- | ---------------------- | ---- |
| OAuth2LoginAuthenticationFilter          | 处理 OAuth2 认证       | NO   |
| OAuth2AuthorizationCodeGrantFilter       | 处理OAuth2认证中授权码 | NO   |

在`.oauth2Login()`方法中，调用了 OAuth2LoginConfigurer类

```java
	public OAuth2LoginConfigurer<HttpSecurity> oauth2Login() throws Exception {
		return getOrApply(new OAuth2LoginConfigurer<>());
	}
```

而`OAuth2LoginConfigurer类`有`OAuth2LoginAuthenticationFilter`的实现

```java
public final class OAuth2LoginConfigurer<B extends HttpSecurityBuilder<B>>
		extends AbstractAuthenticationFilterConfigurer<B, OAuth2LoginConfigurer<B>, OAuth2LoginAuthenticationFilter>
```

在`OAuth2LoginAuthenticationFilter`中，定义了默认的处理身份验证请求的URI

```java
public static final String DEFAULT_FILTER_PROCESSES_URI = "/login/oauth2/code/*";
```

`OAuth2LoginAuthenticationFilter`中的`attemptAuthentication`方法开始进行授权

![image-20220925205519021](img.assets\image-20220925205519021.png)



