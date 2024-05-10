### BeanUtils

​	BeanUtils是由Apache公司所开发的工具包，目的是为了简化数据封装，方便Java程序员对JavaBean类进行简便的操作。



#### 注意事项：

被封装的JavaBean必须是标准的Java类

- 该JavaBean必须被**public**所修饰
- 必须提供**空构造器**
- 成员变量必须私有化，被**private**所修饰
- 提供公共的**setter**和**getter**方法



#### BeanUtils简单使用

| 方法名称                                                     | 描述信息                                         |
| ------------------------------------------------------------ | ------------------------------------------------ |
| populate(Object bean, Map<String,? extends Object> properties) | 根据指定的名称/值对填充指定Bean的JavaBeans属性。 |
| setProperty(Object bean, String name, Object value)          | 设置指定属性的值                                 |
| copyProperty(Object bean, String name, Object value)         | 拷贝指定的属性值到指定的bean                     |
| **copyProperties(Object dest, Object orig)**                 | **对象的拷贝**                                   |
| cloneBean(Object bean)                                       | 对象的克隆                                       |

```java
//BeanUtils.copyProperties("转换前的类", "转换后的类");
BeanUtils.copyProperties(article,articleVo);
```



1. **BeanUtils.populate(Object bean, Map<String,? extends Object> properties)**

   根据指定的名称/值对填充指定Bean的JavaBeans属性。

   ```java
   User user = new User(); // 被封装的JavaBean
   Map<String, String[]> loginUser = request.getParameterMap(); // 封装的数据Map集合
   /* 等价：
    *     Map<String, String> loginUser = new HasheMap<>();
    *     loginUser.put("username","langkye");
    *     loginUser.put("password","admin");
    */
   BeanUtils.populate(user,loginUser);
   System.out.println(user);
   // 输出：User{username='langkye', password='admin', birthday=null}
   ```



2.**BeanUtils.setProperty(Object bean, String name, Object value)**

​		设置指定属性的值

```java
User user = new User(); // 被封装的JavaBean
BeanUtils.setProperty(user,"username","langkye"); // 设置username属性
BeanUtils.setProperty(user,"password","admin"); // 设置password属性
System.out.println(user); 
// 输出：User{username='langkye', password='admin', birthday=null}
```



3.**BeanUtils.copyProperty(Object bean, String name, Object value)** 

​	拷贝指定的属性值到指定的bean

```java
User user1 = new User();
BeanUtils.copProperties(user1, "username","langkye");
System.out.println(user1); // 输出：User{username='langkye', password=null, birthday=null}
```



***4.BeanUtils.copyProperties(Object dest, Object orig) 最主要使用*** 

​	对象的拷贝

```java
// 对象的拷贝
User user2 = new User();
BeanUtils.copyProperties(user, user2);
System.out.println(user1); 
// 输出：User{username='langkye', password='admin', birthday=null}
```

- b中的存在的属性，a中一定要有，但是a中可以有多余的属性；
- a中与b中相同的属性都会被替换，不管是否有值；
- a、 b中的属性要名字相同，才能被赋值，不然的话需要手动赋值；
- Spring的BeanUtils的CopyProperties方法需要对应的属性有getter和setter方法；

**若对象中的数据类型不同，则需要进行数据转换**

```java
//BeanUtils.copyProperties("转换前的类", "转换后的类");
BeanUtils.copyProperties(article,articleVo);
//vo中时间存储为String类型，数据库中的事件为Long类型，需要进行转换
articleVo.setCreateDate(
    	new DateTime(articleVo.getCreateDate()).toString("yyyy-MM-dd HH:mm"));
```



5.**BeanUtils.cloneBean(Object bean)**

​		对象的克隆

```java
// 克隆对象
User user3 = (User)BeanUtils.cloneBean(user);
System.out.println(user3);
// 输出：User{username='langkye', password='admin', birthday=null}
```

