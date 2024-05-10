### ConvertUtils简单使用



ConvertUtils为类型转换工具类

ConvertUtils可以使字符串和指定类型的实例之间进行转换。 实际上，**BeanUtil 是依赖ConvertUtils来完成类型转换**，但是有时候我们可能需要自定义转换器来完成特殊需求的类型转换；



###  常用方法

| 方法                                       | 描述                               |
| ------------------------------------------ | ---------------------------------- |
| convert(Object value, Class<?> targetType) | 将给定的value转换成指定的Class类型 |
| register(Converter converter, Class clazz) | 给BeanUtils注册指定类型的转换器    |



#### 字符串和Date类型转换

1. **convert(Object value, Class<?> targetType)**

```java
// 将日期转换成字符串
String DateStr = (String)ConvertUtils.convert(new Date(), String.class);
System.out.println(DateStr); // Fri May 20 00:00:00 CST 2050
System.out.println(
    (ConvertUtils.convert(new Date(), String.class)).getClass().getTypeName()); 
// java.lang.String
```



2.**register(Converter converter, Class clazz)**

```java
Map<String,String> paraMap = new HashMap<>().put("birthday","2050-05-20");
// 将字符串转换为日期类型
DateConverter dc = new DateConverter();
dc.setPattern("yyyy-MM-dd");
ConvertUtils.register(dc,Date.class);
BeanUtils.populate(user4,paraMap);
System.out.println(user4); 
// User {username=null,password=null,birthday='Fri May 20 00:00:00 CST 2050'}
```



#### 字符串和Double

```java
// 将字符串转换成Double
Double doubleNumber = (Double)ConvertUtils.convert("2050", Double.class);
System.out.println(doubleNumber); // 2050.0
String dn_typeName = doubleNumber.getClass().getTypeName();
System.out.println(dn_typeName); // java.lang.Double
```

