一、日志框架
===

```java
小张；开发一个大型系统；
      1、System.out.println("");将关键数据打印在控制台；去掉？写在一个文件？
      2、框架来记录系统的一些运行时信息；日志框架；zhanglogging.jar;
      3、日志输出的时候启用异步模式？自动归档？xxxx？ zhanglogging-good.jar?
      4、将以前框架卸下来？换上新的框架，重新修改之前相关的API； zhanglogging-perfect.jar;
      5、JDBC---数据库驱动；
          写了一个统一的接口层；日志门面（日志的一个抽象层）；logging-abstract.jar
          给项目中导入具体的日志实现就行了；我们之前的日志框架都是实现的抽象层；
          
市面上的日志框架；
日志门面：JCL(Jakarta Commons Logging)、SLF4j(Simple Logging Facade for java)、jboss-logging
日志实现：Log4j、JUL(java.util.logging)、Log4j2、Logback
上面选一个门面（抽象层）SLF4j、下面来选一个实现Logback；
```

二、SLF4j使用
===

1、如何在系统中使用SLF4j
---

  以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而实调用日志抽象层里面的方法；<br>
  应该给系统里面导入slf4j的jar和logback的实现jar<br>
  
  [slf4j用户手册](http://www.slf4j.org/manual.html)
  
  ```java
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    public class HelloWorld {
    public static void main(String[] args) {
      Logger logger = LoggerFactory.getLogger(HelloWorld.class);
      logger.info("Hello World");
    }
  }
```
![image](https://github.com/LoveChunHua/springboot/blob/master/images/bindings.png)

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，`配置文件还是做成日志实现框架的配置文件`；<br>

2、遗留问题
===

a(slf4j+logback):Spring(commons-logging)、Hibernate(jboss-logging)、MyBatis、xxxx <br>
统一日志记录，即使是别的框架和我一起统一使用slf4j <br>

![image](https://github.com/LoveChunHua/springboot/blob/master/images/legacy.png)

**如何让系统中所有的日志都统一到slf4j**

```java
1、将系统中其他日志框架先排除出去；
2、用中间包来替换原有的日志框架；
3、我们导入slf4j其他的实现
```

3、SpringBoot日志关系
===

pom.xml文件中右键选择diagrams——show dependencies显示整个依赖框架
```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.2.1.RELEASE</version>
      <scope>compile</scope>
    </dependency>
```
SpringBoot使用它来做日志功能；<br>
```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-logging</artifactId>
      <version>2.2.1.RELEASE</version>
      <scope>compile</scope>
    </dependency>
```
```java
总结：
      1）SpringBoot底层也是使用slf4j+logback的方式进行日志记录
      2）SpringBoot也把其他的日志都替换成了slf4j；
      3）中间替换包：log4j-to-slf4j jul-to-slf4j
      4)、如果我们要引入其他框架？一定要把这个框架的默认日志依赖移除掉？
            Spring框架用的是commons-logging；
      `SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉`
 ```
4、日志使用
===
1、默认配置
---
SpringBoot默认帮我们配置好了日志；
```java
@SpringBootTest
class SpringBoot02ConfigAutoconfigApplicationTests {
    //获取记录器
    Logger logger = LoggerFactory.getLogger(getClass());
    @Test
    void contextLoads() {
        //System.out.println();不要使用sout啦
        //日志的级别；
        //由低到高 trace<debug<info<warn<error
        //可以调整输出的日志级别；日志就只会在这个级别及以后的高级别生效
        logger.trace("这是trace日志...");
        logger.debug("这是debug日志...");
        //SpringBoot默认给我们使用的是info级别的,没有指定级别的就用SpringBoot默认规定的级别；root级别
        logger.info("这是info日志...");
        logger.warn("这是warn日志...");
        logger.error("这是error日志...");
    }
}
```
如果需要修改日志级别，应在配置文件application.properties中修改,示例如下：<br>
而SpringBoot默认的配置路径在springframework\boot\spring-boot\2.2.1.RELEASE\spring-boot-2.2.1.RELEASE.jar!\org\springframework\boot\logging\logback\base.xml
```java
#指定日志级别为trace及更高级别
logging.level.com.atguigu=trace

#当前面过目下生成springboot.log日志，该日志在target目录下
#logging.file.name=springboot.log

#也可以直接指定位置
#logging.file.name=D:/springboot.log

#在当前磁盘的根路径下（C盘）创建spring文件夹和里面的log文件夹；使用spring.log作为默认文件
logging.file.path=/spring/log

#在控制台输出的日志的格式  %d[yyyy-MM]显示日期时间格式 -5左对齐 logger全类名50个字符以内  msg带上消息 %n换行符
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n

#指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

2、指定配置
---
给类路径下resources文件夹下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置<br>
[SpringBoot官方自定义配置](https://docs.spring.io/spring-boot/docs/2.1.10.RELEASE/reference/html/boot-features-logging.html)

```java
Logging System	            Customization(自定义配置)
Logback                       logback-spring.xml, logback-spring.groovy, logback.xml, or logback.groovy
Log4j2                        log4j2-spring.xml or log4j2.xml
JDK (Java Util Logging)       logging.properties
```
logback.xml:直接就被日志框架识别了；<br>
logback-spring.xml:日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能

```java
可以指定某段配置只在某个环境下生效，加在logback-spring.xml中，需要在application.properties中激活某个环境spring.profile.active =?
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
	<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
	<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

5、切换日志框架
===

可以按照slf4j的日志适配图，进行相关的切换；<br>
切换到log4j2依赖如下：
```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
```
