# Swagger+Knife4j

[官方文档](https://doc.xiaominfo.com/)

访问地址:http://ip:port/doc.html

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi2-spring-boot-starter</artifactId>
    <version>4.0.0</version>
</dependency>
```



配置类:

```java
import com.github.xiaoymin.knife4j.spring.extension.OpenApiExtensionResolver;
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
import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;

import java.util.ArrayList;
import java.util.List;

@Configuration
@EnableSwagger2WebMvc
public class SwaggerConfiguration {

    /*引入Knife4j提供的扩展类*/
    private final OpenApiExtensionResolver openApiExtensionResolver;

    @Autowired
    public SwaggerConfiguration(OpenApiExtensionResolver openApiExtensionResolver) {
        this.openApiExtensionResolver = openApiExtensionResolver;
    }

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        String groupName = "emos-wx-api";
        Docket docket = new Docket(DocumentationType.SWAGGER_2)
                .host("http://localhost:8888/emps-wx-api/")
                .apiInfo(apiInfo())
                .groupName(groupName)
                .select()
                //.apis(RequestHandlerSelectors.basePackage("com.xh.emosWxApi.controller"))
                //.paths(PathSelectors.any())
                .build()
                //赋予插件体系
                .extensions(openApiExtensionResolver.buildExtensions(groupName));

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

        return docket;
    }

    private ApiInfo apiInfo() {
        ApiInfoBuilder builder = new ApiInfoBuilder();
        return builder.title("EMOS")
                .contact(new Contact("谢豪","","1693062665@qq.com"))
                .description("EMOS接口")
                .version("1.0")
                .build();
    }
}
```

