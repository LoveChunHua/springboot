一、使用SpringBoot步骤：
===

```java
1)创建SpringBoot应用，选中我们需要的模块；
2)SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来；
3)自己编写业务代码；
```

这就需要了解自动配置原理？了解这个场景SpringBoot帮我们配置了什么？能不能修改？能的修改哪些配置？能不能扩展？等等...<br>
`xxxxAutoConfiguration：帮我们给容器中自动配置组件；`<br>
`xxxxProperties：配置类来封装配置文件的内容；`

二、SpringBoot对静态资源的映射规则
===
```java
//可以设置和静态资源有关的参数，缓存时间等
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS 
    = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
    private String[] staticLocations;
```
```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"})
                    .addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"})
                    .setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern})
                    .addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations()))
                    .setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

            }
        }
        
        //配置欢迎页
          @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext, FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
            WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
            welcomePageHandlerMapping.setInterceptors(this.getInterceptors(mvcConversionService, mvcResourceUrlProvider));
            return welcomePageHandlerMapping;
        }
 ```
 1)、所有/webjars/\*\*，都去classpath:/META-INF/resources/webjars/找资源; webjars：以jar包的方式引入静态资源；<br>
    [webjars参考](https://www.webjars.org/) <br>
    
![image](https://github.com/LoveChunHua/springboot/blob/master/images/webjarsStruct.png)
`路径为localhost:8080/webjars/jquery/3.4.1/jquery.js`<br>
```java
<!--引入jquery-webjar-->在访问的时候只需要写webjars下面资源的名称即可
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.4.1</version>
        </dependency>
```
2）、/**  访问当前项目的任何资源(静态资源的文件夹）需要自己从类路径下自己创建
```java
"classpath:/META-INF/resources/", 
"classpath:/resources/", 
"classpath:/static/",
"classpath:/public/"
```
`localhost:8080/abc ===去静态资源文件夹里面找abc`<br>

3）、欢迎页；静态资源问价下的所有index.html页面；被/\*\*映射<br>
   localhost:8080/ 找index页面<br>
4）、所有的\*\*/favicon.ico都是在静态资源文件下找<br>

3、模板引擎
===
`JSP、Velocity、Freemarker、Thymeleaf`;
1、引入Thymeleaf
---
```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```
2、Thymeleaf使用和语法
---
```java
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    //只要我们把html页面放在"classpath:/templates/"，thymelefa就能帮我们自动渲染了
```

  
