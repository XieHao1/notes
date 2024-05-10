#  @PostConstruct

 `@PostConstruct`是Java自带的注解，在方法上加该注解会在项目启动的时候执行该方法，也可以理解为在`spring容器初始化`的时候执行该方法。

  从Java EE5规范开始，Servlet中增加了两个影响Servlet生命周期的注解，`@PostConstruct`和`@PreDestroy`，这两个注解被用来修饰一个`非静态的void（）`方法。

> 作用

@PostConstruct注解的方法在项目启动的时候执行这个方法，也可以理解为在spring容器启动的时候执行，可作为一些数据的常规化加载，比如数据字典之类的。

> 执行顺序

其实从依赖注入的字面意思就可以知道，要将对象p注入到对象a，那么首先就必须得生成对象a和对象p，才能执行注入。所以，如果一个类A中有个成员变量p被@Autowried注解，那么@Autowired注入是发生在A的构造方法执行完之后的。

如果想在生成对象时完成某些初始化操作，而偏偏这些初始化操作又依赖于依赖注入，那么久无法在构造函数中实现。为此，可以使用@PostConstruct注解一个方法来完成初始化，@PostConstruct注解的方法将会在依赖注入完成后被自动调用。

**`Constructor >> @Autowired >> @PostConstruct`**

```java
	@PostConstruct
    // @PostConstruct是Java自带的注解，在方法上加该注解会在项目启动的时候执行该方法，
    //也可以理解为在spring容器初始化的时候执行该方法。
    public void init(){
        List<SysConfig> sysConfigs = sysConfigMapper.searchAll();
        sysConfigs.forEach(sysConfig -> {
            String paramKey = sysConfig.getParamKey();
            //转换为命名
            paramKey = StrUtil.toCamelCase(paramKey);
            String paramValue = sysConfig.getParamValue();
            //使用反射注入属性
            try {
                Field field = sysConfigConstant.getClass().getDeclaredField(paramKey);
                field.set(sysConfigConstant,paramValue);
            } catch (NoSuchFieldException | IllegalAccessException e) {
                log.warn("执行异常");
            }
        });
    }
```

