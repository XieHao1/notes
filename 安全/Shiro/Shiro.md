# Shiro

​	Apache Shiro 是一个功能强大且易于使用的 Java 安全(权限)框架。Shiro 可以完成：认证、授权、加密、会话管理、与Web集成、缓存等。借助 Shiro 您可以快速轻松 地保护任何应用程序——从最小的移动应用程序到最大的 Web 和企业应用程序。

[官网](https://shiro.apache.org/)

[相关博客](https://www.w3cschool.cn/shiro/andc1if0.html)

[TOC]



# 1.shiro简介

## 1.1 特点

- `易于使用`：使用 Shiro 构建系统安全框架非常简单。就算第一次接触也可以快速掌握。 

- `全面`：Shiro 包含系统安全框架需要的功能，满足安全需求的“一站式服务”。 

- `灵活`：Shiro 可以在任何应用程序环境中工作。虽然它可以在Web、EJB和IoC 环境中工作，但不需要依赖它们。Shiro也没有强制要求任何规范，甚至没有很多依赖项。 强力支持 Web：Shiro 具有出色的 Web 应用程序支持，可以基于应用程序 URL 和 Web 协议（例如 REST）创建灵活的安全策略，同时还提供一组 JSP 库来控制页面输出。 

- `兼容性强`：Shiro 的设计模式使其易于与其他框架和应用程序集成。Shiro 与 Spring、Grails、Wicket、Tapestry、Mule、Apache Camel、Vaadin 等框架无缝集成。

- `社区支持`：Shiro 是 Apache 软件基金会的一个开源项目，有完备的社区支持，文档 支持。如果需要，像 Katasoft 这样的商业公司也会提供专业的支持和服务。



## 1.2 Shiro与SpringSecurity的对比

1、Spring Security 基于 Spring 开发，项目若使用 Spring 作为基础，配合 Spring Security 做权限更加方便，而 Shiro 需要和 Spring 进行整合开发； 

2、Spring Security 功能比 Shiro 更加丰富些，例如安全维护方面； 

3、Spring Security 社区资源相对比 Shiro 更加丰富； 

4、Shiro 的配置和使用比较简单，Spring Security 上手复杂些； 

5、Shiro 依赖性低，不需要任何框架和容器，可以独立运行.Spring Security 依赖 Spring 容器； 

6、shiro 不仅仅可以使用在 web 中，它可以工作在任何应用环境中。在集群会话时 Shiro 最重要的一个好处或许就是它的会话是独立于容器的。



## 1.3 基本功能

![Apache Shiro Features](img.assets\ShiroFeatures.png)

- `Authentication`：身份认证/登录，验证用户是不是拥有相应的身份；
- `Authorization`：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能进行什么操作，如：验证某个用户是否拥有某个角色。或者细粒度的验证 某个用户 对某个资源是否具有某个权限；
- `Session Manager`：会话管理，即用户登录后就是一次会话，在没有退出之前，它的 所有 信息都在会话中；会话可以是普通 JavaSE 环境，也可以是 Web 环境的；
- `Cryptography`：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

在不同的应用程序环境中，还有一些其他功能可以支持和加强这些问题，特别是：

- `Web Support`：Web 支持，可以非常容易的集成到 Web 环境；
- `Caching`：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；
- `Concurrency`：Shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；
- `Testing`：提供测试支持；
- `Run As`：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；
- `Remember Me`：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了



## 1.4 Shiro架构(外部)

![img](img.assets\2.png)

可以看到：应用代码直接交互的对象是 Subject，也就是说 Shiro 的对外 API 核心就是 Subject；其每个 API 的含义：

**Subject**：主体，代表了当前 “用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 Subject，如网络爬虫，机器人等；即一个抽象概念；所有 Subject 都绑定到 SecurityManager，与 Subject 的所有交互都会委托给 SecurityManager；可以把 Subject 认为是一个门面；SecurityManager 才是实际的执行者；

**SecurityManager**：安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与后边介绍的其他组件进行交互，如果学习过 SpringMVC，你可以把它看成 DispatcherServlet 前端控制器；

**Realm**：域，Shiro 从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源。



也就是说对于我们而言，最简单的一个 Shiro 应用：

1. 应用代码通过 Subject 来进行认证和授权，而 Subject 又委托给 SecurityManager；
2. 我们需要给 Shiro 的 SecurityManager 注入 Realm，从而让 SecurityManager 能得到合法的用户及其权限进行判断。

**从以上也可以看出，Shiro 不提供维护用户 / 权限，而是通过 Realm 让开发人员自己注入。**



## 1.5 Shiro架构(内部)

![img](img.assets\3.png)

- **Subject**：主体，可以看到主体可以是任何可以与应用交互的 “用户”；
- **SecurityManager**：相当于 SpringMVC 中的 DispatcherServlet 或者 Struts2 中的 FilterDispatcher；是 Shiro 的心脏；所有具体的交互都通过 SecurityManager 进行控制；它管理着所有 Subject、且负责进行认证和授权、及会话、缓存的管理。
- **Authenticator**：认证器，负责主体认证的，这是一个扩展点，如果用户觉得 Shiro 默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；
- **Authorizer**：授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；
- **Realm**：可以有 1 个或多个 Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是 JDBC 实现，也可以是 LDAP 实现，或者内存实现等等；由用户提供；注意：Shiro 不知道你的用户 / 权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的 Realm；
- **SessionManager**：如果写过 Servlet 就应该知道 Session 的概念，Session 呢需要有人去管理它的生命周期，这个组件就是 SessionManager；而 Shiro 并不仅仅可以用在 Web 环境，也可以用在如普通的 JavaSE 环境、EJB 等环境；所以呢，Shiro 就抽象了一个自己的 Session 来管理主体与应用之间交互的数据；这样的话，比如我们在 Web 环境用，刚开始是一台 Web 服务器；接着又上了台 EJB 服务器；这时想把两台服务器的会话数据放到一个地方，这个时候就可以实现自己的分布式会话（如把数据放到 Memcached 服务器）；
- **SessionDAO**：DAO 大家都用过，数据访问对象，用于会话的 CRUD，比如我们想把 Session 保存到数据库，那么可以实现自己的 SessionDAO，通过如 JDBC 写到数据库；比如想把 Session 放到 Memcached 中，可以实现自己的 Memcached SessionDAO；另外 SessionDAO 中可以使用 Cache 进行缓存，以提高性能；
- **CacheManager**：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能
- **Cryptography**：密码模块，Shiro 提供了一些常见的加密组件用于如密码加密 / 解密的。



# 2.基本使用



## 2.1 环境搭建

1、Shiro不依赖容器，直接创建maven工程即可  

2、添加依赖

```xml
    <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-core</artifactId>
      <version>1.10.1</version>
    </dependency>
	
	<!--日志-->
    <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.2</version>
    </dependency>
```

3.Shiro 获取权限相关信息可以通过数据库获取，也可以通过 `ini 配置文件`获取

`在resource目录下创建shiro.ini文件`

```ini
[users]
# 用户名=密码
root=root
admin=admin
```



## 2.2 认证

在 shiro 中，用户需要提供 `principals` （身份）和 `credentials`（证明）给 shiro，从而应用能验证用户身份：

**principals**：身份，即主体的标识属性，可以是任何东西，如用户名、邮箱等，唯一即可。一个主体可以有多个 `principals`，但只有一个 `Primary principals`，一般是用户名 / 密码 / 手机号。

**credentials**：证明 / 凭证，即只有主体知道的安全值，如密码 / 数字证书等。

最常见的 `principals` 和 `credentials` 组合就是用户名 / 密码了。

另外两个相关的概念是之前提到的 **`Subject`** 及 **`Realm`**，分别是主体及验证主体的数据源。



### 2.2.1 身份认证流程

![img](img.assets\4.png)

流程如下：

1. 首先调用 `Subject.login(token)` 进行登录，其会自动委托给 `Security Manager`，调用之前必须通过 `SecurityUtils.setSecurityManager()` 设置；
2. `SecurityManager` 负责真正的身份验证逻辑；它会委托给 `Authenticator` 进行身份验证；
3. `Authenticator` 才是真正的身份验证者，`Shiro API` 中核心的身份认证入口点，此处可以自定义插入自己的实现；
4. `Authenticator` 可能会委托给相应的 `AuthenticationStrategy` 进行多 `Realm` 身份验证，默认 `ModularRealmAuthenticator` 会调用 `AuthenticationStrategy` 进行多 `Realm` 身份验证；
5. 创建自定义的 Realm 类，继承 org.apache.shiro.realm.`AuthenticatingRealm`类， 实现 `doGetAuthenticationInfo()` 方法
6. `Authenticator `会把相应的 `token` 传入 `Realm`，从 `Realm` 获取身份验证信息，如果没有返回 / 抛出异常表示身份验证成功了。此处可以配置多个 `Realm`，将按照相应的顺序及策略进行访问。



### 2.2.2 认证示例

```java
public class ShiroLogin {
    public static void main(String[] args) {
        //1、获取SecurityManager工厂，此处使用Ini配置文件初始化SecurityManager
        IniSecurityManagerFactory factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        //2、得到SecurityManager实例 并绑定给SecurityUtils
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        //3、得到Subject及创建用户名/密码身份验证Token（即用户身份/凭证）
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("root", "root");
        try {
            //4.登录，即身份验证
            subject.login(token);
            System.out.println("认证成功");
        }catch (AuthenticationException e){
            System.out.println("认证失败");
        }
        //6、退出
        subject.logout();
    }
}
```

- 首先通过 `new IniSecurityManagerFactory` 并指定一个 `ini` 配置文件来创建一个 `SecurityManager` 工厂；
- 接着获取 `SecurityManager` 并绑定到 `SecurityUtils`，这是一个全局设置，设置一次即可；
- 通过 `SecurityUtils` 得到 `Subject`，其会自动绑定到当前线程；如果在 web 环境在请求结束时需要解除绑定；然后获取身份验证的 `Token`，如用户名 / 密码；
- 调用 `subject.login` 方法进行登录，其会自动委托给 `SecurityManager.login` 方法进行登录；
- 如果身份验证失败请捕获 `AuthenticationException` 或其子类，常见的如： `DisabledAccountException`（禁用的帐号）、`LockedAccountException`（锁定的帐号）、`UnknownAccountException`（错误的帐号）、`ExcessiveAttemptsException`（登录失败次数过多）、`IncorrectCredentialsException` （错误的凭证）、`ExpiredCredentialsException`（过期的凭证）等，具体请查看其继承关系；对于页面的错误消息展示，最好使用如 “用户名 / 密码错误” 而不是 “用户名错误”/“密码错误”，防止一些恶意用户非法扫描帐号库；
- 最后可以调用 `subject.logout` 退出，其会自动委托给 `SecurityManager.logout` 方法退出。

**从如上代码可总结出身份验证的步骤**：

1. 收集用户身份 / 凭证，即如用户名 / 密码；
2. 调用 `Subject.login` 进行登录，如果失败将得到相应的 `AuthenticationException` 异常，根据异常提示用户错误信息；否则登录成功；
3. 最后调用 `Subject.logout` 进行退出操作。

如上测试的几个问题：

1. 用户名 / 密码硬编码在 `ini` 配置文件，以后需要改成如数据库存储，且密码需要加密存储；
2. 用户身份 `Token` 可能不仅仅是用户名 / 密码，也可能还有其他的，如登录时允许用户名 / 邮箱 / 手机号同时登录。



## 2.3 授权

授权，也叫访问控制，即在应用中控制谁能访问哪些资源（如访问页面/编辑数据/页面操作等）。在授权中需了解的几个关键对象：主体（Subject）、资源（Resource）、权限（Permission）、角色（Role）。

`主体`:主体，即访问应用的用户，在 Shiro 中使用 `Subject` 代表该用户。用户只有授权后才允许访问相应的资源。

`资源`: 在应用中用户`可以访问的URL`，比如访问 JSP 页面、查看/编辑某些数据、访问某个业务方法、打印文本等等都是资源。用户只要授权后才能访问。

`权限`: 安全策略中的原子授权单位，通过权限我们可以表示在应用中用户有没有操作某个资源的权力。即`权限表示在应用中用户能不能访问某个资源`，如： 访问用户列表页面 查看/新增/修改/删除用户数据（即很多时候都是 CRUD（增查改删）式权限控制）打印文档等。如上可以看出，权限代表了用户有没有操作某个资源的权利，即反映在某个资源上的操作允不允许，不反映谁去执行这个操作。所以后续还需要把权限赋予给用户，即定义哪个用户允许在某个资源上做什么操作（权限），Shiro 不会去做这件事情，而是由实现人员提供。

Shiro 支持粗粒度权限（如用户模块的所有权限）和细粒度权限（操作某个用户的权限，即实例级别的），后续部分介绍。

`角色`: 角色代表了操作集合，可以理解为`权限的集合`，一般情况下我们会赋予用户角色而不是权限，即这样用户可以拥有一组权限，赋予权限时比较方便。典型的如：项目经理、技术总监、CTO、开发工程师等都是角色，不同的角色拥有一组不同的权限。

隐式角色： 即直接通过角色来验证用户有没有操作权限，如在应用中 CTO、技术总监、开发工程师可以使用打印机，假设某天不允许开发工程师使用打印机，此时需要从应用中删除相应代码；再如在应用中 CTO、技术总监可以查看用户、查看权限；突然有一天不允许技术总监查看用户、查看权限了，需要在相关代码中把技术总监角色从判断逻辑中删除掉；即粒度是以角色为单位进行访问控制的，粒度较粗；如果进行修改可能造成多处代码修改。

显示角色： 在程序中通过权限控制谁能访问某个资源，角色聚合一组权限集合；这样假设哪个角色不能访问某个资源，只需要从角色代表的权限集合中移除即可；无须修改多处代码；即粒度是以资源/实例为单位的；粒度较细。

### 2.3.1 授权方式

Shiro 支持三种方式的授权：

编程式：通过写 if/else 授权代码块完成：

```java
Subject subject = SecurityUtils.getSubject();
if(subject.hasRole(“admin”)) {
    //有权限
} else {
    //无权限
}
```

注解式：通过在执行的 Java 方法上放置相应的注解完成：

```java
@RequiresRoles("admin")
public void hello() {
    //有权限
}
```

没有权限将抛出相应的异常；

JSP/GSP 标签：在 JSP/GSP 页面通过相应的标签完成：

```jsp
<shiro:hasRole name="admin">
<!— 有权限 —>
</shiro:hasRole>
```



### 2.3.2 授权流程

![img](img.assets\202101141719562904.png)

流程如下：

1. 首先调用 Subject.isPermitted*/hasRole*接口，其会委托给 SecurityManager，而 SecurityManager 接着会委托给 Authorizer；
2. Authorizer 是真正的授权者，如果我们调用如 isPermitted(“user:view”)，其首先会通过 PermissionResolver 把字符串转换成相应的 Permission 实例；
3. 在进行授权之前，其会调用相应的 Realm 获取 Subject 相应的角色/权限用于匹配传入的角色/权限；
4. Authorizer 会判断 Realm 的角色/权限是否和传入的匹配，如果有多个 Realm，会委托给 ModularRealmAuthorizer 进行循环判断，如果匹配如 isPermitted*/hasRole* 会返回 true，否则返回 false 表示授权失败。

ModularRealmAuthorizer 进行多 Realm 匹配流程：

- 首先检查相应的 Realm 是否实现了实现了 Authorizer；
- 如果实现了 Authorizer，那么接着调用其相应的 isPermitted*/hasRole* 接口进行匹配；
- 如果有一个 Realm 匹配那么将返回 true，否则返回 false。

如果 Realm 进行授权的话，应该继承 AuthorizingRealm，其流程是：

- 如果调用 hasRole*，则直接获取 AuthorizationInfo.getRoles() 与传入的角色比较即可；首先如果调用如 isPermitted(“user:view”)，首先通过 PermissionResolver 将权限字符串转换成相应的 Permission 实例，默认使用 WildcardPermissionResolver，即转换为通配符的 WildcardPermission；
- 通过 AuthorizationInfo.getObjectPermissions() 得到 Permission 实例集合；通过 AuthorizationInfo.getStringPermissions() 得到字符串集合并通过 PermissionResolver 解析为 Permission 实例；然后获取用户的角色，并通过 RolePermissionResolver 解析角色对应的权限集合（默认没有实现，可以自己提供）；
- 接着调用 Permission.implies(Permission p) 逐个与传入的权限比较，如果有匹配的则返回 true，否则 false。



### 2.3.3 添加角色

#### 2.3.3.1 隐式角色 

在 ini 配置文件配置用户拥有的角色（shiro.ini）

```ini
[users]
root=root,role1,role2
admin=admin,role1
```

规则即：“`用户名=密码,角色1，角色2`”，如果需要在应用中判断用户是否有相应角色，就需要在相应的 Realm 中返回角色信息，也就是说 Shiro 不负责维护用户-角色信息，需要应用提供，Shiro 只是提供相应的接口方便验证。

Shiro 提供了 `hasRole/hasRoles` 用于判断用户是否拥有某个角色/某些权限；但是没有提供如 hashAnyRole 用于判断是否有某些权限中的某一个。

```java
//判断用户是否拥有某个角色
boolean role = subject.hasRole("role1");
```



#### 2.3.3.2 显示角色

在 ini 配置文件配置用户拥有的角色及角色-权限关系（shiro.ini）

```ini
[users]
root=root,role1,role2
admin=admin,role1
[roles]
role1=user:create,user:update
role2=user:create,user:delete
```

规则：“`用户名=密码，角色 1，角色 2`”“`角色=权限 1，权限 2`”，即首先根据`用户名找到角色，然后根据角色再找到权限`；即角色是权限集合；Shiro 同样不进行权限的维护，需要我们通过 Realm 返回相应的权限信息。只需要维护“用户——角色”之间的关系即可。



Shiro 提供了 `isPermitted 和 isPermittedAll` 用于判断用户是否拥有某个权限或所有权限，也没有提供如 isPermittedAny 用于判断拥有某一个权限的接口。

```java
subject.isPermitted("user:create");
```

也可以用 `checkPermission` 方法，但没有返回值，没权限抛 AuthenticationException

```java
void checkPermission(String permission) throws AuthorizationException
```



### 2.3.4 Permission

**`字符串通配符权限`**

规则：“资源标识符：操作：对象实例 ID” 即对哪个资源的哪个实例可以进行什么操作。

其默认支持通配符权限字符串，“:”表示资源/操作/实例的分割；“,”表示操作的分割；“*”表示任意资源/操作/实例。

```ini
role41=system:user:update,system:user:delete
```

用户拥有资源“system:user”的“update”和“delete”权限。如上可以简写成：

```ini
role42=“system:user:update,delete"
```

接着可以通过如下代码判断

```java
checkPermissions("system:user:update,delete");
```

通过“system:user:update,delete”验证“system:user:update, system:user:delete”是没问题的，但是反过来是规则不成立。



ini 配置文件（表示角色 5 拥有 system:user 的`所有权限`）

```ini
role52=system:user:*
```

也可以简写为（推荐上边的写法）：

```ini
role53=system:user
```

然后通过如下代码判断

```java
checkPermissions("system:user:*");
checkPermissions("system:user");
```



**`所有资源单个权限`**

ini 配置

```ini
role61=*:view
```

然后通过如下代码判断

```ini
checkPermissions("user:view");
```

用户拥有所有资源的“view”所有权限。假设判断的权限是“"system:user:view”，那么需要“role5=::view”这样写才行



## 2.4 加密

```java
public class ShiroMD5 {
    public static void main(String[] args) {
        String pwd = "root";
        //使用 md5 加密
        Md5Hash md5Hash = new Md5Hash(pwd);
        //还可以转换为 toBase64()/toHex()
        System.out.println(md5Hash);
        //带盐的 md5 加密，盐就是在密码明文后拼接新字符串，然后再进行加密
        Md5Hash md5Hash1 = new Md5Hash(pwd,"salt");
        System.out.println(md5Hash1);
        //为了保证安全，避免被破解还可以多次迭代加密，保证数据安全
        Md5Hash md5Hash2 = new Md5Hash(pwd,"salt",3);
        System.out.println(md5Hash2);
    }
}
```

使用 SHA256 算法生成相应的散列数据，另外还有如 SHA1、SHA512 算法。

Shiro 还提供了通用的散列支持：

```java
String str = "hello";
String salt = "123";
//内部使用MessageDigest
String simpleHash = new SimpleHash("SHA-1", str, salt).toString();  
```



## 2.5 Shiro 自定义登录认证

Shiro 默认的登录认证是不带加密的，如果想要实现加密认证需要自定义登录认证，自定义Realm。

创建自定义的 Realm 类，继承 org.apache.shiro.realm.`AuthenticatingRealm`类， 实现 `doGetAuthenticationInfo()` 方法

```java
public class MyRealm extends AuthenticatingRealm {
    //自定义的登录认证方法，Shiro 的 login 方法底层会调用该类的认证方法完成登录认证
    //需要配置自定义的 realm 生效，在 ini 文件中配置，或 Springboot 中配置
    //该方法只是获取进行对比的信息，认证逻辑还是按照 Shiro 的底层认证逻辑完成认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取身份信息
        String principal = token.getPrincipal().toString();
        //获取用户凭证信息
        String credentials = token.getCredentials().toString();
        System.out.println("用户身份信息:"+principal+",用户凭证信息:"+credentials);

        //从数据库中获取用户密码
        if("root".equals(principal)){
            //数据库中加密三次的密码
            String password = "40f728dee7512396eb1a1b2539228b0c";
            //封装校验逻辑的对象，将要比较的数据给该对象
            return new SimpleAuthenticationInfo(
                    principal,password, ByteSource.Util.bytes("salt"),credentials);
        }
        return null;
    }
}
```

`在ini`文件中注入自定义的认证数据源

```ini
[main]
md5CredentialsMatcher=org.apache.shiro.authc.credential.Md5CredentialsMatcher
md5CredentialsMatcher.hashIterations=3
myrealm=com.xh.shiro.MyRealm
myrealm.credentialsMatcher=$md5CredentialsMatcher
securityManager.realms=$myrealm

[users]
root=40f728dee7512396eb1a1b2539228b0c,role1,role2,role3
```



## 2.6 ini文件配置

Shiro 是从根对象 SecurityManager 进行身份验证和授权的；也就是所有操作都是自它开始的，这个对象是线程安全且整个应用只需要一个即可，因此 Shiro 提供了 SecurityUtils 让我们绑定它为全局的，方便后续操作。

因为 Shiro 的类都是 POJO 的，因此都很容易放到任何 IoC 容器管理。但是和一般的 IoC 容器的区别在于，Shiro 从根对象 securityManager 开始导航；Shiro 支持的依赖注入：public 空参构造器对象的创建、setter 依赖注入。

ini 配置文件类似于 Java 中的 properties（key=value），不过提供了将 key/value 分类的特性，key 是每个部分不重复即可，而不是整个配置文件。如下是 INI 配置分类：

```ini
[main]
\#提供了对根对象securityManager及其依赖的配置
securityManager=org.apache.shiro.mgt.DefaultSecurityManager
…………
securityManager.realms=$jdbcRealm
[users]
\#提供了对用户/密码及其角色的配置，用户名=密码，角色1，角色2
username=password,role1,role2
[roles]
\#提供了角色及权限之间关系的配置，角色=权限1，权限2
role1=permission1,permission2
[urls]
\#用于web，提供了对web url拦截相关的配置，url=拦截器[参数]，拦截器
/index.html = anon
/admin/** = authc, roles[admin], perms["permission1"]
```

**[main] 部分**

提供了对根对象 securityManager 及其依赖对象的配置。

**[users] 部分**

配置用户名 / 密码及其角色，格式：“用户名 = 密码，角色 1，角色 2”，角色部分可省略。如：

```ini
[users]
zhang=123,role1,role2
wang=123
```

**[roles] 部分**

配置角色及权限之间的关系，格式：“角色 = 权限 1，权限 2”；如：

```ini
[roles]
role1=user:create,user:update
role2=*
```

如果只有角色没有对应的权限，可以不配 roles

**[urls] 部分**

配置 url 及相应的拦截器之间的关系，格式：“url = 拦截器 [参数]，拦截器 [参数]，如：

```ini
[urls]
/admin/** = authc, roles[admin], perms["permission1"]
```



# 3.springBoot集成

```xml
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring-boot-web-starter</artifactId>
            <version>1.11.0</version>
        </dependency>
```



## 3.1 自定义Realm

继承`AuthorizingRealm`，重写`doGetAuthenticationInfo`认证和`doGetAuthorizationInfo`授权方法

```java
public abstract class AuthorizingRealm extends AuthenticatingRealm
```

`AuthorizingRealm`类是`AuthenticatingRealm`的扩展，增加了授权方法

```java
@Component
public class MyRealm extends AuthorizingRealm {

    private final UserService userService;

    @Autowired
    public MyRealm(UserService userService) {
        this.userService = userService;
    }

    //自定义授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        return null;
    }

    //自定义认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取用户身份
        String principal = token.getPrincipal().toString();
        //获取用户凭证
        String credentials = token.getCredentials().toString();
        //查询数据库获取用户密码
        String password = userService.getPasswordByUsername(principal);
        //封装认证对象
        return new SimpleAuthenticationInfo(
                principal,password, ByteSource.Util.bytes("salt"),credentials
        );
    }
}
```

## 3.2 定义shiro配置类

配置`SecurityManager`

```java
@Configuration
public class ShiroConfig {

    private final MyRealm myRealm;

    @Autowired
    public ShiroConfig(MyRealm myRealm) {
        this.myRealm = myRealm;
    }

    //配置 SecurityManager
    @Bean
    public DefaultWebSecurityManager defaultWebSecurityManager() {
        //1 创建 defaultWebSecurityManager 对象
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        //2 创建加密对象，并设置相关属性
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
        //2.1 采用 md5 加密
        matcher.setHashAlgorithmName("md5");
        //2.2 迭代加密次数
        matcher.setHashIterations(3);
        //3 将加密对象存储到 myRealm 中
        myRealm.setCredentialsMatcher(matcher);
        //4 将 myRealm 存入 defaultWebSecurityManager 对象
        defaultWebSecurityManager.setRealm(myRealm);
        //5 返回
        return defaultWebSecurityManager;
    }

    //配置 Shiro 内置过滤器拦截范围
    @Bean
    public DefaultShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition definition = new DefaultShiroFilterChainDefinition();
        //设置不认证可以访问的资源
        definition.addPathDefinition("/myController/**", "anon");
        definition.addPathDefinition("/login", "anon");
        //设置需要进行登录认证的拦截范围
        definition.addPathDefinition("/**", "authc");
        return definition;
    }
}
```



### 3.2.1 shiro过滤器属性

#### 3.2.1.1 身份验证相关

| 配置缩写   | 对应的过滤器                  | 功能                                                         |
| ---------- | ----------------------------- | ------------------------------------------------------------ |
| anon       | AnonymousFilter               | 指定url可以匿名访问                                          |
| authc      | FormAuthenticationFilter      | 基于表单的拦截器；如“/**=authc”，如果没有登录会跳到相应的登录页面登录；主要属性：usernameParam：表单提交的用户名参数名（ username）； passwordParam：表单提交的密码参数名（password）； rememberMeParam：表单提交的密码参数名（rememberMe）； loginUrl：登录页面地址（/login.jsp）；successUrl：登录成功后的默认重定向地址； failureKeyAttribute：登录失败后错误信息存储key（shiroLoginFailure） |
| authcBasic | BasicHttpAuthenticationFilter | Basic HTTP身份验证拦截器，主要属性： applicationName：弹出登录框显示的信息（application） |
| logout     | authc.LogoutFilter            | 退出拦截器，主要属性：redirectUrl：退出成功后重定向的地址（/） |
| user       | UserFilter                    | 用户拦截器，用户已经身份验证/记住我登录的都可                |

#### 3.2.1.2 授权相关

| 配置缩写 | 对应的过滤器                   | 功能                                                         |
| -------- | ------------------------------ | ------------------------------------------------------------ |
| roles    | RolesAuthorizationFilter       | 角色授权拦截器，验证用户是否拥有所有角色；主要属性： loginUrl：登录页面地址（/login.jsp）；unauthorizedUrl：未授权后重定向的地址；示例“/admin/**=roles[admin]” |
| perms    | PermissionsAuthorizationFilter | 权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例“/user/**=perms[“user:create”]” |
| port     | PortFilter                     | 端口拦截器，主要属性：port（80）：可以通过的端口；示例“/test= port[80]”，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样 |
| rest     | HttpMethodPermissionFilter     | rest风格拦截器，自动根据请求方法构建权限字符串（GET=read, POST=create,PUT=update,DELETE=delete,HEAD=read,TRACE=read,OPTIONS=read, MKCOL=create）构建权限字符串；示例“/users=rest[user]”，会自动拼出“user:read,user:create,user:update,user:delete”权限字符串进行权限匹配（所有都得匹配，isPermittedAll） |
| ssl      | SslFilter                      | SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口（443）；其他和port拦截器一样 |

| noSessionCreation | NoSessionCreationAuthorizationFilter | 需要指定权限才能访问 |
| ----------------- | ------------------------------------ | -------------------- |



## 3.3 多个 realm 的认证策略设置

### 3.3.1 多个realm实现原理

当应用程序配置多个 `Realm` 时，例如：用户名密码校验、手机号验证码校验等等。 Shiro 的 `ModularRealmAuthenticator` 会使用内部的 `AuthenticationStrategy` 组件判断认 证是成功还是失败。 

AuthenticationStrategy 是一个无状态的组件，它在身份验证尝试中被询问 4 次（这 4 次交互所需的任何必要的状态将被作为方法参数):

1）在所有 Realm 被调用之前 

2）在调用 Realm 的 getAuthenticationInfo 方法之前 

3）在调用 Realm 的 getAuthenticationInfo 方法之后 

4）在所有 Realm 被调用之后 



### 3.3.2 三种认证策略

认证策略的另外一项工作就是聚合所有 Realm 的结果信息封装至一个 AuthenticationInfo 实例中，并将此信息返回，以此作为 Subject 的身份信息。 

Shiro 中定义了 3 种认证策略的实现：

| AuthenticationStrategy class | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| AtLeastOneSuccessfulStrategy | 只要有一个（或更多）的 Realm 验证成功，那么认证将视为成功    |
| FirstSuccessfulStrategy      | 第一个 Realm 验证成功，整体认证将视为成功，且后续 Realm 将被忽略 |
| AllSuccessfulStrategy        | 所有 Realm 成功，认证才视为成功                              |

ModularRealmAuthenticator 内置的认证策略默认实现是 `AtLeastOneSuccessfulStrategy` 方式。可以通过配置修改策略



### 3.3.3 多个realm的实现

在`SecurityManager`的配置类中进行配置

```java
//配置 SecurityManager
@Bean
public DefaultWebSecurityManager defaultWebSecurityManager(){
	//1 创建 defaultWebSecurityManager 对象
	DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();

	//2 创建认证对象，并设置认证策略
	ModularRealmAuthenticator modularRealmAuthenticator = new ModularRealmAuthenticator();
	modularRealmAuthenticator.setAuthenticationStrategy(new AllSuccessfulStrategy());
	defaultWebSecurityManager.setAuthenticator(modularRealmAuthenticator);

	//3 封装 myRealm 集合
	List<Realm> list = new ArrayList<>();
	list.add(myRealm);
	list.add(myRealm2);

	//4 将 myRealm 存入 defaultWebSecurityManager 对象
 	defaultWebSecurityManager.setRealms(list);
 	//5 返回
 	return defaultWebSecurityManager;
}
```



## 3.4 remember me

Shiro 提供了记住我（RememberMe）的功能，比如访问一些网站时，关闭了浏览器， 下次再打开时还是能记住你是谁， 下次访问时无需再登录即可访问。

**基本流程如下：**

1. 首先在登录页面选中 RememberMe 然后登录成功；如果是浏览器登录，一般会把 RememberMe 的 `Cookie` 写到客户端并保存下来；
2. 关闭浏览器再重新打开；会发现浏览器还是记住你的；
3. 访问一般的网页服务器端还是知道你是谁，且能正常访问；
4. 但是比如我们访问淘宝时，如果要查看我的订单或进行支付时，此时还是需要再进行身份认证的，以确保当前用户还是你。



代码实现:

1.SecurityManager配置类

```java
@Configuration
public class ShiroConfig {
 	@Autowired
 	private MyRealm myRealm;
 	//配置 SecurityManager
 	@Bean
 	public DefaultWebSecurityManager defaultWebSecurityManager(){
 	//1 创建 defaultWebSecurityManager 对象
 	DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
 	//2 将 myRealm 存入 defaultWebSecurityManager 对象
 	defaultWebSecurityManager.setRealm(myRealm);
 	//3 设置 rememberMe
 	defaultWebSecurityManager.setRememberMeManager(rememberMeManager());
 	//4.返回
 	return defaultWebSecurityManager;
 }
    
 //cookie 属性设置
 public SimpleCookie rememberMeCookie(){
 	SimpleCookie cookie = new SimpleCookie("rememberMe");
 	//设置跨域
 	//cookie.setDomain(domain);
 	cookie.setPath("/");
 	cookie.setHttpOnly(true);
	cookie.setMaxAge(30*24*60*60);
 	return cookie;
 }
    
 //创建 Shiro 的 cookie 管理对象
 public CookieRememberMeManager rememberMeManager(){
 	CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
 	cookieRememberMeManager.setCookie(rememberMeCookie());
 	cookieRememberMeManager.setCipherKey("1234567890987654".getBytes());
 	return cookieRememberMeManager;
 }
    
 //配置 Shiro 内置过滤器拦截范围
 @Bean
 public DefaultShiroFilterChainDefinition shiroFilterChainDefinition(){
 	DefaultShiroFilterChainDefinition definition = new 
	DefaultShiroFilterChainDefinition();
 	//设置不认证可以访问的资源
	definition.addPathDefinition("/myController/userLogin","anon");
 	//设置需要进行登录认证的拦截范围
 	definition.addPathDefinition("/**","authc");
 	//添加存在用户的过滤器（rememberMe）
 	definition.addPathDefinition("/**","user");
 	return definition;
}
```



## 3.5 用户登出

用户登录后，配套的有登出操作。直接通过Shiro过滤器即可实现登出

```java
//修改配置类，添加 logout 过滤器-->配置 Shiro 内置过滤器拦截范围
@Bean
public DefaultShiroFilterChainDefinition shiroFilterChainDefinition(){
	DefaultShiroFilterChainDefinition definition = new DefaultShiroFilterChainDefinition();
 	//设置不认证可以访问的资源
 	definition.addPathDefinition("/myController/userLogin","anon");
 	//配置登出过滤器
 	definition.addPathDefinition("/logout","logout");
 	//设置需要进行登录认证的拦截范围
	 definition.addPathDefinition("/**","authc");
 	//添加存在用户的过滤器（rememberMe）
 	definition.addPathDefinition("/**","user");
 	return definition;
}
```



## 3.6 授权、角色认证

### 3.6.1 授权

用户登录后，需要验证是否具有指定角色指定权限。Shiro也提供了方便的工具进行判断。 

这个工具就是Realm的`doGetAuthorizationInfo`方法进行判断。触发权限判断的有两种方式 

（1） 在页面中通过shiro:****属性判断 

（2） 在接口服务中通过注解@Requires****进行判断

### 3.6.2 后端接口服务注解  

通过给接口服务方法添加注解可以实现权限校验，可以加在控制器方法上，也可以加在业务方法上，一般加在控制器(`Controller`)方法上。

常用注解如下： 

（1）`@RequiresAuthentication`  验证用户是否登录，等同于方法subject.isAuthenticated() 

（2）`@RequiresUser`  验证用户是否被记忆： 

​					登录认证成功subject.isAuthenticated()为true

​					登录后被记忆subject.isRemembered()为true 

（3）`@RequiresGuest`  验证是否是一个guest的请求，是否是游客的请求

​					  此时subject.getPrincipal()为null 

（4）`@RequiresRoles`  验证subject是否有相应角色，有角色访问方法，没有则会抛出异常 AuthorizationException。 

```java
@RequiresRoles("aRoleName") 
void someMethod(); //只有subject有aRoleName角色才能访问方法someMethod() 
```

（5）`@RequiresPermissions` 验证subject是否有相应权限，有权限访问方法，没有则会抛出异常 AuthorizationException。 

```java
@RequiresPermissions ("file:read","wite:aFile.txt") 
void someMethod(); //subject必须同时含有file:read和wite:aFile.txt权限才能访问方法someMethod()
```



### 3.6.3 角色认证

`认证之后，才授权`

```java
public @interface RequiresRoles {
    String[] value();
    //指定了多个角色时权限检查的逻辑操作。AND 是默认值
    Logical logical() default Logical.AND; 
}
```

编写controller

```java
    @GetMapping("/userRoles")
    @RequiresRoles(value = "admin",logical = Logical.AND)
    //只有角色是admin的才可以访问该接口
    public String userRoles(){
        return "验证角色成功";
    }
```

自定义角色认证

```java
@Component
@Slf4j
public class MyRealm extends AuthorizingRealm {

    //自定义授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        log.info("进入授权方法");
        //获取当前用户身份信息
	    String principal = principals.getPrimaryPrincipal().toString();
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //在数据库中查询用户对应的角色信息，添加角色
        simpleAuthorizationInfo.addRoles(Arrays.asList("admin","root"));
        return simpleAuthorizationInfo;
    }
 }
```



### 3.6.4 获取权限进行验证

Controller

```java
    @GetMapping("/userPermissions")
    @RequiresPermissions(value = {"user:update","user:insert"} ,logical = Logical.OR)
    public String userPermissions(){
        return "验证权限成功";
    }
```

自定义权限认证

```java
    //自定义授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        log.info("进入授权方法");
        //获取用户信息
        String userName = principals.getPrimaryPrincipal().toString();
        //通过用户名查询用户角色
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //在数据库中查询用户对应的角色信息，添加角色
        simpleAuthorizationInfo.addRoles(Arrays.asList("admin","root"));
        //在数据库中查询用户对应的角色信息，添加权限
        simpleAuthorizationInfo.addStringPermission("user:insert");
        return simpleAuthorizationInfo;
    }
```



### 3.6.5 授权异常

#### 3.6.5.1 AuthenticationException

异常是Shiro在登录认证过程中，认证失败需要抛出的异常。 AuthenticationException包含以下子类：



- CredentitalsException       凭证异常
  - IncorrectCredentialsException       不正确的凭证	
  - ExpiredCredentialsException         凭证过期



- AccountException    账号异常
  - ConcurrentAccessException   	并发访问异常（多个用户同时登录时抛出）
  - UnknownAccountException         未知的账号
  - ExcessiveAttemptsException      认证次数超过限制
  - DisabledAccountException        禁用的账号
  - LockedAccountException          账号被锁定
  - pportedTokenException          使用了不支持的Token



#### 3.6.5.2 AuthorizationException

- UnauthorizedException    抛出以指示请求的操作或对请求的资源的访问是不允许的。

- UnanthenticatedException  当尚未完成成功认证时，尝试执行授权操作时引发异常。

## 3.7 实现缓存



### 3.7.1 缓存工具EhCache 

EhCache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存,Java EE和轻量级容器。可以和大部分Java项目无缝整合，例如：Hibernate中的缓存就是基于EhCache实现的。 

EhCache支持内存和磁盘存储，默认存储在内存中，如内存不够时把缓存数据同步到磁 盘中。EhCache支持基于Filter的Cache实现，也支持Gzip压缩算法。

EhCache直接在JVM虚拟机中缓存，速度快，效率高； 

EhCache缺点是缓存共享麻烦，集群分布式应用使用不方便;

```xml
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.9.2</version>
</dependency>
```

在`resource`文件夹下新建`ehcache.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
    <!--磁盘的缓存位置-->
    
    <!--
        缓存对象存放路径
        java.io.tmpdir：默认的临时文件存放路径。
        user.home：用户的主目录。
        user.dir：用户的当前工作目录，即当前程序所对应的工作路径。
        其它通过命令行指定的系统属性，如“java –DdiskStore.path=D:\\abc ……”。
	-->
    <diskStore path="java.io.tmpdir/ehcache"/>
    <!--默认缓存-->
    <defaultCache
            maxEntriesLocalHeap="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxEntriesLocalDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
    <!--helloWorld 缓存-->
    <cache name="HelloWorldCache"
           maxElementsInMemory="1000"
           eternal="false"
           timeToIdleSeconds="5"
           timeToLiveSeconds="5"
           overflowToDisk="false"
           memoryStoreEvictionPolicy="LRU"/>
    <!--
    defaultCache：默认缓存策略，当 ehcache 找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
    -->
    <!--
    name:缓存名称。
    maxElementsInMemory:缓存最大数目
    maxElementsOnDisk：硬盘最大缓存个数。
    eternal:对象是否永久有效，一但设置了，timeout 将不起作用。
    overflowToDisk:是否保存到磁盘，当系统宕机时
    timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当
    eternal=false 对象不是永久有效时使用，可选属性，默认值是 0，也就是可闲置时间无穷大。
    timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当 eternal=false 对象不是永久有效时使用，默认是 0.也就是对象存活时间无穷大。
    diskPersistent：是否缓存虚拟机重启期数据
    diskSpoolBufferSizeMB：这个参数设置 DiskStore（磁盘缓存）的缓存区大小。默认是 30MB。每个 Cache 都应该有自己的一个缓冲区。
    diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120 秒。
    memoryStoreEvictionPolicy：当达到 maxElementsInMemory 限制时，Ehcache 将会根据指定的策略去清理内存。默认策略是 LRU（最近最少使用）。你可以设置为 FIFO（先进先出）或是 LFU（较少使用）。
    clearOnFlush：内存数量最大时是否清除。
    memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
    	FIFO,first in first out,这个是大家最熟的，先进先出。
    	LFU, Less Frequently Used,就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个 hit 属性，hit 值最小的将会被清出缓存。
    	LRU, Least Recently Used, 最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳当前时间最远的元素将被清出缓存。
    -->
</ehcache>
```

==demo:==

```java
public class EhCacheTest {
    public static void main(String[] args) {
        //获取编译目录下的资源流对象
        InputStream resource = EhCacheTest.class.getClassLoader().getResourceAsStream("ehcache.xml");
        //获取EhCache的缓存管理对象
        CacheManager cacheManager = new CacheManager(resource);
        //获取缓存对象，和XML文件中的缓存对象保持一致
        Cache cache = cacheManager.getCache("HelloWorldCache");
        //创建缓存数据
        Element element = new Element("name","张三");
        //存入缓存
        cache.put(element);
        //从缓存中取出
        Element name = cache.get("name");
        System.out.println(name.getObjectValue());
    }
}
```



### 3.7.2 Shiro整合EhCache

```xml
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>1.7.1</version>
        </dependency>
        
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.11.0</version>
        </dependency>
```

在`resource`文件夹下新建`ehcache-shiro.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache name="ehcache" updateCheck="false">
    <!--磁盘的缓存位置-->
    <diskStore path="java.io.tmpdir"/>
    <!--默认缓存-->
    <defaultCache
            maxEntriesLocalHeap="1000"
            eternal="false"
            timeToIdleSeconds="3600"
            timeToLiveSeconds="3600"
            overflowToDisk="false">
    </defaultCache>
    <!--登录认证信息缓存：缓存用户角色权限-->
    <cache name="loginRolePsCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="600"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true"/>
</ehcache>
```

修改配置类SecurityManager,`启动缓存`

```java
//配置 SecurityManager
@Bean
public DefaultWebSecurityManager defaultWebSecurityManager() {
     // 创建 defaultWebSecurityManager 对象
     DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        
      //设置缓存管理
      defaultWebSecurityManager.setCacheManager(getEhCacheManager());
      
      return defaultWebSecurityManager;
 }
 
 
//缓存管理器
public EhCacheManager getEhCacheManager(){
       EhCacheManager ehCacheManager = new EhCacheManager();
       InputStream is = null;
       try {
            s = ResourceUtils.getInputStreamForPath("classpath:ehcache-shiro.xml");
        } catch (IOException e) {
            e.printStackTrace();
        }
        CacheManager cacheManager = new CacheManager(is);
        ehCacheManager.setCacheManager(cacheManager);
        return ehCacheManager;
}
```



## 3.8 会话管理



### 3.8.1 SessionManager

会话管理器，负责创建和管理用户的会话（Session）生命周期，它能够在任何环境中 在本地管理用户会话，即使没有Web/Servlet/EJB容器，也一样可以保存会话。默认情况下，Shiro会检测当前环境中现有的会话机制（比如Servlet容器）进行适配，如果没有（比如独立应用程序或者非Web环境），它将会使用内置的企业会话管理器来提供相应的会话管理服务，其中还涉及一个名为`SessionDAO`的对象。SessionDAO负责Session的持久化操作（CRUD），允许Session数据写入到后端持久化数据库。



### 3.8.2 会话管理实现

SessionManager由SecurityManager管理。Shiro提供了三种实现

![img](img.assets\15.png)

Shiro 提供了三个默认实现：

**DefaultSessionManager**：DefaultSecurityManager 使用的默认实现，用于 JavaSE 环境；

**ServletContainerSessionManager**：DefaultWebSecurityManager 使用的默认实现，用于 Web 环境，其直接使用 Servlet 容器的会话；

**DefaultWebSessionManager**：用于 Web 环境的实现，可以替代 ServletContainerSessionManager，自己维护着会话，直接废弃了 Servlet 容器的会话管理。



### 3.8.3 获得session方式 

（1）实现 Session 

```java
session = SecurityUtils.getSubject().getSession(); 
session.setAttribute(“key”,”value”) 
```

（2）说明 

- Controller 中的 request，在 shiro 过滤器中的 doFilerInternal 方法，被包装成 ShiroHttpServletRequest。 

- SecurityManager 和 SessionManager 会话管理器决定 session 来源于 ServletRequest 还是由 Shiro 管理的会话。 

- 无论是通过 request.getSession 或 subject.getSession 获取到 session，操作 session，两者都是等价的
