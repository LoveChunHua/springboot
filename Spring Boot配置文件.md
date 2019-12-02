一、Spring Boot配置文件
=
Spring Boot使用一个全局的配置文件<br>
    application.properties<br>
    application.yml<br>

配置文件的作用：修改SpringBoot自动配置的默认值；SpringBoot在底层都给我们配置好；<br>
YAML(YAML Ain't Markup Language)<br>
  YAML A Markup Language:是一个标记语言；<br>
  YAML isn't Markup Language:不是一个标记语言；<br>
标记语言：<br>
  以前的配置文件大多都使用的是xxxx.xml文件<br>
  YAML:以数据为中心，比json、xml等更适合做配置文件<br>
  YAML:配置例子<br>
  server:<br>
   port:8081<br>
   
   XML:<br>
   server.port=8090<br>
   
 二、YAML语法
 =
 1、基本语法
 --
 
 k:(空格)v:表示一对键值对（空格必须有）；<br>
 以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的<br>
 属性和值也是大小写敏感；<br>
 
 2、值的写法
 --
 字面量：普通的值（数字、字符串、布尔）<br>
 k: v:字面量直接来写；<br>
    字符串默认不用加上单引号或者双引号；<br>
    "":双引号：不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思<br>
        name:"zhangsan \n lisi"；输出：zhangsan 换行 lisi<br>
    '':单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据<br>
        name:'zhangsan \n lisi';输出：zhangsan \n lisi<br>
 对象、Map（属性和值）（键值对）：<br>
    k: v:在下一行来写对象的属性和值的关系；注意缩进<br>
      对象还是k: v的方式<br>
      friends:<br>
          lastName:zhangsan<br>
          age:20<br>
行内写法：<br>
friends:{lastName: zhangsan,age: 18}<br>
 数组（List、Set):<br>
 用-值表示数组中的一个元素<br>
 pets:<br>
 - cat<br>
 - dog<br>
 - pig<br>
 行内写法<br>
 pets: [cat,dog,pig]<br>
 
 3、配置文件值注入值
 --
 配置文件<br>
 ```java
    person:
      lastName: zhangsan
      age: 18
      boss: false
      birth: 2017/12/12
      maps: {k1: v1,k2: 12}
      lists:
        - lisi
        - zhaoliu
      dog:<br>
        name: peiqi
        age: 2
```
javaBean:<br>
```java
**
 * 将配置文件中配置的每一个属性的值映射到这个组件中
 * @ConfigurationProperties:告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 * prefix = "person":配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private  String lastName;
    private  Integer age;
    private  Boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

我们可以导入配置文件处理器，以后编写配置就可以有提示了<br>
```xml
 <!--导入配置文件处理器，配置文件进行绑定就会有提示-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

@Value获取值和@configurationProperties获取值比较<br>

```java
               @configuration           @Value
功能           批量注入配置文件中的属性    一个个指定
松散语法       支持                      不支持
SpEl          不支持                     支持
JSR303校验     支持                      不支持        使用@Validated注解
复杂类型封装   支持                       不支持
```
配置文件yml还是properties他们都能获取到值<br>
如果说，我们只是在某个业务逻辑中需要获取以下配置文件中的某项值，使用@Value<br>
如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties

配置文件注入值数据校验<br>

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    /**
     * <bean class="Person">
     *          <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEl}"></property>
     *     </bean>
     */

    //lastName必须是邮箱格式
    @Email
    //@Value("${person.last-name}")
    private  String lastName;
    //@Value("#{11*2}")
    private  Integer age;
    //@Value("true")
    private  Boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

@PropertySource 和 @ImportSource<br>

@ConfigurationProperties:加载全局配置文件中的值<br>
@PropertySource：加载指定的配置文件；<br>

```java
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
public class Person {

    /**
     * <bean class="Person">
     *          <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEl}"></property>
     *     </bean>
     */

    //lastName必须是邮箱格式
    @Email
    //@Value("${person.last-name}")
    private  String lastName;
    //@Value("#{11*2}")
    private  Integer age;
    //@Value("true")
    private  Boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

@ImportResource:导入Spring的配置文件，让配置文件里面的内容生效<br>
SpringBoot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；<br>
想让Spring的配置文件生效，加载进来；@**ImportResource**标注在一个配置类上<br>
```java
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class SpringBoot02ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot02ConfigApplication.class, args);
    }

}
```

不来编写Spring的配置文件
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class ="com.atguigu.springboot.service.HelloService"></bean>
</beans>
```

SpringBoot推荐给容器中添加组件的方式：推荐使用全注解的方式<br>
1、配置类====Spring配置文件<br>
2、使用@Bean给容器中添加组件
```java
/**
 * @Configuration:指明当前类是一个配置类；就是来替代之前的Spring配置文件
 *
 * 在配置文件中用<bean></bean>标签添加组件
 */
@Configuration
public class MyAppConfig {
    //将方法的返回值添加到容器中，容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了。。。");
        return new HelloService();
    }
}
```

四、置文件占位符
===

1、随机数
---

```java
${random.value}、${random.int}、${random.long}
${random.int(10)、${random.int[1024,65536]}
```

2、占位符获取之前配置的值，如果没有可以使用：指定默认值
---

```java
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/12
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=12
```

五、Profile
===

1、多Profile文件
---

我们在主配置文件编写的时候，文件名可以是application-{profile}.properties/yml <br>
默认使用application.properties <br>

```java
server.port=8081
#激活哪一个配置文件
spring.profiles.active=d
#idea，properties配置文件utf-8编码
# 配置person的值

person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/12
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=12
```

2、yml支持多文档块方式
---

```java
server:
  port: 8081
spring:
  profiles:
    active: prod
---

server:
  port: 8083
spring:
  profiles: d
---

server:
  port: 8084
spring:
  profiles: prod
```

3、激活指定Profile
---

1、在配置文件中指定spring.profiles.active=? <br>
2、命令行：--spring.profiles.active=? 在idea右上角运行程序点开有一个Edit Configurations 填写program arguments 或者打包之后cmd命令行运行<br>
  java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=d 可以直接在测试的时候，配置传入命令行参数<br>
3、虚拟机参数<br>
  在idea右上角运行程序点开有一个Edit Configurations 填写VM options:-Dspring.profiles.active=?<br>

六、配置文件加载位置
===

springboot启动会扫描以下位置的application.properties或者application.yml文件作为Springboot的默认配置文件<br>
```java
-file:./config/
-file:./
-classpath:/config/
-classpath:/
```
优先级由高到低，高优先级的配置会覆盖低优先级的配置；SpringBoot会从这四个位置全部加载主配置文件；互补配置<br>
我们还可以通过spring.config.location来改变默认的配置文件位置：项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件得新位置；<br>
指定配置文件和默认加载的这些配置文件共同起作用形成互补配置<br>

七、外部配置加载顺序
===

SpringBoot也可以从一下位置加载配置；优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置<br>
```java
1、命令行参数
2、来自java:comp.env的JNDI属性
3、Java系统属性（System.getProperties())
4、操作系统环境变量
5、RandomValuePropertySource配置的random.*属性值

优先加载带profile，由jar包外向jar包内加载
6、jar包外部的application-{profile}.properties或application.yml（带spring.profile）配置文件
7、jar包内部的application-{profile}.properties或application.yml（带spring.profile）配置文件
8、jar包外部的application.properties或application.yml（不带spring.profile）配置文件
9、jar包内部的applicationproperties或application.yml（不带spring.profile）配置文件

10、@Configuration注解类上的@PropertySource
11、通过SpringApplication.setDefaultProperties指定的默认属性
```

八、自动配置原理
===


