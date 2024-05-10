响应ajax请求的步骤：

1.加入处理json的工具库依赖，SpringMVC默认为jackson

2.在SpringMVC配置文件中加入注解驱动 --(将数据转换为json,xml,二进制等)

```xml
<mvc:annotation-driven>
```

3.在处理器方法的上面加入@ResponseBody注解（将数据写入响应体中）

```java
@RequestMapping(value = "/returnJSON.do",method = RequestMethod.GET)

@ResponseBody

public List<Student> show(HttpServletRequest request, String name, Integer age){
    String name1 = request.getParameter("name");
    String age1 = request.getParameter("age");
    List<Student> students = new ArrayList<>();
    Student student = new Student();
    student.setName(name1);
    student.setAge(Integer.valueOf(age1));
    students.add(student);
    students.add(new Student("张三",20));
    return students;
}
```

实现原理：

```xml
<mvc:annotation-driven>
```

功能：1.完成java对象到json，xml，text，二进制等数据格式的转换。

  		2.会自动创建HttpMessageConveter接口的7个实现类对象（例如：MappingJackson2HttpMessageConveter--使用jackson工具库中的objectMapper实现java对象转换为json）



<img src="C:\Users\86177\AppData\Local\YNote\data\qqDF41D122736AEA611532EEA277014B95\06107c86c77b4850abab3939ab3a065f\clipboard.png" alt="img" style="zoom:80%;" />

