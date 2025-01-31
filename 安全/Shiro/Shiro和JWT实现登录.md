# Shiro和JWT实现登录

## 1.导入依赖

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.2.1</version>
</dependency>

<dependency>
     <groupId>org.apache.shiro</groupId>
     <artifactId>shiro-spring-boot-web-starter</artifactId>
     <version>1.11.0</version>
</dependency>
```

```yaml
emos:
  jwt:
    #密钥
    secret-key: asf@afxv
    #token过期时间
    expire-time: 7
    #缓存过期时间
    cache-expire-time: 15
```



## 2.JWT工具类

```java
mport com.auth0.jwt.JWT;
import com.auth0.jwt.JWTCreator;
import com.auth0.jwt.algorithms.Algorithm;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.time.Instant;
import java.time.LocalDate;
import java.util.Map;
import java.util.Set;

@Component
public class JWTUtils {

    @Value("${emos.jwt.secret-key}")
    //密钥
    private String secretKey;

    @Value("${emos.jwt.expire-time}")
    //过期时间
    private String expireTime;

    /**
     * 生成token
     * @param map 需要保存到payload中的数据,需要保存key为userId的用户名id
     * @return token字符串
     */
    // token --> header(默认).payload(用户的信息，过期时间).sigNature(加密算法和盐)
    public String getToken(Map<String,Object> map){

        JWTCreator.Builder builder = JWT.create();

        Set<Map.Entry<String, Object>> entries = map.entrySet();
        for(Map.Entry<String,Object> entry : entries){
            //往payload中添加数据
            builder.withClaim(entry.getKey(),entry.getValue().toString());
        }

        //添加过期时间
        LocalDate localDate = LocalDate.now().plusDays(Long.parseLong(expireTime));
        builder.withExpiresAt(Instant.from(localDate));

        //设置加密密钥
        return builder.sign(Algorithm.HMAC256(secretKey));
    }

    /**
     * 解析token
     * @param token token令牌
     */
    public void verify(String token){
        //解析算法要和加密的算法相同
        JWT.require(Algorithm.HMAC256(secretKey)).build().verify(token);
    }

    /**
     * 获取用户id
     * @param token token令牌
     * @return 用户id
     */
    public Integer getUserId(String token){
        return JWT.decode(token).getClaim("userId").asInt();
    }
}
```



## 3.把令牌封装成认证对象

![image-20230228140348523](img.assets\image-20230228140348523.png)

客户端提交的Token不能直接交给Shiro框架，需要先封装成`AuthenticationToken`类型的对象，所以我们我们需要先创建`AuthenticationToken`的实现类。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class OAuth2Token implements AuthenticationToken {

    private String token;

    @Override
    public Object getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}
```





## 4.刷新令牌机制

我们在定义JwtUtil工具类的时候，生成的`Token`都有过期时间。那么问题来了，假设Token过期时间为15天，用户在第14天的时候，还可以免登录正常访问系统。但是到了第15天，用户的Token过期，于是用户需要重新登录系统。
`Httpsession`的过期时间比较优雅，默认为15分钟。如果用户连续使用系统，只要间隔时间不超过15分钟，系统就不会销毁HttpSession对象。JWT的令牌过期时间能不能做成`HttpSession`那样超时时间，只要用户间隔操作时间不超过15天，系统就不需要用户重新登录系统。实现这种效果的方案有两种:`双Token`和`Token缓存`，这里重点讲一下Token缓存方案。

![image-20230228144400209](img.assets\image-20230228144400209.png)

Token缓存方案是把Token缓存到Redis，然后设置Redis里面`缓存的Token过期时间为正常Token的1倍`,然后根据情况刷新Token的过期时间。



### 4.1 缓存令牌机制

- 令牌缓存到Redis上面

- 缓存的令牌过期时间是客户端令牌的一倍
- 如果客户端令牌过期，缓存令牌没有过期，则生成新的令牌
- 如果客户端令牌过期，缓存令牌也过期了，则用户必须重新登录



**Token失效，缓存也不存在的情况**

当第15天，用户的 `Token`失效以后，我们让Shiro程序到Redis查看是否存在缓存的`Token`，如果这个`Token`不存在于Redis里面，就说明用户的操作间隔了15天，需要重新登录。

**Token失效，但是缓存还存在的情况**

如果Redis中存在缓存的 `Token` ，说明当前`Token`失效后，间隔时间还没有超过15天，不应该让用户重新登录。所以要生成新的`Token`返回给客户端，并且把这个`Token`缓存到Redis里面，这种操作成为刷新`Token`过期时间。



### 4.2 如何更新令牌

服务端刷新Token过期时间，其实就是生成一个新的Token给客户端。那么客户端怎么知道这次响应带回来的Token是更新过的呢?

![image-20230301204019550](img.assets\image-20230301204019550.png)

只要用户成功登陆系统，当后端服务器更新Token的时候，就在响应中添加Token。客户端那边判断每次Ajax响应里面是否包含Token，如果包含，就把Token保存起来就可以了。



### 4.3 在响应中添加令牌

![image-20230301204212528](img.assets\image-20230301204212528.png)

我们定义`OAuth2Filter`类拦截所有的HTTP请求，一方面它会把请求中的 `Token`字符串提取出来，封装成对象交给Shiro框架;另一方面，它会检查`Token` 的有效性。如果 Token 过期，那么会生成新的Token，分别存储在`ThreadLocalToken`和 `Redis`中。

**==之所以要把新令牌保存到ThreadLocalToken里面，是因为要向AOP切面类传递这个新令牌==。**

虽然OAuth2Filter中有`doFilterInternal()`方法，我们可以得到响应并且写入新令牌。但是这样非常麻烦，首先我们要通过IO流读取响应中的数据，然后还要把数据解析成JSON对象，最后再放入这个新令牌。

**==如果我们定义了AOP切面类，拦截所有Web方法返回的响应对象，然后在响应对象里面添加新令牌，非常简单==。**

但是OAuth2Filter和AOP切面类之间没有调用关系，所以我们很难把新令牌传给AOP切面类。

这里使用到了ThreadLocal，**只要是同一个线程，往ThreadLocal里面写入数据和读取数据是完全相同的。**在Web项目中，从 OAuth2Filter到AoP切面类，都是由同一个线程来执行的，中途不会更换线程。所以我们可以放心的把新令牌保存都在ThreadLocal里面，AOP切面类可以成功的取出新令牌，然后往R对象里面添加新令牌即可。



## 5.创建ThreadLocalToken类

ThreadLocalToken是自定义的类，里面包含了ThreadLocal类型的变量，可以用来保存线程安全的数据，而且避免了使用**线程锁。**

```java
@Component
public class ThreadLocalToken {

    private final ThreadLocal<String> threadLocal = new ThreadLocal<>();

    /**
     * 放入token
     * @param token token令牌
     */
    public void setToken(String token){
        threadLocal.set(token);
    }

    /**
     * 获取token令牌
     * @return 返回Token令牌
     */
    public String getToken(){
        return threadLocal.get();
    }

    /**
     * 清除本地线程中保存的对象，防止内存溢出
     */
    public void clear(){
        threadLocal.remove();
    }

}
```



## 6.自定义过滤器AuthenticatingFilter

==1.判断哪些请求应该被Shiro处理:==

- options请求直接放行,其余所有请求都要被Shiro处理
  - 提交application/json数据
  - 请求被分成options和post两次

==2.判断Token是真过期还是假过期:==

- 真过期，返回提示信息，让用户重新登录
- 假过期，就生成新的令牌，返回给客户端

==3.存储新令牌==

- ThreadLocalToken
- Redis



>自定义过滤器一般实现org.apache.shiro.web.filter.authc.AuthenticatingFilter类；
>`AuthenticatingFilter`继承AccessControlFilter ；
>
>- `isAccessAllowed`方法配置允许访问的方法
>
>- `onAccessDenied`方法是校验的入口，定义校验的流程；
>
>- `executeLogin`方法定义校验的过程；
>
>- 在`executeLogin`，执行`createToken`方法，封装token；
>
>然后，执行登录过程；
>
>- 如果登录成功，执行`onLoginSuccess`；
>
>- 如果登录失败，执行`onLoginFailure`；
>
> 登录失败，一般抛出AuthenticationException异常；
>
>
>
>在response的流写入提示信息，交给前端处理即可；

```java
@Component
//springBean默认创建是单列
//@Scope("prototype")通知springBean创建对象为多例
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class OAuth2Filter extends AuthenticatingFilter {

    private final JWTUtils jwtUtils;

    private final RedisTemplate<String,Object> redisTemplate;

    private final ThreadLocalToken threadLocalToken;

    //redis中token过期时间
    @Value("${emos.jwt.cache-expire-time}")
    private String cacheExpireTime;

    @Autowired
    public OAuth2Filter(JWTUtils jwtUtils, RedisTemplate<String, Object> redisTemplate,ThreadLocalToken threadLocalToken) {
        this.jwtUtils = jwtUtils;
        this.redisTemplate = redisTemplate;
        this.threadLocalToken = threadLocalToken;
    }

    /**
     * 获取请求头中的token封装给Shiro
     * @param servletRequest 请求对象
     * @param servletResponse 响应对象
     * @return 封装了token的认证对象
     */
    @Override
    protected AuthenticationToken createToken(ServletRequest servletRequest, ServletResponse servletResponse){
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        String token = this.getRequestToken(request);
        if(StrUtil.isBlank(token)){
            return null;
        }
        return new OAuth2Token(token);
    }

    /**
     * 判断那种请求需要shiro对象进行处理
     * @param request 请求对象
     * @param response 响应对象
     * @param mappedValue 映射值
     * @return OPTIONS请求允许访问，返回true，其它类型都需要shiro进行处理，返回false
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        //如果请求方式是options方法，则允许访问
        return RequestMethod.OPTIONS.name().equals(httpServletRequest.getMethod());
    }

    /**
     * 访问被拒绝后(除了OPTIONS请求),都交给该方法处理
     * 1.验证token
     * 2.将token给ThreadLocalToken
     * @param servletRequest 请求对象
     * @param servletResponse 响应对象
     * @return token无效或者token验证失败，则返回false
     * @throws Exception 异常信息
     */
    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        //设置响应类型和响应字符集编码,跨域
        HttpServletResponse response = this.setServletResponseConfig((HttpServletResponse) servletResponse);
        //获取token
        String token = this.getRequestToken(request);
        //如果token为null,则返回错误信息
        if(StrUtil.isBlank(token)){
            this.printErrorTokenResponse(response,Status.TOKEN_VOID);
            return false;
        }
        //手动处理异常
        try{
            jwtUtils.verifyToken(token);
            //如果token过期，则去redis中查询
        }catch (TokenExpiredException e){
            //如果redis中的缓存还在，则删除该缓存，重新生成token并且保存
            if(Boolean.TRUE.equals(redisTemplate.hasKey(token))){
                //删除缓存
                redisTemplate.delete(token);
                //从token中获取用户数据
                Integer userId = jwtUtils.getUserId(token);
                //重新生成token
                Map<String,Object> map = new HashMap<>();
                map.put("userId",userId);
                String newToken = jwtUtils.getToken(map);
                //重新保存token
                redisTemplate.opsForValue().set(newToken,userId,Long.parseLong(cacheExpireTime), TimeUnit.DAYS);
                threadLocalToken.setToken(token);
            }else {
                //如果redis中缓存不存在，则提示用户重新登录
                this.printErrorTokenResponse(response,Status.TOKEN_EXPIRED);
                return false;
            }
          //如果JWT验证失败，则直接返回令牌无效提示
        }catch (JWTDecodeException e){
            this.printErrorTokenResponse(response,Status.TOKEN_VOID);
            return false;
        }
        //token验证成功后，执行登录方法,如果登录失败，则需要自定义登录失败后的方法，重新onLoginFailure类
        return this.executeLogin(request, response);
    }

    /**
     * 自定义登录失败后的方法
     * @param token 身份验证令牌
     * @param e 异常信息
     * @param request 请求对象
     * @param response 响应对象
     * @return 验证失败，返回false
     */
    @SneakyThrows
    @Override
    protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e,
                                     ServletRequest request, ServletResponse response) {
        //设置响应信息
        HttpServletResponse httpServletResponse = this.setServletResponseConfig((HttpServletResponse)response);
        //打印登录失败错误信息
        this.printErrorTokenResponse(httpServletResponse,Status.LOGIN_ERROR);
        return false;
    }

    /**
     * 作用和servlet中的过滤器方法相同
     * @param request 请求对象
     * @param response 响应对象
     * @param chain 过滤器链，执行放行方法
     * @throws ServletException servlet异常
     * @throws IOException IO异常
     */
    @Override
    public void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
        super.doFilterInternal(request, response, chain);
    }

    /**
     * 从请求对象中获取token令牌
     * @param request 请求对象
     * @return token令牌
     */
    private String getRequestToken(HttpServletRequest request){
        //从请求头中获取
        String token = request.getHeader("token");
        //如果请求头中没有，则从请求体中获取
        if(StrUtil.isBlank(token)){
            token = request.getParameter("token");
        }
        return token;
    }

    /**
     * 设置响应对象字符集，返回类型以及跨域
     * @param response 响应对象
     * @return 设置后的响应对象
     */
    private HttpServletResponse setServletResponseConfig(HttpServletResponse response){
        //设置响应类型和响应字符集编码
        response.setContentType("application/json;charset=UTF-8");
        response.setCharacterEncoding("UTF-8");
        //设置响应跨域
        response.setHeader("Access-Control-Allow-Credentials", "true");
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.addHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
        response.addHeader("Access-Control-Allow-Headers", "Content-Type");
        return response;
    }

    /**
     * 打印验证token失败信息
     * @param response 响应对象
     * @param status 失败信息状态码和描述
     * @throws IOException IO异常
     */
    private void printErrorTokenResponse(HttpServletResponse response,Status status) throws IOException {
        Result<String> result = Result.error(status.getCode(),status.getMessage());
        String resultJSON = JSONUtil.toJsonStr(result);
        response.getWriter().print(resultJSON);
    }
}
```



## 7.实现认证与授权的方法-Realm类

继承`AuthorizingRealm`，重写`doGetAuthenticationInfo`认证和`doGetAuthorizationInfo`授权方法

### 7.1 实现认证方法

![image-20230308204744731](img.assets\image-20230308204744731.png)

#### 7.1.1 查询用户信息

因为在认证方法里面要返回认证对象，认证对象创建的时候要传入用户信息和令牌，所以我们这里就要`查询用户信息`，然后判断用现在是在职还是离职状态。如果是在职状态，那就可以创建认证对象，反之就抛出异常。

```sql
    <select id="searchUserById" resultType="com.xh.emosWxApi.entity.domain.TbUser">
        SELECT
        <include refid="Base_Column_List"/>
        FROM tb_user
        WHERE id = #{userId} AND status = 1
    </select>
```

#### 7.1.2 重写`doGetAuthenticationInfo`认证方法

```java
@Component
public class OAuth2Realm extends AuthorizingRealm {

    private final TbUserService tbUserService;

    private final JWTUtils jwtUtils;

    @Autowired
    public OAuth2Realm(TbUserService tbUserService, JWTUtils jwtUtils) {
        this.tbUserService = tbUserService;
        this.jwtUtils = jwtUtils;
    }
    /**
     * 自定义认证方法
     * @param token 用户身份信息
     * @return 自定义认证信息封装对象
     * @throws AuthenticationException 认证异常
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取登录时的token
        String accessToken = (String) token.getPrincipal();
        //从token中拿取用户id
        Integer userId = jwtUtils.getUserId(accessToken);
        //获取用户信息
        TbUser tbUser = tbUserService.searchUserById(userId);
        if(Objects.isNull(tbUser)){
            throw new LockedAccountException();
        }
        //封装简单身份认证信息给shiro
        return new SimpleAuthenticationInfo(tbUser,accessToken, this.getName());
    }

}

```



### 7.2 实现授权方法

![image-20230308204744731](img.assets\image-20230308204744731.png)

Shiro每次验证权限之前，都要执行授权方法，把用户具有的权限封装成权限对象，然后放行请求。接下来Web方法的`@RequiresPermissions`注解，会从权限对象中提取权限数据，跟要求的权限作比较。如果用户具有该Web方法要求的权限，那么Web方法就会正常执行。反之则返回异常消息。修改`OAuth2Realm`中的`doGetAuthorizationInfo()`授权方法。

#### 7.2.1 获取用户权限

```sql
    <select id="searchUserPermissionsByUserId" resultType="java.lang.String">
        SELECT permission_name
        FROM tb_user u
                 JOIN tb_role r ON JSON_CONTAINS(u.role, CAST(r.id AS CHAR))
                 JOIN tb_permission p ON JSON_CONTAINS(r.permissions, CAST(p.id AS CHAR))
        WHERE u.id = #{userId}
          AND u.status = 1
    </select>
```



#### 7.2.2 重写`doGetAuthorizationInfo`授权方法

```java
@Component
public class OAuth2Realm extends AuthorizingRealm {

    private final TbUserService tbUserService;

    private final JWTUtils jwtUtils;

    @Autowired
    public OAuth2Realm(TbUserService tbUserService, JWTUtils jwtUtils) {
        this.tbUserService = tbUserService;
        this.jwtUtils = jwtUtils;
    }

    /**
     * 自定义授权方法
     * @param principals 当前用户身份信息
     * @return 自定义授权信息封装对象
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		//获取用户信息
        TbUser tbUser = (TbUser) principals.getPrimaryPrincipal();
        //中获取用户id
        Integer userId = tbUser.getId();
		//获取用户权限
        Set<String> permissionsSet = tbUserService.searchUserPermissions(userId);
        //获取用户角色
        Set<String> rolesSet = tbUserService.searchUserRoles(userId);
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setRoles(rolesSet);
        simpleAuthorizationInfo.addStringPermissions(permissionsSet);
        return simpleAuthorizationInfo;
    }
}

```

#### 7.2.3 Controller上添加注解

```java
    @RequiresPermissions(value = {"ROOT","USER:ADD"},logical = Logical.OR)
```



### 7.3 配置令牌检查

在`AuthorizingRealm`的继承类中重写`supports`方法，自定义令牌检查规则。

![image-20230309151154250](img.assets\image-20230309151154250.png)

```java
    /**
     * 可以被子类覆盖，以进行更复杂的令牌检查
     * 默认返回：
     * token != null && getAuthenticationTokenClass().isAssignableFrom(token.getClass())
     * 若不修改，则返回true,
     * !realm.supports(token)为false，无法通过令牌检查
     *
     * @param token 自定义令牌
     * @return 如果该令牌是自定义令牌，则返回true
     */
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof OAuth2Token;
    }
```



## 8.配置ShiroConfig

- 把Filter和Realm添加到Shiro框架
- 创建四个对象返回给SpringBoot
  - SecurityManager -用于封装Realm对象
  - ShiroFilterFactoryBean -用于封装Filter对象
  - LifecycleBeanPostProcessor -管理Shiro对象生命周期
  - AuthorizationAttributeSourceAdvisor - AOP切面类，Web方法执行前，验证权限

```java
@Configuration
public class ShiroConfig {

    /**
     * 自定义认证，授权源配置
     *
     * @param oAuth2Realm 自定义认证，授权数据源
     * @return 将自定义Realm注册到shiro中
     */
    @Bean
    //使用DefaultSecurityManager,shiro启动时会自动创建一个"securityManager"的bean，造成冲突
    //或者自定义名字
    public DefaultSecurityManager defaultSecurityManager(OAuth2Realm oAuth2Realm) {
        //使用DefaultWebSecurityManager
        //否则会报The security manager does not implement the WebSecurityManager interface.错误
        DefaultWebSecurityManager defaultSecurityManager = new DefaultWebSecurityManager();
        //封装Realm对象
        defaultSecurityManager.setRealm(oAuth2Realm);
        //取消remember操作
        defaultSecurityManager.setRememberMeManager(null);
        return defaultSecurityManager;
    }

    /**
     * 将自定义认证授权数据源和过滤器注册到Shiro过滤器中
     *
     * @param securityManager 自定义认证，配置数据源
     * @param auth2Filter     自定义过滤器
     * @return 将自定义认证授权数据源和过滤器注册到Shiro过滤器中
     */
    @Bean("shiroFilterFactoryBean")
    //指定bean的名字为shiroFilterFactoryBean，启动shiro时需要该名字的Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(@Qualifier("defaultSecurityManager") SecurityManager securityManager,
                                                         OAuth2Filter auth2Filter) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置自定义认证授权数据源
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        //设置自定义过滤器
        Map<String, Filter> map = new HashMap<>();
        map.put("oauth2", auth2Filter);
        shiroFilterFactoryBean.setFilters(map);
        //设置不认证可以访问的资源
        Map<String, String> filterMap = new LinkedHashMap<>();
        filterMap.put("/webjars/**", "anon");
        filterMap.put("/druid/**", "anon");
        filterMap.put("/app/**", "anon");
        filterMap.put("/sys/login", "anon");
        filterMap.put("/swagger/**", "anon");
        filterMap.put("/v2/api-docs", "anon");
        filterMap.put("/swagger-ui.html", "anon");
        filterMap.put("/swagger-resources/**", "anon");
        //Knife4j
        filterMap.put("/doc.html", "anon");
        filterMap.put("/captcha.jpg", "anon");
        filterMap.put("/user/register", "anon");
        filterMap.put("/user/login", "anon");
        filterMap.put("/test/**", "anon");
        filterMap.put("/meeting/recieveNotify", "anon");
        filterMap.put("/**", "oauth2");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
        return shiroFilterFactoryBean;
    }

    /**
     * shiro生命周期管理
     *
     * @return 将该类注入到Bean中
     */
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    /**
     * AOP切面类，Web方法执行前，验证权限
     *
     * @param securityManager 自定义认证授权
     * @return 将该类注入到Bean中
     */
    @Bean
    public ShiroAnnotationProcessorAutoConfiguration shiroAnnotationProcessorAutoConfiguration(@Qualifier("defaultSecurityManager") SecurityManager securityManager) {
        ShiroAnnotationProcessorAutoConfiguration shiroAnnotationProcessorAutoConfiguration = new ShiroAnnotationProcessorAutoConfiguration();
        shiroAnnotationProcessorAutoConfiguration.authorizationAttributeSourceAdvisor(securityManager);
        return shiroAnnotationProcessorAutoConfiguration;
    }
}
```



## 9.创建AOP切面类返回新的token

- 拦截所有的Web方法返回值
- 判断是否刷新生成新令牌
  - 检查ThreadLocal中是否保存令牌
  - 把新令牌绑定到Result对象中

```java
@Aspect
@Component
@Order(1)
public class AopToken {

    private final ThreadLocalToken threadLocalToken;

    @Autowired
    public AopToken(ThreadLocalToken threadLocalToken) {
        this.threadLocalToken = threadLocalToken;
    }

    /**
     * 切入点
     */
    @Pointcut("execution(public * com.xh.emosWxApi.controller.*Controller.*(..)))")
    public void pointcut() {
    }

    /**
     * 环绕通知，将新生成的token携带返回
     *
     * @param point 切入点
     * @return 携带了token返回的新数据
     */
    @Around("pointcut()")
    public Object aroundToken(ProceedingJoinPoint point) throws Throwable {
        Result<Object> proceed = (Result) point.proceed();
        String token = threadLocalToken.getToken();
        if (Objects.nonNull(token)) {
            //将token放入的要返回的数据中
            Map<String, Object> map = proceed.getData();
            map.put("token", token);
            proceed.setData(map);
            //清除token，保证内存不溢出
            threadLocalToken.clear();
        }
        return proceed;
    }
}
```



