springboot注解EnableAutoConfiguration(开启自动配置功能)，使用AutoConfigurationPackage自动配置包，用到spring底层注解@Import ，给容器中导入一个组件；<br>
导入的组件由AutoConfigurationPackages.Registrar.class<br>
将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器。<br>
@Import({AutoConfigurationImportSelector.class}) <br>
 给容器中导入组件？<br>
 AutoConfigurationImportSelector.class：导入哪些选择器<br>
 将所有需要导入的组件以全类名的方式返回，这些组件会被加入到容器中<br>
 会给容器中导入非常多的自动配置类（XXAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件，现有124个<br>
![部分配置类](https://github.com/LoveChunHua/springboot/blob/master/1575271274.png)
 有了自动配置类，就省去了我们手动编写配置注入功能组件的工作.<br>
 
使用Spring Initializer快速创建Spring Boot项目：<br>
 选择我们需要的模块；向导会联网创建Spring Boot项目；<br>
 默认生成的Spring Boot项目；<br>
 .主程序已经生成好了，我们只需要我们自己的逻辑<br>
 .resources文件夹中目录结构<br>
  。static：保存所有的静态资源；js css images;<br>
  。templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）;可以使用模板引擎（freemarker、thymeleaf);<br>
  。application.properties:Spring Boot应用配置的文件；
