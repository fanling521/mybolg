# Spring-Boot教程

## 什么是Spring-Boot

## Spring-Boot的搭建

以Spring-Boot2为例

## Spring-Boot技巧

### 使用lombok简化代码

详细信息查看**lombok-教程**

### 配置文件中使用数组

（1）yml文件中的写法

```yml
intelligentsearch:
  name: 智能检索服务
  version: 1.0.0
  whiteList[0]: 192.168.1.2
  whiteList[1]: 192.168.1.3
```

（2）配置类中的写法

```java
@Configuration
@ConfigurationProperties(prefix = "intelligentsearch")
@Setter
@Getter
public class SystemConfig {
    private String name;
    private String version;
    private List<String> whiteList;
}
```

### 自定义拦截器

以ip白名单为例说明

（1）自定义WebConfiguration

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {
    @Bean
    public HandlerInterceptor getMyInterceptor() {
        return new IPInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(getMyInterceptor()).addPathPatterns("/**");
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //.addResourceLocations("classpath:/static/")
        registry.addResourceHandler("/**");
    }

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new DateFormatter("yyyy-MM-dd HH:mm:ss"));
    }
}
```

（2）自定义HandlerInterceptor的实例

```java
public class IPInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //...你的逻辑
        return true;
    }
```

### SpringBoot @Value 设置默认值

添加个冒号，指定默认值。

```java
@Component
@Getter
@Setter
public class SolrConfig {
    @Value("${solr.zk.zktimeout:60000}")
    private String zkTimeOut;
}
```

注意：配置类只能使用`@Autowired`，不能使用new的方式。

