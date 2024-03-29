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
    
![image](https://github.com/LoveChunHua/springboot/blob/master/images/webjarsStruct.png)<br>
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
`JSP、Velocity、Freemarker、Thymeleaf`;<br>

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
    public static final String DEFAULT_PREFIX = "classpath:/templates/";//拼前缀
    public static final String DEFAULT_SUFFIX = ".html";//拼后缀
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    //只要我们把html页面放在"classpath:/templates/"，thymelefa就能帮我们自动渲染了
```
使用：<br>
1、导入Thymeleaf的名称空间<br>
`<html lang="en" xmlns:th="http://www.thymeleaf.org">`<br>
2、使用Thymeleaf的语法<br>
```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text将div里面的文本内容设置为指定的值    -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>
```
3、语法规则
---
1）、th:text;改变当前元素里面的文本内容；<br>
    th：任意html属性；来替换原生属性的值<br>
  [th内容参照](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#attribute-precedence)<br>
2）、表达式<br>
  [表达式参照](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax)
```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    @ResponseBody
    public String hello(){
        return "Hello World~";
    }
    //查出一些数据在页面展示
    @RequestMapping("/success")
    public String success(Map<String,Object> map){
        map.put("hello","你好~");
        map.put("users", Arrays.asList("zhangsan","lisi","wangwu"));
        return "success";
    }
}

<body>
    <h1>成功！</h1>
    <!--th:text将div里面的文本内容设置为指定的值    -->
    <div id="div01" class="myDiv" th:id="${hello}" th:class="${hello}" th:text="${hello}">这是显示欢迎信息</div>
    <hr/>
    <div th:text="${hello}"></div>
    <div th:utext="${hello}"></div>
    <hr/>
    <!-- th:each每次遍历都会生成当前这个标签：3个h4-->
    <h4 th:text="${user}" th:each="user:${users}"></h4>
    <hr/>
    <h4>
        <span th:each="user:${users}">[[${user}]]</span>
    </h4>
</body>
```

4、SpringMVC自动配置
===

[开发参照](https://docs.spring.io/spring-boot/docs/2.1.10.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc)<br>

```java
Spring Boot自动配置好了SpringMVC
以下是SpringBoot对SpringMVC的默认配置：
Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.
自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View）,视图对象决定如何渲染（转发?重定向？）
ContentNegotiatingViewResolver：组合所有的视图解析器的；
如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来
@SpringBootApplication
public class SpringBoot04WebRestfulcrudApplication {
    public static void main(String[] args) {                SpringApplication.run(SpringBoot04WebRestfulcrudApplication.class, args);
    }    
    @Bean
    public ViewResolver myViewResolver(){
        return new myViewResolver();
    }    
    public static class myViewResolver implements ViewResolver{
        @Override
        public View resolveViewName(String s, Locale locale) throws Exception {
            return null;
        }
    }    
}

Support for serving static resources, including support for WebJars (covered later in this document)).静态资源文件夹路径，webjars；
Automatic registration of Converter, GenericConverter, and Formatter beans.
    Converter：转换器；public String hello(User user):类型转换使用Converter；
    Formatter：格式化器；2019.12.3==Date
    自己添加的格式化器只需要放到容器中即可
Support for HttpMessageConverters (covered later in this document).
    HttpMessageConverters:SpringMVC用来转换http请求和响应的，user==json
    HttpMessageConverters是从容器中确定；获取所有的HttpMessageConverter;
    自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component)
Automatic registration of MessageCodesResolver (covered later in this document).
Static index.html support.静态首页访问
Custom Favicon support (covered later in this document).支持图标favicon.ico
Automatic use of a ConfigurableWebBindingInitializer bean (covered later in this document).
If you want to keep Spring Boot MVC features and you want to add additional MVC configuration (interceptors, formatters, view controllers, and other features), you can add your own @Configuration class of type WebMvcConfigurer but without @EnableWebMvc. If you wish to provide custom instances of RequestMappingHandlerMapping, RequestMappingHandlerAdapter, or ExceptionHandlerExceptionResolver, you can declare a WebMvcRegistrationsAdapter instance to provide such components.

If you want to take complete control of Spring MVC, you can add your own @Configuration annotated with @EnableWebMvc.
```

2、扩展SpringMVC
---
```java
    <mvc:view-controller path="/hello" view-name="success"/>
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/hello"/>
            <bean></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```
编写一个配置类（@Configuration），是WebMvcConfigurer类型；不能标注@EnableWebMvc，因为这个注解让我们完全接管了SpringMVC导致静态资源无法访问;既保留了所有的自动配置，也能用我们扩展的配置；
```java
//使用WebMvcConfigurer可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //浏览器发送 /atguigu 请求来到 sucess页面
        registry.addViewController("/atguigu").setViewName("success");
    }
}
```
5、如何修改SpringBoot的默认配置
===

模式：<br>
```java
1)、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean，@Component)如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver)将用户配置的和自己默认的组合起来；
2）、在SpringBoot中会有非常多的xxxxConfigurer帮助我们进行扩展配置
```
6、RestfulCRUD
===
1)、默认访问首页
---
```java
    //所有的WebMvcConfigurer组件都会一起起作用,该方法在配置类中（标注@Configuration的类中）
    @Bean//将组件注册在容器，很重要
    public WebMvcConfigurer webMvcConfigurer(){
        WebMvcConfigurer adapter = new WebMvcConfigurer() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("index");
                registry.addViewController("/index.html").setViewName("index");
            }
        };
        return adapter;
    }
}
```
引入bootstrap从webjars官网<br>
```java
        <!--引入bootstrap-->
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>bootstrap</artifactId>
            <version>4.3.1</version>
        </dependency>
```
2)、国际化
---
步骤：<br>
```java
1、编写国际化配置文件，抽取页面需要显示的国际化
```
![image](https://github.com/LoveChunHua/springboot/blob/master/images/%E5%9B%BD%E9%99%85%E5%8C%96.png)

2、SpringBoot自动配置好了管理国际化资源文件的组件；<br>
```java
 @Bean
    @ConfigurationProperties(
        prefix = "spring.messages"
    )
    public MessageSourceProperties messageSourceProperties() {
        return new MessageSourceProperties();
    }

    @Bean
    public MessageSource messageSource(MessageSourceProperties properties) {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        if (StringUtils.hasText(properties.getBasename())) {
        //setBasename获取国际化资源文件的基础名（去掉语言国家代码的）
        //我们的配置文件可以直接放在类路径下叫messages.properties;
        messageSource.setBasenames(StringUtils.commaDelimitedListToStringArray(StringUtils.trimAllWhitespace(properties.getBasename())));
        }

        if (properties.getEncoding() != null) {
            messageSource.setDefaultEncoding(properties.getEncoding().name());
        }

        messageSource.setFallbackToSystemLocale(properties.isFallbackToSystemLocale());
        Duration cacheDuration = properties.getCacheDuration();
        if (cacheDuration != null) {
            messageSource.setCacheMillis(cacheDuration.toMillis());
        }

        messageSource.setAlwaysUseMessageFormat(properties.isAlwaysUseMessageFormat());
        messageSource.setUseCodeAsDefaultMessage(properties.isUseCodeAsDefaultMessage());
        return messageSource;
    }
```
```java
<body class="text-center">
		<form class="form-signin" action="dashboard.html">
			<img class="mb-4" th:src="@{/asserts/img/bootstrap-solid.svg}" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
			<h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
			<label class="sr-only" th:text="#{login.username}">Username</label>
			<input type="text" class="form-control" placeholder="Username" required="" autofocus="">
			<label class="sr-only" th:text="#{login.password}">Password</label>
			<input type="password" class="form-control" placeholder="Password" required="">
			<div class="checkbox mb-3">
				<label>
          <input type="checkbox" value="remember-me"/> [[#{login.remember}]]
        </label>
			</div>
			<button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
			<p class="mt-5 mb-3 text-muted">© 2017-2018</p>
			<a class="btn btn-sm">中文</a>
			<a class="btn btn-sm">English</a>
		</form>
```
效果：根据浏览器语言设置的信息切换了国际化；<br>
原理：国际化Locale（区域信息对象）；LocaleResolver（获取区域信息对象）<br>
默认的就是根据请求头带来的区域信息获取LocaleResolver进行国际化
```java
/*
* 可以在链接上携带区域信息
* */

<a class="btn btn-sm" th:href="@{/index.html(l='zh_CN')}">中文</a>
<a class="btn btn-sm" th:href="@{/index.html(l='en_US')}">English</a>
            
public class MyLocaleResolver implements LocaleResolver
{
    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        String l = httpServletRequest.getParameter("l");
        Locale locale =Locale.getDefault();
        if(!StringUtils.isEmpty(l)){
            String[] split = l.split("_");
            locale = new Locale(split[0],split[1]);
        }
        return locale;
    }
    
     @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
```
3)、登录
---
开发期间模板引擎页面修改以后，需要实时生效<br>
1、禁用缓存 spring.thymeleaf.cache=false <br>
2、ctrl + F9重新编译<br>
```java
//控制登录
@Controller
public class LoginController {
//    @RequestMapping(value = "/user/login",method = RequestMethod.POST)
//    @DeleteMapping
//    @PutMapping
//    @GetMapping
    @PostMapping(value = "/user/login")
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Map<String,Object> map){
        if(!StringUtils.isEmpty(username) &&"123456".equals(password)){
            //登录成功，防止表单重复提交，可以重定向到主页
            return "redirect:/main.html";
        }else{
            map.put("msg","用户名密码错误");
            return "login";
        }
    }
```
登录错误提示演示
```java
	<h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
	<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```  
4）、拦截器进行登录检查
---
```java
/*登录检查
*
* */
public class LoginHandlerInterceptor implements HandlerInterceptor {
    //目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("loginUser");
        if(user == null){
            //未登录，返回登录页面
            request.setAttribute("msg","没有权限请先登录");
            request.getRequestDispatcher("/index.html").forward(request,response);
            return false;
        }else{
            //已登录，放行请求
            return true;
        }
    }
```
```java
 //注册拦截器,从Myconfig中
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                //SpringBoot已经做好了静态资源映射，不需要拦截静态资源
              registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**").excludePathPatterns("/","/index.html","/user/login");
            }
```
5）、CRUD员工列表
---
```java
实验要求：
1、RestfulCRUD：CRUD满足Rest风格；
URI:/资源名称/资源标识 HTTP请求方式区分对资源CRUD操作
             普通CRUD               	RestfulCRUD
查询          getEmp                	emp---GET
添加	    addEmp?xxx                  emp---POST
修改          updateEmp?id=xxx&xxx=xx     emp/{id}---PUT
删除          deleteEmp?id=1              emp/{id}---DELETE

2)、实验的请求架构
                请求URI              请求方式
查询所有员工     emps                 GET
查询某个员工     emp/{id}             GET
来到添加页面     emp                  GET
添加员工         emp                  POST
来到修改页面     emp/{id}             GET
修改员工         emp                  PUT
删除员工         emp/{id}             DELETE
3)、员工列表
thymeleaf公共页面元素抽取
1、抽取公共片段
<body>
    <div th:fragment="copy">
      &copy; 2011 The Good Thymes Virtual Grocery
    </div> 
  </body>
2、引入公共片段 抽取标签的意思
<div th:insert="~{footer :: copy}"></div>
"~{templatename::selector}" 模板名：：选择器
~{templatename::fragmentname} 模板名：：片段名

三种引入功能片段的th属性
th:insert:将公共片段整个插入到声明引入的元素中
th:replace：将声明引入的元素替换为公共片段
th:include：将被引入的片段的内容包含进这个标签中
如果使用th:insert等属性进行引入，可以不用写~{} 行内写法可以加上:[[~{}]]
