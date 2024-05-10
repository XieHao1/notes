# JWT

![image-20200726102546868](JWT.assets/image-20200726102546868.png)

## 1.什么是JWT

```markdown
# 1.官方翻译
-  官网地址: https://jwt.io/introduction/
-  翻译: jsonwebtoken（JWT）是一个开放标准（rfc7519），它定义了一种紧凑的、自包含的方式，用于在各方之间以JSON对象安全地传输信息。此信息可以验证和信任，因为它是数字签名的。jwt可以使用秘密（使用HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名

# 2.通俗解释
- JWT简称JSON Web Token,也就是通过JSON形式作为Web应用中的令牌,用于在各方之间安全地将信息作为JSON对象传输。在数据传输过程中还可以完成数据加密、签名等相关处理。
```

### token验证用户身份的流程

```markdown
- 1.客户端使用用户名和密码请求登录
- 2.服务端收到请求，验证用户名和密码
- 3.验证成功后，服务端会签发一个token，再把这个token返回给客户端
- 4.客户端收到token后可以把它存储起来，比如放到sessionStrong,localStrong中
- 5.客户端每次向服务端请求资源时需要携带服务端签发的token，可以在header中携带
- 6.服务端收到请求，然后去验证客户端请求里面带着的token，如果验证成功，就向客户端返回请求数据
```



## 2. JWT能做什么

```markdown
# 1.授权
- 这是使用JWT的最常见方案。一旦用户登录，每个后续请求将包括JWT，从而允许用户访问该令牌允许的路由，服务和资源。x，因为它的开销很小并且可以在不同的域中轻松使用。

# 2.信息交换
- JSON Web Token是在各方之间安全地传输信息的好方法。因为可以对JWT进行签名（例如，使用公钥/私钥对），所以您可以确保发件人是他们所说的人。此外，由于签名是使用标头和有效负载计算的，因此您还可以验证内容是否遭到篡改。
```

## 3.为什么是JWT

​		传统session是保存在服务器内存中的，而JWT是保存在浏览器本地的，使用JWT的好处就是不占用服务器的内存，使用session的话每一个用户都占用一点服务器的内存。

### (1).基于传统的Session认证

```markdown
# 1.认证方式
- 我们知道，http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了,这就是传统的基于session认证。

# 2.认证流程
```

![image-2020072610 3959013](JWT.assets/image-20200726103959013.png)

```markdown
# 3.暴露问题
- 1.每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大

- 2.用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。

- 3.因为是基于cookie来进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。

- 4.在前后端分离系统中就更加痛苦:如下图所示
	也就是说前后端分离在应用解耦后增加了部署的复杂性。
	通常用户一次请求就要转发多次。如果用session每次携带sessionid到服务器，服务器还要查询用户信息。同时如果用户很多。这些信息存储在服务器内存中，给服务器增加负担。
	还有就是CSRF（跨站伪造请求攻击）攻击，session是基于cookie进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。
	还有就是sessionid就是一个特征值，表达的信息不够丰富。不容易扩展。
	而且如果你后端应用是多节点部署。那么就需要实现session共享机制。不方便集群应用。
```

![image-20200804212240422](JWT.assets/image-20200804212240422.png)

### (2).基于JWT认证

![image-20200726183248298](JWT.assets/image-20200726183248298.png)

```markdown
# 1.认证流程
- 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。

- 后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT(Token)。形成的JWT就是一个形同lll.zzz.xxx的字符串。 
	token head.payload.singurater

- 后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即可。

- 前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题) HEADER

- 后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己（可选）。

- 验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。

# 2.jwt优势

- 简洁(Compact): 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快

- 自包含(Self-contained)：负载中包含了所有用户所需要的信息，避免了多次查询数据库

- 因为Token是以JSON加密的形式保存在客户端的，所以JWT是跨语言的，原则上任何web形式都支持。

- 不需要在服务端保存会话信息，特别适用于分布式微服务。

- 基于token的认证方式相比传统的session认证方式更节约服务器资源，并且对移动端和分布式更加友好
```

## 4.JWT的结构

```java
JWTString  =  Base64(Header)
    		 .Base64(Payload)
    		 .HMACSHA256(base64UrlEncode(header) +"."+base64UrlEncode(payload),secret)
```

```markdown
token string  ====>  header.payload.singnature   token   

# 1.令牌组成
- 1.标头(Header)
- 2.有效载荷(Payload)
- 3.签名(Signature)
- 因此，JWT通常如下所示:xxxxx.yyyyy.zzzzz   Header.Payload.Signature

- 头部：主要设置一些规范信息，签名部分的编码格式就在头部中声明。
- 载荷：token中存放有效信息的部分，比如用户名，用户角色，过期时间等，但是不要放密码，会泄露！
- 签名：将头部与载荷分别采用base64编码后，用“.”相连，再加入盐，最后使用头部声明的编码类型进行编码，就得到 -了签名。
```

```markdown
# 2.Header
- 标头通常由两部分组成：令牌的类型（即JWT）和所使用的签名算法，例如HMAC SHA256或RSA。它会使用Base64 编码组成 JWT 结构的第一部分。

- 注意:Base64是一种编码，也就是说，它是可以被翻译回原来的样子来的。它并不是一种加密过程。
```

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

```markdown
# 3.Payload
- 令牌的第二部分是有效负载，其中包含声明。声明是有关实体（通常是用户）和其他数据的声明。同样的，它会使用 Base64 编码组成 JWT 结构的第二部分

- 请注意，默认情况下JWT是未加密的，因为只是采用base64算法，拿到JWT字符串后可以转换回原本的JSON数据，任何人都可以解读其内容，因此不要构建隐私信息字段，比如用户的密码一定不能保存到JWT中，以防止信息泄露。JWT只是适合在网络中传输一些非敏感的信息
```

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

```markdown
# 4.Signature
- 前面两部分都是使用 Base64 进行编码的，即前端可以解开知道里面的信息。Signature需要使用编码后的 header 和 payload 以及我们提供的一个密钥，然后使用 header 中指定的签名算法（HS256）进行签名。签名的作用是保证 JWT 没有被篡改过
- 如:
	HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload),secret);

# 签名目的
- 最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。

# 信息安全问题
- 在这里大家一定会问一个问题：Base64是一种编码，是可逆的，那么我的信息不就被暴露了吗？

- 是的。所以，在JWT中,不应该在负载里面加入任何敏感的数据。在上面的例子中，我们传输的是用户的User ID。这个值实际上不是什么敏感内容，一般情况下被知道也是安全的。但是像密码这样的内容就不能被放在JWT中了。如果将用户的密码放在了JWT中，那么怀有恶意的第三方通过Base64解码就能很快地知道你的密码了。因此JWT适合用于向Web应用传递一些非敏感信息。JWT还经常用于设计用户认证和授权系统，甚至实现Web应用的单点登录。
```

![image-20200726181136113](JWT.assets/image-20200726181136113.png)

```markdown
# 5.放在一起
- 输出是三个由点分隔的Base64-URL字符串，可以在HTML和HTTP环境中轻松传递这些字符串，与基于XML的标准（例如SAML）相比，它更紧凑。
- 简洁(Compact)
	可以通过URL, POST 参数或者在 HTTP header 发送，因为数据量小，传输速度快
- 自包含(Self-contained)
	负载中包含了所有用户所需要的信息，避免了多次查询数据库
```

![image-20200726124257203](JWT.assets/image-20200726124257203.png)

## 5.JWT的种类

​		其实JWT(JSON Web Token指的是一种规范，这种规范允许我们使用JWT在两个组织之间传递安全可靠的信息，JWT的具体实现可以分为以下几种：

### (1).nonsecure JWT

未经过签名，不安全的JWT。其`header`部分没有指定签名算法,并且也没有`Signature`部分

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

### (2).JWS 最常用 

JWS ，也就是JWT Signature，其结构就是在之前nonsecure JWT的基础上，在头部声明签名算法，并在最后添加上签名。**创建签名，是保证jwt不能被他人随意篡改**。我们通常使**用的JWT一般都是JWS**,为了完成签名，除了用到header信息和payload信息外，还需要算法的密钥，也就是`secretKey`。加密的算法一般有2类：

```markdown
- 对称加密：secretKey指加密密钥，可以生成签名与验签
- 非对称加密：secretKey指私钥，只用来生成签名，不能用来验签(验签用的是公钥)
```

JWT的密钥或者密钥对，一般统一称为JSON Web Key，也就是`JWK`

到目前为止，jwt的签名算法有三种：

```markdown
- HMAC【哈希消息验证码(对称)】：HS256/HS384/HS512
- RSASSA【RSA签名算法(非对称)】（RS256/RS384/RS512）
- ECDSA【椭圆曲线数据签名算法(非对称)】（ES256/ES384/ES512）
```

### (3).JWE

`payload`部分经过加密的JWT



## 6.使用JWT

```markdown
# 1.引入依赖
```

```xml
<!--引入jwt-->
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>java-jwt</artifactId>
  <version>3.4.0</version>
</dependency>
```

```markdown
# 2.生成token
```

```java
 Map<String,Object> map = new HashMap<>();
     //设置令牌在60秒后过期
     Calendar install = Calendar.getInstance();
     install.add(Calendar.SECOND,60);

      String token = JWT.create()
            .withHeader(map)//header 一般不设置header，使用默认配置
             //放入用户的不敏感的信息（密码等敏感信息不要放入）
             .withClaim("userId", 21)//payload
             .withClaim("userName", "zs")
             //指定令牌的过期时间
             .withExpiresAt(install.getTime())
             //放入盐进行算法加密
             .sign(Algorithm.HMAC256("token!@$%@%@@**&!"));//Signature签名
 System.out.println(token);
```

```markdown
- 生成结果
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyTmFtZSI6InpzIiwiZXhwIjoxNjQ1NzcyNTI4LCJ1c2VySWQiOjIxfQ.VWS2o3W_vsPjlsl31rgy06s4WGMiSrlfa0SyqtPoKfI
```

```markdown
# 3.根据令牌和签名解析数据
```

```java
//创建验证对象，要和加密算法和盐保持一致
JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("token!@$%@%@@**&!")).build();
DecodedJWT verify = jwtVerifier.verify(  "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyTmFtZSI6InpzIiwiZXhwIjoxNjQ1NzcyMjI5LCJ1c2VySWQiOjIxfQ.nF865C27x1jKD3DqSmb6kcL8t4pfoP8jCSf_2w4dUm8");

//取出令牌中保存的信息
System.out.println(verify.getClaim("userID").asInt());
System.out.println(verify.getClaim("userName").asString());
//取出令牌中的过期时间
System.out.println(verify.getExpiresAt());
```

````markdown
# 4.常见异常信息
- SignatureVerificationException:				签名不一致异常
- TokenExpiredException:    					令牌过期异常
- AlgorithmMismatchException:					算法不匹配异常
- InvalidClaimException:						失效的payload异常
````

![image-20200805184517282](JWT.assets/image-20200805184517282.png)

## 7.封装工具类

```java
public class JWTUtils {
    //盐
    private final static String SIGN = "@$CAJN@^@!SSS";

    private JWTUtils(){}

    // token --> header(默认).payload(用户的信息，过期时间).sigNature(加密算法和盐)
    public static String getToken(Map<String,String> map){
        
        //设置7天过期
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.DATE,7);

        //创建jwt builder
        JWTCreator.Builder builder = JWT.create();

        //payload 放入map集合中的数据
        Set<Map.Entry<String, String>> entrySet = map.entrySet();
        for (Map.Entry<String,String> m : entrySet){
            builder.withClaim(m.getKey(),m.getValue());
        }

        //sigNature+过期时间
        return builder.withExpiresAt(calendar.getTime()).sign(Algorithm.HMAC256(SIGN));
    }

    //验证token并且返回验证通过后的对象
    public static DecodedJWT verify(String token){
        //如果验证不通过，则会抛出异常
        return JWT.require(Algorithm.HMAC256(SIGN)).build().verify(token);
    }
}
```

## 8.整合springboot

```markdown
# 0.搭建springboot+mybatis+jwt环境
- 引入依赖
- 编写配置
```

```xml
<!--引入jwt-->
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>java-jwt</artifactId>
  <version>3.4.0</version>
</dependency>

<!--引入mybatis-->
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>2.1.3</version>
</dependency>

<!--引入lombok-->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.12</version>
</dependency>

<!--引入druid-->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.1.19</version>
</dependency>

<!--引入mysql-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.38</version>
</dependency>
```

```properties
server.port=8989
spring.application.name=jwt

spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/jwt?characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=root

mybatis.type-aliases-package=com.baizhi.entity
mybatis.mapper-locations=classpath:com/baizhi/mapper/*.xml

logging.level.com.baizhi.dao=debug
```

```markdown
# 1.开发数据库
- 这里采用最简单的表结构验证JWT使用
```

![image-20200805212226422](JWT.assets/image-20200805212226422.png)

```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(80) DEFAULT NULL COMMENT '用户名',
  `password` varchar(40) DEFAULT NULL COMMENT '用户密码',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

```markdown
# 2.开发entity
```

```java
@Data
@Accessors(chain=true)
public class User {
    private String id;
    private String name;
    private String password;
}
```

![image-20200805213539471](JWT.assets/image-20200805213539471.png)

```markdown
# 3.开发DAO接口和mapper.xml
```

```java
@Mapper
public interface UserDAO {
    User login(User user);
}
```

![image-20200805213603655](JWT.assets/image-20200805213603655.png)

```xml
<mapper namespace="com.baizhi.dao.UserDAO">
    <select id="login" parameterType="User" resultType="User">
        select * from user where name=#{name} and password = #{password}
    </select>
</mapper>
```

![image-20200805213628252](JWT.assets/image-20200805213628252.png)

```markdown
# 4.开发Service 接口以及实现类
```

```java
public interface UserService {
    User login(User user);//登录接口
}
```

![image-20200805213648147](JWT.assets/image-20200805213648147.png)

````java
@Service
@Transactional
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDAO userDAO;
    @Override
    @Transactional(propagation = Propagation.SUPPORTS)
    public User login(User user) {
        User userDB = userDAO.login(user);
        if(userDB!=null){
            return userDB;
        }
        throw new RuntimeException("登录失败~~");
    }
}
````

![image-20200805213704965](JWT.assets/image-20200805213704965.png)

```markdown
# 5.开发controller
```

```java
@RestController
@Slf4j
public class UserController {
    @Autowired
    private UserService userService;
    @GetMapping("/user/login")
    public Map<String,Object> login(User user) {
        Map<String,Object> result = new HashMap<>();
        log.info("用户名: [{}]", user.getName());
        log.info("密码: [{}]", user.getPassword());
        try {
            User userDB = userService.login(user);
            Map<String, String> map = new HashMap<>();//用来存放payload
            map.put("id",userDB.getId());
            map.put("username", userDB.getName());
            String token = JWTUtils.getToken(map);
            result.put("state",true);
            result.put("msg","登录成功!!!");
            result.put("token",token); //成功返回token信息
        } catch (Exception e) {
            e.printStackTrace();
            result.put("state","false");
            result.put("msg",e.getMessage());
        }
        return result;
    }
}
```

![image-20200805214235426](JWT.assets/image-20200805214235426.png)

```markdown
# 6.数据库添加测试数据启动项目
```

![image-20200805214324868](JWT.assets/image-20200805214324868.png)

![image-20200805214424786](JWT.assets/image-20200805214424786.png)

```markdown
# 7.通过postman模拟登录失败
```

![image-20200805214610668](JWT.assets/image-20200805214610668.png)

```markdown
# 8.通过postman模拟登录成功
```

![image-20200805214749469](JWT.assets/image-20200805214749469.png)

```markdown
# 9.编写测试接口
```

```java
@PostMapping("/test/test")
public Map<String, Object> test(String token) {
  Map<String, Object> map = new HashMap<>();
  try {
    JWTUtils.verify(token);
    map.put("msg", "验证通过~~~");
    map.put("state", true);
  } catch (TokenExpiredException e) {
    map.put("msg", "Token已经过期!!!");
  } catch (SignatureVerificationException e){
    map.put("msg", "签名错误!!!");
  } catch (AlgorithmMismatchException e){
    map.put("msg", "加密算法不匹配!!!");
  } catch (Exception e) {
    e.printStackTrace();
    map.put("msg", "无效token~~");
  }
  map.put("state", false);
  return map;
}
```

![image-20200805215119590](JWT.assets/image-20200805215119590.png)

```markdown
# 10.通过postman请求接口
```

![image-20200805215337303](JWT.assets/image-20200805215337303.png)

![image-20200805215451442](JWT.assets/image-20200805215451442.png)

```markdown
# 11.问题?
- 使用上述方式每次都要传递token数据,每个方法都需要验证token代码冗余,不够灵活? 如何优化
- 使用拦截器进行优化
```

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
  String token = request.getHeader("token");
  Map<String,Object> map = new HashMap<>();
  try {
    JWTUtils.verify(token);
    return true;
  } catch (TokenExpiredException e) {
    map.put("msg", "Token已经过期!!!");
  } catch (SignatureVerificationException e){
    map.put("msg", "签名错误!!!");
  } catch (AlgorithmMismatchException e){
    map.put("msg", "加密算法不匹配!!!");
  } catch (Exception e) {
    e.printStackTrace();
    map.put("msg", "无效token~~");
  }
  map.put("state", false);
  String json = new ObjectMapper().writeValueAsString(map);
  response.setContentType("application/json;charset=UTF-8");
  response.getWriter().println(json);
  return false;
}
```

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new JwtTokenInterceptor()).
          excludePathPatterns("/user/**")
          .addPathPatterns("/**");
    }
}
```

