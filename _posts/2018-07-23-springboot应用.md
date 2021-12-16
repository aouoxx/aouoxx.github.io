---
layout: post
title: springboot应用
categories: [后端服务, springboot]
description: springboot的应用
keywords: 后端服务，spring
---

<meta name="referrer" content="no-referrer"/>

### springboot 日志配置

```java
默认情况下,Spring boot会用Logback来记录日志,并用INFO级别输出到控制台。

2018-07-08 17:52:11.006 DEBUG 6324 --- [main] o.s.b.c.c.ConfigFileApplicationListener  : Skipped config file
                                              'file:./config/application.properties' resource not found
2018-07-08 17:52:11.006 DEBUG 6324 --- [ main] o.s.b.c.c.ConfigFileApplicationListener  : Skipped config file
                                               'file:./config/application.xml' resource not found

**) 从上面可以看出,日志输出内容元素具体如下:
    1>  时间日期,精确到毫秒
    2>  日志级别:ERROR,WARN,INFO,DEBUG,TRACE
    3>  进程ID
    4>  分割符:---表示实际日志的开始
    5>  进程名[方括号括起来,可能会截断控制台输出]
    6>  Logger名:通常使用源代码的类名
    7>  日志内容

**) springboot的日志依赖spring-boot-starter-logging
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactIf>spring-boot-starter-logging</artifactId>
    </dependency>
    实际开发中我们不需要直接添加该依赖,spring-boot-starter中包含了spring-boot-starter-logging
    该依赖的内容就是springboot默认的日志框架logback。
**) 日志级别的解析:
    日志级别从低到高分别为 TRACE<DEBUG<INFO<WARN<ERROR<FATAL
    如果设置为WARN,则低于WARN的信息都不会输出
    springboot中默认配置ERROR,WARN,INFO级别的日志输出到控制台
    我们可以通过启动应用程序--debug 表示来启动"调试"模式(开发的时候推荐开启),以下两种方式皆可
    1> 在运行命令后加入--debug表示,如:"java -jar springtest.jar --debug"
    2> 在application.properties中配置debug=true,该属性置为true的时候,核心Logger(包含嵌入式容器,hibernate,spring)
       会输出很多内容,但是我们自己的应用日志并不会输出为DEBUG级别


**) 使用springboot的配置文件进行简单的场景配置
    如配置保存路径,日志格式,复杂的场景(区分info,error等日志,每天产生一个日志文件,主要自定义配置)
    logging:
       pattern:
          console: "%d - %msg%n"          //格式化,只输出日期和内容
       path: /var/log/tomcat/             //日志输出路径,
       file: /var/log/tomcat/sell.log     //指定输出文件名,这里也可以是相对路径 logging.file=my.log
       1> 如果只配置logging.file 会在项目的当前路径下生成一个xx.log日志文件
       2> 如果只配置logging.path,会在/var/log文件夹下生成一个日志文件为spring.log
       3> 注意,两者不能同时使用,如果同时使用,则只有logging.file生效
          默认情况下,日志文件的大小达到10MB的时候会切分一次,产生新的日志文件,默认级别为:ERROR,WARN,INFO

    配置级别控制,所有支持的日志记录系统都可以在spring环境中设置记录级别(例如application.properties中)
    格式为:'logging.level.* =LEVEL'
       logging.level: 日志级别控制前缀,*为包名或Logger名
       LEVEL: 选项TRACE,DEBUG,INFO,WARN,ERROR,FATAL,OFF
       logging.level.com.dudu=DEBUG  >>com.dudu包下所有class以DEBUG级别输出
       logging.level.root=WARN  >>root日志以WARN级别输出
       ----------------------------------------------------------------------------------------
       logging:
          pattern:
            console: "%d - %msg%n"
          file: /var/log/tomcat/sell.log
          level: com.aouo.LoggerTest: debug  //指定包名或者类名,日志级别


```

```java
spring中可以配置logging信息

=========== 日志配置·简易（spring boot已经集成logback日志）=========
#controller层日志 WARN级别输出
#logging.level.com.liyan.controller=WARN
#mapper层 sql日志 DEBUG级别输出
#logging.level.com.liyan.mapper=DEBUG
#logging.file=logs/spring-boot-logging.log
#logging.pattern.console=%d{yyyy/MM/dd-HH:mm:ss} [%thread] %-5level %logger- %msg%n
#logging.pattern.file=%d{yyyy/MM/dd-HH:mm} [%thread] %-5level %logger- %msg%n
#打印运行时sql语句到控制台
#spring.jpa.show-sql=true

通过外部文件引入标准的logback日志配置
==================== 日志配合·标准  ============================
#logging.config=classpath:logback-boot.xml

```

#### 自定义日志配置

```java
**) 根据不同的日志系统,我们可以按如下规则配置文件名,就可以被正确加载
   > Logback: logback-spring.xml,logback-spring.groovy,logback.xml,logback.groovy
   >   Log4j: log4j-spring.properties, log4j-spring.xml,log4j.properties,log4j.xml
   >  Log4j2: log4j2-spring.xml,log4j2.xml
   > JDK: logging.properties
    springboot官方推荐优先使用带有-spring的文件名作为你的日志配置(如使用logback-spring.xml,而不是logback.xml)
    命名为logback-spring.xml的日志配置文件,springboot可以为它添加一些springboot特有的配置项
    默认的命名规则,并且放在src/main/resources下面集合
    如果我们已经掌握了日志配置,又不想使用logback.xml作为logback配置的名称,
    application.yml可以通过logging.config属性指定自定义的名字:
        logging.config=classpath:logging-config.xml

**) 如果你想针对不同运行时profile使用不同的日志配置
     我们可以在logback-spring.xml中使用springProfile配置
     spring:
         profile:
            active: dev
     <configuration>
         ....
         <!-- 测试环境+开发环境,多个使用逗号隔开 -->
         <springProfile name="test,dev">
             <logger name="com.example.demo.controller" level="DEBUG" additivity="false">
                 <appender-ref ref="consolelog" />
             </logger>
         </springProfile>
         <!-- 生成环境 -->
         <springProfile name="prod">
             <logger name="com.example.demo.controller" level="DEBUG" additivity="false">
                 <appender-ref ref="consolelog" />
             </logger>
         </springProfile>
     </configuration>




```

### springboot 数据源配置

```java
springboot 默认支持4中数据源类型,定义在org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration中分别是:
 org.apache.tomcat.jdbc.pool.DataSource
 com.zaxxer.hikari.HikariDataSource
 org.apahce.commons.dbcp.BasicDataSource
 org.apache.commons.dbcp2.BasicDataSource
对于这4种数据源,当classpath下有相应的类存在的时候,springboot会通过自动配置为其生成DataSource Bean,DataSource Bean默认只会生成一个。
四种数据源类型的生效先后顺序如下：Tomcat->Hikari->Dbcp->Dbcp2

添加依赖于配置
在springboot 使用JDBC的时候,可以直接添加官方提供的spring-boot-start-jdbc或spring-boot-start-jpa依赖。
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.1.RELEASE</version>
</parent>
<dependencies>
    <!-- 添加MySQL依赖 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!-- 添加JDBC依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
</dependencies>

在核心配置application.properties或application.yml文件中添加数据源相关配置。
application.properties文件中添加如下配置
    spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
    spring.datasource.driverClassName=com.mysql.jdbc.Driver
    spring.datasource.username=root
    spring.datasource.password=123456
 使用application.yml 文件中添加如下配置:
    spring:
        datasource:
            url:jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncodeing=utf-8
            driverClassName:com.mysql.jdbc.Driver
            username:root
            passowrd:123456

 按照上述配置启动时,springboot 会读取核心配置中的配置并自动装配一个Tomcat DataSource,如果要切换成其他默认支持的数据源类型,有如下两种方式:
    方式一> 排除其他的数据源依赖,进保留需要的数据源依赖
    方式二> 通过在核心配置中配置spring.datasource.type 属性执行数据源的类型。

 spring 默认支持的四种数据源的配置
     // 添加Tomcat-JDBC依赖
     <dependency>
         <groupId>org.apache.tomcat</groupId>
         <artifactId>tomcat-jdbc</artifactId>
     </dependency>
     // 添加HikariCP 依赖
     <dependency>
         <groupId>com.zaxxer</groupId>
         <artifactId>HikeriCP</artifactId>
     </dependency>
     // 添加DBCP依赖
     <dependency>
         <groupId>commons-dbcp</groupId>
         <artifactId>commons-dbcp</artifactId>
     </dependency>
    // 添加DBCP2依赖
     <dependency>
         <groupId>org.apache.commons</groupId>
         <artifactId>commons-dbcp2</artifactId>
     </dependency>


```

```java
当我们引入spring-boot-start-jdbc依赖的时候,其实里面已经包含了Tomcat-jdbc的依赖,如果我们想切换其他的数据源类型
需要先将Tomcat-JDBC依赖排除,再添加上需要的数据源的依赖,以使用HikariCP数据源为例,配置如下:
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
        <exclusions>
            <!--排除Tomcat-JDBC依赖-->
            <exclusion>
                <groupId>org.apache.tomcat</groupId>
                <artifactId>tomcat-jdbc</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
   <!-- 添加HikariCP依赖 -->
   <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
    </dependency>
```

```java
通过在核心配置中,添加spring.datasource.type=[数据源类型]来指定数据源的类型
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
# spring.datasource.type=org.apache.tomcat.jdbc.pool.DataSource
# spring.datasource.type=org.apache.commons.dbcp.BasicDataSource
# spring.datasource.type=org.apache.commons.dbcp2.BasicDataSource
```

```java
第三方数据源
    如果不想使用springboot默认支持的4种数据源,还可以选择使用其他第三方的数据源,如Durid,c3p0等
    下面我们以Durid数据源为例:
    添加依赖与配置,在pom文件中引入第三方数据源依赖
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
    </parent>
    <dependencies>
        //添加mySql依赖
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        //添加JDBC依赖
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        //添加Druid依赖
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.6</version>
        </dependency>
    </dependencies>
    //核心配置文件中添加数据源相关配置与使用默认数据源时的配置相同

定义数据源
    使用注解@Bean创建一个DataSource Bean并将其纳入到spring容器中进行管理即可。
    @Configuration
    public class DataSourceConfig{
        @Autowired
        private Environment env;

        @Bean
        public DataSource getDataSource(){
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setUrl(env.getProperty("spring.datasource.url"));
            dataSource.setUsername(env.getProperty("spring.datasource.username"));
            dataSource.setPassword(env.getProperty("spring.datasource.password"));
            return dataSource;
        }
    }
    -------------------------------------------------------------------------------------------------
    @Configuration
    @ConfigurationProperties(prefix="spring.datasource")
    public class DataSourceConfig{
        private String url;
        private String username;
        private String password;
        @Bean
        public DataSource getDataSource(){
            DuridDataSource datasource = new DuridDataSource();
            datasource.setUrl(url);
            datasource.setUsername(username); //用户名
            datasource.setPassword(password); //密码
        }
        //...getter/setter....
    }

```

### springboot 与 mybatis

```java
<dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>

<dependency>
      <groupId>MySQL</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>6.0.4</version>
</dependency>
```

```java
springboot 在启动的时候先执行完成mybatis-starter中的MybatisAutoConfiguration,这时在spring容器中sqlSessionFactory已经注册好了,
然后

```

### springboot 定时任务

```java
springboot中使用定时任务很简单.
    我们在启动类中加入@EnableScheduling来开启定时任务
    @EnableScheduling
    @SpringBootApplication
    public class RuoundApplication{
        public static void main(String[] args){
            SpringApplication.run(RoundApplication.class,args);
        }
    }
```

```java
@Scheduled
    我们之前使用crom表达式来指定每分钟启动一次定时器,除此之外我们还可以使用fixedRate,fixedDelay等来作为时间配置.
    @Schedule(fixedRate=5000)
    public void timerToZpp{
        System.out.println("---timerToZpp");
    }
    @Schedule(fixedDelay=5000)
    public void timerToZpp{
        System.out.println("---timerToZpp");
    }
    @Schedule(initialDelay=50000,fixedRate=5000)
    public void timerToZpp{
        System.out.println("---timerToZpp");
    }

 >>> cron: cron表达式,指定任务在特定时间执行
     @Scheduled(cron = "${jobs.cron}") 通过配置文件,配置运行参数

 >>> fixedRate:上一次启动时间点之后,X秒执行一次,单位毫秒
 >>> fixedRateString: 与fixedRateString 含义一样,只是参数类型变为String
 >>> fixedDelay:上一次结束时间点之后,每X秒执行一次
 >>> fixedDelayString: 与fixedDelay含义一样,只是参数类型变为String
 >>> initialDelay:第一次延迟X秒执行,之后按照fixedRate的规则每X秒执行,单位ms
 >>> intitalDelayString: 与initialDelay含义一样,只是将参数类型变为String


```

​

```java
>>>简单实例
    @SpringBootApplication
    @Import(Configuration.class)
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class,args);
        }
    }
    //配置信息
    @org.springframework.context.annotation.Configuration
    @EnableScheduling
    public class Configuration {
    }
    //具体执行的任务
    @Component
    public class MyTask  {
        private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        @Scheduled(cron = "0/5 * * * * *")
        public void run1() {
            System.out.println("----ssgao----"+format.format(new Date()));
        }
        @Scheduled(cron = "0/2 * * * * *")
        public void run2() {
            System.out.println("----chenlin----"+format.format(new Date()));
        }
    }

Thread[pool-1-thread-1,5,main]----chenlin----2018-07-26 07:21:02
Thread[pool-1-thread-1,5,main]----chenlin----2018-07-26 07:21:04
Thread[pool-1-thread-1,5,main]----ssgao----2018-07-26 07:21:05


上面的结果可以看出,两个定时使用的是同一个线程,因为SpringTask默认是单线程的,在实例开发中我们不希望所有的任务都在一个线程中.
    需要使用多线程的方式来运行,需要给SpringTask提供一个多线的TashScheduler即可,spring已经有默认的实现。
    @Configuration
    @EnableScheduling
    public class Configure {
        @Bean
        public TaskScheduler taskScheduler(){
            ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
            // 这里线程池的数目最好等于需要运行的任务数,大于的话,可能一个任务被分配很多个线程执行
            scheduler.setPoolSize(2);
            scheduler.setThreadNamePrefix("spring-task-demo");
            return scheduler;
       }
    }

```

### springboot 拦截器集成

```java
使用注解@Configuration 配置拦截器
继承WebMvcConfigurerAdapter


@Configuration
public class WebMvcConfigurer extends WebMvcConfigurerAdapter{
    // 拦截器按照顺序执行
    @Override
    public void addInterceptors(InterceptorRegistry registry){
        // 设置拦截的路径信息
        registry.addInterceptor(new OneItercepter()).addPathPatterns("/one/**");
        super.addInterceptors(registry);
    }
}


public class OneInterceptor implements HandlerInterceptor{
    /** 在请求处理之前调用(Controller)方法调用之前 */
    @Override
    public boolean preHandler(HttpServletRequest request,
                HttpServletResponse response,Object object) throws Exception{
        System.out.println("被one拦截,放行...");
        return true;
        // returnErrorResponse(response,IMoocJsonResult.errorMsg("被one拦截..."));
    }
    /** 请求处理之后进行调用,但是在视图渲染之前(Controller方法调用之后)*/
    @Override
    public void postHandle(HttpServletRequst request,HttpServletResponse response,
                           Object obj,ModelAndView mv) throws Exception{
        //TODO Auto-generated method stub
    }
    /**
     * 在整个请求结束之后被调用,也就是在DispatcherServlet 渲染了对应视图之后执行
     * (主要用于进行资源清理工作)
     */
    @Override
    public void afterCompletion(HttpServletRequest request,HttpServletResponse response,
                           Object obj,Exception ex) throws Exception{
        //TODO Auto-generated method stub
    }

    public void returnErrorResponse(HttpServletResponse response,IMoocJSONResult result){
        response.setChararterEncoding("utf-8");
        response.setContentType("text/json");
        try(OutputStream out = resonse.getOutputStream()){
            out.write(JsonUtils.objectToJson(result).getBytes("utf-8"));
            out.flush();
        }catch(Exception e){
            e.printStack()
        }
    }

}



```

### springboot 与 tomcat 部署

```java
使用外部的tomcat部署方式
1> pom.xml文件,dependencies中添加
   将spring-boot-starter-tomcat是spring boot默认就会配置的,即上面说到的内嵌tomcat,将其设置为provided实在打包时会将该包(依赖)排除
   因为是要放到独立的tomcat中运行,spring boot内嵌的tomcat是不需要用到的.
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>

2> 在pom.xml文件中project下面packaging标签改为
    <packaging>war</packaging>
3> 将项目中的启动类Application.java基础SpringBootServletInitializer并重写configure方法
    // 若要部署到外部servlet容器,需要继承SpringBootServletInitializer并重写configure()
    @SpringBootApplication
    public class Application extends SpringBootServletInitializer{
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder application){
            // 设置启动类,用于独立tomcat运行的入口
            return application.source(Application.class);
        }

        public static void main(String[] args) throws Exception{
            SpringApplication.run(Application.class,args);
        }
    }

这样通过上面三步,就可以打成war包,并且部署到tomcat中了.
    需要注意的是这样部署的request url需要在端口后加上项目名称,才能正常访问.
    spring-boot更加强大的一点就是,即使项目是以上配置,依然可以使用内嵌的tomcat来吊事,启动命名和之前一样为mvn spring-boot:run

如果需要在springboot中加上request前缀,需要在application.properties中添加server.contextPath=/prefix/即可,其中prefix为前缀名
前缀名会在war包中失效,取而代之的是war包名称,如果war包名称和prefix相同的话,那么调试环境和正式部署环境就是一个request地址了.


```

### springboot 与 dubbo 集成
