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
 ```
 1)所有/webjars/**，都去classpath:/META-INF/resources/webjars/找资源; webjars：以jar包的方式引入静态资源；<br>
    [webjars参考](https://www.webjars.org/)
    
 
    
  
