# Swagger

Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务的接口文档



**springBoot2.6.X版本使用swagger3.0.0版本**

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

pom文件:

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
            <scope>provided</scope>
        </dependency>

        <!--mybatis-plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided </scope>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
        </dependency>
```

yml文件：

```yaml
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
```



swagger配置：

```java
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spi.service.contexts.SecurityContext;
import springfox.documentation.spring.web.plugins.ApiSelectorBuilder;
import springfox.documentation.spring.web.plugins.Docket;

@Configuration
//swagger3.0.0版本使用@EnableOpenApi而不是使用EnableSwagger2
@EnableOpenApi
public class SwaggerConfig implements WebMvcConfigurer {
    @Bean
    public Docket webApiConfig(){
         Docket docket = new Docket(DocumentationType.SWAGGER_2)
                .host("http://localhost:8888/emps-wx-api/")
                .apiInfo(webApiInfo())
                .groupName(groupName)
                .select()
                //.apis(RequestHandlerSelectors.basePackage("com.xh.emosWxApi.controller"))
                //.paths(PathSelectors.any())
                .build();

        //将该项目所有路径下带有@ApiOperation注解的方法放入swagger中
        ApiSelectorBuilder apiSelectorBuilder = docket.select();
        apiSelectorBuilder.paths(PathSelectors.any());
        apiSelectorBuilder.apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class));
        docket = apiSelectorBuilder.build();

        //让Swagger支持JWT
        //name:请求参数名，keyname:请求描述,passAs:放在请求头中
        ApiKey apiKey = new ApiKey("token","token","header");
        List<ApiKey> apiKeyList = new ArrayList<>();
        apiKeyList.add(apiKey);
        docket.securitySchemes(apiKeyList);

        //设置JWT的作用域
        //设置作用域为global,可以访问一切
        AuthorizationScope authorizationScope = new AuthorizationScope("global","accessEverything");
        AuthorizationScope[] scopes  = {authorizationScope};
        SecurityReference reference = new SecurityReference("token",scopes);
        List<SecurityReference> securityReferenceList = new ArrayList<>();
        securityReferenceList.add(reference);
        SecurityContext securityContext = SecurityContext.builder().securityReferences(securityReferenceList).build();
        List<SecurityContext> securityContextList = new ArrayList<>();
        securityContextList.add(securityContext);
        docket.securityContexts(securityContextList);
    }


    private ApiInfo webApiInfo(){
        return new ApiInfoBuilder()
                .title("网站-课程中心API文档")
                .description("本文档描述了课程中心微服务接口定义")
                .version("1.0")
                .contact(new Contact("xh", "http://xh.com","1693062665@qq.com"))
                .build();
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/swagger-ui/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/springfox-swagger-ui/")
                .resourceChain(false);
    }
}

```

2. 添加Bean

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;
import org.springframework.util.ReflectionUtils;
import org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMapping;
import springfox.documentation.spring.web.plugins.WebFluxRequestHandlerProvider;
import springfox.documentation.spring.web.plugins.WebMvcRequestHandlerProvider;
 
import java.lang.reflect.Field;
import java.util.List;
import java.util.stream.Collectors;
 
@Component
public class SwaggerBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof WebMvcRequestHandlerProvider || bean instanceof WebFluxRequestHandlerProvider)
        {
            List<RequestMappingInfoHandlerMapping> handlerMappings = getHandlerMappings(bean);
            customizeSpringfoxHandlerMappings(handlerMappings);
        }
        return bean;
    }
 
    private <T extends RequestMappingInfoHandlerMapping> void customizeSpringfoxHandlerMappings(List<T> mappings) {
        List<T> copy = mappings.stream()
                .filter(mapping -> mapping.getPatternParser() == null)
                .collect(Collectors.toList());
        mappings.clear();
        mappings.addAll(copy);
    }
    
    @SuppressWarnings("unchecked")
    private List<RequestMappingInfoHandlerMapping> getHandlerMappings(Object bean) {
        try {
            Field field = ReflectionUtils.findField(bean.getClass(), "handlerMappings");
            field.setAccessible(true);
            return (List<RequestMappingInfoHandlerMapping>) field.get(bean);
        }
        catch (IllegalArgumentException | IllegalAccessException e) {
            throw new IllegalStateException(e);
        }
    }
}
```



 **访问 http://{ip}:{port}/swagger-ui/index.html**





# Swagger3.0 注解

```
@Api：用在请求的类上，表示对类的说明
    tags="说明该类的作用，可以在UI界面上看到的注解"
    value="该参数没什么意义，在UI界面上也看到，所以不需要配置"

@ApiOperation：用在请求的方法上，说明方法的用途、作用
    value="说明方法的用途、作用"
    notes="方法的备注说明"

@ApiImplicitParams：用在请求的方法上，表示一组参数说明
    @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面
        name：参数名
        value：参数的汉字说明、解释
        required：参数是否必须传
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · div（不常用）
            · form（不常用）    
        dataType：参数类型，默认String，其它值dataType="Integer"       
        defaultValue：参数的默认值

@ApiResponses：用在请求的方法上，表示一组响应
    @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：抛出异常的类

@ApiModel：用于响应类上，表示一个返回响应数据的信息
            （这种一般用在post创建的时候，使用@RequestBody这样的场景，
            请求参数无法使用@ApiImplicitParam注解进行描述的时候）
    @ApiModelProperty：用在属性上，描述响应类的属性
```





```java
@RestController
@RequestMapping("/teacher")
@Api(tags = "讲师管理")
public class TeacherController {

    @Resource
    private TeacherService teacherService;

    @GetMapping("/findAll")
    @ApiOperation(value = "查询所有讲师")
    public List<Teacher> findAll(){
        return teacherService.findAll();
    }

    @PostMapping("/delete/{id}")
    @ApiImplicitParams(@ApiImplicitParam(name = "id",value = "讲师id",required = true,paramType = "query"))
    @ApiOperation(value = "根据id删除讲师")
    public void deleteById(@PathVariable("id") String id){
        teacherService.deleteById(id);
    }
}
```



# 注意：

```
Illegal DefaultValue null for parameter type integer
```

在使用 swagger 时需要注意数值类型的参数（integer、long）需要添加一个 **example 属性**

```
Unable to interpret the implicit parameter configuration with dataType: , dataTypeClass: class java.lang.Void
```

指定dataTypeClass属性
