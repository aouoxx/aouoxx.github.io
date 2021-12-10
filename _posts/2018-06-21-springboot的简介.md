---
layout: post
title: springboot简介
categories: [后端服务, springboot]
description: springboot的简介
keywords: 后端服务，spring
---

<meta name="referrer" content="no-referrer"/>
​

​

#### springboot 的概述

```java
化繁为简，简化配置
备受关注，是下一代框架
微服务入门级微框架

springboot 特性理解
 > 为基于spring开发提供更快的入门体验
 > 开箱即用,没有代码生成,也无需xml配置,同时可以修改默认值来满足特定需求。
 > 提供一些大型项目中常见的非功能性特性,如嵌入式服务器,安全,指标,健康监测,外部配置等。
 > spring boot并不是对spring功能上的增强,而是提供了一种快速使用spring的方式

spring boot的优势
 > 创建独立的Spring应用程序
 > 嵌入tomcat 无需部署war文件
 > 简化Maven配置
 > 自动配置spring
 > 提供生成就绪型功能,如指标,健康检查和外部配置
 > 开箱即用,没有代码生成,也无需xml配置




我们搭建一个spring-web的服务的时候,需要进行如下几个主要步骤的设置:
 1> 配置web.xml,加载spring和springmvc
 2> 配置数据库连接,配置spring事物
 3> 配置加载配置文件的读取,开启注解
 4> 配置日志文件
配置完成之后,部署tomcat调试
 现在非常流行为微服务,如果我这个项目仅仅只是需要发送一个邮件,如果我的项目仅仅是产生一个消费积分,都需要这么折腾一遍
 springboot的优势就体现出来,我们只需要非常少的几个配置就可以迅速方便的搭建起来一套web项目或者构建一个微服务。
```

​

```java
依赖配置信息
 // springboot 父节点依赖,引入这个之后相关的引入就不需要添加version配置,spring boot会自动选择最合适的版本进行添加
 <parent>
     <groupId> org.springframework.boot </groupId>
     <artifactId> spring-boot-starter-parent </artifactId>
     <version> 1.4.1.RELEASE </version >
 </parent>

 <properties>
     <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
     // 指定一下jdk版本,这里我们使用jdk1.8,默认是1.6
     <java.version>1.8</java.version>
 </properties>
 // 添加spring-boot-starter-web依赖
 <dependencies>
     <dependency>
         <groupId> org.springframework.boot </groupId>
         <artifactId> spring-boot-starter-web </artifactId>
     </dependency>
 </dependencies>
```

##### springboot 搭建

```java
<pre name="code" class="plain"><?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-spring-boot</artifactId>
    <version>0.1.0</version>

    /**
     * 作用和功能:
     * spring-boot-starter-parent
     *  1>  在dependency中依赖的信息可以不用填写version信息
     *  2> 可以识别application.properties、application.yml这类文件
     */
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.2.7.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

```java
@RestController
public class HelloController {
    @Value("${customer.name}")
    private String name;
    @RequestMapping(value = "hello")
    public String hello(){
        return "Hello World!"+name;
    }
}
```

```java
@SpringBootApplication
public class HelloWorldApplication {
	public static void main(String[] args) {
		SpringApplication.run(HelloWorldApplication.class, args);
	}
}

spring-boot的启动
进入对应的maven项目使用'mvn spring-boot:run'
ssgao:spring_boot aouo$ mvn spring-boot:run
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building springboot_start 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------

先通过 mvn package进行打包
通过java -jar命令运行对应的jar包
ssgao:target aouo$ java -jar springboot_start-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.7.RELEASE)

这种启动方式可以添加参数
java -jar springboot_start-0.0.1-SNAPSHOT.jar --spring.profile.active=prod //启动生产环境
```

##### maven 创建 springboot

```java
ssgao:spring_boot aouo$ mvn archetype:generate -DatchetypeCatalog=internal
[INFO] Scanning for projects...
...
10: internal -> org.apache.maven.archetypes:maven-archetype-webapp (An archetype which contains a sample Maven Webapp project.)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 7: 10
Define value for property 'groupId': com.aouo
Define value for property 'artifactId': boot_starta
Define value for property 'version' 1.0-SNAPSHOT: : 1.0
Define value for property 'package' com.aouo: : main
Confirm properties configuration:
groupId: com.aouo
artifactId: boot_starta
version: 1.0
package: main
 Y: : Y
....
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 43.062 s
[INFO] Finished at: 2017-11-30T09:18:34+08:00
[INFO] Final Memory: 14M/224M
[INFO] ------------------------------------------------------------------------

"导入到idea中配置pom.xml文件"
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.aouo</groupId>
  <artifactId>boot_starta</artifactId>
  <packaging>war</packaging>
  <version>1.0</version>
  <name>boot_starta Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
  </parent>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>boot_starta</finalName>
  </build>
</project>

```

##### springboot-starter 依赖

```java
spring boot starter 依赖包以及作用

spring-boot-starter
 spring boot的核心启动类,包含了自动配置,日志和YAML

spring-boot-starter-amqp
 spring-rabbit来支持AMQP协议(advanced message queuing protocol)

spring-boot-starter-aop
 springboot支持面向切面的编程,aop,包括spring-aop和AspectJ

spring-boot-starter-artemis
 通过Apache artemis 支持JMS的api

spring-boot-starter-batch
 支持spring Batch,包括HSQLDB数据库

spring-boot-starter-cache
 支持sping的cache抽象

spring-boot-starter-cloud-connectors
 支持spring Cloud connectors 简化了在像Cloud Foundry或HeroKu这样的云平台上连接服务

spring-boot-starter-test
    支持常规的测试依赖，包括JUnit、Hamcrest、Mockito以及spring-test模块。

spring-boot-starter-data-elasticsearch
    支持ElasticSearch搜索和分析引擎，包括spring-data-elasticsearch。


spring-boot-starter-data-rest
   通过spring-data-rest-webmvc，支持通过REST暴露Spring Data数据仓库。
spring-boot-starter-jdbc
    支持JDBC数据库

spring-boot-starter-web
  支持全栈式Web开发，包括Tomcat和spring-webmvc。

spring-boot-starter-velocity
  支持Velocity模板引擎。

spring-boot-starter-thymeleaf
  支持Thymeleaf模板引擎，包括与Spring的集成。

spring-boot-starter-websocket
    支持WebSocket开发。

spring-boot-starter-actuator
    增加了面向产品上线相关的功能，比如测量和监控。

spring-boot-starter-remote-shell
  增加了远程ssh shell的支持。
  最后，Spring Boot应用启动器还有一些替换技术的启动器，具体如下：

spring-boot-starter-jetty
   引入了Jetty HTTP引擎（用于替换Tomcat. 。
spring-boot-starter-tomcat
   引入了Spring Boot默认的HTTP引擎Tomcat。
spring-boot-starter-log4j
    支持Log4J日志框架。

```

### springboot 配置文件

#### springboot 中的 profile 的使用

```java
application.properties


application-dev.yml //注意:后面需要有个空格
  server:
    port: 8081
    context-path: /ssagao
  cupsize: B
  age: 18
  content: "cupSize: ${cupsize}, age: ${age}"  //配置文件中使用使用配置

  girl:
    age: 18
    name: chenlin
    addr: hangzhou

// 将配置接入到一个类中
@Component
@ConfigurationProperties(prefix="girl")
public class Girl{
    private String name;
    private String addr;
    private int age;

    //..getter/setter..
}
```

```java
profile 是spring对不同环境提供不同配置功能的支持,可以通过激活,指定参数等方式快速切换环境。
    1> 多profile文件形式
        application-{profile}.properties
        application-dev.yml
        application.yml
            spring:
                prifiles:
                    active: dev
    2> 激活方式
        命令行   spring.profile.active=dev
        配置文件 spring.profile.active=dev
        JVM参数  -Dspring.profile.active=dev

 按Profile不同环境读取不同配置
     dev环境下的配置配置在application-dev.properties中
     prod环境下的配置配置在application-prod.properties中
 在application.properties中指定使用哪一个文件
     spring.profiles.active=dev
 我们也可以在运行的时候手动指定
     java -jar myproject.jar --spring.profile.active=dev

 //同时使用多个profile文件
     spring.profile.active=dev,host



```

#### 配置文件

```java
springboot的properties的文件

server:
  port: 8090
  context-path: /ssgao
person:
  name: ssgao
  age: 30
  addrs:
    - hangzhou
    - shangdong

------------------------List,Map,Array--------------
myprops: # 自定义的属性和值
  simple-prop: simplePropValue
  array-props: 1,2,3,4,5
  list-prop1:
    - name: abc
      value: abcValue
    - name: efg
      value: efgValue
  list-prop2:
    - config2Value1
    - config2Vavlue2
  map-props:
    key1: value1
    key2: value2
// String类型的一定需要setter来接收属性值,maps,collections和arrays不需要
public class MyProps {
    private String simpleProp;
    private String[] arrayProps;
    /** 接收prop1里面的属性值 */
    private List<Map<String, String>> listProp1 = new ArrayList<>();
    /** 接收prop2里面的属性值 */
    private List<String> listProp2 = new ArrayList<>();
    /** 接收prop1里面的属性值 */
    private Map<String, String> mapProps = new HashMap<>();
}

注意:
   1> 注意命名规则,根节点不能采用驼峰方式,否则会导致识别失败,最好全部小写,剩余的其他字段都可以采用驼峰方式命名,也可以使用"-"方式来修改
   2> 注意查看日志的提示信息

在springboot 中使用application.yml进行项目配置,会产生一个自动提示,这个自动提示可以自已定义开发的

```

```java
创建管理配置的实体类
    @ConfigurationProperties(locations="classpath:config/my-web.properties",prefix="web")
    @Component
    public class MyWebConfig{
        private String name;
        private String verison;
        //...
    }

@ConfigurationProperties注释中有两个属性
    > locations： 指定配置文件所在的位置
    > prefix: 指定配置文件键名称的前缀(这里配置文件中所有的键名都是以web.开头)
使用@Component是让该类能够在其他地方被依赖使用,即使用@Autowire注释来创建实例

springboot1.5 以上的版本@ConfigurationProperties取消了location注解后的替代方案
@ConfigurationProperties(profix="ssgao.info",location="classpath:/config/myconf.properties")
public class MyConf{
    private String email;
}
我们需要换一种方式
   > @EnableConfigurationProperties取消激活自定义的配置类(重要)
   > 在配置中采用@Component的方式注册为组件,然后使用@PropertySource来指定自定义的资源目录
   @Component //因为ConfigurationProperties取消了location属性,所以这里需要@Component注册
   @ConfigurationProperties(prefix="surpass.info")
   @PropertySource("classpath:/config/myConf.properties")
   public class MyConf{
       private String email;
   }
```

```java
使用Environment方式
    @RestController
    public class WebController{
        //这种方式依赖注入Environment来完成,在创建的成员变量private Enviroment env上加上@Autowired注解
        //即可完成依赖注入,然后使用env.getProperty("键名")即可读取出对应的值
        @Autowire
        private Environment env;

        @RequestMapping(value="index2",method=RequestMethod.GET)
        public String index2(){
            return "The Way 2:"+env.getProperty("test.msg");
        }

    }
```

### springboot 注解大全

```java
@SpringBootApplication 包含了@ComponentScan @Configuration @EnableAutoConfiguration
        其中@ComponentScan 让sprint boot 扫描到Configuration类并把它加入到程序上下文
@Configuration 等同于spring的xml配置文件
@EnableAutoConfiguration 自动配置
@ComponentScan 组件扫描,自动发现和装配一些Bean

@Component
@RestController 注解是@Controller和@ResponseBody的合集,表示这个是一个控制器Bean,并且是将函数的返回值直接填入到Http响应体中,是Rest风格的控制器
@Autowired 自动导入
@PathVariable 获取参数
@JsonBackReference 解决前台外链问题
```

```java
@EnableAutoConfiguration
    spring boot 自动配置(auto-configuration):尝试根据添加的jar依赖自动配置spring应用
    例如,如果你的classpath下存在HSQLDB,并且没有手动配置任何数据库连接bean

@ComponentScan
   表示该类自动发现扫描组件。
   如果扫描到有@Component,@Controller,@Service等这些注解的类,并注册为Bean,可以自动收集所有spring组件,包括@Configuration类。
   我们通常使用@ComponentScan注解搜索Beans,并结合@Autowired注解导入。
   如果没有配置的话,spring boot 会扫描启动类所在包下以及子包下的使用了@Service,@Repository等注解的类

@Configuration
    相当于传统的xml配置文件,如果有些第三方库需要用到xml文件,建议仍然通过@Configuration类作为项目的配置主类。
    可以使用@ImportResource注解加载xml配置文件

@Import
    用来导入其他配置类
@ImportResource
    用来加载xml配置文件
@Autowired
    自动导入依赖的bean

@Bean
    用@Bean标注方法等价于xml中配置的bean
@Value
    spring boot application.properties/application.yml配置的属性值
    @Value(value="${xx}")
    private String message;



@Inject
    等价于@Autowired,只是没有required属性
@Component
    组件标注

```

```java
@ConfigurationProperties(prefix="person")
    将配置文件中前缀为person的属性注入到执行的类中

@Component
@ConfigurationProperties(locations="classpath:mail.properties",
                         ignoreUnknownFileds=false, //告诉spring boot在有属性不能匹配到声明域的时候抛出异常
                         prefix="person" ) // 用来选择哪个属性的prefix 名字来绑定
public class Person{
    private String name;
    private boolean auth;
    private int age;
}
-----------------------------------------------------------------------------------------------------------------------
@ConfigurationProperties + @Bean注解在配置类的Bean定义方法上
   @Data
   public class Bean2{
     private String name;
   }
   @SpringBootApplication
   public class Application{

       @Bean
       // @Bean注解在该方法上定义一个Bean,这种基于方法的Bean定义不一定非要出现在@SpringBootApplication注解类中
       // 还可以出现在任何@Configuration注解了的类中
       @ConfigurationProperties(prefix="section2")
       // 使用配置文件中前缀为section2的属性的值初始化这里bean定义所产生的bean实例同名属性
       public Bean2 bean(){
           return new Bean2();
       }
       public static void main(String[] args) throws Exception{
           SpringBootApplication.run(Application.class,args);
       }
   }
-----------------------------------------------------------------------------------------------------------------------
@ConfigurationProperties注解到普通类然后通过@EnableConfigurationProperties定义为bean
    @ConfigurationProperties(prefix="section")
    @Data
    public class Bean3{
        private String name;
    }
    //使用@EnableConfigurationProperties将上述类定义为一个bean
    @SpringBootApplication
    @EnableConfigurationProperties({Bean3.class})
    //@EnableConfigurationProperties注解将会导致,注解了ConfigurationProperties类Bean3
    //使用配置文件中前缀为section3的同名属性值来填充
    public class Application{
        public static void main(String[] args) throws Exception{
            SpringApplication.run(Application.class,args);
        }
    }

```

### springboot 其他应用

#### springboot 热部署

```java
devtools 可以实现页面热部署
(即页面修改后立即生效,这个可以直接在application.properties文件中配置spring.thymeleaf.cache=false 来实现)
实现类文件热部署(类文件修改后不会立即生效),实现对属性文件的热部署
-->即:devtools会监听classpath下的文件变动,并且会立即重启应用,发生在保存时机。(因为器采用的是虚拟机机制,该项重启是很快的)
   1) base classloader(Base类加载器):加载不改变的Class,例如第三方提供的jar包
   2) restart classloader(Restart类加载器): 加载正在开发的class
-->重启很快,因为重启的时候只是加载了在开发的Class, 没有重新加载第三方的jar包
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifaceId>spring-boot-devtools</artifactId>

   </dependency>

```

**_使用 springloader 进行热部署_**

```java
方式一：
通过maven启动方式:在pom.xml文件添加依赖
    1) 配置maven-springboot-plugin (注意：依赖添加到带有main方法的模块下的pom.xml文件中)
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <dependencies>
                       <dependency>
                           <groupId>org.springframework</groupId>
                           <artifactId>springloaded</artifactId>
                           <version>1.2.6.RELEASE</version>
                       </dependency>
                   </dpendencies>
               </plugin>
           </plugins>
       </build>
      2) 切换到指定的运行目录下: mvn spring-boot:run
      3) 多模块的情况下可能出错
         the pom for xx.xxx.jar is missing
         //需要将相关依赖的jar进行按照,安装在本地的命令
         // mvn -Dmaven.test.skip -U clean install

      4) 缺点,无法直接进行debug调试,需要借助远程调试才行




方式二:
首先在idea中需要进行简单的设置,通过"run as —— Java application"
    1) file>set>Builder>Compiler
       将Make project automatically 选中
    2) shift+ctrl+alt+/ > Registry
       将compiler.automake.allow.when.app.running 选中
    3) Run/Debug Configurations > VM Options:
       -javaagent:G:/workspace/springloaded-1.2.7.RELEASE.jar -noverify (指定springloaded的jar包位置)


```
