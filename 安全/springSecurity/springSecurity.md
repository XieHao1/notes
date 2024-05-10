# SpringSecurity



[TOC]



# 一.权限管理

基本上涉及到用户参与的系统都要进行权限管理，权限管理属于系统安全的范畴，权限管理实现<font color='red'>`对用户访问系统的控制`</font>，按照安全规则或者安全策略控制用户可以访问而且只能访问自己被授权的资源。

权限管理包括用户**身份认证**和**授权**两部分，简称**认证授权**。对于需要访问控制的资源用户首先经过身份认证，认证通过后用户具有该资源的访问权限方可访问。



## 1.1 认证

**`身份认证`** ，就是判断一个用户是否为合法用户的处理过程。最常用的简单身份认证方式是系统通过核对用户输入的用户名和口令，看其是否与系统中存储的该用户的用户名和口令一致，来判断用户身份是否正确。对于采用指纹等系统，则出示指纹；对于硬件Key等刷卡系统，则需要刷卡。



## 1.2 授权

**`授权`**，即访问控制，控制谁能访问哪些资源。主体进行身份认证后需要分配权限方可访问系统的资源，对于某些资源没有权限是无法访问的。



## 1.3 解决方案

和其他领域不同，在 Java 企业级开发中，安全管理框架非常少，目前比较常见的就是：

- Shiro
  - Shiro 本身是一个老牌的安全管理框架，有着众多的优点，例如轻量、简单、易于集成、可以在JavaSE环境中使用等。不过，在微服务时代，Shiro 就显得力不从心了，在微服务面前和扩展方面，无法充分展示自己的优势。
- 开发者自定义
  - 也有很多公司选择自定义权限，即自己开发权限管理。但是一个系统的安全，不仅仅是登录和权限控制这么简单，我们还要考虑种各样可能存在的网络政击以及防彻策略，从这个角度来说，开发者白己实现安全管理也并非是一件容易的事情，只有大公司才有足够的人力物力去支持这件事情。
- [Spring Security]([Spring Security](https://spring.io/projects/spring-security))
  - Spring Security,作为spring 家族的一员，在和 Spring 家族的其他成员如 Spring Boot,Spring Cloud等进行整合时，具有其他框架无可比拟的优势，同时对 OAuth2 有着良好的支持，再加上Spring Cloud对 Spring Security的不断加持（如推出 Spring Cloud Security )，让 Spring Security 不知不觉中成为微服务项目的首选安全管理方案。





# 二.SpringSecurity简介

Spring Security 是一个功能强大且高度可定制的身份验证和访问控制框架。它是保护基于 Spring 的应用程序的事实标准。 

Spring Security 是一个专注于为 Java 应用程序提供身份验证和授权的框架。像所有 Spring 项目一样，Spring Security 的真正强大之处在于它可以轻松扩展以满足自定义需求。

**特征:**

- 对身份验证和授权的全面且可扩展的支持 
- 防止会话固定、点击劫持、跨站点请求伪造等攻击
- Servlet API 集成
- Spring Web MVC 的可选集成



SpringSecurity是一个功能强大，可高度定制的<font color="blue">身份验证</font>和<font color="blue">访问控制</font>的框架,或者说用来实现权限系统中权限管理的框架。



## 2.1 整体架构

在SpringSecurity的架构设计中，**`认证(Authentication)`**和**`授权(Authorization)`** 是分开的，无论使用什么样的认证方式。都不会影响授权，这是两个独立的存在，这种独立带来的好处之一，就是可以非常方便地整合一些外部的解决方案。

![image-20220110112541559](img.assets\image-20220110112541559.png)



## 2.2 认证接口

### 2.2.1 AuthenticationManager

在Spring Security中认证是由`AuthenticationManager`接口来负责的，接口定义为：

![image-20220110104531129](img.assets\image-20220110104531129.png)

```java
public interface AuthenticationManager { 
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

- 返回 Authentication 表示认证成功
- 返回 AuthenticationException 异常，表示认证失败。



### 2.2.2 ProviderManager

AuthenticationManager 主要实现类为 `ProviderManager`，在 ProviderManager 中管理了众多 **AuthenticationProvider** 实例。在一次完整的认证流程中，Spring Security 允许存在多个 AuthenticationProvider ，用来实现多种认证方式，这些 AuthenticationProvider 都是由 ProviderManager 进行统一管理的。

![image-20220110103518334](img.assets\image-20220110103518334.png)



### 2.2.3 Authentication

认证以及认证成功的信息主要是由 Authentication 的实现类进行保存的，其接口定义为：

![image-20220110104815645](img.assets\image-20220110104815645.png)

```java
public interface Authentication extends Principal, Serializable {
    //getAuthorities 获取用户权限信息
    Collection<? extends GrantedAuthority> getAuthorities();
    //getCredentials 获取用户凭证信息，一般指密码
    Object getCredentials();
    //getDetails 获取用户详细信息
    Object getDetails();
    //getPrincipal 获取用户身份信息，用户名、用户对象等
    Object getPrincipal();
    //isAuthenticated 用户是否认证成功
    boolean isAuthenticated();
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

- getAuthorities 获取用户权限信息
- getCredentials 获取用户凭证信息，一般指密码
- getDetails 获取用户详细信息
- getPrincipal 获取用户身份信息，用户名、用户对象等
- isAuthenticated 用户是否认证成功



### 2.2.4 SecurityContextHolder

SecurityContextHolder <font color="red">用来获取登录之后用户信息</font>。Spring Security 会将登录用户数据保存在 Session 中。但是，为了使用方便,Spring Security在此基础上还做了一些改进，其中最主要的一个变化就是**线程绑定**。当用户登录成功后,Spring Security 会将登录成功的**用户信息(Authentication类)保存到 SecurityContextHolder** 中。

SecurityContextHolder 中的数据保存<font color="red">默认是通过ThreadLocal 来实现的</font>，使用 ThreadLocal 创建的变量只能被当前线程访问，不能被其他线程访问和修改，也就是<font color="red">用户数据和请求线程绑定在一起</font>。

当**登录请求处理完毕后**，Spring Security 会将 SecurityContextHolder 中的<font color="red">数据拿出来保存到 Session 中，同时将 SecurityContexHolder 中的数据清空</font>。以后每当**有请求到来时**，Spring Security 就会<font color="red">先从 Session 中取出用户登录数据，保存到 SecurityContextHolder 中</font>，方便在该请求的后续处理过程中使用，同时**在请求结束时**将 SecurityContextHolder 中的数据拿出来保存到 Session 中，然后将 Security SecurityContextHolder 中的数据清空。

这一策略非常方便用户在 Controller、Service 层以及任何代码中获取当前登录用户数据。



## 2.3 授权接口

### 2.3.1 AccessDecisionManager

AccessDecisionManager (访问决策管理器)，用来**决定此次访问是否被允许**。---决定是否放权

![image-20220110110946267](img.assets\image-20220110110946267.png)



### 2.3.2 AccessDecisionVoter

AccessDecisionVoter (访问决定投票器)，投票器会**检查用户是否具备应有的角色，进而投出赞成、反对或者弃权票。**---投票

![image-20220110111011018](img.assets\image-20220110111011018.png)

AccesDecisionVoter 和 AccessDecisionManager 都有众多的实现类，在 **AccessDecisionManager 中会换个遍历 AccessDecisionVoter**，进而决定是否允许用户访问，因而 AccessDecisionVoter 和 AccessDecisionManager 两者的关系类似于 AuthenticationProvider 和 ProviderManager 的关系。



### 2.3.3 ConfigAttribute

ConfigAttribute，**用来保存授权时的角色信息**

![image-20220110111037603](img.assets\image-20220110111037603.png)

在 Spring Security 中，用户请求一个资源(通常是一个接口或者一个 Java 方法)需要的角色会被封装成一个 ConfigAttribute 对象，在 ConfigAttribute 中只有一个 getAttribute方法，该方法返回一个 String 字符串，就是**角色的名称**。一般来说，角色名称都带有一个 `ROLE_` 前缀，投票器 AccessDecisionVoter 所做的事情，其实就是比较用户所具各的角色和请求某个 资源所需的 ConfigAtuibute 之间的关系。



## 2.4 实现原理

 Spring Security 的 Servlet 支持基于 Servlet Filters

官方文档：[Spring Security Reference](https://docs.spring.io/spring-security/site/docs/5.5.4/reference/html5/#servlet-architecture)

![filterchain](img.assets\filterchain.png)

客户端向应用程序发送请求，容器创建一个 FilterChain，其中包含应根据请求 URI 的路径处理 HttpServletRequest 的过滤器和 Servlet。在 Spring MVC 应用程序中，Servlet 是 DispatcherServlet 的一个实例。最多一个 Servlet 可以处理一个 HttpServletRequest 和 HttpServletResponse。但是，可以使用多个过滤器来：

- ​	防止调用下游过滤器或 Servlet。在这种情况下，过滤器通常会写入 HttpServletResponse。
- ​	修改下游Filters和Servlet使用的HttpServletRequest或HttpServletResponse



### 2.4.1 DelegatingFilterProxy

**将servlet中的请求通过DelegatingFilterProxy代理交给spring security中的过滤器执行**

Spring 提供了一个名为 DelegatingFilterProxy 的过滤器实现，它允许在 Servlet 容器的生命周期和 Spring 的 ApplicationContext 之间进行桥接。 Servlet 容器允许使用自己的标准注册过滤器，但它不知道 Spring 定义的 Bean。 DelegatingFilterProxy 可以通过标准的 Servlet 容器机制注册，但将所有工作委托给实现 Filter 的 Spring Bean。

![delegatingfilterproxy](img.assets\delegatingfilterproxy.png)

DelegatingFilterProxy 的另一个好处是它允许延迟查找 Filter bean 实例。这很重要，因为容器需要在容器启动之前注册过滤器实例。但是，Spring 通常使用 ContextLoaderListener 来加载 Spring Bean，直到需要注册 Filter 实例之后才会这样做。



### 2.4.2 FilterChainProxy

Spring Security 的 Servlet 支持包含在 FilterChainProxy 中。 FilterChainProxy 是 Spring Security 提供的一个特殊 Filter，它允许通过 SecurityFilterChain 委托给**多个 Filter 实例**。由于 FilterChainProxy 是一个 Bean，它通常被包装在一个 DelegatingFilterProxy 中。

![filterchainproxy](img.assets\filterchainproxy.png)

### 2.4.3 SecurityFilterChain

FilterChainProxy 使用 SecurityFilterChain 来确定应该为此请求调用哪些 Spring Security Filters。

![securityfilterchain](img.assets\securityfilterchain.png)

- SecurityFilterChain 中的安全过滤器通常是 Bean，但它们是使用 FilterChainProxy 而不是 DelegatingFilterProxy 注册的。 FilterChainProxy 为直接注册到 Servlet 容器或 DelegatingFilterProxy 提供了许多优势。首先，它为 Spring Security 的所有 Servlet 支持提供了一个起点。出于这个原因，如果您尝试对 Spring Security 的 Servlet 支持进行故障排除，在 FilterChainProxy 中添加一个调试点是一个很好的起点。
- 其次，由于 **FilterChainProxy 是 Spring Security 使用的核心**，它可以执行不被视为可选的任务。例如，它清除 SecurityContext 以避免内存泄漏。它还应用 Spring Security 的 HttpFirewall 来保护应用程序免受某些类型的攻击。
- 此外，它在确定何时应调用 SecurityFilterChain 时提供了更大的灵活性。在 Servlet 容器中，仅根据 URL 调用过滤器。但是，FilterChainProxy 可以通过利用 RequestMatcher 接口基于 HttpServletRequest 中的任何内容来确定调用。
- 实际上，**FilterChainProxy 可以用来确定应该使用哪个 SecurityFilterChain**。这允许为应用程序的不同部分提供完全独立的配置。

![multi securityfilterchain](img.assets\multi-securityfilterchain.png)

FilterChainProxy 决定应该使用哪个 SecurityFilterChain。**只有第一个匹配的 SecurityFilterChain 才会被调用**。

如果请求 /api/messages/ 的 URL，它将首先匹配 SecurityFilterChain0 的 /api/** 模式，因此即使在 SecurityFilterChainn 上也匹配，也只会调用 SecurityFilterChain0。

如果请求 /messages/ 的 URL，它将与 SecurityFilterChain0 的 /api/** 模式不匹配，因此 FilterChainProxy 将继续尝试每个 SecurityFilterChain。假设没有其他，SecurityFilterChain 实例匹配 SecurityFilterChainn 将被调用。

请注意，SecurityFilterChain0 仅配置了三个安全过滤器实例。但是，SecurityFilterChainn 配置了四个安全过滤器。需要注意的是，每个 SecurityFilterChain 都可以是唯一的并且可以单独配置。事实上，如果应用程序希望 Spring Security 忽略某些请求，SecurityFilterChain 可能具有零安全过滤器。

### 2.4.4 Security Filters

使用 SecurityFilterChain API 将安全过滤器插入到 FilterChainProxy 中。过滤器的顺序很重要。通常不需要知道 Spring Security 的过滤器的顺序。但是，有时知道排序是有益的,以下是 Spring Security Filter 排序的完整列表：

| 过滤器                                                       | 过滤器作用                                               | 默认是否加载 |
| ------------------------------------------------------------ | -------------------------------------------------------- | ------------ |
| ChannelProcessingFilter                                      | 过滤请求协议 HTTP 、HTTPS                                | NO           |
| <font color="red">WebAsyncManagerIntegrationFilter</font>    | 将 WebAsyncManger 与 SpringSecurity 上下文进行集成       | YES          |
| <font color="red">SecurityContextPersistenceFilter</font>    | 在处理请求之前,将安全信息加载到 SecurityContextHolder 中 | YES          |
| <font color="red">HeaderWriterFilter</font>                  | 处理头信息加入响应中                                     | YES          |
| CorsFilter                                                   | 处理跨域问题                                             | NO           |
| <font color="red">CsrfFilter</font>                          | 处理 CSRF 攻击                                           | YES          |
| <font color="red">LogoutFilter</font>                        | 处理注销登录                                             | YES          |
| OAuth2AuthorizationRequestRedirectFilter                     | 处理 OAuth2 认证重定向                                   | NO           |
| Saml2WebSsoAuthenticationRequestFilter                       | 处理 SAML 认证                                           | NO           |
| X509AuthenticationFilter                                     | 处理 X509 认证                                           | NO           |
| AbstractPreAuthenticatedProcessingFilter                     | 处理预认证问题                                           | NO           |
| CasAuthenticationFilter                                      | 处理 CAS 单点登录                                        | NO           |
| OAuth2LoginAuthenticationFilter                              | 处理 OAuth2 认证                                         | NO           |
| Saml2WebSsoAuthenticationFilter                              | 处理 SAML 认证                                           | NO           |
| <font color="red">UsernamePasswordAuthenticationFilter</font> | 处理表单登录                                             | YES          |
| OpenIDAuthenticationFilter                                   | 处理 OpenID 认证                                         | NO           |
| <font color="red">DefaultLoginPageGeneratingFilter</font>    | 配置默认登录页面                                         | YES          |
| <font color="red"> DefaultLogoutPageGeneratingFilter</font>  | 配置默认注销页面                                         | YES          |
| ConcurrentSessionFilter                                      | 处理 Session 有效期                                      | NO           |
| DigestAuthenticationFilter                                   | 处理 HTTP 摘要认证                                       | NO           |
| BearerTokenAuthenticationFilter                              | 处理 OAuth2 认证的 Access Token                          | NO           |
| <font color="red">BasicAuthenticationFilter</font>           | 处理 HttpBasic 登录                                      | YES          |
| <font color="red">RequestCacheAwareFilter</font>             | 处理请求缓存                                             | YES          |
| <font color="red">SecurityContextHolderAwareRequestFilter</font> | 包装原始请求                                             | YES          |
| JaasApiIntegrationFilter                                     | 处理 JAAS 认证                                           | NO           |
| RememberMeAuthenticationFilter                               | 处理 RememberMe 登录                                     | NO           |
| <font color="red">AnonymousAuthenticationFilter</font>       | 配置匿名认证                                             | YES          |
| OAuth2AuthorizationCodeGrantFilter                           | 处理OAuth2认证中授权码                                   | NO           |
| <font color="red">SessionManagementFilter</font>             | 处理 session 并发问题                                    | YES          |
| <font color="red">ExceptionTranslationFilter</font>          | 处理认证/授权中的异常                                    | YES          |
| <font color="red">FilterSecurityInterceptor</font>           | 处理授权相关                                             | YES          |
| SwitchUserFilter                                             | 处理账户切换                                             | NO           |

可以看出，Spring Security 提供了 30 多个过滤器。默认情况下Spring Boot 在对 Spring Security 进入自动化配置时，会**创建一个名为 SpringSecurityFilerChain 的过滤器，并注入到 Spring 容器中，这个过滤器将负责所有的安全管理，包括用户认证、授权、重定向到登录页面等**。

具体可以参考`WebSecurityConfiguration`的源码:

![image-20220830164409734](img.assets\image-20220830164409734.png)

通过遍历过滤器链可以看到：

![image-20220830165028407](img.assets\image-20220830165028407.png)



### 2.4.5 总结

默认过滤器并不是直接放在 Web 项目的原生过滤器链中，而是通过一个 **FilterChainProxy** 来统一管理。FilterChainProxy 本身是通过 Spring 框架提供的 DelegatingFilterProxy 整合到原生的过滤器链中。Spring Security 中的过滤器链通过 FilterChainProxy 嵌入到 Web项目的原生过滤器链中。**FilterChainProxy 作为一个顶层的管理者**，将统一管理 Security Filter。



- SpringSecurity中不是用原生Web的Filter
- 既然如此就需要代理类DelegatingFilterProxy注册Filter到Spring中
- FilterChainProxy作为最顶层的代理托管整个SecurityFilter链（支持多个Filter组合拦截）
- SpringSecurity中支持根据请求路径自定义FilterChain



# 三.环境搭建

- Spring boot
- spring Security
  - 认证：判断用户是否是系统合法用户的过程
  - 授权：判断系统用户可以访问或者具有哪些访问资源的权限

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

**引入该依赖后，默认对所有的资源进行保护，无需任何配置**

启动项目可以发现:

```
Using generated security password: aca8376d-3b84-4c24-bc6d-48f0dacb8c04
```

在浏览器中输入请求地址,请求地址将自动变成`http://localhost:8080/login`,同时出现以下界面

![image-20220830151230952](img.assets\image-20220830151230952.png)

默认用户名为：`user`,默认密码为控制台打印的：`Using generated security password:`后面字符串



## 3.1 默认配置

自动配置官方文档:[Spring Security Reference](https://docs.spring.io/spring-security/site/docs/5.5.4/reference/html5/#servlet-hello-auto-configuration)

Spring Boot 自动启动：

- 启用 Spring Security 的默认配置，该配置将 **servlet 过滤器创建为名为 springSecurityFilterChain 的 bean**。此 bean 负责应用程序中的所有安全性（保护应用程序 URL、验证提交的用户名和密码、重定向到登录表单等）。
- 使用用户名 user 和随机生成的密码创建一个 UserDetailsService bean，该密码记录到控制台。
- 使用 Servlet 容器为**每个请求注册一个名为 springSecurityFilterChain 的 bean 的过滤器**。



Spring Boot 配置不多，但做了很多。功能总结如下：

- 需要经过身份验证的用户才能与应用程序进行任何交互
- 为您生成默认登录表单
- 让登录到控制台的用户名user和密码的用户进行基于表单的身份验证
- 使用 BCrypt 保护密码存储
- 让用户退出
- CSRF 攻击预防
- session 固定保护
- 安全标头集成
  - 用于安全请求的 HTTP 严格传输安全性 
  - X-Content-Type-Options 集成 
  - 缓存控制（稍后可以由您的应用程序覆盖以允许缓存您的静态资源） 
  - X-XSS-保护集成 
  - X-Frame-Options 集成有助于防止点击劫持
- 与以下 Servlet API 方法集成：
  - `HttpServletRequest#getRemoteUser()`
  - `HttpServletRequest.html#getUserPrincipal()`
  - `HttpServletRequest.html#isUserInRole(java.lang.String)`
  - `HttpServletRequest.html#login(java.lang.String, java.lang.String)`
  - `HttpServletRequest.html#logout()`



### 3.1.1 @Conditional注解

@Conditional是Spring4新提供的注解，它的作用是**按照一定的条件进行判断，满足条件给容器注册bean。**



### 3.1.2 SpringBootWebSecurityConfiguration

SpringBootWebSecurityConfiguration是springboot自动配置类，通过这个源码可知，在默认情况下对所有的请求进行权限控制

```java
@Configuration(proxyBeanMethods = false)
//Security默认使用条件
@ConditionalOnDefaultWebSecurity
//基于Web应用程序的条件--是一个servlet容器 -> 当启动的容器为servlet时，该注解生效
@ConditionalOnWebApplication(type = Type.SERVLET)
class SpringBootWebSecurityConfiguration {

	@Bean
	@Order(SecurityProperties.BASIC_AUTH_ORDER)
	SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        //authorizeRequests()开启请求权限认证
        //anyRequest() 任何请求
        //authenticated() 认证
        //and() 开启哪些认证
        //formLogin() 表单登录验证
        //httpBasic() 没有界面，直接输入账号和密码
		http.authorizeRequests().anyRequest().authenticated().and().formLogin().and().httpBasic();
		return http.build();
	}

}
```

**@ConditionalOnDefaultWebSecurity:**

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
//默认的SecurityCondition类条件
@Conditional(DefaultWebSecurityCondition.class)
public @interface ConditionalOnDefaultWebSecurity {

}
```

**DefaultWebSecurityCondition：**

```java
class DefaultWebSecurityCondition extends AllNestedConditions {

	DefaultWebSecurityCondition() {
		super(ConfigurationPhase.REGISTER_BEAN);
	}
	
    //在当前类路径中发现有SecurityFilterChain类和HttpSecurity类
    //引入依赖后必有上述两个类
	@ConditionalOnClass({ SecurityFilterChain.class, HttpSecurity.class })
	static class Classes {}

    //在当前项目中没有找到 WebSecurityConfigurerAdapter,SecurityFilterChain类的实例
	@ConditionalOnMissingBean({ WebSecurityConfigurerAdapter.class, SecurityFilterChain.class })
	static class Beans {}
}

```



### 3.1.3 总结

**SpringSecurity默认配置的条件:**

- ​	启动的容器为servlet容器
- ​    classpath中没有SecurityFilterChain类和HttpSecurity类
- ​    没有自定义的 WebSecurityConfigurerAdapter,SecurityFilterChain类的实例
- ​    默认情况下条件都是满足的

​	

若是想要自定义Security，需要自定义WebSecurityConfigurerAdapter或者SecurityFilterChain类的实现,**WebSecurityConfigurerAdapter**实现的功能更加完全，**推荐使用**,**SpringSecurity中的核心配置都在WebSecurityConfigurerAdapter中**

```java
	protected void configure(HttpSecurity http) throws Exception {
		this.logger.debug("Using default configure(HttpSecurity). "
				+ "If subclassed this will potentially override subclass configure(HttpSecurity).");
        //如果子类化，这可能会覆盖子类配置
		http.authorizeRequests((requests) -> requests.anyRequest().authenticated());
		http.formLogin();
		http.httpBasic();
	}
```

SecurityFilterChain用于扩展自定义Filter：

```java
public interface SecurityFilterChain {

	boolean matches(HttpServletRequest request);

	List<Filter> getFilters();

}
```



## 3.2 默认登录界面

登录流程:

![image-20220906164222971](img.assets\image-20220906164222971.png)

1.请求/hello接口，在引入spring security之后会经过一些过滤器。

2.在请求到达FilterSecurityInterceptor时，发现请求并未认证，请求拦截下来，并且抛出AccessDeniedException异常。

3.抛出AccessDeniedException的异常会被ExceptionTransactionFilter捕获，这个Filter中会调用LoginUrlAuthenticationEntry#commence方法给客户端返回302，要求客户端进行重定向到/login页面。

4.客户端发送/login请求。

5./login请求会再次被拦截器中的DefaultLoginPageGeneratingFilter拦截到，并且在拦截器中返回登录界面。

**通过这种方式，springSecurity默认过滤器中生成了登录页面，并且返回**



DefaultLoginPageGeneratingFilter中的doFilter方法,其中页面的显示在 generateLoginPageHtml方法中：

```java
	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		boolean loginError = isErrorPage(request);
		boolean logoutSuccess = isLogoutSuccess(request);
		if (isLoginUrlRequest(request) || loginError || logoutSuccess) {
			String loginPageHtml = generateLoginPageHtml(request, loginError, logoutSuccess);
			response.setContentType("text/html;charset=UTF-8");
			response.setContentLength(loginPageHtml.getBytes(StandardCharsets.UTF_8).length);
			response.getWriter().write(loginPageHtml);
			return;
		}
		chain.doFilter(request, response);
	}
```

默认登录页面html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="">
    <title>Please sign in</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-/Y6pD6FV/Vv2HJnA6t+vslU6fwYXjCFtcEpHbNJ0lyAFsXTsjBbfaDjzALeQsN6M" crossorigin="anonymous">
    <link href="https://getbootstrap.com/docs/4.0/examples/signin/signin.css" rel="stylesheet" crossorigin="anonymous"/>
  </head>
  <body>
     <div class="container">
      <form class="form-signin" method="post" action="/login">
        <h2 class="form-signin-heading">Please sign in</h2>
        <p>
          <label for="username" class="sr-only">Username</label>
          <input type="text" id="username" name="username" class="form-control" placeholder="Username" required autofocus>
        </p>
        <p>
          <label for="password" class="sr-only">Password</label>
          <input type="password" id="password" name="password" class="form-control" placeholder="Password" required>
        </p>
	<input name="_csrf" type="hidden" value="1e852105-8bf3-43ce-939d-72cbee14d7ba" />
        <button class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>
      </form>
</div>
</body></html>
```



## 3.3 默认用户登录

1.查看SpringBootWebSecurityConfiguration#defaultSecurityFilterChain中的formLogin方法表单登录

![image-20220906171638769](img.assets\image-20220906171638769.png)

2.处理登录为FormLoginConfigurer类中的UsernamePasswordAuthenticationFilter类的实例

![image-20220906171756459](img.assets\image-20220906171756459.png)

3.查看类中的UsernamePasswordAuthenticationFilter中的attemptAuthentication方法得知实际调用的AuthenticationManager中的authenticate方法

![image-20220906172044805](img.assets\image-20220906172044805.png)

4.调用ProviderManager类中的方法authenicate方法

![image-20220906173549112](img.assets\image-20220906173549112.png)

5.调用AuthenticationProvider实现类中的AbstractUserDetailsAuthenticationProvider类中的方法

![image-20220906174708085](img.assets\image-20220906174708085.png)

6.最终调用DaoAuthenticationProvider类中的getUserDetailsService方法进行比较

![image-20220906174952597](img.assets\image-20220906174952597.png)

由此可知，**默认实现是基于InMemoryUserDetailsManager类,基于内存的方式实现**

noop -- 未加密

![image-20220906175125532](img.assets\image-20220906175125532.png)



### 3.3.1 UserDetailsService

UserDetailsService是顶层父接口，接口中的loadUserByUsername方法是用在认证时进行用户名认证方法，**默认实现使用的是内存实现，如果想要修改数据库实现，只需要自定义UserDetailsService实现，最终返回userDetatils实例即可。**

```java
public interface UserDetailsService {
   UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

![image-20220906180137359](img.assets\image-20220906180137359.png)

### 3.3.2 UserDetailsServiceAutoConfiguation

UserDetatilServiceAutoConfiguation是UserDetatilService的自动配置

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(AuthenticationManager.class)
@ConditionalOnBean(ObjectPostProcessor.class)
@ConditionalOnMissingBean(
      value = { AuthenticationManager.class, AuthenticationProvider.class, UserDetailsService.class,
            AuthenticationManagerResolver.class },
      type = { "org.springframework.security.oauth2.jwt.JwtDecoder",
            "org.springframework.security.oauth2.server.resource.introspection.OpaqueTokenIntrospector",
            "org.springframework.security.oauth2.client.registration.ClientRegistrationRepository" })
public class UserDetailsServiceAutoConfiguration {
    
    private static final String NOOP_PASSWORD_PREFIX = "{noop}";

	private static final Pattern PASSWORD_ALGORITHM_PATTERN = Pattern.compile("^\\{.+}.*$");

	private static final Log logger = LogFactory.getLog(UserDetailsServiceAutoConfiguration.class);
    
   @Bean
   @Lazy
   public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,
         ObjectProvider<PasswordEncoder> passwordEncoder) {
      SecurityProperties.User user = properties.getUser();
      List<String> roles = user.getRoles();
      return new InMemoryUserDetailsManager(
            User.withUsername(user.getName()).password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
                  .roles(StringUtils.toStringArray(roles)).build());
   }
    //…………
}
```

结论:

1.从自动配置源码中得知当classpath下存在AuthenticationManager类

2.spring中的ObjectPostProcesso实例

3.当前项目中，没有自定义**AuthenticationManager，AuthenticationProvider, UserDetailsService, AuthenticationManagerResolve**r这四个类的实例

**默认情况下都会满足，此时springSecurity会提供一个InMemoryUserDetailManger实例**



### 3.3.3 修改默认用户和密码

![image-20220906182319514](img.assets\image-20220906182319514.png)

在SecurityProperties中的user内部类中可以看到定义了默认的用户名和密码

```java
@ConfigurationProperties(prefix = "spring.security")
public class SecurityProperties {

	private final User user = new User();
    
    public User getUser() {
		return this.user;
	}

    public static class User {

		private String name = "user";

		private String password = UUID.randomUUID().toString();
        
        //角色信息
        private List<String> roles = new ArrayList<>();
	}
}
```

通过源码可以得知，@ConfigurationProperties(prefix = "spring.security")--》**可以通过属性注入的方式修改默认的用户名和密码,只需要在配置文件中添加以下配置就可以经行修改**

```properties
spring.security.user.name=root
spring.security.user.password=root
spring.security.user.roles=admin,user
```



## 3.4 默认登录流程图

![image-20220821175458416](img.assets\image-20220821175458416.57c8e429.png)





![img](img.assets\认证流程.png)



## 3.5 总结

- AuthenticationManager,ProviderManger以及AuthenticationProvider的关系

![image-20220906184553871](img.assets\image-20220906184553871.png)

- WebSecurityConfigurerAdapter扩展SpringSecurity所有的默认配置

![image-20220906185053688](img.assets\image-20220906185053688.png)

- UserDetailsService用来修改默认认证的数据源信息

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```



# 四.自定义认证

## 1.自定义资源权限规则

- /index 公共资源
- /hello 受保护资源

WebSecurityConfigurerAdapter中的`configure`方法可以实现自定义资源权限规则

```java
	protected void configure(HttpSecurity http) throws Exception {
		this.logger.debug("Using default configure(HttpSecurity). "
				+ "If subclassed this will potentially override subclass configure(HttpSecurity).");
		http.authorizeRequests((requests) -> requests.anyRequest().authenticated());
		http.formLogin();
		http.httpBasic();
	}
```

自定义认证

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //authorizeRequests() 开启请求认证
        //mvcMatchers() 请求匹配
        //permitAll() 允许所有
        //authenticated() 认证
        //formLogin() 表单认证
        http.authorizeRequests()
                //放行的资源要放在所有认证资源的前面
                .mvcMatchers("/index").permitAll()
                //.mvcMatchers("/hello").authenticated()
                .anyRequest().authenticated()
                .and()
                .formLogin();
    }
}
```

```markdown
# 说明
- permitAll() 表示放行该资源，该资源是为公共资源，无需认证和授权可以直接访问
- anyRequest().authenticated() 表示所有请求，必须认证之后才能访问
- formLogin() 返回表单认证
## 放行的资源要放在所有认证资源的前面
```



## 2.自定义登录界面

使用默认拦截器拦截`UsernamePasswordAuthenticationFilter`跳转

```java
	public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";

	public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";

	private static final AntPathRequestMatcher DEFAULT_ANT_PATH_REQUEST_MATCHER = new AntPathRequestMatcher("/login",
			"POST");
	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		username = (username != null) ? username : "";
		username = username.trim();
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

**使用要求:**

1.请求方式必须为POST

2.请求参数必须为username，和password

3.请求路径必须为/login

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            	//login.html 放行跳转到登录界面的请求
                .mvcMatchers("/index","/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
            	//loginPage 指定在需要登录时将用户发送到的URL
            	//loginProcessingUrl 指定处理登录的url请求，前端发送的请求路径为/login，使用自定义登录界面必须指定,否则会无限重定向
                .formLogin().loginPage("/login.html").loginProcessingUrl("/login")
                .and()
                .csrf().disable();//禁止跨域请求保护
    }
}
```

```html
<!--默认登录界面中的隐藏输入框，指定跨越请求保护id -->
<input name="_csrf" type="hidden" value="1e852105-8bf3-43ce-939d-72cbee14d7ba" />
```

指定跳转到的**界面**

```java
@Controller
public class LoginController {

    @RequestMapping("/login.html")
    public String login() {
        return "login";
    }

}
```

引入thymeleaf依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
</head>
<body>
<h1>用户登录</h1>
    <!--
		1.请求方式必须为POST

		2.请求参数必须为username，和password

		3.请求路径必须为/login
	-->
<form method="post" th:action="@{/login}">
    用户名:<input name="username" placeholder="root" type="text"> <br>
    密码:<input name="password" placeholder="root" type="password"> <br>
    <input type="submit" value="登录">
</form>
</body>
</html>
```

### 2.1 修改用户名，密码，请求路径

**表单属性必须和配置中的属性一一对应**

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                //.mvcMatchers("/hello").authenticated()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login.html")
                //指定处理登录的方法
                .loginProcessingUrl("/doLogin")
                //指定用户名name属性
                .usernameParameter("name")
                //指定密码name属性
                .passwordParameter("passwd")
                .and()
                .csrf().disable();//禁止跨域请求保护
    }
}

```

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
</head>
<body>
<h1>用户登录</h1>
<form method="post" th:action="@{/doLogin}">
    用户名:<input name="name" placeholder="root" type="text"> <br>
    密码:<input name="passwd" placeholder="root" type="password"> <br>
    <input type="submit" value="登录">
</form>
</body>
</html>
```



## 3.自定义认证成功处理

### 3.1 defaultSuccessUrl

 defaultSuccessUrl("/hello")  默认使用 **redirect** 跳转,如果之前有请求路径，**认证成功后会优先跳转到之前的请求路径，也可以使用第二个参数进行修改**

```java
    http.authorizeRequests()
            .mvcMatchers("/index", "/login.html").permitAll()
            .anyRequest().authenticated()
            .and()
            .formLogin().loginPage("/login.html").loginProcessingUrl("/doLogin")
        	.usernameParameter("name")
            .passwordParameter("passwd")
        	//默认使用redirect跳转
            .defaultSuccessUrl("/hello")
            .and()
            .csrf().disable();
}
```

请求路径发送改变，说明为重定向跳转:

![image-20220909155802458](img.assets\image-20220909155802458.png)



### 3.2 successForwardUrl

successForwardUrl("/index") 默认使用**forward**跳转,而且**默认跳转到指定页码**

```java
    http.authorizeRequests()
            .mvcMatchers("/index", "/login.html").permitAll()
            .anyRequest().authenticated()
            .and()
            .formLogin().loginPage("/login.html").loginProcessingUrl("/doLogin")
        	.usernameParameter("name")
            .passwordParameter("passwd")
        	//默认使用Forward跳转
            .successForwardUrl("/index")
            .and()
            .csrf().disable();
}
```

请求路径为发送改变，说明为请求转发跳转，且默认转发的路径为/index

**使用请求转发时请求方式必须和第一次请求的请求方式相同**

![image-20220909160346155](img.assets\image-20220909160346155.png)

### 3.3  successHandler 主要使用

以上两种方式都不适合前后端分离开放，这时需要使用 **successHandler**方法经行处理

```java
	public final T successHandler(AuthenticationSuccessHandler successHandler) {
		this.successHandler = successHandler;
		return getSelf();
	}
```

![image-20220909175631389](img.assets\image-20220909175631389.png)

自定义AuthenticationSuccessHandler接口

```java
/**
 * 自定义认证成功后的处理
 */
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)
            throws IOException, ServletException {
        Map<String, Object> map = new HashMap<>();
        map.put("msg", "验证成功");
        map.put("code", 200);
        map.put("authentication", authentication);
        response.setContentType("application/json;charset=UTF-8");
        final String asString = new ObjectMapper().writeValueAsString(map);
        response.getWriter().print(asString);
    }
}

```

使用

```java
http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login.html")
                .loginProcessingUrl("/doLogin")
                .usernameParameter("name")
                .passwordParameter("passwd")
    			//前后分离开发返回json数据
                .successHandler(new MyAuthenticationSuccessHandler())
                .and()
                .csrf().disable();//禁止跨域请求保护
    }
```

结果:

```json
{
	"msg": "验证成功",
	"code": 200,
	"authentication": {
		"authorities": [ ],
		"details": {
			"remoteAddress": "0:0:0:0:0:0:0:1",
			"sessionId": "F9EF27C22A719BFD718A9F515468CFEA"
			},
		"authenticated": true,
		"principal": {
			"password": null,
			"username": "root",
			"authorities": [ ],
			"accountNonExpired": true,
			"accountNonLocked": true,
			"credentialsNonExpired": true,
			"enabled": true
			},
		"credentials": null,
		"name": "root"
	}
}
```



## 4.自定义认证失败处理

当输入错误的账号和密码后，通过debug可以发现

![image-20220909183008747](img.assets\image-20220909183008747.png)

unsuccessfulAuthentication中的`failureHandler.onAuthenticationFailure`方法执行

```java
protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
			AuthenticationException failed) throws IOException, ServletException {
		SecurityContextHolder.clearContext();
		this.logger.trace("Failed to process authentication request", failed);
		this.logger.trace("Cleared SecurityContextHolder");
		this.logger.trace("Handling authentication failure");
		this.rememberMeServices.loginFail(request, response);
		this.failureHandler.onAuthenticationFailure(request, response, failed);
	}
```

![image-20220909184058791](img.assets\image-20220909184058791.png)

进入saveException方法，可以发现，将身份验证异常存入到session中:

![image-20220909184302286](img.assets\image-20220909184302286.png)



> **使用redirect进行跳转**，将信息存入到`session`中
>
> 若使用forward跳转，将信息存入到`request`中



属性key：

```java
public static final String AUTHENTICATION_EXCEPTION = "SPRING_SECURITY_LAST_EXCEPTION";
```



**默认使用redirect进行跳转**

![image-20220909185825170](img.assets\image-20220909185825170.png)



### 4.1 failureForwardUrl 

failureForwardUrl 认证失败后通过 `forward` 方式跳转,**将信息存入request中**

```java
 http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login.html")
                .loginProcessingUrl("/doLogin")
                .usernameParameter("name")
                .passwordParameter("passwd")
     			//认证失败后通过 forward 方式跳转
                .failureForwardUrl("/login.html")
                .and()
                .csrf().disable();
```

![image-20220909190527599](img.assets\image-20220909190527599.png)

在**request**中获取错误信息

```html
<h2 th:text="${#request.getAttribute('SPRING_SECURITY_LAST_EXCEPTION')+'--->request'}"></h2>
```

```
org.springframework.security.authentication.BadCredentialsException: 用户名或密码错误--->request
```



### 4.2  failureUrl

failureUrl 认证失败后通过 `redirect` 方式跳转 ，**将信息存入session中**

```java
 http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login.html")
                .loginProcessingUrl("/doLogin")
                .usernameParameter("name")
                .passwordParameter("passwd")
     			//认证失败后通过 redirect 方式跳转
                .failureUrl("/login.html")
                .and()
                .csrf().disable();
```

![image-20220909190739992](img.assets\image-20220909190739992.png)

在**session**中获取错误信息

```html
<h2 th:text="${#session.getAttribute('SPRING_SECURITY_LAST_EXCEPTION')+'--->session'}"></h2>
```

```txt
org.springframework.security.authentication.BadCredentialsException: 用户名或密码错误--->session
```



### 4.3 failureHandler 主要使用

以上两种方式都不适合前后端分离开放，这时需要使用 **failureHandler**方法经行处理

```java
	public final T failureHandler(AuthenticationFailureHandler authenticationFailureHandler) {
		this.failureUrl = null;
		this.failureHandler = authenticationFailureHandler;
		return getSelf();
	}
```

![image-20220909193402648](img.assets\image-20220909193402648.png)

自定义AuthenticationFailureHandler：

```java
/**
 * 自定义验证失败后的处理
 */
public class MyAuthenticationFailureHandle implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) 
            throws IOException, ServletException {
        Map<String, Object> map = new HashMap<>();
        map.put("msg", "验证失败"+exception.getMessage());
        map.put("code", 200);
        response.setContentType("application/json;charset=UTF-8");
        final String asString = new ObjectMapper().writeValueAsString(map);
        response.getWriter().print(asString);
    }
}
```

使用：

```java
 http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login.html")
                .loginProcessingUrl("/doLogin")
                .usernameParameter("name")
                .passwordParameter("passwd")
     			//前后端分离自定义验证错误处理
                .failureHandler(new MyAuthenticationFailureHandle())
                .and()
                .csrf().disable();
```

```json
{
	"msg": "验证失败用户名或密码错误",
	"code": 200
}
```



## 5.注销登录

LogoutFilter，处理注销登录，是springsecurity默认开启的

在禁止CSRF之后，在浏览器中使用`get`方式发送`/logout`即可注销登录

![image-20220911160358325](img.assets\image-20220520145953421.png)

自定义注销登录

```java
http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                //提供注销支持
                .logout()
                //触发注销的URL（默认为“/logout”）。如果启用了CSRF保护（默认），则请求也必须是POST。
                //这意味着默认情况下，需要POST“/logout”来触发注销。如果禁用CSRF保护，则允许任何HTTP方法。
                .logoutUrl("/logout")
                //注销时使HttpSession无效。
                .invalidateHttpSession(true)
                //是否应在注销时清除身份验证
                .clearAuthentication(true)
                //注销后要重定向到的URL,默认值为“/login？logout”
                .logoutSuccessUrl("/login.html")
                .and()
                .csrf().disable();//禁止跨域请求保护
```

> logout() 开启注销登录配置
>
> logoutUrl() 指定退出登录请求地址，路径为/logout
>
> invalidateHttpSession（） 退出时是否使session失效，默认为true
>
> clearAuthentication（） 退出时是否清除认证信息，默认为true
>
> logoutSuccessUrl 退出登录后跳转地址



### 5.1 配置多个注销登录请求

想使用HTTP GET，可以使用`logoutRequestMatcher`（新的AntPathRequestMatcherLogoutURL，“GET”)

```java
http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                //提供注销支持
    			.login()
                //OrRequestMatcher 使用任意一种方式退出
                .logoutRequestMatcher(new OrRequestMatcher(
                        //退出登录url为/aa,方式为GET
                        new AntPathRequestMatcher("/aa", "GET"),
                        //退出登录url为/bb,方式为POST
                        new AntPathRequestMatcher("/bb", "POST")
                ))
                //注销时使HttpSession无效。
                .invalidateHttpSession(true)
                //是否应在注销时清除身份验证
                .clearAuthentication(true)
                //注销后要重定向到的URL,默认值为“/login？logout”
                .logoutSuccessUrl("/login.html")
                .and()
                .csrf().disable();//禁止跨域请求保护
```



### 5.2 logoutSuccessHandler

如果是前后端分离开发，注销成功之后就不需要跳转页面了。只需要将注销成功后的信息返回给前端即可，这时可以使用` logoutSuccessHandler`来自定义注销之后的信息

```java
	public LogoutConfigurer<H> logoutSuccessHandler(LogoutSuccessHandler logoutSuccessHandler) {
		this.logoutSuccessUrl = null;
		this.customLogoutSuccess = true;
		this.logoutSuccessHandler = logoutSuccessHandler;
		return this;
	}
```

自定义实现

```java
/**
 * 自定义验证失败后的处理
 */
public class MyAuthenticationFailureHandle implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception)
            throws IOException, ServletException {
        Map<String, Object> map = new HashMap<>();
        map.put("msg", "验证失败" + exception.getMessage());
        map.put("code", 200);
        response.setContentType("application/json;charset=UTF-8");
        final String asString = new ObjectMapper().writeValueAsString(map);
        response.getWriter().print(asString);
    }
}

```

使用：

```java
http.authorizeRequests()
                .mvcMatchers("/index", "/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                //提供注销支持
    			.login()
     			.logoutSuccessHandler(new MyLogoutSuccessHandler())
                //OrRequestMatcher 使用任意一种方式退出
                .logoutRequestMatcher(new OrRequestMatcher(
                        //退出登录url为/aa,方式为GET
                        new AntPathRequestMatcher("/aa", "GET"),
                        //退出登录url为/bb,方式为POST
                        new AntPathRequestMatcher("/bb", "POST")
                ))
                //注销时使HttpSession无效。
                .invalidateHttpSession(true)
                //是否应在注销时清除身份验证
                .clearAuthentication(true)
                .and()
                .csrf().disable();//禁止跨域请求保护
```

```json
{
    "msg":"注销成功",
    "code":200,
    "authentication":{
        "authorities":[

        ],
        "details":{
            "remoteAddress":"0:0:0:0:0:0:0:1",
            "sessionId":"72B9800FA0E77B322EF0EC11A238C76B"
        },
        "authenticated":true,
        "principal":{
            "password":null,
            "username":"root",
            "authorities":[

            ],
            "accountNonExpired":true,
            "accountNonLocked":true,
            "credentialsNonExpired":true,
            "enabled":true
        },
        "credentials":null,
        "name":"root"
    }
}
```



# 五.获取用户信息

## 5.1  SecurityContextHolder

`SecurityContextHolder` <font color="red">用来获取登录之后用户信息</font>。Spring Security 会将登录用户数据保存在 Session 中。但是，为了使用方便,Spring Security在此基础上还做了一些改进，其中最主要的一个变化就是**线程绑定**。当用户登录成功后,Spring Security 会将登录成功的**用户信息(Authentication类)保存到 SecurityContextHolder** 中。

`SecurityContextHolder` 中的数据保存<font color="red">默认是通过ThreadLocal 来实现的</font>，使用 ThreadLocal 创建的变量只能被当前线程访问，不能被其他线程访问和修改，也就是<font color="red">用户数据和请求线程绑定在一起</font>。

当**登录请求处理完毕后**，Spring Security 会将 SecurityContextHolder 中的<font color="red">数据拿出来保存到 Session 中，同时将 SecurityContexHolder 中的数据清空</font>。以后每当**有请求到来时**，Spring Security 就会<font color="red">先从 Session 中取出用户登录数据，保存到 SecurityContextHolder 中</font>，方便在该请求的后续处理过程中使用，同时**在请求结束时**将 SecurityContextHolder 中的数据拿出来保存到 Session 中，然后将 Security SecurityContextHolder 中的数据清空。

实际上，**`SecurityContextHolder`中存储的是`SecurityContext`,`SecurityContext`中存储的是Authenication**

![image-20220911170012777](img.assets\image-20220911170012777.png)

SecurityContextHolder采用了典型的策略设计模式

```java
public class SecurityContextHolder {
    //本地线程模式,默认
	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
	//父子线程模式
	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
	//全局模式
	public static final String MODE_GLOBAL = "MODE_GLOBAL";
	//初始化模式
	private static final String MODE_PRE_INITIALIZED = "MODE_PRE_INITIALIZED";

	public static final String SYSTEM_PROPERTY = "spring.security.strategy";

	private static String strategyName = System.getProperty(SYSTEM_PROPERTY);

	private static SecurityContextHolderStrategy strategy;
    
    private static void initializeStrategy() {
		if (MODE_PRE_INITIALIZED.equals(strategyName)) {
			Assert.state(strategy != null, "When using " + MODE_PRE_INITIALIZED
					+ ", setContextHolderStrategy must be called with the fully constructed strategy");
			return;
		}
		if (!StringUtils.hasText(strategyName)) {
			// Set default
			strategyName = MODE_THREADLOCAL;
		}
		if (strategyName.equals(MODE_THREADLOCAL)) {
			strategy = new ThreadLocalSecurityContextHolderStrategy();
			return;
		}
		if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
			strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
			return;
		}
		if (strategyName.equals(MODE_GLOBAL)) {
			strategy = new GlobalSecurityContextHolderStrategy();
			return;
		}
		// Try to load a custom strategy
		try {
			Class<?> clazz = Class.forName(strategyName);
			Constructor<?> customStrategy = clazz.getConstructor();
			strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
		}
		catch (Exception ex) {
			ReflectionUtils.handleReflectionException(ex);
		}
        
        //获取 SecurityContext 
        public static SecurityContext getContext() {
			return strategy.getContext();
		}
	}
```

- `MODE_THREADLOCAL`:这种存放策略是**将SecurityContext存放到ThreadLocal中**，ThreadLocal的特点是在哪个线程中存就在哪个线程中取，在默认情况下，一个请求无论经历多少个Filter到达Selvet，都在一个线程中，也是SpringSecurity默认的存储策略。如果在实际业务中，在子线程中获取用户的登录数据，将无法获取。

- `MODE_INHERITABLETHREADLOCAL`:**这种存储方式适用于多线程环境**，如果希望在子线程中获取用户登录的数据，可以使用这种存储方式。

- `MODE_GLOBAL`:这种存储方式是**将数据保存在这个静态变量中**，很少使用。



## 5.2 SecurityContextHolderStrategy

SecurityContextHolderStrategy接口用来定义存储策略方法

```java
public interface SecurityContextHolderStrategy {
    // 使用该方法来清除存储的SecurityContext对象
	void clearContext();
	//使用该方法来获取存储的SecurityContext对象
	SecurityContext getContext();
	//使用该方法来获取修改的SecurityContext对象
	void setContext(SecurityContext context);
	////使用该方法来获取创建一个空的的SecurityContext对象
	SecurityContext createEmptyContext();
}
```

![image-20220911172534802](img.assets\image-20220911172534802.png)



## 5.3 获取用户信息

### 5.3.1 默认从本地线程中获取信息

通过SecurityContextHolder可知，先获取SecurityContext，在获取Authentication

```java
@GetMapping("/hello")
public String hello() {
    //获取认证信息
    final Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    System.out.println("身份信息:" + authentication.getPrincipal());
    System.out.println("权限信息:" + authentication.getAuthorities());
    System.out.println("凭证:" + authentication.getCredentials());
    System.out.println("用户名:" + authentication.getName());
    System.out.println("其它信息:" + authentication.getDetails());
    return "hello spring security";
}
```

```txt
身份信息:org.springframework.security.core.userdetails.User [Username=root, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[]]
权限信息:[]
凭证:null
用户名:root
其它信息:WebAuthenticationDetails [RemoteIpAddress=0:0:0:0:0:0:0:1, SessionId=6C062B250709F5A764A93A3D485505D9]
```

getPrincipal()方法返回的是一个User对象，可以将其强转为User对象

```java
import org.springframework.security.core.userdetails.User;
User user = (User) authentication.getPrincipal();
System.out.println(user.getUsername());
System.out.println(user.getPassword());
```

```
root
null
```

**默认情况下无法从子线程中获取用户信息**

```java
new Thread(() -> {
            final Authentication authentication1 = SecurityContextHolder.getContext().getAuthentication();
            System.out.println("子线程"+authentication1);
}).start();
```

```
子线程null
```



### 5.3.2 从子线程中获取用户信息

将SecurityContextHolder的策略模式替换为`MODE_INHERITABLETHREADLOCAL`

```java
public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
public static final String SYSTEM_PROPERTY = "spring.security.strategy";
private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
```

通过源码可知，SecurityContextHolder是通过系统配置`System.getProperty()`来替换不同的策略模式的

修改系统配置

```
-Dspring.security.strategy=MODE_INHERITABLETHREADLOCAL
```

![image-20220911175233374](img.assets\image-20220911175233374.png)

启动测试:

```java
 new Thread(() -> {
            final Authentication authentication1 = SecurityContextHolder.getContext().getAuthentication();
            System.out.println("子线程:" + authentication1);
        }).start();
```

```
子线程:
	UsernamePasswordAuthenticationToken [Principal=org.springframework.security.core.userdetails.User [Username=root, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[]], Credentials=[PROTECTED], Authenticated=true, Details=WebAuthenticationDetails [RemoteIpAddress=0:0:0:0:0:0:0:1, SessionId=C1D9789EE5EE15FBAA8CD30ABA7C2EDB], Granted Authorities=[]]
```



## 5.4 在页面中直接获取用户信息

引入thymeleaf直接操作springsecurity依赖

```xml
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
        </dependency>
```

在页面中添加命名空间

```html
<html lang="zh" xmlns:th="http://www.thymeleaf.org" 
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security5">
```

获取：

```html
<ul>
    <li sec:authentication="principal.username"></li>
    <li sec:authentication="principal.authorities"></li>
    <li sec:authentication="principal.accountNonExpired"></li>
    <li sec:authentication="principal.accountNonLocked"></li>
    <li sec:authentication="principal.password"></li>
    <li sec:authentication="principal.credentialsNonExpired"></li>
</ul>
```



# 六.自定义认证数据源

## 6.1 认证流程分析

![abstractauthenticationprocessingfilter](img.assets\abstractauthenticationprocessingfilter.png)

- 当用户提交他们的凭据时， AbstractAuthenticationProcessingFilter 从 HttpServletRequest 创建一个 Authentication 以进行身份验证。创建的 Authentication 类型取决于 AbstractAuthenticationProcessingFilter 的子类。例如，UsernamePasswordAuthenticationFilter 根据在 HttpServletRequest 中提交的用户名和密码创建 UsernamePasswordAuthenticationToken。

- 接下来，将 Authentication 传递给 **AuthenticationManager 进行身份验证**。

- 如果身份验证失败，则失败:

  ​		1.SecurityContextHolder 被清除。

  ​		2.调用 RememberMeServices.loginFail。如果记住我没有配置，这是一个空操作。 

  ​		3.AuthenticationFailureHandler 被调用。

- 如果身份验证成功，则为 Success。

​			1. SessionAuthenticationStrategy 收到新登录通知。

​		    2.Authentication 在 SecurityContextHolder 上设置。稍后 SecurityContextPersistenceFilter 将 SecurityContext 保存到 HttpSession。 

​			3.调用 RememberMeServices.loginSuccess。如果记住我没有配置，这是一个空操作。

​			4.ApplicationEventPublisher 发布一个 InteractiveAuthenticationSuccessEvent。

​		    5. AuthenticationSuccessHandler 被调用。



## 6.2 认证核心三类

​	 **AuthenticationManager是认证的核心类，但在实际上底层真正认证的是ProviderManager和 AuthenticationProvider**

- `AuthenticationManager`是一个认证管理器，它定义了springSecurity过滤器要执行的认证操作。
- `ProviderManager`是AuthenticationManager的实现类，SpringSecurity默认使用的是providerManager。
- `AuthenticationProvider`是针对不同身份类型执行的具体的身份认证（认证方式）。



### 6.2.1 AuthenticationManager和ProviderManager

![image-20220110103518334](img.assets\image-20220110103518334.png)

ProviderManager 是 AuthenticationManager 最常用的实现。 



### 6.2.2 ProviderManager和 AuthenticationProvider

​	在springSecurity中，允许系统同时支持多种不同的认证方式，例如同时支持用户名密码，RememberMe认证，手机号码动态认证等，而不同的认证方式对应了不同的 AuthenticationProvider ，所以一个完整的认证流程可能有多个AuthenticationProvider实现。

​	多个AuthenticationProvider组成一个列表，交给ProviderManager代理，也就是说，在ProviderManager 中存在一个AuthenticationProvider列表，在ProviderManager中遍历列表中的的每一个AuthenticationProvider去执行身份认证，最终得到认证结果（只要有一个通过就成功）。

```java
	
	private List<AuthenticationProvider> providers = Collections.emptyList();	
	
	//遍历列表中的的每一个AuthenticationProvider去执行身份认证
	for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}
			if (logger.isTraceEnabled()) {
				logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
						provider.getClass().getSimpleName(), ++currentPosition, size));
			}
			try {
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException ex) {
				prepareException(ex, authentication);
				throw ex;
			}
			catch (AuthenticationException ex) {
				lastException = ex;
			}
		}
		
		/**/
		public List<AuthenticationProvider> getProviders() {
			return this.providers;
		}
```

​	ProviderManager还本身也可以配置一个AuthenticationManager作为Parent（由于AuthenticationManager的实现只有ProviderManager一个，也可以看作配置了一个ProviderManager为parent ），这样当**ProviderManager认证失败后，就可以进入到parent中进行再次认证**。理论上说，ProviderManager的parent可以实任何类型的AuthenticationManager，但是通常由ProviderManager来扮演patent，也就是说由ProviderManager来扮演ProviderManager的父亲。

```java
private AuthenticationManager parent;

if (result == null && this.parent != null) {
			try {
                //由父类的ProviderManager再次进行验证
				parentResult = this.parent.authenticate(authentication);
				result = parentResult;
			}
			catch (ProviderNotFoundException ex) {
			}
			catch (AuthenticationException ex) {
				parentException = ex;
				lastException = ex;
			}
		}
```

​	ProviderManager本身也可以由很多个，**多个ProviderManager公用一个parent**。有时一个应用程序有受保护资源的逻辑组(列如，所有符合路径模式的网络资源，如/api/*)，每个组都可以由自己专用的AuthenticationProvider。通常，每一个组都是一个ProviderManager，它们共享一个父级，**父级是一个全局资源**，作为所有提供者的后备资源。

![image-20220912185413698](img.assets\image-20220912185413698.png)



**官网说明：**

ProviderManager 委托给一个 AuthenticationProviders 列表。每个 AuthenticationProvider 都有机会指示身份验证应该成功、失败或指示它不能做出决定并允许下游 AuthenticationProvider 做出决定。如果配置的 AuthenticationProviders 都不能进行身份验证，则身份验证将失败并出现 ProviderNotFoundException，这是一个特殊的 AuthenticationException，表明 ProviderManager 未配置为支持传递给它的身份验证类型。

![providermanager](img.assets\providermanager.png)

实际上，每个 AuthenticationProvider 都知道如何执行特定类型的身份验证。例如，一个 AuthenticationProvider 可能能够验证用户名/密码，而另一个可能能够验证 SAML 断言。这允许每个 AuthenticationProvider 执行非常特定类型的身份验证，同时支持多种类型的身份验证并且只公开单个 AuthenticationManager bean。 ProviderManager 还允许配置一个可选的父 AuthenticationManager，在没有 AuthenticationProvider 可以执行身份验证的情况下进行咨询。父级可以是任何类型的 AuthenticationManager，但它通常是 ProviderManager 的一个实例。

![providermanager parent](img.assets\providermanager-parent.png)

事实上，多个实例可能共享同一个 parent 。这在有多个 SecurityFilterChain 实例具有一些共同身份验证（共享父级）但也有不同身份验证机制（不同实例）的场景中有些常见。

![providermanagers parent](.\img.assets\providermanagers-parent.png)

默认情况下，将尝试从成功的身份验证请求返回的对象中清除任何敏感的凭据信息。这可以防止密码等信息在 `ProviderManager` `Authentication` `HttpSession` 中保留的时间超过必要的时间 。

当您使用用户对象的缓存时，这可能会导致问题，例如，为了提高无状态应用程序的性能。如果 包含对缓存中的对象（例如实例）的引用并且这已删除其凭据，则将不再可能针对缓存的值进行身份验证。如果您使用缓存，则需要考虑到这一点。一个明显的解决方案是首先制作对象的副本，无论是在缓存实现中还是在创建返回对象的方法中。或者，您可以禁用以上的属性。



### 6.2.3  默认认证数据源获取

通过debug可知,在默认情况下，使用表单(`UsernamePasswordAuthenticationToken`)的方式进行验证,而默认验证方式AuthenticationProvider是`AnonymousAuthenticationProvider`

![image-20220912190926341](img.assets\image-20220912190926341.png)

验证不支持，直接跳出循环,进入parent中进行验证

![image-20220912191510079](img.assets\image-20220912191510079.png)

在parent的AuthenticationProvider中可以看到，验证的方式为`DaoAuthenticationProvider`

![image-20220912191643338](img.assets\image-20220912191643338.png)

该验证支持表单验证,并且开始认证

![image-20220912192336818](img.assets\image-20220912192336818.png)

进入到`authenticate`方法中，可以发现是**AbstractUserDetailsAuthenticationProvider**类中的方法,其子类并没有重写该方法

![image-20220912193018038](img.assets\image-20220912193018038.png)

![image-20220912192818758](img.assets\image-20220912192818758.png)

在AbstractUserDetailsAuthenticationProvider中`retrieveUser`方法为抽像方法，所有调用的是子类**DaoAuthenicationProvider**重写后的`retrieveUser`方法 

```java
protected abstract UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
      throws AuthenticationException;
```

在`retrieveUser`方法中，我们可以看到调用了**UserDetailsService**类,而该类默认是基于内存`InMemoryUserDerailsManager`实现的，并且`loadUserByUsername`方法只对用户名进行访问，没有对密码进行访问

![image-20220912193442016](img.assets\image-20220912193442016.png)

在`additionalAuthenticationChecks`方法中进行密码的验证

![image-20220912194159900](img.assets\image-20220912194159900.png)

根据不同的加密算法进行匹配

![image-20220912194305692](img.assets\image-20220912194305692.png)

就此完成了所有的默认验证流程



所以：

  在默认情况下`AuthenticationProvider`是由`DaoAuthenicationProvider`类来进行实现的，在通过`DaoAuthenicationProvider`认证时又通过`UserDetalisService`完成数据校验的

![img](img.assets\认证流程.png)



### 6.2.4 总结

  `AuthenticationManager`是认证管理器，在Spring Security中有**全局**的`AuthenticationManager`,也可以有**局部**的`AuthenticationManager`。全局的AuthenticationManager用来对全局认证进行处理，局部的AuthenticationManager用来对某些特殊的资源进行认证处理。无论是全局的还是局部的都是由ProviderManger进行实现，每一个ProviderManger中都代理一个AuthenticationManager的列表，列表中每一个实现代表一种身份的认证方式，**认证时底层需要调用数据源`UserDetailsService`来实现**。



## 6.3 全局配置AuthenticationManager

全局定义的两种方式

### 6.3.1 springboot默认配置

默认自动配置全局AuthenticationManager，默认找当前项目中是否存在自定义的`UserDetailService`实例，若有，则将当前项目的`UserDetailService`设置为数据源。**使用时在工厂中直接在代码注入即可**

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

   ... // web stuff here

  @Autowired
  public void initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

配置之后将使用自定义之后的用户名和密码:

```java
    @Autowired
    public void initialize(AuthenticationManagerBuilder builder) throws Exception {
        System.out.println("springboot 默认配置" + builder);
        InMemoryUserDetailsManager userDetailsService = new InMemoryUserDetailsManager();
        //{noop} 明文密码
        userDetailsService.createUser(User.withUsername("aaa").password("{noop}123").roles("admin").build());
        builder.userDetailsService(userDetailsService);
    }
```

通过`UserDetailsServiceAutoConfiguation`自动配置类可知，只要项目中有 `UserDetailsService`接口的实例，就会采用自定义的配置,所以以上代码等同于,

**只需要配置UserDetailService的Bean即可**

```java
@Bean
public UserDetailsService userDetailsService() {
    InMemoryUserDetailsManager userDetailsService = new InMemoryUserDetailsManager();
    userDetailsService.createUser(User.withUsername("aaa").password("{noop}123").roles("admin").build());
    return userDetailsService;
}
```



### 6.3.2 自定义AuthenticationManager

**自定义AuthenticationManager之后会对springboot默认配置的AuthenticationManager实现覆盖**，推荐使用自定义AuthenticationManager

**一旦使用自定义AuthenticationManager，必须在实现中指定认证的数据源UserDetailService**。

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  @Autowired
  DataSource dataSource;

   ... // web stuff here

  @Override
  public void configure(AuthenticationManagerBuilder builder) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

```

```java
    @Override
    //使用该方式必须手动指定userDetailsService
    public void configure(AuthenticationManagerBuilder builder) throws Exception {
        builder.userDetailsService(userDetailsService());
    }
```

这种方式创建的AuthenticationManager对象工厂**是内部本地的一个AuthenticationManager对象**，这将是全局方法的子对象。在 Spring Boot 应用程序中，您可以将全局 bean 放入另一个 bean，但您不能对本地 bean 执行此操作，除非您自己显式公开它。

```
A component required a bean of type 'org.springframework.security.authentication.AuthenticationManager' that could not be found.
```

```java
    @Override
    //用来将自定义的AuthenticationManager在工厂中进行暴露，可以在任何位置注入
	@Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
```



#### 6.3.2.1 自定义数据源

User类中定义了一些字段可以用作数据库表构建,其中`username`,`password`为必须字段

```java
	private String password;

	private final String username;

	private final Set<GrantedAuthority> authorities;

	private final boolean accountNonExpired;

	private final boolean accountNonLocked;

	private final boolean credentialsNonExpired;

	private final boolean enabled;
```

用户表:

```mysql
create table user
(
    id                      int          not null
        primary key,
    username                varchar(32)  null,
    password                varchar(255) null,
    enabled                 tinyint(1)   null,
    account_non_expired     tinyint(1)   null,
    account_non_locked      tinyint(1)   null,
    credentials_non_expired tinyint(1)   null
);
```

权限表:

```mysql
create table role
(
    id      int         not null
        primary key,
    name    varchar(32) null,
    name_zh varchar(32) null
);
```

用户和权限关联表

```mysql
create table user_role
(
    id  int not null
        primary key,
    uid int null,
    rid int null
);
```



##### 6.3.2.1.1 用户实体类实现UserDetails接口

在UserDetailsService接口中`loadUserByUsername`方法返回UserDetails接口的实现类，该类保存了用户的信息

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

```java
@Data
public class User implements UserDetails {

    private Integer id;

    private String password;

    private String username;
    //权限信息
    private Set<GrantedAuthority> authorities;
    //用户是否过期
    private boolean accountNonExpired;
    //用户是否锁定
    private boolean accountNonLocked;
    //凭证是否过期
    private boolean credentialsNonExpired;
    //能否使用
    private boolean enabled;
    //保存用户权限信息
    private List<Role> roles;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Set<SimpleGrantedAuthority> authorities = new HashSet<>();
        roles.forEach(role -> {
            SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(role.getName());
            authorities.add(simpleGrantedAuthority);
        });
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```



##### 6.3.2.1.2 自定义UserDetailsService类，实现UserDetailsService接口

在此处直接使用字段注入为null，不清楚原因！！

```java
@Component
@Slf4j
public class MyUserDetailService implements UserDetailsService {
	
    private final UserMapper userMapper;

    @Autowired
    public MyUserDetailService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userMapper.loadUserByUsername(username);
        if (Objects.isNull(user)) {
            throw new UsernameNotFoundException("用户名未找到");
        }
        final Integer id = user.getId();
        List<Role> roles = userMapper.loadUserRole(id);
        user.setRoles(roles);
        log.info("" + user);
        return user;
    }
}
```



##### 6.3.2.1.3 自定义AuthenticationManager

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    private final MyUserDetailService myUserDetailService;

    @Autowired
    public WebSecurityConfigurer(MyUserDetailService myUserDetailService) {
        this.myUserDetailService = myUserDetailService;
    }

    @Override
    public void configure(AuthenticationManagerBuilder builder) throws Exception {
        builder.userDetailsService(myUserDetailService);
    }

    @Override
    //用来将自定义的AuthenticationManager在工厂中进行暴露，可以在任何位置注入
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
 }
```



##### 6.3.2.1.4 SQL

```mysql
	<select id="loadUserByUsername" resultType="com.example.springsecurity.domain.User">
        select id, username, password, enabled, account_non_expired, account_non_locked, credentials_non_expired
        from user
        where username = #{username}
    </select>

    <select id="loadUserRole" resultType="com.example.springsecurity.domain.Role">
        select r.id, name, name_zh nameZh
        from role r,
             user_role ur
        where r.id = ur.id
          and ur.uid = #{id}
    </select>
```



#### 6.3.2.2 web开发认证

自定义认证规则和数据源

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final MyUserDetailsService myUserDetailsService;

    @Autowired
    public SecurityConfig(MyUserDetailsService myUserDetailsService) {
        this.myUserDetailsService = myUserDetailsService;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailsService);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login.html")
                .usernameParameter("username")
                .passwordParameter("password")
                .loginProcessingUrl("/doLogin")
                .defaultSuccessUrl("/index.html")
                .and()
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login.html")
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .and()
                .csrf().disable();
    }
}
```

数据源

```java
@Component
public class MyUserDetailsService implements UserDetailsService {

    private final UserMapper userMapper;

    @Autowired
    public MyUserDetailsService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userMapper.loadUserByUsername(username);
        if (Objects.isNull(user)) {
            throw new UsernameNotFoundException("用户不存在");
        }
        final Integer uid = user.getId();
        List<Role> roles = userMapper.selectRolesByUid(uid);
        user.setRoles(roles);
        return user;
    }
}
```

数据实体类

```java
@Data
public class User implements UserDetails {
    private String username;
    private Set<GrantedAuthority> authorities;
    private boolean accountNonExpired;
    private boolean accountNonLocked;
    private boolean credentialsNonExpired;
    private boolean enabled;
    private Integer id;
    private String password;
    private List<Role> roles;
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Set<SimpleGrantedAuthority> authorities = new HashSet<>();
        roles.forEach(role -> {
            SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(role.getName());
            authorities.add(simpleGrantedAuthority);
        });
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```

```java
@Data
public class Role implements Serializable {
    private Integer id;
    private String name;
    private String nameZh;
}
```



#### 6.3.2.3 前后端分离开发

在默认情况下，表单的验证都是交给`UsernamePasswordAuthenticationFilter`过滤器来进行的

```java
	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		username = (username != null) ? username : "";
		username = username.trim();
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
		setDetails(request, authRequest);
		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

在源码中我们可以看到，方式请求的方式为POST，通过`request.getParameter()`方法来获取表单数据

但是在前后端分离的情况下，都是通过JSON来传递数据，所以我们**需要对`UsernamePasswordAuthenticationFilter`中的`attemptAuthentication`进行重写，而且要保证SpringSecurity过滤器的顺序不改变**



**前后端分离开发要求:**

- 自定义Filter重写UsernamePasswordAuthenticationFilter中的attemptAuthentication方法，并且**将默认的UsernamePasswordAuthenticationFilter替换为自定义的Filter**

- 验证请求方式为POST
- 从JSON中获取username和password
- 进行验证



##### 6.3.2.3.1 重写attemptAuthentication方法

```java
/**
 * 自定义前后端分离开发
 */
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        //1.先判度是否为POST请求
        if (!request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        //2.判断请求的方式是否为JSON
        if (request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_VALUE)) {
            //从json中获取用户名和密码进行认证
            try {
                final Map<String, String> userInfo = new ObjectMapper().readValue(request.getInputStream(), Map.class);
                //不要将用户名和密码写死，使用父类中的方法获取
                //可以在注入工厂时进行手动设置
                final String username = userInfo.get(this.getUsernameParameter());
                final String password = userInfo.get(this.getPasswordParameter());
                UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
                setDetails(request, authRequest);
                //可以在工厂进行自定义认证规则
                return this.getAuthenticationManager().authenticate(authRequest);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return super.attemptAuthentication(request, response);
    }
}
```

##### 6.3.2.3.2 替换自定义Filter

**替换为自定义过滤器之后默认配置将不在生效，需要在自定义过滤器中定义认证请求地址以及认证规则等**

```java
 http.addFilterAt(loginFilter(), UsernamePasswordAuthenticationFilter.class);
//将自定义的过滤器和UsernamePasswordAuthenticationFilter进行替换，保持过滤器链顺序不变
//At 用某个过滤器替换指定的过滤器
//After 在已知Filter类之一之后添加Filter器
//Before 在已知Filter类之一之前添加Filter器
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final MyUserDetailService myUserDetailService;

    @Autowired
    public SecurityConfig(MyUserDetailService myUserDetailService) {
        this.myUserDetailService = myUserDetailService;
    }

    @Override
    //自定义数据源
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailService);
    }

    @Override
    @Bean
    //将自定义认证实例暴露到工厂
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    //将自定义的过滤器交给工厂
    @Bean
    public LoginFilter loginFilter() throws Exception {
        final LoginFilter loginFilter = new LoginFilter();
        //认证请求地址
        loginFilter.setFilterProcessesUrl("/doLogin");
        loginFilter.setUsernameParameter("username");
        loginFilter.setPasswordParameter("password");
        //设置自定义认证规则
        loginFilter.setAuthenticationManager(this.authenticationManagerBean());
        //认证成功
        loginFilter.setAuthenticationSuccessHandler((request, response, authentication) -> {
            Map<String, Object> map = new HashMap<>();
            map.put("msg", "验证成功");
            map.put("code", HttpStatus.OK.value());
            map.put("authentication", authentication);
            response.setContentType("application/json;charset=UTF-8");
            final String asString = new ObjectMapper().writeValueAsString(map);
            response.getWriter().print(asString);
        });
        //认证失败
        loginFilter.setAuthenticationFailureHandler((request, response, exception) -> {
            Map<String, Object> map = new HashMap<>();
            map.put("msg", exception.getMessage());
            map.put("code", HttpStatus.INTERNAL_SERVER_ERROR.value());
            response.setContentType("application/json;charset=UTF-8");
            final String asString = new ObjectMapper().writeValueAsString(map);
            response.getWriter().print(asString);
        });
        return loginFilter;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //退出认证
                .logout()
                //退出登录方式
                .logoutRequestMatcher(new OrRequestMatcher(
                        new AntPathRequestMatcher("/logout", HttpMethod.DELETE.name()),
                        new AntPathRequestMatcher("/logout", HttpMethod.GET.name())
                ))
                .logoutSuccessHandler((request, response, authentication) -> {
                    Map<String, Object> map = new HashMap<>();
                    map.put("msg", "退出成功");
                    map.put("code", HttpStatus.OK.value());
                    map.put("authentication", authentication);
                    response.setContentType("application/json;charset=UTF-8");
                    final String asString = new ObjectMapper().writeValueAsString(map);
                    response.getWriter().print(asString);
                })
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .and()
                .csrf().disable();
        http.addFilterAt(loginFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```

##### 6.3.2.3.3 自定义数据源和数据实体

数据源

```java
@Component
public class MyUserDetailService implements UserDetailsService {

    private final UserMapper userMapper;

    @Autowired
    public MyUserDetailService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userMapper.loadUserByUsername(username);
        if (Objects.isNull(user)) {
            throw new UsernameNotFoundException("用户不存在");
        }
        final Integer uid = user.getId();
        List<Role> roles = userMapper.selectRolesByUid(uid);
        user.setRoles(roles);
        return user;
    }
}
```

数据实体

```java
@Data
public class User implements UserDetails {

    private Integer id;

    private String password;

    private String username;

    private Set<GrantedAuthority> authorities;

    private boolean accountNonExpired;

    private boolean accountNonLocked;

    private boolean credentialsNonExpired;

    private boolean enabled;

    private List<Role> roles;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Set<SimpleGrantedAuthority> authorities = new HashSet<>();
        roles.forEach(role -> {
            SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(role.getName());
            authorities.add(simpleGrantedAuthority);
        });
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```

```java
@Data
public class Role implements Serializable {
    private Integer id;
    private String name;
    private String nameZh;
}
```



### 6.3.3 验证码认证

#### 6.3.3.1 获取验证码

```xml
        <dependency>
            <groupId>com.github.penggle</groupId>
            <artifactId>kaptcha</artifactId>
            <version>2.3.2</version>
        </dependency>
```

验证码配置:

```java
@Configuration
public class KaptchaConfig {
    @Bean
    public DefaultKaptcha defaultKaptcha() {
        DefaultKaptcha dk = new DefaultKaptcha();
        Properties properties = new Properties();
        // 图片边框
        properties.setProperty("kaptcha.border", "yes");
        // 边框颜色
        properties.setProperty("kaptcha.border.color", "105,179,90");
        // 字体颜色
        properties.setProperty("kaptcha.textproducer.font.color", "red");
        // 图片宽
        properties.setProperty("kaptcha.image.width", "110");
        // 图片高
        properties.setProperty("kaptcha.image.height", "40");
        // 字体大小
        properties.setProperty("kaptcha.textproducer.font.size", "30");
        // session key
        properties.setProperty("kaptcha.session.key", "code");
        // 验证码长度
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        //验证码选项
        properties.setProperty("kaptcha.textproducer.char.string", "0123456789");
        // 字体
        properties.setProperty("kaptcha.textproducer.font.names", "宋体,楷体,微软雅黑");
        Config config = new Config(properties);
        dk.setConfig(config);
        return dk;
    }
}
```

获取验证码

```java
import com.google.code.kaptcha.impl.DefaultKaptcha;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.imageio.ImageIO;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.awt.image.BufferedImage;
import java.io.IOException;

@Controller
public class VerifyCodeController {

    private final DefaultKaptcha defaultKaptcha;

    @Autowired
    public VerifyCodeController(DefaultKaptcha defaultKaptcha) {
        this.defaultKaptcha = defaultKaptcha;
    }

    @RequestMapping("/code")
    //获取图片
    public void verifyCode(HttpServletResponse response, HttpSession session) {
        //生成验证码
        final String text = defaultKaptcha.createText();
        //将验证码保持在session中,可以将其保持在Redis中
        session.setAttribute("code", text);
        //生成图片
        final BufferedImage image = defaultKaptcha.createImage(text);
        //将图片返回
        try {
            response.setContentType("image/png");
            final ServletOutputStream outputStream = response.getOutputStream();
            //使用流返回
            ImageIO.write(image, "jpg", outputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    @RequestMapping("/code/base")
    public String verifyCode(HttpServletResponse response, HttpSession session) {
        //生成验证码
        final String text = defaultKaptcha.createText();
        //将验证码保持在session中
        session.setAttribute("code", text);
        //生成图片
        final BufferedImage image = defaultKaptcha.createImage(text);
        //将图片转换为byte数组
        FastByteArrayOutputStream fastByteArrayOutputStream = new FastByteArrayOutputStream();
        try {
            ImageIO.write(image, "jpg", fastByteArrayOutputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //以Base64返回
        return Base64.encodeBase64String(fastByteArrayOutputStream.toByteArray());
    }
}
```

请求放行

```java
 http.authorizeRequests().mvcMatchers("/code").permitAll()
```



#### 6.3.3.2 传统web认证

认证流程，先验证验证码，在验证用户名和密码，所以只需要扩展**UsernamePasswordAuthenticationFilter中的attemptAuthentication方法**，即可

##### 6.3.3.2.1 扩展attemptAuthentication方法

```java
//自定义验证码filter
public class KaptchaFilter extends UsernamePasswordAuthenticationFilter {

    private static final String FROM_KAPTCHA_KEY = "kaptcha";

    private String kaptchaParameter = FROM_KAPTCHA_KEY;

    public String getKaptchaParameter() {
        return kaptchaParameter;
    }

    public void setKaptchaParameter(String kaptchaParameter) {
        this.kaptchaParameter = kaptchaParameter;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        //首先判断是否为post请求
        if (!request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        //从请求中获取验证码
        final String kaptchaCode = request.getParameter(this.getKaptchaParameter());
        //从session中获取验证码
        final String sessionCode = (String) request.getSession().getAttribute("code");
        //比较
        if (!ObjectUtils.isEmpty(kaptchaCode) && !ObjectUtils.isEmpty(sessionCode) && kaptchaCode.equalsIgnoreCase(sessionCode)) {
            return super.attemptAuthentication(request, response);
        }
        throw new RuntimeException("验证码异常");
    }
}
```

##### 6.3.3.2.2 替换为自定义的Filter

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final MyUserDetailsService myUserDetailsService;

    @Autowired
    public SecurityConfig(MyUserDetailsService myUserDetailsService) {
        this.myUserDetailsService = myUserDetailsService;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailsService);

    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    public KaptchaFilter kaptchaFilter() throws Exception {
        final KaptchaFilter kaptchaFilter = new KaptchaFilter();
        kaptchaFilter.setKaptchaParameter("kaptcha");
        kaptchaFilter.setUsernameParameter("username");
        kaptchaFilter.setPasswordParameter("password");
        kaptchaFilter.setFilterProcessesUrl("/doLogin");
        //指定认证管理器
        kaptchaFilter.setAuthenticationManager(authenticationManagerBean());
        //指定认证成功后的处理
        kaptchaFilter.setAuthenticationSuccessHandler((request, response, authentication) -> {
            response.sendRedirect("/index.html");
        });
        //指定认证失败收的处理
        kaptchaFilter.setAuthenticationFailureHandler((request, response, exception) -> {
            response.sendRedirect("/login.html");
        });
        return kaptchaFilter;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/code").permitAll()
                .mvcMatchers("/login.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login.html")
                .and()
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login.html")
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .and()
                .csrf().disable();
        //替换
        http.addFilterAt(kaptchaFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```



#### 6.3.3.3 前后端分离开发

步骤和上述一致,但是在前后端分离开发时，不能直接将验证码以图片返回，而是应该采用`base64`的方式进行返回

```java
    @RequestMapping("/code")
    public String verifyCode(HttpSession session) {
        //生成验证码
        final String text = defaultKaptcha.createText();
        //将验证码保持在session中
        session.setAttribute("code", text);
        //生成图片
        final BufferedImage image = defaultKaptcha.createImage(text);
        //将图片转换为byte数组
        FastByteArrayOutputStream fastByteArrayOutputStream = new FastByteArrayOutputStream();
        try {
            ImageIO.write(image, "jpg", fastByteArrayOutputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //以Base64返回
        return Base64.encodeBase64String(fastByteArrayOutputStream.toByteArray());
    }
```

##### 6.3.3.3.1 扩展attemptAuthentication方法

```java
**
 * 自定义前后端分离开发
 */
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private static final String FROM_KAPTCHA_KEY = "kaptcha";

    private String kaptchaParameter = FROM_KAPTCHA_KEY;

    public String getKaptchaParameter() {
        return kaptchaParameter;
    }

    public void setKaptchaParameter(String kaptchaParameter) {
        this.kaptchaParameter = kaptchaParameter;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        //1.先判度是否为POST请求
        if (!request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        //2.判断请求的方式是否为JSON
        if (request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_VALUE)) {
            //从json中获取用户名和密码进行认证
            try {
                final Map<String, String> userInfo = new ObjectMapper().readValue(request.getInputStream(), Map.class);
                //不要将用户名和密码写死，使用父类中的方法获取
                //可以在注入工厂时进行收手动设置
                final String username = userInfo.get(this.getUsernameParameter());
                final String password = userInfo.get(this.getPasswordParameter());
                final String kaptchaCode = userInfo.get(this.getKaptchaParameter());
                //验证验证码
                final String sessionCode = (String) request.getSession().getAttribute("code");
                if (!ObjectUtils.isEmpty(kaptchaCode) && !ObjectUtils.isEmpty(sessionCode) && sessionCode.equalsIgnoreCase(kaptchaCode)) {
                    //验证用户名和密码
                    UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
                    setDetails(request, authRequest);
                    //可以在工厂进行自定义认证规则
                    return this.getAuthenticationManager().authenticate(authRequest);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        throw new RuntimeException("验证失败");
    }
}
```

##### 6.3.3.2.2 替换为自定义的Filter

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final MyUserDetailService myUserDetailService;

    @Autowired
    public SecurityConfig(MyUserDetailService myUserDetailService) {
        this.myUserDetailService = myUserDetailService;
    }

    @Override
    //自定义数据源
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailService);
    }

    @Override
    @Bean
    //将自定义认证实例暴露到工厂
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    //将自定义的过滤器交给工厂
    @Bean
    public LoginFilter loginFilter() throws Exception {
        final LoginFilter loginFilter = new LoginFilter();
        //认证请求地址
        loginFilter.setFilterProcessesUrl("/doLogin");
        loginFilter.setUsernameParameter("username");
        loginFilter.setPasswordParameter("password");
        loginFilter.setKaptchaParameter("kaptcha");
        //设置自定义认证规则
        loginFilter.setAuthenticationManager(this.authenticationManagerBean());
        //认证成功
        loginFilter.setAuthenticationSuccessHandler((request, response, authentication) -> {
            Map<String, Object> map = new HashMap<>();
            map.put("msg", "验证成功");
            map.put("code", HttpStatus.OK.value());
            map.put("authentication", authentication);
            response.setContentType("application/json;charset=UTF-8");
            final String asString = new ObjectMapper().writeValueAsString(map);
            response.getWriter().print(asString);
        });
        //认证失败
        loginFilter.setAuthenticationFailureHandler((request, response, exception) -> {
            Map<String, Object> map = new HashMap<>();
            map.put("msg", exception.getMessage());
            map.put("code", HttpStatus.INTERNAL_SERVER_ERROR.value());
            response.setContentType("application/json;charset=UTF-8");
            final String asString = new ObjectMapper().writeValueAsString(map);
            response.getWriter().print(asString);
        });
        return loginFilter;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/code").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //退出认证
                .logout()
                //退出登录方式
                .logoutRequestMatcher(new OrRequestMatcher(
                        new AntPathRequestMatcher("/logout", HttpMethod.DELETE.name()),
                        new AntPathRequestMatcher("/logout", HttpMethod.GET.name())
                ))
                .logoutSuccessHandler((request, response, authentication) -> {
                    Map<String, Object> map = new HashMap<>();
                    map.put("msg", "退出成功");
                    map.put("code", HttpStatus.OK.value());
                    map.put("authentication", authentication);
                    response.setContentType("application/json;charset=UTF-8");
                    final String asString = new ObjectMapper().writeValueAsString(map);
                    response.getWriter().print(asString);
                })
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .and()
                .csrf().disable();
        //将自定义的过滤器和UsernamePasswordAuthenticationFilter进行替换，保持过滤器链顺序不变
        //At 用某个过滤器替换指定的过滤器
        //After 在已知Filter类之一之后添加Filter器
        //Before 在已知Filter类之一之前添加Filter器
        http.addFilterAt(loginFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```



# 七.密码加密

## 7.1 passwordEncoder

在默认情况下，SpringSecurity使用passwordEncoder进行密码匹对

![image-20220919172405879](img.assets\image-20220919172405879.png)

**passwordEncoder接口**

```java
public interface PasswordEncoder {
	
    //加密
	String encode(CharSequence rawPassword);
	//密码配对
	boolean matches(CharSequence rawPassword, String encodedPassword);
	//密码升级
	default boolean upgradeEncoding(String encodedPassword) {
		return false;
	}
}
```

实现类:

![image-20220919175022766](img.assets\image-20220919175022766.png)

![image-20220919172713312](img.assets\image-20220919172713312.png)

通过debug可以发现，在默认情况下，passwordEncoder的实现是**DelegatingPasswordEncoder**，不在上述实现类中

![image-20220919172859222](img.assets\image-20220919172859222.png)

进入到matches方法中，可以看到全部的加密方式

![image-20220919173528109](img.assets\image-20220919173528109.png)

最后在**NoOpPasswordEncoder类**中完成密码的匹对

![image-20220919174230158](img.assets\image-20220919174230158.png)



## 7.2 DelegatingPasswordEncoder

在SpringSecurity5.0之后，默认的密码加密方式为`DelegatingPasswordEncoder`,`DelegatingPasswordEncoder`是一个代理类，主要用来代理不同的密码加密方案。默认使用密码相对最安全的加密方式。

使用`DelegatingPasswordEncoder`的原因：

- 兼容性：使用`DelegatingPasswordEncoder`可以帮助许多使用旧密码加密的方式的系统顺利迁徙的SpringSecurity中，它允许同一个系统中存在多种不同的密码加密方案。
- 便捷性：密码储存的方案不能一直不变，如果使用`DelegatingPasswordEncoder`作为默认的加密方案时，只需要改动很小一部分代码就可以实现。



## 7.3 使用自定义加密方式

### 7.3.1 直接使用

```java
    public static void main(String[] args) {
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        //$2a$10$ODqrDuG8AbgY5lgk5qu6JOI0GlCko1WhYzLfmEQZt8NYtxTYwPIMG
        final String encode = bCryptPasswordEncoder.encode("123");
        System.out.println(encode);
    }
```

加密后将密码替换源替换为指定加密方式{bcrypt}

```java
inMemoryUserDetailsManager.createUser(User.withUsername("root").password("{bcrypt}$2a$10$ODqrDuG8AbgY5lgk5qu6JOI0GlCko1WhYzLfmEQZt8NYtxTYwPIMG").roles("root").build());
```

通过debug可以发现解密方式发送改变

![image-20220919181622313](img.assets\image-20220919181622313.png)



### 7.3.2 全局使用

在WebSecurityConfigurerAdapter中有一个内部类`LazyPasswordEncoder`，其中有一个方法创建`passwordEncoder`

```java
passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

createDelegatingPasswordEncoder方法

```java
public static PasswordEncoder createDelegatingPasswordEncoder() {
		String encodingId = "bcrypt";
		Map<String, PasswordEncoder> encoders = new HashMap<>();
		encoders.put(encodingId, new BCryptPasswordEncoder());
		encoders.put("ldap", new org.springframework.security.crypto.password.LdapShaPasswordEncoder());
		encoders.put("MD4", new org.springframework.security.crypto.password.Md4PasswordEncoder());
		encoders.put("MD5", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("MD5"));
		encoders.put("noop", org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance());
		encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
		encoders.put("scrypt", new SCryptPasswordEncoder());
		encoders.put("SHA-1", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("SHA-1"));
		encoders.put("SHA-256",
				new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("SHA-256"));
		encoders.put("sha256", new org.springframework.security.crypto.password.StandardPasswordEncoder());
		encoders.put("argon2", new Argon2PasswordEncoder());
		return new DelegatingPasswordEncoder(encodingId, encoders);
	}
```

通过源码可知如果在**工厂中指定了PasswordEncoder，则就会使用自定义的PasswordEncoder**，否则就会使用默认的PasswordEncoder

```java
	private PasswordEncoder getPasswordEncoder() {
			if (this.passwordEncoder != null) {
				return this.passwordEncoder;
			}
			PasswordEncoder passwordEncoder = getBeanOrNull(PasswordEncoder.class);
			if (passwordEncoder == null) {
				passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
			}
			this.passwordEncoder = passwordEncoder;
			return passwordEncoder;
		}

		private <T> T getBeanOrNull(Class<T> type) {
			try {
				return this.applicationContext.getBean(type);
			}
			catch (NoSuchBeanDefinitionException ex) {
				return null;
			}
		}
```

#### 自定义PasswordEncoder

**不推荐使用**

```java
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
```

**使用该方式在密码中就不用添加{byrypt}**

```java
inMemoryUserDetailsManager.createUser(User.withUsername("root").password("$2a$10$ODqrDuG8AbgY5lgk5qu6JOI0GlCko1WhYzLfmEQZt8NYtxTYwPIMG").roles("root").build());
```



## 7.4 密码自动升级

 在`DaoAuthenticationProvider`中的 createSuccessAuthentication方法定义了密码的升级

```java
	@Override
	protected Authentication createSuccessAuthentication(Object principal, Authentication authentication,
			UserDetails user) {
		boolean upgradeEncoding = this.userDetailsPasswordService != null
				&& this.passwordEncoder.upgradeEncoding(user.getPassword());
		if (upgradeEncoding) {
			String presentedPassword = authentication.getCredentials().toString();
			String newPassword = this.passwordEncoder.encode(presentedPassword);
			user = this.userDetailsPasswordService.updatePassword(user, newPassword);
		}
		return super.createSuccessAuthentication(principal, authentication, user);
	}
```

在该类中，使用了`UserDetailsPasswordService`接口，该接口定义了密码的升级

```java
public interface UserDetailsPasswordService {
	UserDetails updatePassword(UserDetails user, String newPassword);
}
```

![image-20220919202038889](img.assets\image-20220919202038889.png)

该接口只有`InMemoryUserDetailsManager`类一个实现

所以，要实现密码的自动升级，只需要实现`UserDetailsPasswordService`接口即可



### 实现UserDetailsPasswordService接口

```java
@Component
public class MyUserDetailsService implements UserDetailsService, UserDetailsPasswordService {

    private final UserMapper userMapper;

    @Autowired
    public MyUserDetailsService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        final User user = userMapper.loadUserByUsername(username);
        if (!Objects.isNull(user)) {
            final Integer id = user.getId();
            final List<Role> roles = userMapper.selectRolesByUid(id);
            user.setRoles(roles);
        }
        return user;
    }

    @Override
    //默认使用DelegatingPasswordEncoder --> 现阶段是Bcrypt加密
    public UserDetails updatePassword(UserDetails user, String newPassword) {
        Integer i = userMapper.updatePassword(user.getUsername(), newPassword);
        User myUser = (User) user;
        if (i == 1) {
            myUser.setPassword(newPassword);
        }
        return myUser;
    }
}
```

```mysql
    <update id="updatePassword">
        update user
        set password = #{newPassword}
        where username = #{username}
    </update>
```

添加自定义数据源:

```java
    private final MyUserDetailsService myUserDetailsService;

    @Autowired
    public SecurityConfig(MyUserDetailsService myUserDetailsService) {
        this.myUserDetailsService = myUserDetailsService;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailsService);
```

在数据库中观察密码的变化,已经发送改变

![image-20220919205444952](img.assets\image-20220919205444952.png)
