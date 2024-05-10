拦截器的使用步骤：（AOP的思想）

1.定义类实现Handlerinterceptor接口

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    //预处理方法
    //Object handler 被拦截的控制器对象
    //返回值为boolean
    //特点：方法在控制器方法之前先执行，用户的请求首先到达此方法
    //在这个方法中我们可以获取请求的信息，验证请求是否符合要求
    //可以验证用户是否登录，验证用户是否有权限访问某个链接地址（url）
    //如果验证失败，可以截断请求，请求不能被处理，验证成功，可以放行请求
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        return false;
    }

    @Override
    //后处理方法
    //Object handler 被拦截的控制器对象
    //ModelAndView modelAndView 处理器方法的返回值
    //特点：在处理器方法之后执行
    //     可以获取到处理器方法的返回值，可以修改modelAndView中的视图和数据，可以影响到最后的执行结果
    //作用：对原理执行的结果做二次修正
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    //最后执行的方法
    //Object handler 被拦截的控制器对象
    //Exception ex 程序中发送的异常
    //在请求处理完成后执行的：当视图处理完成后（forward）
    //一般做资源回收工作的，程序请求过程中创建了一些对象，在这里可以删除
    //
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {

    }
}
```

2.在SpringMVC的配置文件中，声明拦截器

```xml
<!-- 声明拦截器:拦截器可以有0个或多个 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 指定拦截的请求uri地址
              path：uri地址，使用通配符-->
        <mvc:mapping path="/**"/>
        <!-- 声明拦截器对象  -->
        <bean class = "com.interceptor.MyInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```