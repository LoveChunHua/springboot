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
 ```java
 pets:
 - cat
 - dog
 - pig
 ```
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

配置文件能配置的属性具体参考[Springboot官方文档](https://docs.spring.io/spring-boot/docs/2.1.10.RELEASE/reference/html/common-application-properties.html)<br>

**1、自动配置原理**

```java
1)、springboot启动的时候加载主配置类，开启自动配置功能`@EnableAutoConfiguration`
2)、@EnableAutoConfiguration作用：
        利用AutoConfigurationImportSelector给容器中导入一些组件
        可以查看selectImports看导入哪些内容；该方法调用getAutoConfigurationEntry方法获取List<String> configurations =            this.getCandidateConfigurations(annotationMetadata, attributes);接下来调用
        SpringFactoriesLoader.loadFactoryNames
        扫描所有jar包类路径下 classLoader.getResources("META-INF/spring.factories")
        把扫描到的这些文件的内容包装成properties对象
        从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加到容器中

**将类路径下META-INF/spring.factories里面配置的所有EnableAutoConfiguration的值加入到了容器中**

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration

每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置
3)、每一个自动配置类进行自动配置功能    
4)、以**httpEncodingAutoConfiguration**为例解释自动配置原理
            @Configuration(   //表示这是一个配置类，以前编写的配置文件一样
            proxyBeanMethods = false
        )
        @EnableConfigurationProperties({HttpProperties.class})//启动指定类的ConfigurationProperties功能；将配置文件中对应的值和httpEncodingProperties绑定起来；并把httpEncodingProperties加入到ioc容器中
        @ConditionalOnWebApplication(//Spring底层@conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；判断当前应用是否是web应用，如果是，当前配置类生效
            type = Type.SERVLET
        )
        @ConditionalOnClass({CharacterEncodingFilter.class})//判断当前项目有没有这个类； CharacterEncodingFilter是SpringMVC中进行乱码解决的过滤器
        @ConditionalOnProperty(//判断配置文件中是否存在某个配置 spring.http.encoding;如果不存在，判断也是成立的
        //即使我哦们配置文件中不配置spring.http.encoding.enabled=true，也是默认生效的；
            prefix = "spring.http.encoding",
            value = {"enabled"},
            matchIfMissing = true
        )
    
    public class HttpEncodingAutoConfiguration {
    //他已经和springboot的配置文件映射了
    private final Encoding properties;
    //只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(HttpProperties properties) {
        this.properties = properties.getEncoding();
    }
    @Bean//给容器中添加一个组件，这个组件的某些值要从properties中获取
    @ConditionalOnMissingBean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
        return filter;
    }
    
根据当前不同的条件判断，决定这个配置类是否生效？
一旦这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；
5)、所有在配置文件中能配置的属性都是在xxxxProperties类中封装着；配置文件能配置什么就可以参照某个能对应的这个属性类

        @ConfigurationProperties(  //从配置文件中获取指定的值和bean的属性进行绑定
    prefix = "spring.http"
)
public class HttpProperties {
    private boolean logRequestDetails;
    private final HttpProperties.Encoding encoding = new HttpProperties.Encoding();

精髓：
    1）、SpringBoot启动会加载大量的自动配置类
    2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类
    3）、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）
    4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以再配置文件中指定这些属性的值；
    
xxxxAutoConfiguration：自动配置类；
给容器中添加组件：
xxxxProperties:封装配置文件中相关属性

```

**2、细节**

       
    @Conditional派生注解（Spring注解版原生的@Conditional作用）<br>
    作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效<br>
    
    ！[Conditional扩展](https://github.com/LoveChunHua/springboot/blob/master/157530027.png)
    
    自动配置类必须在一定的条件下才能生效；<br>
    我们怎么知道哪些自动配置类生效？我们可以通过在application.properties中启用debug=true属性来让控制台打印自动配置报告
    
    
