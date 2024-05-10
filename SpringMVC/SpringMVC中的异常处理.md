1.新建一个自定义异常，在定义它的子类

2.在cotroller中子类抛出异常

3.创建一个普通类，做全局异常处理类（aop）

1）.在类的上面加入ControllerAdvice

2）.在类中定义方法，在方法的上面加入@ExecptionHandler

```java
//控制器增强，给控制器类增加功能
//在类的上面使用
//特点：必须让框架知道这个注解所在的包名，需要在配置文件中使用组件扫描器
@ControllerAdvice
public class ExceptionHandlerTest {
    //定义方法，处理发生的异常
    //处理异常的方法和控制器方法的定义一样
    //形参是一个Exception，表示Controller中抛出的异常对象
    //可以获取发生异常的信息
    @ExceptionHandler(value = NameException.class)
    public ModelAndView doNameException(Exception ex) {
        //处理NameException异常
        //处理异常的逻辑：
        //      1.将异常记录下来，记录到数据库或日志文件
        //          记录发生的时间，那个方法发生的，异常错误的内容
        //      2.发送通知，把异常的信息通过邮件，短信等发送给相关人员
        //      3.给用户友好的提示
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","你的姓名必须是张三");
        mv.addObject("ex",ex);
        mv.setViewName("nameError");
        return mv;
    }
    //处理其它异常
    @ExceptionHandler()
    //不加value值处理其它的异常，只能有一个
    public ModelAndView doOtherException(Exception ex) {
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","其它的异常");
        mv.addObject("ex",ex);
        mv.setViewName("otherError");
        return mv;
    }
```



4.在SpringMVC配置文件中声明组件扫描器：扫描Controller和ControllerAdvice注解，声明注解驱动

```xml
<!-- 声明组件扫描器 -->
<context:component-scan base-package="com.xh"/>
<!-- 声明ControllerException的扫描器 -->
<context:component-scan base-package="com.handler"/>
```

