# SpringSecurity



[TOC]



# 八.RememberMe

RememberMe是一种**服务端的行为**，具体的实现思路为：

​		**通过cookie来记录当前用户的身份**，当用户登录成功后，会通过一定算法，将用户信息，时间戳等进行加密，加密完成后，通过响应头带回前端**返回前端存储在cookie中**，当浏览器的会话过期之后，如果再次访问该网站，会**自动将cookie中的信息发送给服务器**，服务器对Cookie中的信息进行校验分析，进而确定出用户的身份信息，Cookie中所保存的用户信息也是有时效性的，一般为三天，一周等



## 8.1 开启rememberMe

```java
 	@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //开启记住我功能
                .rememberMe()
                .and()
                .csrf().disable();
    }
```

在开启之后，默认登录界面会显示记住我单选框

![image-20220920111937157](img.assets\image-20220920111937157.png)

查看页面源码,发现多了一个单选框，名字默认叫做`remember-me`

```html
<input type='checkbox' name='remember-me'/> Remember me on this computer.</p>
```

在请求数据中，多了`remember-me`请求数据

![image-20220920113253909](img.assets\image-20220920113253909.png)

查询cookie，发现多了`remember-me`的键值对，而且默认保存时间为15天

![image-20220920112144646](img.assets\image-20220920112144646.png)

通过解码可以发现

remember-me的值中包含**用户名，过期时间戳，以及用户名、令牌过期时间、用户密码以及 key 组成一个字符串，中间用 " : " 割开，然后通过MD5消息摘要算法对该字符串进行加密后的令牌三部分**

```java
public static void main(String[] args) {
        final byte[] cookie = Base64Utils.decodeFromString("cm9vdDoxNjY0ODc0MTU0NzY1Ojk0MDZiOTRmYWFlNTU3NzUxZTcyMDVjYzIwOTFjMzNm");
        System.out.println(new String(cookie));
        //root:1664874154765:9406b94faae557751e7205cc2091c33f
    }
```



## 8.2 RememberMe原理

![img](img.assets\webp)

### 8.2.1 RememberMeAuthenticationFilter

在开启 RememberMe功能之后，SpringSecurity会自动添加`RememberMeAuthenticationFilter`过滤器来处理记住登录功能

在`RememberMeAuthenticationFilter`类中的过滤方法`doFilter`源码，**RememberMe的认证是通过RemeberMeService来实现的**

![image-20220920161423478](img.assets\image-20220920161423478.png)

- (1) 请求达到过滤器之后，首先判断 SecurityContextHolder 是否有值 , 没有值的话表示用户尚未登录，此时调用 autoLogin 方法进行自动登录。
- (2) 当自动登录成功后返回 rememberMeAuth 不为null 时 ，表示自动登录成功，此时调用 authenticate 方法对 key进行效验，并且将登录成功信息保存到 SecurityContextHolder 对象中，然后调用登录成功贵点，并发布登录成功事件。需要主要的是，登录成功的回调并不包含RememberMeMeService 中的loginSuccess 方法。
- (3) 如果自动登录失败，则调用 `remeberMeService.loginFail `方法处理登录失败回调。onUnsuccessfulAuthentication 和 onSuccessfulAuthentication 都是该过滤器中定义的空方法，并没有任何实现这就是 RememberMeAuthenticationFilter 过滤器所做的事情，成功将RememberMeServices 的服务集成进来。



### 8.2.2 RememberService

**RememberService是进行记住我登录的重要接口**

```java
public interface RememberMeServices {
    //autoLogin 方法可以从请求中提取出需要的参数，完成自动登录功能。
	Authentication autoLogin(HttpServletRequest request, HttpServletResponse response);
    //loginFail 方法是自动登录失败的回调。
	void loginFail(HttpServletRequest request, HttpServletResponse response);
    //loginSuccess 方法是自动登录成功的回调。
	void loginSuccess(HttpServletRequest request, HttpServletResponse response,
			Authentication successfulAuthentication);
}
```

![image-20220920162851633](img.assets\image-20220920162851633.png)



#### 8.2.2.1 TokenBaseRememberMeServices

通过debug可知，自动登录的方式是用**TokenBaseRememberMeServices实现类来实现的**

![image-20220920163847449](img.assets\image-20220920163847449.png)

进入autoLogin方法中，发现使用的是父类`AbstractRememberMeServices`类中的author方法，在该方法中，通过`processAutoLoginCookie`方法来提交的持久登录 cookie。

![image-20220920164540547](img.assets\image-20220920164540547.png)



##### 8.2.2.1.1 processAutoLoginCookie方法

**processAutoLoginCookie** 方法主要用来验证Cookie中的令牌信息是否合法：

1. 首先判断cookieTokens 长度是否为3，如果不为3说明格式不对，则直接抛出异常。
2. 从cookieTokens数组中取出第2项，也就是过期时间，判断令牌是否过期，如果已经过期，则抛出异常。
3. 根据用户名(cookieTokens 数组的第1项)查询当前用户对象。
4. 调用 makeTokenSignature 方法生成一个签名，**签名的生成过程如下：首先将用户名、令牌过期时间、用户密码以及 key 组成一个字符串，中间用 " : " 割开，然后通过MD5消息摘要算法对该字符串进行加密**，并将加密结果转为一个字符串返回。
5. 判断第四步生成的签名和通过Cookie 传过来的签名是否相等（即cookieTokens数组的第三项），如果相等，表示令牌合法，则直接返回用户对象，否则抛出异常

![image-20220920165701774](img.assets\image-20220920165701774.png)

##### 8.2.2.1.2 令牌的生成方法

![image-20220920171123184](img.assets\image-20220920171123184.png)

**自定义key**：

```java
//开启记住我功能
.rememberMe()
//自定义加密key
.key(UUID.randomUUID().toString())
```



#### 8.2.2.2 loginSuccess

 loginSuccess方法在`RememberService`的实现类`AbstractAuthenticationProcessingFilter`里调用 loginSuccess，**loginSuccess 中调用的onLoginSuccess** 

![image-20220920172350748](img.assets\image-20220920172350748.png)

**rememberMeRequested:**

![image-20220920172840788](img.assets\image-20220920172840788.png)

**开启总是记住:**

```java
 //开启记住我功能
.rememberMe()
//自定义加密key
.key(UUID.randomUUID().toString())
//开启总是记住
.alwaysRemember(true)
```

获取的请求参数默认为`remember-me`

![image-20220920173207632](img.assets\image-20220920173207632.png)

```java
public static final String DEFAULT_PARAMETER = "remember-me";
private String parameter = DEFAULT_PARAMETER;
```

**修改请求参数:**

```java
//开启记住我功能
.rememberMe()
//自定义加密key
.key(UUID.randomUUID().toString())
//开启总是记住
.alwaysRemember(true)
//修改cookie的key
.rememberMeParameter("remember-me")
```



##### 8.2.2.2.1 onLoginSuccess

`onLoginSuccess`在`TokenBaseRememberMeServices`类中调用，**在用户认证成功后**，还么有生成 **remeber-me 的cookie时**，进行生成remember-me cookie ，并将cookie写回给前端。



1. 在这个回调中，首先获取用户经和密码信息，如果用户密码在用户登录成功后successfulAuthentication 对象中擦除，则从数据库中重写加载出用户密码。
2. 计算出令牌的过期时间，令牌的有效期是两周。
3. 根据令牌的过期时间、用户名以及用户密码，计算出一个签名。
4. 调用setCookie 方法设置Cookie ，第一个参数是一个数组，数组中一共包含三项。用户名、过期时间以及签名，在setCookie 方法中会将数组转为字符串，并进行**Base64** 编码后响应给前端。

![image-20220920173930336](img.assets\image-20220920173930336.png)

###### 8.2.2.2.1.1 生成cookie

![image-20220920174053400](img.assets\image-20220920174053400.png)



### 8.2.3 总结

​		当⽤户通过⽤户名/密码的形式登录成功后，系统会根据**⽤户的⽤户名、密码以及令牌的过期时间计算出⼀个签名**，这个签名使⽤ MD5 消息摘要算法⽣成，是不可逆的。然后再将**⽤户名、令牌过期时间以及签名拼接成⼀个字符串，中间⽤“:” 隔开，对拼接好的字符串进⾏Base64 编码**，然后将编码后的结果返回到前端，也就是我们在浏览器中看到的令牌。当会话过期之后，访问系统资源时会⾃动携带上Cookie中的令牌，服务端拿到 Cookie中的令牌后，先进⾏ Bae64解码，解码后分别提取出令牌中的三项数据：接着根据令牌中的数据**判断令牌是否已经过期**，如果没有过期，则根据令牌中的⽤户名**查询出⽤户信息**：接着再**计算出⼀个签名和令牌中的签名进⾏对⽐**，如果⼀致，表示会牌是合法令牌，⾃动登录成功，否则⾃动登录失败。

![img](img.assets\bf60e804ad432520e816e6c4ebe082e2.png)



## 8.3 提高Remember的安全性

 		默认使用的remebermeService 的实现类 `TokenBasedRememberMeServices `的做法的比较简单，对用户名 、过期时间+生成令牌 进行Base64的加密，在放入cookie，还又就是解码比较认证，没有什么安全措施，如果cookie被外部拦截到，那么就会肆无忌惮的进行攻击服务器，是比较不安全的。spring Security 也给我们提供了比较安全的实现方式，使用 `PersistentTokenBasedRememberMeServices `进行remeberme的功能实现。

![image-20220920193039889](img.assets\image-20220920193039889.png)



### 8.3.1 PersistentTokenBasedRememberMeServices

核心方法`processAutoLoginCookie`:

![image-20220920195044680](img.assets\image-20220920195044680.png)

1. 不同于 TokenBasedRemornberMeServices 中的 processAutologinCookie 方法，这里cookieTokens 数组的长度为2，**第一项是servies，第二项是token** 。
2. 从cookieTokens 数组中分别提取出series 和 token ，然后根据 series 去内存中查询出一个 PersistentRememberMeToken 对象。如果查询出来的对象为null，表示内存中并没有series 对应的值，本次自动登录失败。如果查询出来的token和从 cookieTokens 中解析出来的 token不相同，说明自动登录令牌已经泄露（恶意用户利用令牌登录后，内存中的token变了），此时移除当前用户的所有自动登录记录并抛出异常。
3. 根据数据库中查询出来的结果判断令牌是否过期，如果过期就抛出异常。
4. 生成一个新的PersistentRememberMeToken 对象，**用户名和 series 不变，token 重新生成，date 也是使用当前时间。newToken生成后，根据series 去修改内存中的token 和 date (即每次自动登录后都会产生新的 token 和 date)**
5. 调用 addCookie 方法添加 Cookie，在addCookie 方法中，会调用到我们前面所说的setCookie方法，但是要注意第一个数组参数中只有两项：series和 token (即**返回到前端的令牌是通过对series 和 token 进行 Base64 编码得到的)**
6. 最后将根据用户名查询用户对象并返回。



### 8.3.2 使用

```java
 .rememberMeServices(rememberMeServices()) //指定rememberMeServices的实现
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        inMemoryUserDetailsManager.createUser(User.withUsername("root").password("{noop}123").roles("ROOT").build());
        return inMemoryUserDetailsManager;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //开启记住我功能
                .rememberMe()
                //指定rememberMeServices的实现
                .rememberMeServices(rememberMeServices())
                .and()
                .csrf().disable();
    }
    
    @Bean
    public RememberMeServices rememberMeServices(){
        //key 加密key
        //userDetailsService 自定义数据源
        //tokenRepository token存储方式
        return new PersistentTokenBasedRememberMeServices(UUID.randomUUID().toString(),userDetailsService(),new InMemoryTokenRepositoryImpl());
    }
}
```

观察cookie的值：发送改变

![image-20220920200226870](img.assets\image-20220920200226870.png)



## 8.4 持久化令牌

### 8.4.1 PersistentTokenRepository

PersistentTokenRepository 是一个接口，它的作用就是存取生成的认证信息的。

```java
public interface PersistentTokenRepository {
	//创建新的token
	void createNewToken(PersistentRememberMeToken token);
	//更新token
	void updateToken(String series, String tokenValue, Date lastUsed);
	//获取Series
	PersistentRememberMeToken getTokenForSeries(String seriesId);
	//移除token
	void removeUserTokens(String username);
}

```

token的默认存储方式是`InMemoryTokenRepositoryImpl`内存实现的，所以我们要修改为`JdbcTokenRepositoryImpl`将token存入到数据库中进行实现.

![image-20220920201129945](img.assets\image-20220920201129945.png)

![image-20220920201450418](img.assets\image-20220920201450418.png)



### 8.4.2 JdbcTokenRepositoryImpl

通过源码可知,这个类已经将类的结构的sql写好，直接拿来用就可以 。默认是通过 JDBCTemplate进行实现的，所以需要引入对应的 jdbc 相关依赖。

![image-20220920202038116](img.assets\image-20220920202038116.png)

这里使用MySQL

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.url=jdbc:mysql://localhost:3306/sucerity
mybatis-plus.mapper-locations=classpath:mapper/*.xml
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

#### 8.4.2.1 自己创建表

```mysql
CREATE TABLE persistent_logins (
	username VARCHAR ( 64 ) NOT NULL,
	series VARCHAR ( 64 ) PRIMARY KEY,
token VARCHAR ( 64 ) NOT NULL,
last_used TIMESTAMP NOT NULL)
```

配置` JdbcTokenRepositoryImpl`

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final DataSource dataSource;

    @Autowired
    public SecurityConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        inMemoryUserDetailsManager.createUser(User.withUsername("root").password("{noop}123").roles("ROOT").build());
        return inMemoryUserDetailsManager;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //开启记住我功能
                .rememberMe()
                //指定rememberMeServices的实现
                .rememberMeServices(rememberMeServices())
                .and()
                .csrf().disable();
    }

    @Bean
    public RememberMeServices rememberMeServices() {
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        //指定数据库数据源
        jdbcTokenRepository.setDataSource(dataSource);
        return new PersistentTokenBasedRememberMeServices(UUID.randomUUID().toString(), userDetailsService(), jdbcTokenRepository);
    }
}

```

数据库中数据:

![image-20220920203941341](img.assets\image-20220920203941341.png)



#### 8.4.2.2 自动创建表

**这种方式不能指定加密的key和认证数据源**

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final DataSource dataSource;

    @Autowired
    public SecurityConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        inMemoryUserDetailsManager.createUser(User.withUsername("root").password("{noop}123").roles("ROOT").build());
        return inMemoryUserDetailsManager;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //开启记住我功能
                .rememberMe()
                //指定token持久化方式
                .tokenRepository(persistentTokenRepository())
                .and()
                .csrf().disable();
    }

    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        //启动时创建表结构,建议关闭，否则启动后会重新创建
        jdbcTokenRepository.setCreateTableOnStartup(true);
        return jdbcTokenRepository;
    }
```



## 8.5 自定义记住我

### 8.5.1 web开发

通过源码可知，在进行记住我时，**需要一个请求参数为remember-me(可以自定义),值为true,on,yes,1四个中的一个**

![image-20220920220708130](img.assets\image-20220920220708130.png)

所以，我们只需要在表单中添加一个复选框，`name=remember-me,value=true|on|yes|1`即可

```html
<input type="checkbox" name="remember-me" value="true">记住我 <br>
```



### 8.5.2 前后端分离开发

**实现方式：认证成功后记住cookie到客户端，只有cookie写入成功才能实现自动登录功能**

在源码中可以看到，默认是从请求参数中获取值的，所以我们要将获取的方式修改为从json中获取值

![image-20220920225324668](img.assets\image-20220920225324668.png)

所以我们只需要将`rememerMeRequest方法重写即可`,**在之前的过滤器中将remember-me的值放入到requset作用域中**

```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (!request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        if (request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_VALUE)) {
            try {
                //流中的数据只能使用一次
                final Map<String, String> usernamePasswordMap = new ObjectMapper().readValue(request.getInputStream(), Map.class);
                String username = usernamePasswordMap.get(this.getUsernameParameter());
                username = (username != null) ? username : "";
                String password = usernamePasswordMap.get(this.getPasswordParameter());
                password = (password != null) ? password : "";
                //获取remember-me的值
                final String rememberValue = usernamePasswordMap.get(AbstractRememberMeServices.DEFAULT_PARAMETER);
                //放入request作用域中
                if (Objects.nonNull(rememberValue)) {
                    request.setAttribute(AbstractRememberMeServices.DEFAULT_PARAMETER, rememberValue);
                }
                UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
                setDetails(request, authRequest);
                return this.getAuthenticationManager().authenticate(authRequest);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return super.attemptAuthentication(request, response);
    }
}
```

自定义`PersistentTokenBasedRememberMeServices`，**重写`rememerMeRequest`,从request作用域中获取值**

```java
public class MyPersistentTokenBasedRememberMeServices extends PersistentTokenBasedRememberMeServices {

    public MyPersistentTokenBasedRememberMeServices(String key, UserDetailsService userDetailsService, PersistentTokenRepository tokenRepository) {
        super(key, userDetailsService, tokenRepository);
    }

    @Override
    protected boolean rememberMeRequested(HttpServletRequest request, String parameter) {
        //从请求作用域中获取
        String paramValue = request.getAttribute(AbstractRememberMeServices.DEFAULT_PARAMETER).toString();
        if (paramValue != null) {
            if (paramValue.equalsIgnoreCase("true") || paramValue.equalsIgnoreCase("on")
                    || paramValue.equalsIgnoreCase("yes") || paramValue.equals("1")) {
                return true;
            }
        }
        this.logger.debug(
                LogMessage.format("Did not send remember-me cookie (principal did not set parameter '%s')", parameter));
        return false;
    }
}

```

开启记住我，定义token保存位置:

注意，需要在认证成功之后对cookie进行解码操作，所有要在两处设置自定义的`RememberService`--**开启remember之后进行cookie的生成，认证成功后进行cookie的解码**

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final SecurityUserDetailsService userDetailsService;

    private final DataSource dataSource;

    @Autowired
    public SecurityConfig(SecurityUserDetailsService userDetailsService, DataSource dataSource) {
        this.userDetailsService = userDetailsService;
        this.dataSource = dataSource;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService);
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    public LoginFilter loginFilter() throws Exception {
        final LoginFilter loginFilter = new LoginFilter();
        loginFilter.setFilterProcessesUrl("/login");
        loginFilter.setAuthenticationManager(authenticationManagerBean());
        loginFilter.setUsernameParameter("username");
        loginFilter.setPasswordParameter("password");
        //设置认证成功时使用自定义的remember的RememberMeServices
        //对cookie进行解码
        loginFilter.setRememberMeServices(getMyPersistentTokenBasedRememberMeServices());
        loginFilter.setAuthenticationSuccessHandler((request, response, authentication) -> {
            Map<String, Object> map = new HashMap<>();
            map.put("code", HttpStatus.OK.value());
            map.put("msg", "认证成功");
            map.put("data", authentication);
            map.put("success", true);
            response.setContentType("application/json;charset=UTF-8");
            final String result = new ObjectMapper().writeValueAsString(map);
            response.getWriter().print(result);
        });
        loginFilter.setAuthenticationFailureHandler((request, response, exception) -> {
            Map<String, Object> map = new HashMap<>();
            map.put("code", HttpStatus.OK.value());
            map.put("msg", exception.getMessage());
            map.put("data", null);
            map.put("success", false);
            response.setContentType("application/json;charset=UTF-8");
            final String result = new ObjectMapper().writeValueAsString(map);
            response.getWriter().print(result);
        });
        return loginFilter;
    }

    @Bean
    public MyPersistentTokenBasedRememberMeServices getMyPersistentTokenBasedRememberMeServices() {
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        return new MyPersistentTokenBasedRememberMeServices(UUID.randomUUID().toString(), userDetailsService, jdbcTokenRepository);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .rememberMe()
                //生成cookie
                .rememberMeServices(getMyPersistentTokenBasedRememberMeServices())
                //设置过期时间
                .tokenValiditySeconds(60 * 60 * 24 * 7)
                .and()
                .exceptionHandling()
                .authenticationEntryPoint((request, response, authException) -> {
                    response.setContentType("application/json;charset=UTF-8");
                    response.getWriter().print("请先验证");
                })
                .and()
                .csrf().disable();
        http.addFilterAt(loginFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```



# 九.会话管理

当浏览器去调用登录接口登录成功之后，服务器会和浏览器之间建立一个会话(session)，在每一次发送请求时都会携带一个SessionId，服务器根据这个SessionId来判断用户身份，当浏览器关闭后，服务器的Session并不会自动销毁，需要手动调用Session的销毁方法，或者等待Session过期时间到了之后自动销毁（默认时间为半个小时）。

在springSecurity中，与httpSession相关的功能由` SessionManagementFilter`和`SessionAuthenticationStrategy`接口来处理，` SessionManagementFilter`过滤器将Session的相关操作委托给`SessionAuthenticationStrategy`接口完成。

在`SessionManagementFilter`的doFilter中可以看到实际上是通过`SessionAuthenticationStrategy`接口来完成

![image-20220921093850417](img.assets\image-20220921093850417.png)



## 9.1 会话并发管理

会话并发管理就是指在当前系统中，同一个用户可以同时创建多个会话，如果一台设备对应一个会话，那么也可以理解为同一个用户可以同时在

多少台设备上同时登陆**，在默认情况下，同一个用户在多少设备上登录没有限制**，不过开发者可以在Spring Security中进行设置。



### 9.1.1 开启会话管理

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable()
                //开启会话管理
                .sessionManagement()
                //指定最大的会话数
                .maximumSessions(1);
    }

    //在新版的Security中HttpSessionEventPublisher可以不用进行配置
    @Bean
    public HttpSessionEventPublisher httpSessionEventPublisher(){
        return new HttpSessionEventPublisher();
    }
```

使用两个浏览器进行登录，刷新后发现

```
This session has been expired (possibly due to multiple concurrent logins being attempted as the same user).
```

> - sessionManagement():用来开启会话管理
> - maximumSessions(1)：用来指定最大的会话数为1
> - HttpSessionEventPublisher ：提供一个HttpSessionEventPublisher 实例，SpringSecurity中通过一个Map集合来维护当前的Http Session数，进而实现会话的并发管理。当用户登录成功后，就向Map集合中添加一条HttpSession记录数，当会话销毁时，就从集合中移出一条HttpSession数。HttpSessionEventPublisher 实现了HttpSessionListener接口，**可以 监听到HttpSession的创建和销毁事件**，并且将HttpSession的销毁/创建事件发布出去，这样，springSecurity就可以感知到当前事件



### 9.1.2 会话失效处理

#### 9.1.2.1 web方式处理

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable()
                .sessionManagement()
                .maximumSessions(1)
                //如果用户尝试访问资源并且他们的会话由于当前用户的会话过多而已过期，
                //则重定向到的 URL。默认是在响应中写入一个简单的错误消息
                .expiredUrl("/login");
    }

```

#### 9.1.2.2 前后端分离处理

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable()
                //开启会话管理
                .sessionManagement()
                //指定最大的会话数
                .maximumSessions(1)
                //确定检测到过期会话时的行为
                .expiredSessionStrategy(event -> {
                    final HttpServletResponse response = event.getResponse();
                    response.setContentType("application/json;charset=UTF-8");
                    Map<String, Object> map = new HashMap<>();
                    map.put("code", HttpStatus.INTERNAL_SERVER_ERROR.value());
                    map.put("msg", "会话失效,请重新登录");
                    final String result = new ObjectMapper().writeValueAsString(map);
                    response.getWriter().print(result);
                });
    }
```

![image-20220921102923699](img.assets\image-20220921102923699.png)

## 9.2 禁止再次登录

默认的效果是一种被”挤下线“的效果，后面登录的用户会把前面登录的用户”挤下线 “。

**还有一种是禁止后来者登录，即一旦当前用户登录成功，后来在无法再次使用相同的用户登录，直到当前用户主动注销登录**

```java
http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable()
                .sessionManagement()
                .maximumSessions(1)
                //一旦登录禁止再次登录
                .maxSessionsPreventsLogin(true);
```

![image-20220921103804375](img.assets\image-20220921103804375.png)



## 9.3 会话共享

如果当前环境是集群环境，前面所述的处理方案就会失效，此时可以利用**spring-session结合redis来实现**

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

redis配置

```properties
spring.redis.host=121.41.112.246
spring.redis.port=6379
spring.redis.password=13883981813xH
spring.redis.connect-timeout=5000
spring.redis.timeout=5000
spring.redis.database=0
```

将session交给redis储存

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final FindByIndexNameSessionRepository findByIndexNameSessionRepository;

    @Autowired
    public SecurityConfig(FindByIndexNameSessionRepository findByIndexNameSessionRepository) {
        this.findByIndexNameSessionRepository = findByIndexNameSessionRepository;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable()
                .sessionManagement()
                .maximumSessions(1)
                .maxSessionsPreventsLogin(true)
                //控制使用的SessionRegistry实现。默认是SessionRegistryImpl ，它是一个内存实现
                //把session交给谁管理
                .sessionRegistry(sessionRegistry());
    }

    //创建session同步到redis中的bean中的方案
    @Bean
    public SpringSessionBackedSessionRegistry sessionRegistry() {
        return new SpringSessionBackedSessionRegistry(findByIndexNameSessionRepository);
    }
}
```

redis中的数据

![image-20220921110332522](img.assets\image-20220921110332522.png)





# 十.CSRF漏洞保护

CSRF(Cross-Site Request Forgery跨站请求伪造)，也可称为一键式攻击(one-click-attack),通常缩写为`CSRF`或者`XSRF`

**CSRF攻击是一种挟持用户在当前已登录的浏览器上发送恶意请求的攻击方法。**相对于XSS利用用户对指定网站的信任，CSRF则是利用网站对用户网页浏览器的信任。简单来说，CSRF是攻击者通过一些技术手段欺骗用户的浏览器，去访问一个用户曾经认证过的网站并执行恶意请求，例如发送邮件、发消息、甚至财产操作(如转账和购买商品)。由于客户端(浏览器)已经在该网站上认证过，所以该网站会认为是真正用户在操作而执行请求(实际上这个并非用户的本意)。



**例如**：

假设zkt现在登录了某银行的网站准备完成一项转账操作，转账的链接如下:

https://bank.xxx.com/withdrwa?account=zkt&amount=1000&for=zhangsan

可以看到，这个连接是想从zkt这个账户下转账1000元到张三账户下，假设zkt没有注销登录该银行的网站，就在同一个浏览器新的选项卡中打开了一个危险网站，这个危险网站中有一幅图片，代码如下：

![img](https://bank.xxx.com/withdrwa?account=zkt&amount=1000&for=lisi)

一旦用户打开了这个网站，这个图片链接中的请求就会被发送出去。由于是同一个浏览器并且用户尚未注销登录，所以该请求**会自动携带上对应的有效的cookie信息**，进而完成一次转账操作。这就是跨站请求伪造。



## 10.1 CSRF防御配置

​		CSRF攻击的根源在于**浏览器默认的身份验证机制(自动携带当前网站的Cookie信息)**，这种机制虽然可以保证请求是来自用户的某个浏览器，但是无法确保这请求是用户授权发送。攻击者和用户发送的请求一模一样，这意味着我们没有办法去直接拒绝这里的某一个请求。如果能在合法请求中额外携带一个攻击者无法获取的参数，就可以成功区分出两种不同的请求，进而直接拒绝掉恶意请求。在SpringSecurity中就提供了这种机制来防御CSRF攻击，这种机制我们称之为**令牌同步模式**。



### 10.1.1 令牌同步模式

​		这是目前主流的CSRF攻击防御方案。具体的操作方式就是在每一个HTTP请求中，除了默认自动携带的Cookie参数之外，再**提供一个安全的、随机生成的字符串，我们称之为CSRF令牌**。这个**CSRF令牌由服务端生成，生成后再HttpSession中保存一份**。当前端请求到达后，将请求携带的CSRF令牌信息和服务端中保存的令牌进行对比，如果两者不相等，则拒绝掉该HTTP请求。

> 注意：考虑到会有一些外部站点链接到我们的网站，所以我们要求请求是幂等的，这样对于HEAD、OPTIONS、TRACE等方法就没有必要使用CSRF令牌了，强行使用可能会导致令牌泄露。



### 10.1.2 开启CSRF防御

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
            	//开启CSRF防御，默认情况下是自动开启的
                .csrf()
        }
```

在默认的登录页面中，可以看到添加了一个hidden隐藏框

```html
<input name="_csrf" type="hidden" value="a5049303-02cc-4d70-87f3-31cbf450e8e8" />
```



## 10.2 使用CSRF

### 10.2.1 web方式

开启CSRF防御后会自动在提交的表单中加入如下代码，如果不能自动加入，需要在开启之后手动加入如下代码，并随着请求提交。获取服务端令牌方式如下：

```html
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
```



### 10.2.2 前后端分离方式

在前后端分离开发时，**只需要将生成的csrf放入cookie中**，并且在请求时获取cookie中令牌信息进行提交即可

![image-20220921160340731](img.assets\image-20220921160340731.png)

```java
	@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf()
                //指定csrf的存储库
                //CookieCsrfTokenRepository将令牌保存在cookie中
                //withHttpOnlyFalse 允许前端获取cookie
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
    }
```

登录之后，我们可以在cookie中查看到`XSRF-TOKEN`的值,但是无法完成登录认证

![image-20220921162743882](img.assets\image-20220921162743882.png)

CSRF的认证是通过`CsrfFilter`过滤器中的`doFilterInternal`方法来执行，所以我们对其进行debug

![image-20220921163644282](img.assets\image-20220921163644282.png)

请求头的参数为`X-XSRF-TOKEN`

![image-20220921163703536](img.assets\image-20220921163703536.png)

继续往下,如果从请求头中无法获取值，则从请求参数中获取名为`_scrf` 的值，但是前后端分离都是通过json传递，也无法获取

![image-20220921164020828](img.assets\image-20220921164020828.png)

所以在请求头中添加`X-XSRF-TOKEN`的参数

![image-20220921164619270](img.assets\image-20220921164619270.png)

**在开启CSRF之后，处理HEAD、OPTIONS、TRAC、GET方式之外的请求，都需要在请求头中添加`X-XSRF-TOKEN`，否则无法通过**

![image-20220921165034905](img.assets\image-20220921165034905.png)



**发送请求携带令牌即可**

- 请求参数中携带令牌,修改`doFilterInternal`方法中获取参数的方式

```
key: "_csrf"
value: "xxx"
```

- 请求头中携带令牌

```
X-XSRF-TOKEN:value
```



# 十一.跨域

​		CORS(Cross-Origin Resource Sharing)是由W3C制定的一种跨域资源共享技术标准，其目的就是为了解决前端的跨域请求。在JavaEE开发中，最常见的前端跨域请求解决方案是早起的JSONP，但是JSONP只支持GET请求，这是一个很大的缺陷，而CORS则支持多种HTTP请求方法，也是目前主流的跨域解决方案。

​		CORS中新增了一组HTTP请求头字段，通过这些字段，服务器高炉浏览器，哪些网站通过浏览器有权限访问哪些资源。同时规定，对那些可能修改服务器数据的HTTP请求方法(如GET以外的HTTP请求等)，**浏览器必须首先使用OPTIONS方法发起一个预检请求(prenightst)，预检请求的目的是查看服务端是否支持即将发起的跨域请求**，如果服务端允许，才发送实际的HTTP请求。在预检请求的返回中，服务端也可以通知客户端，是否需要携带身份凭证(如Cookies、HTTP认证信息等)。

> CORS: 同源/同域 = 协议 + 主机ip + 端口



## 11.1 简单请求

GET请求为例，如果需要发起一个跨域请求，则请求头如下：

```http
Host: localhost:8080
Origin: http://localhost:8081
Referer: http://localhost:8081/index.html
```

如果服务端支持该跨域请求，那么返回的响应头中将包含如下字段：

```http
Access-Control-Allow-Origin: http://localhost:8081
```

`Access-Control-Allow-Origin`字段用来告诉浏览器可以访问该资源的域，**当浏览器收到这样的响应头信息之后，提取出Access-Control-Allow-Origin字段中的值，发现该值包含当前页面所在的域**，就知道这个跨域是被允许的，因此就不再对前端的跨域请求进行限制。这属于简单请求，即不需要进行预检请求的跨域。



## 11.2 非简单请求

对于一些非简单请求，**会首先发送一个预检请求**。预检请求类似下面这样：

```http
OPTIONS /put HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Accept: */*
Access-Control-Request-Method: PUT
Origin: http://localhost:8081
Referer: http://localhost:8081/index.html
```

**请求方法是OPTIONS**,请求头Origin就告诉服务端当前页面所在域，请求头Access-Control-Request-Methods告诉服务器端即将发起的跨域请求所使用的方法。服务端对此进行判断，如果允许即将发起的跨域请求，则会给出如下响应：

```http
HTTP/1.1 200
Access-Control-Allow-Origin:http://localhost: 8081
Access-Control-Request-Methods: PUT
Access-Control-Max-Age: 3600
```

`Access-Control-Allow-Methods`字段表示允许的跨域方法,`Access-Control-Max-Age`字段表示预检请求的有效期，单位为秒，在有效期内如果发起该跨域请求，则不用再次发起预检请求。预检请求结束后，接下来就会发起一个真正的跨域请求，跨域请求和前面的简单请求跨域步骤类似。



## 11.3 Spring跨域解决方案

### 11.3.1 @CrossOrigin

​	Spring中第一种处理跨域的方式是通过@CrossOrigin注解来标记支持跨域，**该注解可以添加在方法上，也可以添加在Controller上**。当添加在Controller上时，表示Controller中的所有接口都支持跨域，具体配置如下：

```java
@RestController
public class IndexController {

    @GetMapping("/hello")
    @CrossOrigin(origins = "http://localhost:8081")
    public String hello(){
        System.out.println("hello ok");
        return "hello ok";
    }
}
```

@CrossOrigin注解各属性含义如下：

- allowCredentials：浏览器是否应当发送凭证信息，如Cookie。
- allowedHeaders：请求被允许的请求头字段，*表示所有字段。
- exposedHeaders：哪些响应头可以作为相应的一部分暴露出来。

> **注意，这里只可以一一列举，通配符 \* 在这里是无效的。**

- maxAge：预检请求的有效期，有效期内不必再次发送预检请求，默认是1800秒
- methods：允许的请求方法，*表示允许的所有方法。
- **origins：允许的域，*表示允许所有域。**



### 11.3.2 addCrossMapping自定义mvc配置

@CrossOrigin注解需要添加在不同的Controller上。所以还有一种全局配置方法，就是通过重写**WebMvcConfigurerComposite#addCorsMappings**方法来实现，具体配置如下：

```java
//自定义mvc配置类
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    //用来全局处理跨域
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")   //对哪些请求进行跨域
                .allowedMethods("*")
                .allowedOrigins("*")
                .allowedHeaders("*")
                .allowCredentials(false)
                .exposedHeaders("")
                .maxAge(3600);
    }
```



### 11.3.3 CorsFilter

CorsFilter是Spring Web中提供的一个处理跨域的过滤器，开发者也可以通过该过滤器处理跨域。

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.Arrays;

//自定义mvc配置类
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    
    @Bean
    FilterRegistrationBean<CorsFilter> corsFilter(){
        FilterRegistrationBean<CorsFilter> registrationBean = new FilterRegistrationBean<>();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.setAllowedHeaders(Arrays.asList("*"));
        corsConfiguration.setAllowedMethods(Arrays.asList("*"));
        corsConfiguration.setAllowedOrigins(Arrays.asList("*"));
        corsConfiguration.setMaxAge(3600L);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**",corsConfiguration);
        registrationBean.setFilter(new CorsFilter(source));
        registrationBean.setOrder(-1);
        return registrationBean;
    }

}
```



## 11.4 SpringSecurity跨域解决方案

当我们为项目添加了Spring Security依赖后，发现上面三种跨域方式有的失效了，有的则可以继续使用。

通过@CrossOrigin注解或者重写addCorsMappings方法配置跨域，统统失效了，通过**CorsFilter配置的跨域，有没有失效则要看过滤器的优先级**，如果**过滤器优先级高于SpringSecurity过滤器，即先于Spring Security过滤器执行，则CorsFilter所配置的跨域处理依然有效**；如果过滤器优先级地域Spring Security过滤器，则CorsFilyer所配置的跨域处理就会失效。

以下是Filter、DispatcherServlet以及Interceptor执行顺序:

![image-20220921202328678](img.assets\image-20220921202328678.png)

由于非简单请求都要首先发送一个预检请求(request),而**预检请求并不会携带认证信息**，所以预检请求就有被Spring Security拦截的可能。因此通过@CrossOrigin注解或者重写addCorsMappings方法配置跨域就会失效。如果使用CorsFilter配置的跨域，只要过滤器优先级高于SpringSecurity过滤器就不会有问题，反之同样会出现问题。



### SpringSecurity跨域设置

```java
public CorsConfigurationSource configurationSource() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        //设置请求头
        corsConfiguration.setAllowedHeaders(Arrays.asList("*"));
        //设置请求参数
        corsConfiguration.setAllowedMethods(Arrays.asList("*"));
        //设置请求域
        corsConfiguration.setAllowedOrigins(Arrays.asList("*"));
        //设置遇检请求的时间
        corsConfiguration.setMaxAge(3600L);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        //设置请求路径
        source.registerCorsConfiguration("/**", corsConfiguration);
        return source;
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //添加要使用的CorsFilter.
                //如果提供了名为 corsFilter的bean.则使用该CorsFilter
                //否则，如果定义了corsConfigurationSource，则使用该CorsConfiguration
                //否则，如果SpringMVC在类路径上，则使用HandlerMappingIntrospector
                .cors()//开启springSecurity跨域
                .configurationSource(configurationSource())//跨域配置
                .and()
                .csrf();
    }
```



# 十二.异常处理

## 12.1 springSecurity异常体系

Spring Security 中异常主要分为两大类：

- **AuthenticationException**：认证异常

```java
package org.springframework.security.core;

public abstract class AuthenticationException extends RuntimeException {

	public AuthenticationException(String msg, Throwable cause) {
		super(msg, cause);
	}

	public AuthenticationException(String msg) {
		super(msg);
	}

}

```

继承类:

![image-20220921205857553](img.assets\image-20220921205857553.png)

- **AccessDeniedException**：授权异常

```java
package org.springframework.security.access;

public class AccessDeniedException extends RuntimeException {

	public AccessDeniedException(String msg) {
		super(msg);
	}

	public AccessDeniedException(String msg, Throwable cause) {
		super(msg, cause);
	}

}
```

继承类:

![image-20220921210126280](img.assets\image-20220921210126280.png)

在实际项目开发中，如果默认提供异常无法满足需求时，就需要根据实际需要来自定义异常类。



## 12.2 自定义异常配置

```java
	@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                //允许配置异常处理。这会在使用WebSecurityConfigurerAdapter时自动应用。
                .exceptionHandling()
                //指定认证异常
                .authenticationEntryPoint((request, response, authException) -> {
                     if(authException instanceof UsernameNotFoundException){
                        //自定义异常处理
                    }
                    response.setContentType("application/json;charset=UTF-8");
                    response.getWriter().print("处理认证异常"+authException.getMessage());
                })
                .accessDeniedHandler((request, response, accessDeniedException) -> {
                    response.setContentType("application/json;charset=UTF-8");
                    response.getWriter().print("处理授权"+accessDeniedException.getMessage());
                })
                .and()
                .csrf().disable();
    }
```



# 十三.授权



## 13.1 权限管理



### 13.1.1 认证

身份认证，就是判断一个用户是否为合法用户的处理过程。Spring Security中支持多种不同方式的认证，但是无论开发者使用哪种方式认证，都不会影响授权功能使用。因为Spring Security很好做到了认证和授权解耦。



### 13.1.2 授权

授权，即访问控制，控制谁能访问哪些资源。简单的理解授权就是根据系统提前设置好的规则，给用户分配可以访问某一个资源的权限，用户根据自己所具有的权限，去执行相应操作。



## 13.2 授权管理的核心概念

认证成功之后会将当前登录用户信息保存到Authentication对象中，Authentication对象中有一个`getAuthorities()`方法，用来返回当前登录用户具备的权限信息，也就是当前用户具有权限信息。该方法的返回值为`Collection<? extends GrantedAuthority>`,当需要进行权限判断时，就会根据集合返回权限信息调用相应 方法进行判断。

```java
public interface Authentication extends Principal, Serializable {
	
    //返回当前登录用户具备的权限信息
	Collection<? extends GrantedAuthority> getAuthorities();

	/……/
}
```

基于角色进行权限管理

基于资源进行权限管理 -->权限字符串

R(Role Resources) B(base) A(access) C(controll)

我们针对于授权可以是**基于角色权限管理**和**基于资源权限管理**，从设计层面上来说，角色和权限是两个完全不同的东西：权限是一些具体操作，角色则是某些权限集合。如：READ_BOOK和ROLE_ADMIN是完全不同的。因此至于返回值是什么取决于你的业务设计情况：

- 基于角色权限设计就是：**用户<=>角色<=>资源** 三者关系 返回就是用户的**角色**
- 基于资源权限设计就是： **用户<=>权限<=>资源** 三者关系 返回就是用户的**权限**
- 基于角色和资源权限设计就是: **用户<=>角色<=>权限<=>资源** 返回统称为用户的**权限**

为什么统称为权限，因为从代码层面角色和权限没有太大不同，都是权限，特别是在Spring Security中。角色和权限处理方式基本上都是一样的。唯一区别SpringSecurity在很多时候会自动给**角色添加一个ROLE_前缀，而权限则不会自动添加**。



## 13.3 权限管理策略

可以访问系统中哪些资源(http url method)

Spring Security中提供的权限管理策略主要有两种类型：

- 基于过滤器(URL)的权限管理(FilterSecurityInterceptor)
  - 基于过滤器的权限管理主要用来**拦截HTTP请求**，拦截下来之后，根据HTTP请求地址进行权限校验。
- 基于AOP的权限管理(MethodSecurityInterceptor)
  - 基于AOP权限管理主要是用来**处理方法级别的权限问题**。当需要调用某一个方法时，通过AOP将操作拦截下来，然后判断用户是否具备相关的权限。



### 13.3.1 mvcMatchers,antMatchers,regexMatchers

```java
	//antMathchers
	public C antMatchers(HttpMethod method, String... antPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure antMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.antMatchers(method, antPatterns));
	}	

	public C antMatchers(String... antPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure antMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.antMatchers(antPatterns));
	}

	
	//mvcMatchers
	//使用 Spring MVC 用于匹配的相同规则。例如，路径“/path”的映射通常会匹配“/path”、“/path/”、“/path.html”等。
	//如果 Spring MVC 不会处理当前请求，则使用该模式作为ant模式的合理默认值。
	public abstract C mvcMatchers(String... mvcPatterns);
	public abstract C mvcMatchers(HttpMethod method, String... mvcPatterns);
	
	//regexMatchers
	//支持正则表达式
	public C regexMatchers(HttpMethod method, String... regexPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure regexMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.regexMatchers(method, regexPatterns));
	}
	public C regexMatchers(String... regexPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure regexMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.regexMatchers(regexPatterns));
	}
```

在使用上，三种在使用上区别差距不大

```java
 http.authorizeRequests()
     			//.mvcMatchers 除了匹配/admin,还可以匹配 /admin/，/admin.html
                .mvcMatchers(HttpMethod.GET,"/admin").hasRole("ADMIN") 
     			//.antMatchers 只能匹配/admin
                .antMatchers(HttpMethod.GET,"/admin").hasRole("ADMIN")
     			//.regexMatchers 可以匹配正则表达式
                .regexMatchers(HttpMethod.GET,"/admin").hasRole("ADMIN")
```



### 13.3.2 权限表达式

![image-20220923103333094](img.assets\image-20220923103333094.png)

一般使用前面6个

| 方法                                                         | 说明                                                     |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| hasAuthority(String authority)                               | 当前用户是否具备指定权限                                 |
| hasAnyAuthority(String... authorities)                       | 当前用户是否具备指定权限中的任意一个                     |
| hasRole(String role)                                         | 当前用户是否具备指定角色                                 |
| hasAnyRole(String... roles)                                  | 当前用户是否具备指定角色中的任意一个                     |
| permitAll()                                                  | 放行请求                                                 |
| denyAll()                                                    | 拒绝请求                                                 |
| isAnonymous()                                                | 当前用户是否为一个匿名用户                               |
| isAuthenticated()                                            | 当前用户是否认证成功                                     |
| isRememberMe()                                               | 当前用户是否通过记住我自动登录                           |
| isFullyAuthenticated()                                       | 当前用户是否既不是匿名用户也不是通过RememberMe自动登录的 |
| hasPermission(Object target, Object permission)              | 当前目标用户是否具备指定目标的指定权限信息               |
| hasPermission(Object targetId, String targetType, Object permission) | 当前目标用户是否具备指定目标的指定权限信息               |



### 13.3.3 基于URL权限管理

```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        //roles 设置角色
        inMemoryUserDetailsManager.createUser(User.withUsername("root").password("{noop}123").roles("USER", "ADMIN").build());
        inMemoryUserDetailsManager.createUser(User.withUsername("zs").password("{noop}123").roles("USER").build());
        //authorities 设置权限字符串
        inMemoryUserDetailsManager.createUser(User.withUsername("ls").password("{noop}123").authorities("READ_INFO").build());
        return inMemoryUserDetailsManager;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                //hasRole 指定访问的角色,注意大小写相同
                .mvcMatchers("/admin").hasRole("ADMIN") //拥有ADMIN角色的用户才能访问
                .mvcMatchers("/user").hasRole("USER") //拥有USER角色的才能访问
                //hasAuthority 指定访问的权限字符串
                .mvcMatchers("/getInfo").hasAuthority("READ_INFO")  //拥有READ_INFO权限的才能访问
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable();
    }
}
```

contorller

```java
@RestController
public class Demo {

    @RequestMapping("/admin") //ADMIN角色访问
    public String admin() {
        return "admin ok";
    }

    @RequestMapping("/user") //USER角色访问
    public String user() {
        return "user ok";
    }

    @RequestMapping("/getInfo") //ROLE_INFO权限访问
    public String getInfo() {
        return "getInfo ok";
    }
}

```



### 13.3.4 基于AOP(方法)权限管理

​	基于方法的权限管理主要是**通过AOP**来实现的，Spring Security中通过`MethodSecurityInterceptor`来提供相关的实现。不同在于FilterSecurityInterceptor只是在请求之前进行前置处理**，MethodSecurityInterceptor除了前置处理之外还可以进行后置处理（环绕通知@Around）**。前置处理就是在请求之前判断是否具备相应的权限，后置处理则是对方法的执行结果进行二次过滤。前置处理和后置处理分别对应了不同的实现类。



#### 13.3.4.1 @EnableGlobalMethodSecurity

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled=true,securedEnabled=true,jsr250Enabled=true)
public class SecurityConfig extends WebsecurityConfigurerAdapter{

}
```

- perPostEnabled:开启Spring Security提供的四个权限注解，**@PostAuthorize、@PostFilter、@PreAuthorize以及@PreFilter**。
- securedEnabled：开启Spring Security提供的@Secured注解支持，该注解不支持权限表达式
- jsr250Enabled：开启JSR-250提供的注解，主要是@DenyAll、@PermitAll、@RolesAll,同样这些注解也不支持权限表达式

**权限表达式使用：@PreAuthorize(“hasRole('ADMIN')”)**

```markdown
# 以上注解含义如下：
- @PostAuthorize： 在目标方法执行之后进行权限校验。
- @PostFilter： 在目标方法执行之后对方法的返回结果进行过滤。
- @PreAuthorize： 在目标方法执行之前进行权限校验。--最常用
- @PreFilter： 在目标方法执行之前对方法参数进行过滤。
- @secured： 访问目标方法必须具备相应的角色。
- @DenyAll： 拒绝所有访问。
- @PermitAll： 允许所有访问。
- @RolesAllowed： 访问目标方法必须具备相应的角色。
```

这些基于方法的权限管理相关的注解，一般来说只要设置**<font color = 'red'>prePostEnabled=true</font>**就够用了。



#### 13.4.4.2 基本使用

- 在配置类上添加`@EnableGlobalMethodSecurity`注解,并且设置权限相关注解开启

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true,jsr250Enabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        inMemoryUserDetailsManager.createUser(User.withUsername("root").password("{noop}123").roles("USER", "ADMIN").build());
        inMemoryUserDetailsManager.createUser(User.withUsername("zs").password("{noop}123").roles("USER").build());
        inMemoryUserDetailsManager.createUser(User.withUsername("ls").password("{noop}123").authorities("READ_INFO").build());
        return inMemoryUserDetailsManager;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers(HttpMethod.GET, "/admin").hasRole("ADMIN")
                .mvcMatchers("/user").hasRole("USER")
                .mvcMatchers("/getInfo").hasAuthority("READ_INFO")
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable();
    }
}
```

- 在controller类上或者方法上添加权限注解

**springSecurity注解 ，可以使用权限表达式**

```java
    @GetMapping
    //用户的角色必须为ADMIN并且用户的角色名必须为root
    @PreAuthorize("hasRole('ADMIN') and authentication.name == 'root'")
    public String hello() {
        return "hello";
    }

    @GetMapping("/demo1")
    //验证权限信息
    @PreAuthorize("hasAuthority('READ_INFO')")
    public String demo1() {
        return "demo1";
    }

    @GetMapping("/name")
    //认证后的用户名必须和传参过来的用户名相同
    @PreAuthorize("authentication.name == #name")
    public String demo2(String name) {
        return "name:" + name;
    }

    @GetMapping("/id")
    //返回的参数id必须为1
    @PostAuthorize("returnObject.id == 1")
    public User getUserById(Integer id) {
        return new User(id, "root");
    }

    @PostMapping("/userId")
    //对请求的参数进行过滤
    @PreFilter(value = "filterObject.id%2!=0", filterTarget = "users")
    //filterTarget 必须为一个数组或者集合
    public List<User> addUserId(@RequestBody List<User> users) {
        return users;
    }

    @GetMapping("/list")
    //返回的id必须是奇数，必须为一个数组或者集合
    @PostFilter("filterObject.id%2!=0")
    public List<User> getAll() {
        List<User> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(new User(i, "root" + i));
        }
        return list;
    }
```

**以下注解不支持权限表达式，不常用**

```java
    @GetMapping("/secured")
    //只能判断角色,而且必须补全ROLE_
    @Secured("ROLE_USER")
    public String secured() {
        return "secured";
    }

    @GetMapping("/secured1")
    //只能判断角色.其中一个就行
    @Secured({"ROLE_USER", "ROLE_ADMIN"})
    public String secured1() {
        return "secured1";
    }

    @PermitAll
	//允许所有访问
    @GetMapping("/permitAll")
    public String permitAll(){
        return "PermitAll";
    }

    @DenyAll
	//拒绝访问
    @GetMapping("/denyAll")
    public String denyAll(){
        return "DenyAll";
    }
	
	//访问目标方法必须具备相应的角色
    @RolesAllowed({"ROLE_ADMIN","ROLE_USER"})  //具有其中一个角色即可
    @GetMapping("/rolesAllowed")
    public String rolesAllowed(){
        return "RolesAllowed";
    }
```



## 13.4 授权原理分析

![springSecurity授权](.\img.assets\9e393ee40af44fe19eaeaead9fa3bf13.png)

| 过滤器                                             | 作用         | 是否默认开启 |
| -------------------------------------------------- | ------------ | ------------ |
| <font color="red">FilterSecurityInterceptor</font> | 处理授权相关 | YES          |

- **ConfigAttribute** 在Sring Security中，用户请求一个资源(通常是一个接口或者一个Java 方法)需要的角色会被封装成一个ConfigAttribute对象，在ConfigAttribute中只有一个`getAttribute`方法，该方法返回一个String字符串，就是角色的名称。一般来说，角色名称都带有一个`ROLE_` 前缀，投票器AccessDecisionVoter所做的事情，其实就是比较用户所具有的各个角色和请求某个资源所需的ConfigAttribue之间的关系。
- **AccessDecisionVoter和AccessDecisionManager**都有众多的实现类，在AccessDecisionManager中会逐个遍历AccessDecisionVoter,进而决定是否允许用户访问，因而AccessDecisionVoter和AccessDecisionManager两者的关系类似于AuthenticationProvider和ProviderManager的关系。



### 13.4.1 FilterSecurityInterceptor

FilterSecurityInterceptor使用来处理授权相关的Filter，在过滤方法`dofilter`中调用了`invoke`方法

![image-20220924153253437](img.assets\image-20220924153253437.png)

在`invoke`方法中去调用了父类的`beforeInvocation（）`方法**获取授权相关信息**

在该方法的`obtainSecurityMetadataSource()`中，我们可以看到我们**定义的需要授权的资源**

![image-20220924153438184](img.assets\image-20220924153438184.png)

**在attemptAuthorization方法中完成授权相关操作**

![image-20220924154139346](img.assets\image-20220924154139346.png)

在`attemptAuthorization（）`方法中调用`AccessDecisionManager`授权接口中的decide方法开始进行授权

![image-20220924154627451](img.assets\image-20220924154627451.png)

在`AccessDecisionManager`接口的实现类中调用`AccessDecisionVoter`访问决定投票器接口中的`vote`方法进行投票处理

![image-20220924154558402](img.assets\image-20220924154558402.png)



### 13.4.2 SecurityMetadataSource

从上述的源码中我们可以得知,**要授权的资源的获取**都是通过`AbstractSecurityInterceptor`类中的方法来进行的

```java
Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource().getAttributes(object);
```

该obtainSecurityMetadataSource()方法是调用的子类` FilterSecurityInterceptor`中的方法来获取，并且通过set方法来赋值

```java
	@Override
	public SecurityMetadataSource obtainSecurityMetadataSource() {
		return this.securityMetadataSource;
	}
```

![image-20220924153833627](img.assets\image-20220924153833627.png)



#### 13.4.2.1 FilterInvocationSecurityMetadataSource

该接口继承了`SecurityMetadataSource`接口

```java
public interface FilterInvocationSecurityMetadataSource extends SecurityMetadataSource {}
```

**SecurityMetadataSource**接口:

```java
public interface SecurityMetadataSource extends AopInfrastructureBean {
	//this.obtainSecurityMetadataSource().getAttributes(object) 调用的就是此处的方法来获取授权资源信息
	Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException;

	Collection<ConfigAttribute> getAllConfigAttributes();

	boolean supports(Class<?> clazz);

}
```

**所以，我们如果要实现自定义数据源来实现授权资源的获取，应该去实现FilterInvocationSecurityMetadataSource接口,自定义实现**



## 13.5 动态授权

动态管理权限规则就是我们将URL拦截规则和访问URI所需要的权限都保存在数据库中，这样，在不修改源代码的情况下，只需要修改数据库中的数据，就可以对权限进行调整。



### 13.5.1 数据库表设计

用户 <----> 中间表 <----> 角色 <----> 中间表 <---->菜单(资源路径)

![image-20220924162536564](img.assets\image-20220924162536564.png)

#### 13.5.1.1 菜单表

菜单表中保存资源路径

```mysql
create table `menu`(
		`id` int(11) not null auto_increment,
		`pattern` varchar(128) default null,
		primary key(`id`)
)engine=Innodb auto_increment=4 default charset=utf8;

insert into `menu` values(1,'/admin/**');
insert into `menu` values(2,'/user/**');
insert into `menu` values(3,'/guest/**');
```



#### 13.5.1.2 角色表

```mysql
create table `role`(
	`id` int(11) not null auto_increment,
	`name` varchar(32) default null,
	`name_zh` varchar(32) default null,
	primary key(`id`)
)engine=innodb auto_increment=4 default charset=utf8;

insert into `role` values(1,'ROLE_ADMIN','系统管理员');
insert into `role` values(2,'ROLE_USER','普通用户');
insert into `role` values(3,'ROLE_GUEST','游客');
```



#### 13.5.1.3 用户名表

```mysql
create table `user`(
	`id` int(11) not null auto_increment,
	`username` varchar(32) default null,
	`password` varchar(255) default null,
	`enabled` tinyint(1) default null,
	`locked` tinyint(1) default null,
	primary key(`id`)
)engine=InnoDB auto_increment=4 default charset=utf8;

insert into `user` values(1,'admin','{noop}123',1,0);
insert into `user` values(2,'user','{noop}123',1,0);
insert into `user` values(3,'zkt','{noop}123',1,0);
```



#### 13.5.1.4 中间表 

#####   用户和角色连接表

```mysql
create table `user_role`(
		`id` int(11) not null auto_increment,
		`uid` int(11) default null,
		`rid` int(11) default null,
		primary key(`id`),
		key `uid`(`uid`),
		key `rid`(`rid`),
		CONSTRAINT `user_role_ibfk_1` foreign key(`uid`) REFERENCES `user`(id),
		constraint `user_role_ibfk_2` foreign key(`rid`) REFERENCES `role`(`id`)	
	)engine=innodb auto_increment=5 default charset=utf8;
	
insert into `user_role` VALUES(1,1,1);
insert into `user_role` VALUES(2,1,2);
insert into `user_role` VALUES(3,2,2);
insert into `user_role` VALUES(4,3,3);
```

##### 角色和菜单连接表

```mysql
create table `menu_role`(
	`id` int(11) not null auto_increment,
	`mid` int(11) default null,
	`rid` int(11) default null,
	primary key(`id`),
	key `mid`(`mid`),
	key `rid`(`rid`),
	constraint `menu_role_ibfk_1` foreign key(`mid`) REFERENCES `menu`(`id`),
	constraint `menu_role_ibfk_2` foreign key(`rid`) REFERENCES `role`(`id`)
)engine=Innodb auto_increment=5 default charset=utf8;

insert into `menu_role` values(1,1,1);
insert into `menu_role` values(2,2,2);
insert into `menu_role` values(3,3,3);
insert into `menu_role` values(4,3,2);
```



### 13.5.2 实体类

#### 角色

```java
@Data
public class Role {
    private Integer id;
    private String name;
    private String nameZh;
}
```

#### 用户

```java
@Data
public class User implements UserDetails {

    private Integer id;
    private String username;
    private String password;

    private boolean enabled;
    private boolean locked;
    private List<Role> roles;

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
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return !locked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream().map(role -> new SimpleGrantedAuthority(role.getName())).collect(Collectors.toSet());
    }
}
```

#### 菜单

```java
@Data
public class Menu {
    private Integer id;
    private String pattern;
    //资源路径对应的角色信息
    private List<Role> roles;
}
```



### 13.5.3 controller

```java
@RestController
public class Demo {


    /**
     * 访问要求:
     * 用户    角色                   可以访问的资源
     * admin  ROLE_ADMIN,ROLE_USER   /admin/**,/user/**
     * user   ROLE_USER              /user/**
     * xh     ROLE_GUEST             /guest/**
     */

    @GetMapping("/admin/hello")
    public String admin() {
        return "hello admin";
    }

    @GetMapping("/user/hello")
    public String user() {
        return "hello user";
    }

    @GetMapping("/guest/hello")
    public String guest() {
        return "hello guest";
    }

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```



### 13.5.4  认证实现

```java
@Component
public class SecurityUserDetailsService implements UserDetailsService, UserDetailsPasswordService {

    private final UserDetailsMapper userDetailsMapper;

    @Autowired
    public SecurityUserDetailsService(UserDetailsMapper userDetailsMapper) {
        this.userDetailsMapper = userDetailsMapper;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userDetailsMapper.loadUserByUsername(username);
        if (Objects.nonNull(user)) {
            final Integer uid = user.getId();
            List<Role> roles = userDetailsMapper.findRolesByUserId(uid);
            if (!roles.isEmpty()) {
                user.setRoles(roles);
            }
        }
        return user;
    }

    @Override
    public UserDetails updatePassword(UserDetails user, String newPassword) {
        User myUser = (User) user;
        final Integer id = myUser.getId();
        Integer result = userDetailsMapper.updatePasswordById(id, newPassword);
        if (result == 1) {
            myUser.setPassword(newPassword);
            return myUser;
        }
        return null;
    }
}

```

SQL

```mysql
	<update id="updatePasswordById">
        update security_authorize.user
        set password = #{newPassword}
        where id = #{id}
    </update>

    <select id="loadUserByUsername" resultType="com.xh.dynamicAuthorize.domain.User">
        select id, username, password, enabled, locked
        from security_authorize.user
        where username = #{username}
    </select>

    <select id="findRolesByUserId" resultType="com.xh.dynamicAuthorize.domain.Role">
        select r.id, name, nameZh
        from security_authorize.role r,
             security_authorize.user_role ur
        where r.id = ur.rid
          and ur.uid = #{uid}
    </select>
```



### 13.5.5 授权实现

```java
@Component
public class CustomerSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {

    private final MenuMapper menuMapper;

    //比较当前菜单和请求路径
    AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Autowired
    public CustomerSecurityMetadataSource(MenuMapper menuMapper) {
        this.menuMapper = menuMapper;
    }

    /**
     * 自定义动态资源元数据信息
     *
     * @param object
     * @return
     * @throws IllegalArgumentException
     */
    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {
        //1.object为当前的请求对象
        final HttpServletRequest request = ((FilterInvocation) object).getRequest();
        //查询所有菜单
        final List<Menu> allMenu = menuMapper.getAllMenu();
        for (Menu menu : allMenu) {
            //如果当前请求路径和菜单中的路径匹配
            if (antPathMatcher.match(menu.getPattern(), request.getRequestURI())) {
                //获取菜单中的的角色信息并且将其转换为string数组
                final String[] roles = menu.getRoles().stream().map(Role::getName).toArray(String[]::new);
                //创建角色信息
                return SecurityConfig.createList(roles);
                //认证流程 : 资源路径匹对 --> 判断登录用户的角色信息
            }
        }
        return null;
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return FilterInvocation.class.isAssignableFrom(clazz);
    }
}
```

SQL:

```mysql
	<resultMap id="MenuResultMap" type="com.xh.dynamicAuthorize.domain.Menu">
        <id property="id" column="id"/>
        <result property="pattern" column="pattern"/>
        <collection property="roles" ofType="com.xh.dynamicAuthorize.domain.Role">
            <id property="id" column="id"/>
            <result property="name" column="name"/>
            <result property="nameZh" column="nameZh"/>
        </collection>
    </resultMap>

    <select id="getAllMenu" resultMap="MenuResultMap">
        select m.id, m.pattern, r.id, r.name, r.nameZh
        from menu m
                 left join menu_role mr on m.id = mr.mid
                 left join security_authorize.role r on mr.rid = r.id
    </select>
```



### 13.5.6 配置类

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    private final SecurityUserDetailsService userDetailsService;

    private final CustomerSecurityMetadataSource customSecurityMetadataSource;

    @Autowired
    public MySecurityConfig(SecurityUserDetailsService userDetailsService, CustomerSecurityMetadataSource customSecurityMetadataSource) {
        this.userDetailsService = userDetailsService;
        this.customSecurityMetadataSource = customSecurityMetadataSource;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //自定义授权对象
        //1.获取工厂对象
        ApplicationContext applicationContext = http.getSharedObject(ApplicationContext.class);
        //2.设置自定义url权限处理
        http.apply(new UrlAuthorizationConfigurer<>(applicationContext))
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O object) {
                        object.setSecurityMetadataSource(customSecurityMetadataSource);
                        //是否拒绝公共资源访问
                        object.setRejectPublicInvocations(false);
                        return object;
                    }
                });
        //认证
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .csrf().disable();
    }
}
```

