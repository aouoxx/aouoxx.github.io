---
layout: post
title: httpinvoke远程服务
categories: [分布式, rpc, httpinvoke]
description: httpinvoke远程服务
keywords: 分布式, rpc, httpinvoke
---

<meta name="referrer" content="no-referrer"/>
​

### httpinvoke

```java
httpInvoker是一个新的远程调用模型,作为spring框架的一部分,能够执行基于http的远程调用，也就是说可以通过防火墙,并使用java的序列化机制在网络间传递对象。客户端可以很轻松的像调用本地对象一样调用远程服务器上的对象,要注意的一点是,服务端,客户端都是使用spring框架。

httpInvoker的整体流程：
客户端：
> 向服务器发送远程调用请求.
    远程调用信息 -> 封装为远程调用对象 ->序列化写入远程调用http请求中 ->向服务端发送
> 接收服务端返回的远程调用结果
   服务器端返回远程调用结果HTTP响应 ->反序列化为远程调用结果对象
服务端:
> 接收客户端远程调用请求
   客户端发送的远程调用HTTP请求 -> 反序列化为远程调用对象 -> 调用服务器端目标对象的目标处理方法
> 向客户端返回远程调用结果
   服务器端目标对象方法的处理结果 -> 序列化写入远程调用结果http响应中 -> 返回给客户端

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639104027140-9c0beed4-5aea-49e1-9b2b-f33eb9dc0cba.png#clientId=u0142570d-3485-4&from=paste&height=227&id=ufc41d127&margin=%5Bobject%20Object%5D&name=image.png&originHeight=454&originWidth=1102&originalType=binary&ratio=1&size=28170&status=done&style=none&taskId=u7c5ea74c-150b-412f-9247-404f5e9a293&width=551)

#### HttpInovketServiceExporter 导出 http 服务

```java
HttpInvokerServiceExporter的工作方式与HessianServiceExporter类似。
HttpInvokerServiceExporter也是一个spring的mvc控制器，它通过DispatcherServelt接收来自客户端的请求，并这这些请求转换成对实现
服务的pojo的方法调用。

@Bean
public HttpInvokerServiceExporter httpInvokerServiceExporter (HelloService helloService){
  HttpInvokerServiceExporter httpInvokerServiceExporter = new HttpInvokerServiceExporter();
  httpInvokerServiceExporter.setService(helloService);
  httpInvokerServiceExporter.setServiceInterface(HttpInvokerServiceExporter.class);
  return httpInvokerServiceExporter;
}
httpInvokerServiceExporter是一个Spring mvc的控制器,我们需要建立一个URL处理器，映射到HTTP URL到对应的服务器上
@Bean
public HandlerMapping httpInvokerMapping(){
 SimpleUrlHandlerMapping mappging = new SimpleUrlHandlerMapping();
 Properties mappings = new Properties();
 mappings.put("/*.service","httpInvokerServiceExporter");
 mapping.setMappings(mappings);
 return mapping;
}
```

#### 服务端代码

```java
'首先服务端是一个spring mvc的工程,因为httpinvoker是一个controller,通handlerMapping对DispatcherServlet转发的请求进行映射'
'spring-mvc的配置文件'
  <bean id="helloService" class="remote.HelloServiceImpl" />
    <bean id="httpInvokerServiceExporter"
           class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
        <property name="service" ref="helloService" />
        <property name="serviceInterface">
            <value>remote.HelloService</value>
        </property>
    </bean>
    <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="mappings">
           <props value-type="String" >
               <prop key="/hello.service">httpInvokerServiceExporter</prop>
           </props>
        </property>
    </bean>
'服务端代码'
public interface HelloService {
    public String sayHello(String name);
}

public class HelloServiceImpl implements HelloService {
    @Override
    public String sayHello(String name) {
        return "Hello "+name;
    }
}
```

#### 客户端调用

```java
为了把基于Http invoker的远程服务装配到我们的客户端spring应用上下文中,我们必须将HttpInvokerProxyFactoryBean 配置为一个bean来代理它
> serviceInterface属性:用来标识服务所实现的接口
> serviceUrl属性:用来标识远程服务的位置


'spring的配置文件'
<bean id="invokerFacotory" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
        <property name="serviceUrl" value="http://127.0.0.1:8090/hello.service" />
        <property name="serviceInterface">
            <value>remote.HelloService</value>
        </property>
</bean>
'主体测试代码'
public class App {
    public static void main( String[] args ){
        ApplicationContext applicationContext = new
                                ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        HelloService helloIf = (HelloService)
                                applicationContext.getBean(HttpInvokerProxyFactoryBean.class).getObject();
        System.out.println(helloIf.sayHello("ssgao"));
    }
}
输出：say ssgao
```

### httpinoke 实现方式

```java
> 基于Url映射方式,远程系统处理请求的方式同spring mvc的controller类似,
	所有的请求通过在web.xml中的org.springframework.web.servlet.DispathcerServlet统一处理,
	根据url映射去对应[servlet名称-servlet.xml]文件,查询根请求的url匹配的bean配置

>基于servlet方式,由org.springframework.web.context.support.HttpRequestHandlerServlet
	去拦截url-pattern匹配的请求,如果匹配成功
	去applicationContext中查找name与servlet-name一致的bean,完成远程方法调用
```

#### 方式一:服务端

```java
'web.xml配置DispatcherServlet'
<servlet>
   <servlet-name>application</servlet-name>
   <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
   <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
   <servlet-name>application</servlet-name>
   <url-pattern>/*</url-pattern>
</servlet-mapping>
'添spring的配置文件信息applicationcontext.xml的配置文件'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN"
"http://www.springframework.org/dtd/spring-beans-2.0.dtd">
<beans>
	<bean id="userService" class="org.felix.service.impl.UserServiceImpl" />
	<!-- 基于Url映射方式,这个配置，就是把userService接口，提供给远程调用 -->
	<bean id="httpService"  class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
        <property name="service" ref="userService"/>
        <property name="serviceInterface" value="org.felix.service.UserService"/>
    </bean>
    <!-- 远程服务的URL -->
    <bean
        class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
         <property name="mappings">
            <props>
                  <prop key="/test">httpService</prop>
            </props>
        </property>
    </bean>
</beans>
```

#### 方式二服务端

```java
'声明服务'
spring配置文件中声明一个httpinvokerserviceExporter类的bean
<bean name="helloExporter" clas="org.springframework.remoting.httpinvoker,HttpInvokerServiceExporter">
 <property name="service" ref="helloService" />
 <property name="serviceInterface">
   <value>com.ssgao.HelloService</value>
 </property>
</bean>
'服务URL关联'
在web.xml中声明一个与服务和服务名称同名的Servlet(当然这个Servlet类spring已经提供即HttpRequestHandlerServlet,这个家伙的作用就是直接把请求扔给同名的bean),然后声明servlet-mapping将其map到指定url,这样用户就可以通过这个URL访问到对应的服务。
<servlet>
   <servlet-name>helloExporter</servlet-name>
   <servlet-class>
       org.springframework.web.context.support.HttpRequestHandlerServlet
   </servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>helloExporter</servlet-name>
    <url-pattern>/remoting/HelloService</url-pattern>
</servlet-mapping>
```
