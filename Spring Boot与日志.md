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

