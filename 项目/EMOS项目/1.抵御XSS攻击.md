# 抵御XSS攻击

XSS攻击通常指的是通过利用网站系统保存数据的漏洞，通过巧妙的方法把恶意指令注入到网页，用户加载网页的时候就会自动执行恶意脚本。

```javascript
<script>alert('xss')</script>
```

如果黑客能在你的浏览器上执行javaScript，那么就能窃取Cookie或者Token

所以避免XSS攻击最有效的办法就是对用户输入的数据进行转义，然后存储到数据库里面。等到视图层渲染HTML页面的时候。转义后的文字是不会被当做JavaScript执行的，这就可以抵御XSS攻击。



这里使用`Hutool`对请求中的数据进行`转义`

```xml
<dependency>
   <groupId>cn.hutool</groupId>
   <artifactId>hutool-all</artifactId>
   <version>5.8.12</version>
</dependency>
```

`HtmlUtil.filter` 过滤HTML文本，防止XSS攻击

```java
String html = "<alert></alert>";
// 结果为：""
String filter = HtmlUtil.filter(html);
```



## 定义请求包装类

我们平时写Web项目遇到的 `HttpServletRequest`，它其实是个接口。如果我们想要重新定义请求类，扩展这个接口是最不应该的。因为 `HttpServletRequest` 接口中抽象方法太多了，我们逐一实现起来太耗费时间。所以我们应该挑选一个简单一点的自定义请求类的方式。那就是继承**`HttpservletRequestWrapper`**父类。

JavaEE只是一个标准，具体的实现由各家应用服务器厂商来完成。比如说`Tomcat`在实现`Servlet`规范的时候，就自定义了`HttpServletRequest`接口的实现类。同时JavaEE规范还定义了HttpServletRequestwrapper，这个类是请求类的包装类，用上了`装饰器模式`。无论各家应用服务器厂商怎么去实现`HttpServletRequest`接口，用户想要自定义请求，只需要**`继承HttpServletRequestWrapper`**，对应覆盖某个方法即可，然后把请求传入请求包装类，装饰器模式就会替代请求对象中对应的某个方法。

将`HttpServletRequest`中所有从请求对象中获取数据的方法全部进行数据的转义处理:

- getParameter
- getParameters
- getParameterMap
- getHeader
- getInputStream

```java
import cn.hutool.core.util.StrUtil;
import cn.hutool.http.HtmlUtil;
import cn.hutool.json.JSONUtil;

import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.*;
import java.nio.charset.Charset;
import java.util.LinkedHashMap;
import java.util.Map;

public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {
    public XssHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    @Override
    public String getParameter(String name) {
        //获取原value
        String value = super.getParameter(name);
		//判断是否为空或者“”
        if (!StrUtil.hasEmpty(value)) {
            //3. 去掉HTML标签和Script标签
            value = HtmlUtil.filter(value);
        }
        return value;
    }

      /**
     * 重写getParameterValues方法，
     * 遍历每一个值，用HtmlUtil转义后再返回
     */
    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(name);
        if (values != null) {
            for (int i = 0; i < values.length; i++) {
                String value = values[i];
                if (!StrUtil.hasEmpty(value)) {
                    value = HtmlUtil.filter(value);
                }
                values[i] = value;
            }
        }
        return values;
    }
	
      /**
     * 重写getParameterMap方法，
     * 拿到所有的k-v键值对，用LinkedHashMap接收，
     * key不变，value用HtmlUtil转义后再返回
     */
    @Override
    public Map<String, String[]> getParameterMap() {
        Map<String, String[]> parameters = super.getParameterMap();
        LinkedHashMap<String, String[]> map = new LinkedHashMap();
        if (parameters != null) {
            for (String key : parameters.keySet()) {
                String[] values = parameters.get(key);
                for (int i = 0; i < values.length; i++) {
                    String value = values[i];
                    if (!StrUtil.hasEmpty(value)) {
                        value = HtmlUtil.filter(value);
                    }
                    values[i] = value;
                }
                map.put(key, values);
            }
        }
        return map;
    }

    @Override
    public String getHeader(String name) {
        String value = super.getHeader(name);
        if (!StrUtil.hasEmpty(value)) {
            value = HtmlUtil.filter(value);
        }
        return value;
    }

    @Override
    //springMVC通过getInputStream()获取
    public ServletInputStream getInputStream() throws IOException {
		 /**
         * 拿到数据流，通过StringBuffer拼接，
         * 读取到line上，用StringBuffer是因为会有多个线程同时请求，要保证线程的安全
         */
        InputStream in = super.getInputStream();
        InputStreamReader reader = new InputStreamReader(in, StandardCharsets.UTF_8);
        BufferedReader buffer = new BufferedReader(reader);
        StringBuffer body = new StringBuffer();
        String line = buffer.readLine();
        while (line != null) {
            body.append(line);
            line = buffer.readLine();
        }
        buffer.close();
        reader.close();
        in.close();
         /**
         * 拿到数据流，通过StringBuffer拼接，
         * 读取到line上，用StringBuffer是因为会有多个线程同时请求，要保证线程的安全
         */
        Map<String, Object> map = JSONUtil.parseObj(body.toString());
        Map<String, Object> result = new LinkedHashMap<>();
        for (String key : map.keySet()) {
            Object val = map.get(key);
            if (val instanceof String) {
                if (!StrUtil.hasEmpty(val.toString())) {
                    result.put(key, HtmlUtil.filter(val.toString()));
                }
            } else {
                result.put(key, val);
            }
        }
        String json = JSONUtil.toJsonStr(result);
		//匿名内部类，只需要重写read方法，把转义后的值，创建成ServletInputStream对象
        ByteArrayInputStream bain = new ByteArrayInputStream(json.getBytes());
        return new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return bain.read();
            }

            @Override
            public boolean isFinished() {
                return false;
            }

            @Override
            public boolean isReady() {
                return false;
            }

            @Override
            public void setReadListener(ReadListener readListener) {

            }
        };
    }
}

```



## 自定义过滤器

```java
@WebFilter(urlPatterns = "/*",filterName = "xssFilter")
public class XSSFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        //构造包装给定请求的请求对象。
        XSSHttpServletRequestWrapper xssHttpServletRequestWrapper = new XSSHttpServletRequestWrapper(httpServletRequest);
        chain.doFilter(xssHttpServletRequestWrapper, response);
    }
}
```



## 启动类添加@ServletComponentScan

```java
@SpringBootApplication
@ServletComponentScan({"com.xh.emosWxApi.common.xss"})
public class EmosWxApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(EmosWxApiApplication.class, args);
    }
```

