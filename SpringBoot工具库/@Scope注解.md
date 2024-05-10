# @Scope注解

Spring中的Bean默认是`单例模式`，在代码各处注入的一个Bean都是同一个实例；我们将在Spring IoC 创建的Bean对象的请求可见范围称为`作用域`。注解

`@Scope`可用于更改Bean的作用域

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {
    @AliasFor("scopeName")
    String value() default "";

    @AliasFor("value")
    String scopeName() default "";

    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```

> 传入作用域类型时直接传入字符串，例如：`@Scope("prototype")`，但这种方法字符串拼写错误不容易发现；Spring提供了默认的参数：
>
> - `ConfigurableBeanFactory.SCOPE_PROTOTYPE`
>
> - `ConfigurableBeanFactory.SCOPE_SINGLETON`
>
> - `WebApplicationContext.SCOPE_REQUEST`
>
> - `WebApplicationContext.SCOPE_SESSION`
>
>   
>
>   proxyMode也提供了参数:
>
> - `ScopedProxyMode.INTERFACES`
>
> - `ScopedProxyMode.TARGET_CLASS`



- `singleton` : 表示在spring容器中的单例，通过spring容器获得该bean时总是返回唯一的实例
- `prototype`: 表示每次获得bean都会生成一个新的对象
- `request`: 表示在一次http请求内有效（只适用于web应用）
- `session`: 表示在一个用户会话内有效（只适用于web应用）
- `globalSession`:表示在全局会话内有效（只适用于web应用）



## 使用

作用域分为：

基本作用域(`singleton`, `prototype`)

Web作用域(`request`, `session`, `globalssion`)

由于默认为单例模式，所以需要标注时一般都是使用prototype

```java
@Bean
@Scope("prototype")  // @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public MyBean myBean() {
    return new MyBean();
}
```

prototype是原型模式，具体可以看设计模式中的"`原型模式`"，每次通过容器的getBean方法获取Bean时，都将产生一个新的Bean实例。



## 常见误区

常见使用误区，单例调用多例：

```java
@Component
public class SingletonBean {
    
    @Autowired
    private PrototypeBean bean;
    
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    
}
```

上面的代码中，外面的SingletonBean默认单例模式，里面的PrototypeBean设置成原型模式。
这里并不能达到我们每次创建返回新实例的效果。
因为`@Autowired`只会在单例SingletonBean初始化的时候注入一次，再次调用SingletonBean的时候，PrototypeBean不会再创建了(注入时创建，而注入只有一次)。
解决方法1：不使用`@Autowired`，每次调用多例的时候，直接调用Bean;
解决方法2：Spring的解决办法：设置proxyMode, 每次请求的时候实例化

>两种代理模式的区别：
>ScopedProxyMode.INTERFACES: 创建一个JDK代理模式
>ScopedProxyMode.TARGET_CLASS: 基于类的代理模式
>前者只能将其注入一个接口，后者可以将其注入类。