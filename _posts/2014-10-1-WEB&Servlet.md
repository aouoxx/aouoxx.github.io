---
layout: post
title: web基础知识
categories: [web, servlet]
description: web基础知识的介绍,servlet的相关信息
keywords: web, servlet
---

<meta name="referrer" content="no-referrer"/>
​

ccc

### welcome-file-list

```java
当用户在浏览中输入的UTL不包含某个servlet 名或JSP页面的时候，welecome-file-list元素可以指定默认的文件。一般用于默认登陆页面

  <welcome-file-list >
    <welcome-file >index.html </welcome-file>
    <welcome-file >index.htm </welcome-file>
    <welcome-file >index.jsp </welcome-file>
    <welcome-file >default.html </welcome-file>
    <welcome-file >default.htm </welcome-file>
    <welcome-file >default.jsp </welcome-file>
  </welcome-file-list >

welcome-file-list的工作原理是：按照welcome-file的 <welcome-file>一个一个的去检查是否web目录下面有这个文件，如果不存在，继续向下查找。
若存在跳转到对应的页面。
welcome-file不可以是一个直接访问的action（直接跳转到对应的urlpattern），解决的方法就是通过url跳转或jsp的forward跳转到action地址就可以了。

<?xml version= "1.0" encoding ="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
  xsi:schemaLocation= "http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id= "WebApp_ID" version ="3.0">
  <display-name >Servlet</display-name>
  <welcome-file-list >
    <welcome-file >index.html </welcome-file>
    <welcome-file >index.htm </welcome-file>
    <welcome-file >index.jsp </welcome-file>
    <welcome-file >default.html </welcome-file>
    <welcome-file >default.htm </welcome-file>
    <welcome-file >default.jsp </welcome-file>
  </welcome-file-list >
</ web-app>




```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635735274195-54db8535-d745-4032-85e7-b1d34f41de70.png#clientId=uaafaf5ae-53bd-4&from=paste&height=173&id=udaae8d92&margin=%5Bobject%20Object%5D&name=image.png&originHeight=267&originWidth=1189&originalType=binary&ratio=1&size=23120&status=done&style=none&taskId=u270ad6e8-7350-4987-925d-53e1f46ece7&width=770.5)

> _访问页面：http://127.0.0.1/Servlet/_

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635735307992-8b33862c-168b-48dd-81f5-0e13518ab9c4.png#clientId=uaafaf5ae-53bd-4&from=paste&height=136&id=ufc9eaf2f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=207&originWidth=1180&originalType=binary&ratio=1&size=16065&status=done&style=none&taskId=u554bbb8a-bdff-4431-a48e-32141ba2da1&width=776)
​

​

### errorpage 页面跳转

```java
首先需要在Web.xml文件中配置如下标签：
  <error-page >
       <error-code >404 </error-code>
       <location >/error.html </location>
  </error-page >

error-code 是错误代码,location 是转向页面。
如果这个配置成功，当服务器出现这个错误代码的时候就会跳转到location这个页面。
location可以是html文件也可以是jsp页面。

如果Tomcat配置完成后：
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635736020281-e59b8ec1-2d2f-4ec1-ae2a-0aeb21e5f387.png#clientId=uaafaf5ae-53bd-4&from=paste&height=449&id=u79432b1f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=727&originWidth=1336&originalType=binary&ratio=1&size=151073&status=done&style=none&taskId=ub23d8ff9-4500-4bd4-b85e-2e95b90b424&width=825)

> _当访问在当前该 web 服务下一个不存在的页面时,就会跳转到对应的错误处理页面,如果访问的不是当前服务下的页面的时候是不会跳转的( 这一点对 form 表单提交的时候尤其需要注意不要顺便定义)_

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635736082250-935644eb-86df-4bff-966f-13250388375e.png#clientId=uaafaf5ae-53bd-4&from=paste&height=185&id=u8ddc2591&margin=%5Bobject%20Object%5D&name=image.png&originHeight=275&originWidth=1193&originalType=binary&ratio=1&size=23991&status=done&style=none&taskId=ue29a2dd7-77b6-43e6-b079-9a67753e456&width=802.5)

> _访问当前服务下不存在的页面_

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635736106987-ffdc0e09-9600-414c-883b-f6f322314795.png#clientId=uaafaf5ae-53bd-4&from=paste&height=128&id=uaa994c45&margin=%5Bobject%20Object%5D&name=image.png&originHeight=188&originWidth=1181&originalType=binary&ratio=1&size=21298&status=done&style=none&taskId=ua1bfbef2-4ed2-47c6-b64e-d0f190d9b37&width=805.5)

> _访问的不是当前服务下的页面：http://127.0.0.1/accociat_

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635736134293-f20da321-fc92-40fd-b8d6-b2ddcad05048.png#clientId=uaafaf5ae-53bd-4&from=paste&height=137&id=uaae267d4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=225&originWidth=1309&originalType=binary&ratio=1&size=24733&status=done&style=none&taskId=u085f9caf-7493-4de7-9f29-0658f34c7ea&width=797.5)
​

### urlpattern 的匹配

#### urlpattern 的匹配原则

```java
1） 完全匹配
          以“/”开头，以字母（非“*”）结束 如：<url-pattern>/test/list.do</url-pattern>
2) 目录匹配
          以“/”开头且以“/*”结尾 如：<url-pattern>/test/*</url-pattern>
3) 扩展名匹配
          以* 开头，以扩展名结束  如<url-pattern>*.do</url-pattern>
4) 用“/”来表明对应的Servlet为应用默认的Servlet
    这种情况下Serlvet路径是请求的URL去掉上下文路径并且路径信息为NULL

匹配过程：
     当一个请求发送到Servlet容器的时候，
     容器先将请求的URL减去当前应用上下文的路径作为Servlet的映射Url,
     例如我访问的是http://localhost/test/aaa.html我的应用上下就是test（Web的程序名称）
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635735410684-7a122e67-0cd7-438e-b9b9-7f1bdb029a05.png#clientId=uaafaf5ae-53bd-4&from=paste&height=137&id=u41a4117d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=203&originWidth=1174&originalType=binary&ratio=1&size=20040&status=done&style=none&taskId=u5862bcc5-76c8-4e7f-98e0-f0df33d1744&width=793)

```java
 Tomcat 可以通过Service>>Model进行上下文的配置，例如我将我的应用程序名设为Servlet，对应的path为“/”，我的应用上下文就是空的，如果将path设置为Serlvet我的应用上下文为Servlet，Path的
     设置为影响页面中Servlet urlPattern的验证路径
     path=/   @WebServlet (name="annociation",urlPatterns= "/annociation/*" )  Servlet中对应的路径应该为http://127.0.0.1/annociation/*
     path=/Servlet  @WebServlet (name= "annociation",urlPatterns= "/annociation/*" )  Servlet中对应的路径应该为http://127.0.0.1/Servlet/annociation/*

如下form表单的提交路径为 http://127.0.0.1:8081/annocation?test=xx; （ip+端口号+urlpattern）
     如果将Servlet对应的Tomcat中将path调整，但是urlPattern不变，form表单提交的时候需要action=path/+提交路径
A)
     1) path=/
     2）页面提交路径：
          <!DOCTYPE html>
          <html>
               <head>
               <meta charset= "UTF-8">
               <title> Insert title here</title >
               </head>
          <body>
               <form action= "/annociation" method ="get">
                  <input  type= "text"  name ="test"/>
                          <input type= "submit" value ="提交">
               </form>
          </body>
          </html>
     3) Servlet UrlPattern:
           @WebServlet (name= "annociation",urlPatterns= "/annociation/*" )
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635735459683-f62b5ba4-cf7c-4d09-b524-c6a685d43ce5.png#clientId=uaafaf5ae-53bd-4&from=paste&height=120&id=ufa0a437c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=203&originWidth=1355&originalType=binary&ratio=1&size=20248&status=done&style=none&taskId=uc6006a3c-bde8-40d7-bd65-3887735f580&width=799.5)

```java
B)
  1) Path=/Servlet
  2) 页面提交路径：
     <!DOCTYPE html>
     <html>
     <head>
        <meta charset= "UTF-8">
        <title> Insert title here</title >
        </head>
        <body>
             <form action= "/Servlet/annociation" method ="get">
                <input  type= "text"  name ="test"/>
                <input type= "submit" value ="提交">
             </form>
             </body>
      </html>
   3) Servlet UrlPattern:
     @WebServlet (name="annociation",urlPatterns= "/annociation/*" )

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635735517470-37ab7aed-b0b6-4066-bff3-8b1e784411d4.png#clientId=uaafaf5ae-53bd-4&from=paste&height=122&id=uf71b852b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=207&originWidth=1352&originalType=binary&ratio=1&size=20356&status=done&style=none&taskId=u2b3d847d-79e5-497b-845c-3be1a0d37d3&width=795)

```java
由于客户端是通过URL地址访问web服务器中的资源，所以Servlet程序若想被外界访问，必须把servlet程序映射到一个URL地址上，
 这个工作，在web.xml文件中使用<servlet>元素和<servlet-mapping>元素完成。

一个<Servlet-mapping>元素用于映射一个已经注册的Servlet的一个对外访问的路径，
	它包含两个子元素，<servlet-name>和<url-pattern>分别用于指定servlet的注册名称和


servlet的对外访问路径。
	同一个Serlvet可以被映射到多个URL上，
    即多个<servlet-mapping>元素的<servlet-name>子元素的设置值可以是同一个Servlet注册名
在Servlet映射到的URL中也可以使用*通配符，
	但是只能有两种固定的格式：
    		一种格式是"*.扩展名"
    		一种格式是以正斜杠（/）开头并以"/*"结尾。
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635735901686-c132e699-be8d-4f41-beef-4550796dcaec.png#clientId=uaafaf5ae-53bd-4&from=paste&height=151&id=u60c88dc4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=302&originWidth=1752&originalType=binary&ratio=1&size=154278&status=done&style=none&taskId=u5397575e-1828-42c7-b98f-b370228cf7e&width=876)

> _\*匹配任意的字符，所以可以使用任意的 URL 去访问 ServletDemo01 这个 Servlet_

**_映射关系的对应_**​
![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635735833638-147f0ad0-eb00-4430-94c4-45fae2c2016b.png#clientId=uaafaf5ae-53bd-4&from=paste&height=226&id=u0b24d2e7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=452&originWidth=1742&originalType=binary&ratio=1&size=311270&status=done&style=none&taskId=u5135fa1b-2d75-4c45-8d18-9c8eb9e80f0&width=871)

```java
缺省的Servlet，UrlPattern "/"
    如果某个Serlvet的映射路径仅仅是一个正斜杠(/),那么这个Servlet就成为当前web应用程序的缺省,Servlet。凡是在web.xml文件中找不到匹配的<Servlet-mapping>元素的URL。
    它们的访问请求都将交给缺省的Servlet进行处理。也就是说缺省的Servlet用于处理其他Servlet都不处理的访问请求。
<servlet>
     <servlet-name>ServletDemo2</servlet-name>
     <servlet-class>gacl.servlet.study.ServletDemo2</servlet-class>
     <load-on-startup>1</load-on-startup>
/servlet>

<!-- 将ServletDemo2配置成缺省Servlet -->
<servlet-mapping>
  <servlet-name>ServletDemo2</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

####

#### UrlPattern 的使用注意

```java
Web中URL地址的推荐写法
     在JavaWeb开发中，主要是写URL地址，那么建议最好是以“/”开头，也就是使用绝对路径的方法，那么这个 “/”到底代表什么呢？

可以使用如下方式来记忆:
  如果是"/"是给服务器的用的，则代表当前的WEB工程，如果“/”是给浏览器用的，则代表webapps目录

"/”代表当前web工程的常见应用场景
```

```java
1）ServletContext.getRealPath("/download/1.JPG")是用来获取服务器上的某个资源，那么这个"/"就是给服务器用的。
	"/"此时代表的就是整个WEB工程 ServletContext.getRealPath("/download/1.JPG")表示的就是读取web工程下的download文件夹中的1.JPG这个资源
     * 只要明白了"/"代表的具体含义，就可以很快写出要访问的web资源的绝对路径
	this.getServeltContext().getRealPath("/download/1.jpg")'
2）在服务器端forword到其他页面
	forword
	客户端请求某个web资源，服务器跳转到另外一个web资源，这个forword也是给服务器用的
	那个这个“/”就是给服务器用的, 所以此时"/"代表的就是web工程
	this.getServletContext().getRequestDispatcher("/index.jsp").forword(request,response)
3）使用include指令或<jsp：include>标签引入页面
	<%@include file="/jspfragments/head.jspf" %>
	<jsp:include page="/jspfragments/demo.jsp" />
```

> **_"/" 代表 webapps 目录的常见应用场景_**​

```java
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
 <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
 <html>
 <head>
     <title>"/"代表webapps目录的常见应用场景</title>
     <%--使用绝对路径的方式引用js脚本--%>
     <script type="text/javascript" src="${pageContext.request.contextPath}/js/index.js"></script>
     <%--${pageContext.request.contextPath}与request.getContextPath()写法是得到的效果是一样的--%>
     <script type="text/javascript" src="<%=request.getContextPath()%>/js/login.js"></script>
     <%--使用绝对路径的方式引用css样式--%>
      <link rel="stylesheet" href="${pageContext.request.contextPath}/css/index.css" type="text/css"/>
</head>
<body>
       <%--form表单提交--%>
       <form action="${pageContext.request.contextPath}/servlet/CheckServlet" method="post">
            <input type="submit" value="提交">
          </form>
        <%--超链接跳转页面--%>
        <a href="${pageContext.request.contextPath}/index.jsp">跳转到首页</a>
</body>
</html>

```

> **_${pageContext.request.contextPath}的效果等同于 request.getContextPath()，两者获取到的都是"/项目名称"_**​

```java
1） 使用sendRedirect实现请求重定向
response.sendRedirect("/JavaWeb/index.jsp")
服务器发送一个URL地址给浏览器，浏览器拿到地址之后再去请求服务器，所以这个"/"是给浏览器使用的。此时"/"代表的是webapps目录
”/JavaWeb/index.jsp“这个地址指的就是"webapps\JavaWeb\index.jsp"

response.sendRedirect("/项目名称/文件夹目录/页面")
这种写法是将项目名称写死在程序中的做法，不灵活，项目名称变了，此时就得改程序所以推荐的写法：
rfesponse.sendRedirect(request.getContextPath()+"/index.jsp");

request.getContextPath()获取的内容就是”JavaWeb“,这样就比较灵活了，使用request.getContext（）代替项目名称，推荐使用这种方式，灵活方便

2)使用超链接进行跳转
<a href="/JavaWeb/index.jsp" >跳转到首页 </a>
这是客户端浏览器使用超链接跳转，这个 "/" 是给浏览器使用的，此时"/"代表的就是webapps目录
使用超链接访问wen资源，绝对路径的写法推荐使用下面的写法改进：
<a href="${pageContext.request.contextPath}/index.jsp">跳转到首页</a>
这样做的好处是，避免在路径中出现项目的名称


3) Form表单的提交
<form action="/JavaWeb/CheckServlet" method="post">
     <input type="submit" value="提交" />
</form>

<form action="${pageContext.request.contextPath}/servlet/CheckServlet" method="post">
     <input type="submit" value="提交">
</form>
```

ccc

### servlet 相关介绍

#### servlet 缓存技术

```java
HTTP协议中关于缓存的关键字包括 Cache-Control  Pragma  last-Modified  Expires

HTTP1.0 中是通过Param 控制页面缓存的，可以设置 Pragma 或no-cache。如果让浏览器或中间服务器不进行缓存一般设置no-cache,不过这个值不这么保险，通常加上Expires置为0来达到目的
但如果刻意需要浏览器或中间服务器缓存住我们的页面这个值则需要设置为Pragma

HTTP1.1中启用Cache-Control来控制页面的缓存与否，常用的参数如下：
no-cache
	浏览器和缓存服务器都不应该缓存页面信息
public
    浏览器和缓存服务器都可以缓存页面信息
no-store
	请求和响应的信息都不应该被存储在方法的磁盘系统中
must-revalidate
	对于客户机的每次请求，代理服务器必须向服务器验证缓存是否过时
max-age=xxx s-max-age=xxx
	替代Expires 表示应该在xxx秒后认为页面过时，后者指示代理服务器中缓存的页面过期时间

通常我们不需要缓存页面时
	setHeader("Cache-Control","no-cache,no-store,must-revalidate")
需要缓存页面的时候：
	setHeader("Cache-Control","public,max-age,s-max-age");

Last-Modified  指页面最后生成时间  GMT格式
Expires  过期限制，GMT格式，指浏览器或缓存服务器在该时间点后必须从真正服务器中获取新的页面信息
注意:上面两个值如果是这JSP 页面中进行设置需要设置为long型

//本页面允许在浏览器端或缓存服务器中缓存，时间限制为10秒
    java.util.Date date = new java.util.Date();
    response.setDateHeader("Last-Modified",date.getTime());
    response.setDateHeader("Expired",date.getTime()+10000);
    response.setHeader("Cache-Control","public");
    response.setHeader("Pragma","Pragma");

//不允许浏览器端或缓存服务器缓存当前页面信息
    response.setHeader("Pragma","no-cache");
    response.setDateHeader("Expires","0");

    response.addHeader("Cache-Control","no-cache");
    response.addHeader("Cache-Control","no-store");
    response.addHeader("Cache-Control","must-revalidate");
```

#### servlet 实例介绍

```java
1）对于不经常变化的数据在Servlet里可以为其设置合理的缓存时间以避免浏览器频繁向服务器发送请求
	设置缓存时间为3分钟
    public void doGet(HttpServlet req, HttpServlet resp){
        String value = "xxx"；(编码方式和系统的编码方式有关)；详见（GET && POST乱码详解）
        resp.setDataHeader("experice",System.currentTimeMillis()+1000*180);
        resp.getWriter().wirter(value);
    }

2） 如果要实现中高级功能 即客户端请求动态web资源的时候，
	动态web资源发现发送给客户端的数据更新了，就发送给客户端最新的数据，如果没有更新
	动态web资源就要客户访问的缓存数据。此种情况可以复写web资源的（即Servlet）的getLastModify()方法实现
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
              ServletContext context = this.getServletContext();
              InputStream in = context.getResourceAsStream( "annociation.html");
              BufferedReader reader = new BufferedReader(new InputStreamReader(in));
              String line = reader.readLine();
              resp.getWriter().write(line);
        }
        //复写service中的getLastModifid()方法
        protected long getLastModified(HttpServletRequest req) {
               //获取文件
              ServletContext context = this.getServletContext();
              String path = context.getRealPath( "/annociation.html");
              java.io.File file = new File(path);

               return file.lastModified();
       }
```

#### ServletConfig&&ServletContext

```java
ServletConfig 对象
 在Servelt的配置文件中可以使用一个或多个<init-parm>标签为servlet配置一些初始化参数(配置在某个servlet标签下或整个web-app下)
 当Servelt配置了初始化参数以后,web容器在创建servlet实例对象时，会自动将这些初始化参数封装到ServeltConfig中，并在调用Servelt的init方法时
 将ServeltConfig对象传递给Servelt。

Servlet中获取ServletConfig
 1） 创建一个私有变量 private ServletConfig  config=null;
 2） 要重写init()方法，（该方法有两个需要重写下面这个）
    @Override
    public void init(ServletConfig config) throws ServletException {
        // TODO Auto-generated method stub
        super.init(config);
        this.config=config;
    }
  3）获取<init-param>中的配置信息了
    //获取初始化参数
          String value1 = this.config.getInitParameter("x1");
    //获取配置文档中的<init-param>标签下name对应的value
          String value2 = this.config.getInitParameter("x2");
     //获取配置文档中所有的初始化参数使用（Enumeration接受）
           Enumeration e = this.config.getInitParameterNames();
           while(e.hasMoreElements()){
                   String name = (String ) e.nextElement();
                   String value  = this.config.getInitParameter(name);
                    System.out.println(name + "  " + value);
          }


在开发中ServletConfig的作用有如下三个：
1) 获取字符集编码
     String charset = this.config.getInitParameter("charset");

2）获取数据库连接信息
     String url = this.config.getInitParameter("url");
     String username = this.config.getInitParameter("username");
     String password = this.config.getInitParameter("password");

3）获取配置文件
      String configFile = this.config.getInitParameter("config");

WEB.xml文件配置在Servlet标签下：
      <servlet >
               <servlet-name> annociation</servlet-name >
               <servlet-class> Annociation</servlet-class >
               <init-param>
                      <param-name> age</ param-name>
                      <param-value> 27</ param-value>
               </init-param>
   </servlet >

                   程序中获取：
public class Annociation extends HttpServlet {
     ...
     @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
               // TODO Auto-generated method stub
              String age = super.getServletConfig().getInitParameter("age" );
              PrintWriter out = resp.getWriter();
              out.print(age);
              out.println( "=======");
       }
          ....
     }
```

##### ServletContext 对象

```java
ServletContext对象：
 WEB容器启动的时候，它会为每个WEB应用程度都创建一个对应的ServletContext对象，它代表当前WEB应用，ServletConfig对象中维护了ServletContext对象的引用。
 可以通过Servletconfig.getServletContext方法获得ServletContext对象。

 ServletContext对象应用
  1：多个WEB组件之间使用它实现数据共享
    ServletConfig对象中维护了ServletContext对象的引用,开发人员在编写Servlet时可以通过ServletConfig.getServletContext方法获取ServletContext的对象。
    由于一个WEB应用中的所有Servlet共享同一个ServletContext对象，因此Servlet对象之间可以通过ServletContext对象来实现通讯。
    ServeltContext对象通常也被称之为context域对象

   在Servlet中可以使用如下语句来设置数据共享：
      ServletContext context = this.getServletContext();//获取Servlet域对象
      context.setAttribute("data","数据共享")；//设置共享变量
   在另一个Servlet中可以使用如下语句来获取域中的data属性
      ServletContext context context = this.getServletContext();

   通过serveltContext对象获取整个web应用的配置信息
      String url = this. getServeltContext().getInitParameter("url");
      String username = this.getServletContext().getInitParameter("username");
      String password= this.getServletContext().getInitParameter("password")

  通过servletContext对象实现servlet转发
     由于servlet中的java数据不易设置样式，所以serlvet可以将java数据转发到JSP页面中进行处理
          this.getServletContext().setAttribute("data","serlvet数据转发");
          RequestDispatcher rd = this.getServletContext().getRequestDispatcher("/viewdata.jsp");
          rd.forward(request, response);


web.xml文件中配置：
  <context-param >
       <param-name >name </param-name>
       <param-value >shuo ai lin</ param-value>
  </context-param >
在程序中获取的方法：
  public class Annociation extends HttpServlet {
     @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
               // TODO Auto-generated method stub

              String name = super.getServletContext().getInitParameter( "name");
              out.print( name);
              out.println( "=======");
       }
   }
```

```java
getServletConfig 和 getServletContext 实在GenericServlet中进行重写的，HttpServlet又继承了GenericServlet
public ServletConfig getServletConfig()
  {
    return this.config;
  }

  public ServletContext getServletContext()
  {
    return getServletConfig().getServletContext();
  }
```

##### servletcontext 的使用实例

```java
1） ServletContext实现请求转发
<!DOCTYPE html>
<html>
<head>
<meta charset= "UTF-8">
<title> Insert title here</title >
</head>
<body>
<form action= "/Servlet/annociation" method ="get">
        <input  type= "text"  name ="test"/>
        <input type= "submit" value ="提交">
</form>
</body>
</html>
package com.gc;
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
@WebServlet(name="annociation" ,urlPatterns="/annociation/*")
public class Annociation extends HttpServlet {

        private static final long serialVersionUID = -4983121091648494572L;
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
               // TODO Auto-generated method stub
              System. out.println("===============Servlet is running =====================");
              // 获取ServletConfig对象中的内容
              String age = super.getServletConfig().getInitParameter("age" );
              PrintWriter out = resp.getWriter();

              //通过ServletContext获取ServletContext对象中的内容
              String name = super.getServletContext().getInitParameter("name" );
              String ageContext = this.getServletContext().getInitParameter("age" );
              out.print(age+ "   " +name);
              out.println(ageContext);
              out.println( "=====================");
               //获取form表单的元素
              String ID = req.getParameter( "test");
               if(ID.equals("login" )){
                     ##使用ServletContext进行页面跳转
                     RequestDispatcher rd = this.getServletContext().getRequestDispatcher("/web/login");
                     rd.forward(req, resp);
              }

       }

        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
               // TODO Auto-generated method stub
               doGet(req, resp);
       }
}

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635736641883-2d59abf8-495d-4371-841e-1924e2ec05b1.png#clientId=uaafaf5ae-53bd-4&from=paste&height=258&id=udabb99d9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=556&originWidth=1752&originalType=binary&ratio=1&size=314443&status=done&style=none&taskId=ub6f83078-65ec-463f-b5a9-69745322ab0&width=814)

```java
package com.gc;
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
@WebServlet(name="login" , urlPatterns="/web/*")
public class LoginServlet extends HttpServlet {
        private static final long serialVersionUID = 1L;

        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
              PrintWriter writer = resp.getWriter();
              String data = req.getParameter( "test");
              writer.println( "=================登陆页面，中文测试==================" );
              writer.println( "=================Login page==================" );
              writer.println(data);
       }

        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
               // TODO Auto-generated method stub
              doGet(req, resp);
       }
}
```

> *输入"login"页面跳转后的 URL 为*​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635736674165-ebdc29a4-6fa3-48dc-af34-881dcca3cd79.png#clientId=uaafaf5ae-53bd-4&from=paste&height=65&id=uae4a0f92&margin=%5Bobject%20Object%5D&name=image.png&originHeight=94&originWidth=1165&originalType=binary&ratio=1&size=11554&status=done&style=none&taskId=u66c57f12-49a3-4e5a-87a5-7681f05ac63&width=802.5)

```java
发现页面输出乱码：该乱码是使用POST方式传值时导致的，
POST传值乱码是，在接受端设置：
   request.setCharacterEncoding( "UTF-8");
   response.setContentType( "text/html;charset=utf-8");

使用get方式船只或者URL乱码时：
     new String(ID.getBytes("ISO-8859-1"), "UTF-8");

```

#### servlet 跳转

```java
使用标签 meta 跳转到对应的Servlet
     < META HTTP-EQUIV= "Refresh" CONTENT="0; URL=/Servlet/annociation" />
```

#### Servlet 深入解释

```java
Servlet
     Servlet 是一个sun公司提供的一门用于开发web资源的技术
     sun公司在其API提供了一个serlvet接口，用户若想用发一个动态web资源(即开发一个java程序向浏览器输出数据),需要完成以下2个步骤：
     1） 编写一个Java类，实现servlet接口
     2）把开发好的java类部署到web服务器中。
     按照一种约定的称呼习惯，通常我们

Servlet是一个基于 Java技术的 web组件，运行在服务器端，由 servlet容器管理，用于生成动态内容。
可以说Servlet 是子服务器上运行的小程序。一个 Servlet就是一个 Java类 ,并且可以通过 ”请求 -响应 ”编程模型来访问的这个驻留在服务器里的 Servlet程序

1） web服务器首先检查是否已经装载并创建了该Servlet的实例对象。如果是直接执行第4）步，否则，执行第2）步
2）装载并创建该Servlet的一个实例对象
3）调用Servlet实例对象的init方法
4）创建一个用于封装HTTP请求消息的HTTPServletRequest对象和一个代表HTTP响应的HttpServletResponse对象然后调用Servlet的service方法并将请求对象作为参数传递进去
5) WEB应用程序被停止或重新启动之前，Servlet引擎将卸载Servlet,并卸载之前调用Servlet的destory()方法、


```

##### servlet 的优点

```java
ervlet可以很好地替代公共网关接口(Common Gateway Interface，CGI)脚本。通常CGI脚本是用Perl或者C语言编写的，它们总是和特定的服务器平台紧密相关。
而servlet是用Java编写的，所以它们一开始就是平台无关的。这样，Java编写一次就可以在任何平台运行(write once,run anywhere)的承诺就同样可以在服务器上实现了。servlet还有一些CGI脚本所不具备的独特优点：

1、servlet是持久的。servlet只需Web服务器加载一次，而且可以在不同请求之间保持服务(例如一次数据库连接)。与之相反，CGI脚本是短暂的、瞬态的。每一次对CGI脚本的请求，都会使Web服务器加载并执行该脚本。一旦这个CGI脚本运行结束，它就会被从内存中清 除，然后将结果返回到客户端。CGI脚本的每一次使用，都会造成程序初始化过程(例如连接数据库)的重复执行。
2、servlet是与平台无关的。如前所述，servlet是用Java编写的，它自然也继承了Java的平台无关性。
3、servlet是可扩展的。由于servlet是用Java编写的，它就具备了Java所能带来的所有优点。Java是健壮的、面向对象的编程语言，它很容易扩展以适应你的需求。servlet自然也具备了这些特征。
4、servlet是安全的。从外界调用一个servlet的惟一方法就是通过Web服务器。这提供了高水平的安全性保障，尤其是在你的Web服务器有防火墙保护的时候。
5、setvlet可以在多种多样的客户机上使用。由于servlet是用Java编写的，所以你可以很方便地在HTML中使用它们，就像你使用applet一样。

但值得注意的是Servlet并不局限于Web领域(或者说是HTTP协议相关).你可能要自己去扩展javax.servlet.GenericServlet .上面第三点就是这个意思.
比如说:扩展javax.servlet.GenericServlet实现一个Servlet,搞搞电子邮件的smtp(Simple Mail Transfer Protocol 即简单邮件传输协议)服务器,你就是基于SMTP协议扩展了GenericServlet ,也许你会叫它SMTPServlet.我们用的HttpServlet不就这个命名规范嘛.
综上所述,Servlet可以在各种协议下良好的运行应用程序.

也许现在很多人不再写Servlet,但Servlet是心脏,Servlet放在刚出世的时候,还是非常NB的.到现在,各种成熟好用的东西很多了.但不能忘本.....再深点说  为什么要这么设计，体现了什么？我还真是说不上来了,我觉得上面的几点也已经表明为什么了.就是Java对协议的一个设计.....
JSP的本质就是Servlet,或者说(是吧,开始说的是java,汗)JavaWeb开发的本质也就是Servlet+JDBC.任何性质的框架技术最底层的依然是基于他们2个.因此如果自己想写一套如SSH那样的框架技术,Java最底层的东西是必须掌握的.

Servlet被称为"服务器端小程序."是运行在服务器端的程序,用于处理以及响应客户端的请求.
```

##### **Servlet 的生命周期**

```java
Servlet的生命周期：Servlet加载---实例化---服务---销毁
     init()：
     	在Servlet的生命周期中进执行一次init（）方法，它是在服务器装入Servlet时执行的，
     	负责初始化Servlet对象，以在启动服务器或客户机首次访问Servlet时装入Servlet。
        无论有多少客户机访问Serlvet，都不会重复执行init()

     Service():
		它是Servlet的核心，负责相应客户请求，每当一个客户请求一个httpServlet对象，
        该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest对象和一个响应“ServletResponse”对象作为
        参数，在httpServlet中已存在service（）方法。
        默认的服务功能是调用HTTP请求的方法响应的do功能。

     destory()
        仅执行一次，在服务器端停止且卸载servlet时执行该方法，
        当Servlet退出生命周期时，负责释放占用的资源。
        一个Servlet在运行service方法时可能会产生其他的线程，因此调用destory方法时，这些线程已经终止或完成。
```

```java
package javax.servlet;
import java.io.IOException;

public abstract interface Servlet{
  public abstract void init(ServletConfig paramServletConfig) throws ServletException;
  public abstract ServletConfig getServletConfig();
  public abstract void service(ServletRequest paramServletRequest, ServletResponse paramServletResponse) throws ServletException, IOException;
  public abstract String getServletInfo();
  public abstract void destroy();
}
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635736916694-59f389ac-2a5a-460c-95f7-b71c2597ef4c.png#clientId=uaafaf5ae-53bd-4&from=paste&height=282&id=u5eea644f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=478&originWidth=1408&originalType=binary&ratio=1&size=126123&status=done&style=none&taskId=ubd9fceb1-824b-4e8c-91d7-db43aeedf07&width=832)

```java
步骤：
  1） Web Client 向Servlet容器（Tomcat）发出Http请求
  2） Servlet容器接收Web Client的请求
  3）Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中。
  4）Servlet容器创建一个HttpResponse对象
  5）Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给 HttpServlet 对象。
  6）HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息。
  7）HttpServlet调用HttpResponse对象的有关方法，生成响应数据。
  8）Servlet容器把HttpServlet的响应结果传给Web Client。
```

#### Servlet 工作原理

```java
ervlet接受和响应客户请求的过程,首先客户发送一个请求Servlet是调用service方法对请求进行响应的，
通过源代码可见，Service方法中对请求进行了匹配，选择调用doGet和doPost等这些方法
  然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在servlet接口和GenericServlet中是没有doGet（）、doPost( )等这些方法的。
HttpServlet中定义这些方法，都是返回error信息所以我们每次定义Servlet的方法的时候都必须实现doGet和doPost等这些方法

每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，
	分别是上文提到的init(),service(),destroy()方法。
    GenericServlet是一个通用的，不特定于任何协议的Servlet,它实现了Servlet接口。
    而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。
    所以我们定义Servlet的时候只需要继承HttpServlet即可。

Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于HTTP协议的类，
    所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse 强转为HttpRequest 和 HttpResponse。

创建Servlet对象的时机：
       Servlet容器启动时，读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，通过将ServletConfig对象作为参数来调用Servlet对象的init()方法
       在Servlet容器启动后：客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet 对象的service方法。
       Servlet Servlet容器在启动时自动创建Servlet，这是由在web.xml文件中为Servlet设置的<load-on-startup>属性决定的。从中我们也能看到同一个类型的Servlet对象在Servlet容器中以单例的形式存在。
       <servlet>
            <servlet-name>Init</servlet-name>
            <servlet-class>org.xl.servlet.InitServlet</servlet-class>
            <load-on-startup>1</load-on-startup>
 		</servlet>

```

### web 请求和响应

#### 中文乱码问题

```java
乱码的解决方案如下：

post 传值乱码时,在接受端设置 request.setCharacterEncoding("utf-8")，
	最好使用过滤器设置，并设置response.setContextType("text/html;charset=utf-8");
get传值乱码或者URL乱码时，手动设置接受的参数
	String str = new String(request.getParamter("something").getBytes("ISO-8859-1"),"utf-8");
```

##### **post 方法传值乱码**

```java
由于post方式传值通过request存储的，在另一个页面也是通过request.getParameter(String name)来提取信息，所以这种情况下乱码主要是
因为request存储信息的编码设置导致的。
post提交是，如果没有设置提交的编码格式，则会以iso-8859-1方式进行提交，接受的页面却以utf-8的方式接受。
所以使用如下语句即可得到单个正确的中文字符串：
     String str = new String(request.getParamter("something").getBytes("ISO-8869-1"),"UTF-8")
解决方法：
     在接受页面设置request.setCharacterEncoding("utf-8")。
     最好通过过滤实现每个页面都设置为request.setcharacterEncoding("utf-8");
```

##### get 方式传值乱码

```java
get方式传值有两种，
	一种是通过FORM表单get传值，
    一个是url地址传值(实质上这两种方式都是通过url参数方式传值)

表单方式get传值：
     表单方式get传值编码过程为，首先浏览器根据页面charset编码方式对传值进行编码，然后提交至服务器交给tomcat,tomcat会对这个信息进行编码
     采用的编码方式是server.xml文件中URIEncoding设置决定的，也就是说我们使用命令request.getParameter("xxx")获取表单中参数值时，得到的字符串
     经过了charset的编码和URIEncoding的编码。

     所以只要charset的编码和URIEncoding的解码一致，并且支持中文，就能保证没有乱码（ Tomcat 的URIEncoding的默认编码是iso-8859-1）
     第一种解决方法：
     	设置方式修改tomcat的server.xml的 HTTP Connector或者AJP Connector的配置上加上URIEncoding="utf-8"
                 <Connector connectionTimeout= "20000" port= "80" protocol= "HTTP/1.1" redirectPort ="8443" URIEncoding ="utf-8"/>

  	第二种解决方法：
    	 使用useBodyEncodingForURI="true" 这个方法合适你的TOMCAT 实例下需要多跑不同Encoding的程序时
                  < Connector connectionTimeout ="20000" port ="80" protocol ="HTTP/1.1" redirectPort ="8443" useBodyEncodingForURI ="true"/>

在Tomcat配置中，连接器（HTTP Connector）属性中有一个Encoding和useBodyEncodingForURI属性，
  这两个属性设置对URL后附加参数进行URL解码时该如何选择字符集编码
  URIEncoding用于定制URL后的附件参数的字符集编码
  useBodyEncodingForURI则说明是否采用实体内容的字符集编码设置来替代URIEncoding的设置，也就说useBodyEncodingForURI属性设置为true时，
  ServletRequest.setCharsetEncoding方法设置的字符集编码也影响了getParameter等方法对URL地址后的参数进行URL解码的结果

 url 方式get传值乱码
   对于这种方式，浏览器不会采用页面的charset方式对URL中的中文进行编码后提交至服务器（IE，FireFox都一样）而是采用系统的GBK转码为ISO-8859-1之后提交到服务器
   Tomcat,所以这个过程为：
        首先,URL地址中的中文被从GBK转换成ISO-8859-1,交给Tomcat后，
        	又被tomcat根据URIEncoding解码，
            这种情况，只有把URLEncoding设置为gbk才能在request.getParameter("")时不出现乱码。
  但是这样会影响到上面配置，所以一个好的解决方法时，使用java.net.URIEncoding对地址中的中文进行手动编码和解码
```

##### 乱码万全的解决方法

```java
1） 所有页面的charset设置为UTF-8。
2） Tomcat的URIEncoding默认是ISO-8859-1，设置为UTF-8，
      主要是解决中文命名的文件以及请求以get方式提交有可能出现的乱码问题。
3） 添加过滤器，调用request.setCharacterEncoding("utf-8")方法将request的字符集设定为utf-8，解决请求以post方式提交的乱码问题。
4） url地址中存在中文参数时，首先对中文参数使用URLEcoder编码为utf-8，然后在request.getParameter("")接收到参数后再使用URLDecoder还原。
```

#### HttpServletRequest

> _HttpServletRequest 对象代表客户端的请求,当客户端请求通过 HTTP 协议访问服务器时，HTTP 请求头中的所有信息都封装在整个对象中，通过这个对象提供的方法，可以获得客户端请求的所有信息_

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737459606-791ff3e6-53ba-4400-b9be-b447b59a46be.png#clientId=uaafaf5ae-53bd-4&from=paste&height=438&id=u64b34ca0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=876&originWidth=1734&originalType=binary&ratio=1&size=911896&status=done&style=none&taskId=u389b0a09-593f-4a2d-a507-65b5f82cdcb&width=867)

```java
获取客户机信息：
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
               //得到请求的URL地址
              String requestUrl = req.getRequestURL().toString();
               //得到请求的资源
              String requestUri = req.getRequestURI();
               //得到请求的URL地址中附带的参数
              String queryString = req.getQueryString();
               //得到来访者的IP地址
              String remoteAddr = req.getRemoteAddr();
              String remoteHost = req.getRemoteHost();
               int remotePort = req.getRemotePort();
              String remoteUser = req.getRemoteUser();
               //得到请求URL地址时使用的方法
              String method = req.getMethod();
              String pathInfo = req.getPathInfo();
               //获取WEB服务器的IP地址
              String localAddr = req.getLocalAddr();
               //获取WEB服务器的主机名
              String localName = req.getLocalName();
               //设置将字符以"UTF-8"编码输出到客户端浏览器
              req.setCharacterEncoding( "UTF-8");
               //通过设置响应头控制浏览器以UTF-8的编码显示数据，如果不加这句话，那么浏览器显示的将是乱码
               resp.setHeader( "content-type", "text/html;charset=UTF-8" );
               PrintWriter out = resp.getWriter();
               out.write( "获取到的客户机信息如下：" ); ...

       }

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737491404-1e679fee-f9d0-4762-9076-bf97c8f729be.png#clientId=uaafaf5ae-53bd-4&from=paste&height=163&id=u4f5cc4fb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=253&originWidth=1169&originalType=binary&ratio=1&size=25596&status=done&style=none&taskId=uab45cfaf-2a5b-4347-b8a3-50a4c03ff91&width=753.5)

```java
获取请求头的信息：

     getHeader(String name) 方法String
     getHeader(String name) 方法Enumeration
     getHeaderNames( )方法

     @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
              resp.setHeader( "content-type", "text/html;charset=UTF-8" );
              PrintWriter out = resp.getWriter();
               //获取所有请求头
              Enumeration<String> reqHeadInfos = req.getHeaderNames();
              out.println( "获取到的请求头的信息如下：" );
              out.write( "<hr/>");
               while(reqHeadInfos.hasMoreElements()){
                     String name = reqHeadInfos.nextElement();
                      //获取请求头信息
                     String value = req.getHeader(name);
                     out.write(name+ " : "+value);
                     out.write( "<br>");
              }

       }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737523892-51a75c02-3aa7-4fce-b1c0-222b6f86e507.png#clientId=uaafaf5ae-53bd-4&from=paste&height=132&id=uf14d3f6b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=184&originWidth=1103&originalType=binary&ratio=1&size=13345&status=done&style=none&taskId=ue2b6240d-e926-4960-9ee6-6a730585cb0&width=790.5)

```java
获取客户及请求参数
     getParameter(String )
     getParameterValues(String name)
     getParameterNames()方法
     getParamterMap()方法 ( )

     <div>
        <form action= "/Servlet/forminfo" method ="post">
        编 &nbsp;&nbsp;号：< input type ="text" name="userID" value= "NO" size ="2" maxlength="2" ><br>
        用户名： <input type= "text" name ="username" value= "请输入用户名" ><br>
             密码： <input type= "password" name ="userpass" value= "请输入密码" ><br>

        性 &nbsp;&nbsp;别(单选框)：
      <input type="radio" name= "sex" value ="男" checked>男
      <input type="radio" name= "sex" value ="女"> 女< br>
          部 &nbsp;&nbsp;门(下拉框)：
     <select name="dept">
         <option value= "技术部" >技术部 </option>
         <option value= "销售部" SELECTED> 销售部</option >
         <option value= "财务部" >财务部 </option>
     </select ><br>
         兴 &nbsp;&nbsp;趣(复选框)：
            <input type= "checkbox" name ="inst" value="唱歌"> 唱歌
            <input type= "checkbox" name ="inst" value="游泳"> 游泳
            <input type= "checkbox" name ="inst" value="跳舞"> 跳舞
            <input type= "checkbox" name ="inst" value="编程" checked>编程
            <input type= "checkbox" name ="inst" value="上网"> 上网
            <br>
             说 &nbsp;&nbsp;明(文本域)：
           <textarea name= "note" cols ="34" rows="5">
             </textarea>
           <br>
    <input type="hidden" name= "hiddenField" value ="hiddenvalue"/>
    <input type="submit" value= "提交(提交按钮)" >
    <input type="reset" value= "重置(重置按钮)" >
        </form>
  </div>
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737550960-8299b408-5ac0-40ae-a207-b01cff21a46f.png#clientId=uaafaf5ae-53bd-4&from=paste&height=199&id=u66892a5d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=258&originWidth=1023&originalType=binary&ratio=1&size=24469&status=done&style=none&taskId=ubb72f11a-1e13-4906-bf7a-45c1a226e7b&width=788.5)

```java
@Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
              req.setCharacterEncoding( "utf-8");
               String html = null;
               String userid = req. getParameter("userID"); //获取填写的编号
               String username = req. getParameter("username"); //获取填写的用户名
               String userpass = req. getParameter("userpass"); //获取填写的密码
               String sex = req. getParameter("sex"); //获取选中的性别
               String dept = req. getParameter("dept"); //获取选中的部门
               String[] insts = req.getParameterValues( "inst");//获取兴趣
               String note = req. getParameter("note"); //获取填写的说明信息
               String hiddenField = req. getParameter("hiddenField"); //获取隐藏域的内容
               String instStr= "";
               /**
           * 获取数组数据的技巧，可以避免 insts数组为null时引发的空指针异常错误！
           */
               for (int i = 0; insts!=null && i < insts. length; i++) {
               if (i == insts.length -1) {
                      instStr+=insts[i];
                      } else {
                            instStr+=insts[i]+ ",";
                      }
               }
               html= "编号："+userid+"<br>" +"用户名:" +username+"<br>"+ "密码:"+userpass+"<br>" +"性别:" +sex+
                       "<br>"+"部门:" +dept+"<br>" +"兴趣：" +instStr+"<br>"+ "说明信息:" +note+"<br>" +"隐藏域" +hiddenField+"<br>";
               resp.setCharacterEncoding( "UTF-8");//设置服务器端以UTF-8编码输出数据到客户端
               resp.setContentType( "text/html;charset=UTF-8");//设置客户端浏览器以UTF-8编码解析数据
               resp.getWriter().println(html);
       }

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737575867-556093ef-f86c-4964-9c9b-4800e3751f0a.png#clientId=uaafaf5ae-53bd-4&from=paste&height=163&id=u2d4308f3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=220&originWidth=1068&originalType=binary&ratio=1&size=18683&status=done&style=none&taskId=ubdd41170-f559-4ed2-bc01-304fe7f21ea&width=789)

```java
服务器端使用getParameterNames方法接受表单参数，代码如下：
          String html="";
              Enumeration<String> paramNames = req.getParameterNames(); //获取所有的参数名
               while(paramNames.hasMoreElements()){
                     String name= paramNames.nextElement();//得到参数名
                     String value = req.getParameter(name);
                     html  = html + name+ "  :  "+value+"<br>" ;
              }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737609964-04b1751e-dd71-4e57-a42b-2064a6231fe7.png#clientId=uaafaf5ae-53bd-4&from=paste&height=186&id=ufc81d4d0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=233&originWidth=1025&originalType=binary&ratio=1&size=19536&status=done&style=none&taskId=ue5dfbe96-fa32-414c-a2e2-db473d65a52&width=818.5)

```java
使用getParameterMap方法接受表单参数，代码如下：
String html="";
Map<String, String[]> paramMap = req.getParameterMap();
	for(Map.Entry<String, String[]> entry :paramMap.entrySet()){
        String paramName = entry.getKey();
        String paramValue = "";
        String[] paramValueArr = entry.getValue();
        for (int i = 0; paramValueArr!=null && i < paramValueArr.length; i++) {
            if (i == paramValueArr.length -1) {
                paramValue+=paramValueArr[i];
            } else {
                paramValue+=paramValueArr[i]+ ",";
            }
        }
        String data = (MessageFormat.format("<tr><td>{0}</td><td>{1}</td></tr>", paramName,paramValue));
    	html=html+data;
	}
resp.setCharacterEncoding( "UTF-8");//设置服务器端以UTF-8编码输出数据到客户端
resp.setContentType( "text/html;charset=UTF-8");//设置客户端浏览器以UTF-8编码解析数据
resp.getWriter().println( "<table>"+html+"</table>" );
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737663339-ac2ecfa7-7a6b-4821-b2cf-2905298b990d.png#clientId=uaafaf5ae-53bd-4&from=paste&height=156&id=u6ad2c1ff&margin=%5Bobject%20Object%5D&name=image.png&originHeight=194&originWidth=982&originalType=binary&ratio=1&size=17219&status=done&style=none&taskId=u162baf75-7e2c-4a85-b8b0-77560726db8&width=788)

```java
Request对象实现请求转发
  请求转发：指的是一个web资源收到客户端请求后，通知服务器去调用另外一个web资源进行处理
在Servlet 中实现请求转发有两种方式
1) 通过ServletContext的getRequestDispacher(String path)方法
		该方法返回一个RequestDispacher对象，调用这个对象的forword方法可以实现请求转发
   RequestDispacher reqDispacher = this.getServletContext().getRequestDispatcher("/test.jsp");
   reqDispacher.forword(request,reponse)

2) 通过request对象提供的getRequestDispacher(String path)方法，该方法返回一个RequestDispacher对象，调用这个对象的forword方法可以实现请求转发
   request.getRequestDispacher("/test.jsp").forward(request, reponse)

request对象同时也是域对象（Map容器）, 开发人员通过request对象在实现转发时，把数据通过request对象带给其他web资源处理

```

#### HttpServletResponse

```java
web服务器接受到客户端的http请求，会针对每一次请求，分别创建一个用于代表请求的request对象和代表响应的response对象
request和response对象系带代表请求和响应，那么我们获取客户及提交过来的数据，只需要找到request对象就行了。
要向客户机输出数据，只需要找response对象就行了

HttpServletResponse对象代表服务器的响应，这个对象中封装了向客户端发送数据，发送响应头，发送响应状态码的方法。
对应的一些常用API如下:
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635737794211-9b377cec-c911-4231-a8ef-da5d8c4a397d.png#clientId=uaafaf5ae-53bd-4&from=paste&height=256&id=ud736dd0f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=512&originWidth=1764&originalType=binary&ratio=1&size=337882&status=done&style=none&taskId=u84a85763-5883-48dc-9c4f-e7e96608d01&width=882)

** _使用 OutputStream 流向客户端浏览器输出中文数据_**

```java
使用OutPutStream流输出中文注意问题：
     在服务器端数据是以哪个码表输出的，那么就要控制客户端浏览器以相应的码表打开，比如：outputStream.writer("中国"，"utf-8")使用outputStream流向客户端浏览器输出中文以UTF-8进行编码。此时就要控制客户端浏览器
以UTF-8的编码打开，否则显示的时候就会出现中文乱码，那么在服务器端如何控制客户端的浏览器以utf-8的编码显示数据呢？
就需要通过response设置响应头控制浏览器以utf-8的编码进行显示。方法： response.setHeader("content-type","text/html;charset=UTF-8")

   protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {

       //Servlet 内部定义的变量的中文编码为GB2312,（该变量与操作系统的环境有关，中文的操作系统编码为GB2312）
       //前台页面传递的中文参数的编码如果server.xml文件中没有做任何编码配置，则编码为ISO-8859-1
       String data = "中国";
       resp.setHeader( "context-type", "text/html;charset=UTF-8" );
       OutputStream outputStream = resp.getOutputStream();
       //参照Java 运用章节中的获取字符传编码
       outputStream.write( getEncoding(data).getBytes());
       data.getBytes(); //是一个将字符转换为字节数组的过程，这个过程中一定会去查码表，如果是中文的操作系统环境，默认就是GB2312的码表
       //   将字符转换为字节数组的过程就是讲中文字符转换成GB2312码表上对应的数字
       byte[] datas= data.getBytes( "utf-8");
       //PrintWriter writer = resp.getWriter()
       // writer.write(data);
       outputStream.write(datas);
    }
```

**_使用 printWriter 流向客户端浏览器输出中文数据_**

```java
 使用printwrite需要注意的事项： printWriter输出流之前首先使用"response.setCharacterEncoding(charset)"设置字符串以什么样的编码输出到浏览器，如 response.setCharacterEncoding("utf-8")
     设置将字符以“utf-8”编码输出到客户端浏览器，然后再使用response.getWriter；获取PrintWriter进行输出。这两步骤不能颠倒。
   @Override
   protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
     String data = "我爱中国" ;
     resp.setCharacterEncoding( "UTF-8");//设置将字符以"UTF-8"编码输出到客户端浏览器
     /**
      * PrintWriter out = response.getWriter();这句代码必须放在response.setCharacterEncoding("UTF -8");之后
      * 否则response.setCharacterEncoding("UTF -8")这行代码的设置将无效，浏览器显示的时候还是乱码
      */
     PrintWriter out = resp.getWriter(); //获取PrintWriter输出流
     //out.write("<meta http-equiv ='content-type' content='text/html;charset=UTF-8'/>");
     resp.setHeader( "content-type", "text/html;charset=UTF-8" );
     out.write(data);
  }
```

**_使用 outputStream 和 printWriter 输出数字_**

```java
需要将数字转换为对应的字符
 public void outputOneByPrintWriter(HttpServletResponse response) throws IOException{
    response.setHeader( "content-type", "text/html;charset=UTF-8" );
    response.setCharacterEncoding( "UTF-8");
    PrintWriter out = response.getWriter(); //获取PrintWriter输出流
    out.write( "使用PrintWriter流输出数字1：" );
    out.write(1+ "");
 }
```

​

**_文件下载和注意事项_**

```java
使用HttpServletResponse对象实现对文件的下载
文件下载功能的实现思路
1) 获取要下载的文件的绝对路径
2）获取要下载的文件名
3）设置content-disposition响应头 setHeader("content-disposition" , "attachment;filename="+filename);
4）获取要下载的文件输入流，创建数据缓冲去
5）通过response对象获取OutputStream流，将FileInputStream流写入到buffer缓冲区，最后使用OutputStream将缓冲区的数据输出

文件下载几个需要注意的事项：
1） 中文文件名需要使用URLEncoder.encode方法进行编码，（Encoder.encode(fileName,"字符编码")），否则会出现文件名乱名
2） 编写文件下载功能时推荐使用OutputStream流，避免使用PrintWriter流，因为OutputStream流是字节流，可以处理任意类型的数据，而PrintWrite流是字符流，只能处理字符数据，
    如果用字符流处理字节数据，会导致数据丢失。
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException {
    try{
        downloadFileByOutputStream(resp); //下载文件，通过OutputStream流
    } catch(Exception e){
        e.printStackTrace();
    }
}

private void downloadFileByOutputStream(HttpServletResponse response) throws Exception{
    //1.获取要下载的文件的绝对路径
    String realPath = this.getServletContext().getRealPath("/web/login.html" );
    //设置content-disposition响应头控制浏览器以下载的形式打开文件，中文文件名要使用URLEncoder.encode方法进行编码，否则会出现文件名乱码
    String fileName = "下载的文件" ;
    fileName=URLEncoder. encode(fileName,"utf-8");
    response.setHeader( "content-disposition", "attachment;filename=" +fileName);

    InputStream in = new FileInputStream(realPath);
    int len = 0;
    byte[] buffer = new byte[1024];
    OutputStream out = response.getOutputStream();
    while ((len = in.read(buffer)) > 0) {
        //使用OutputStream将缓冲区的数据输出到客户端浏览器
        out.write(buffer,0,len);
    }
    in.close();
}

```

**_客户端缓存 Servlet 的输出_**

```java
为什么要设置缓存
  一个网站往往会有很多的静态web资源，例如，html页面、css文件、jpg图片等，这些资源一旦创建可能永远不会改变
  如果客户端每次访问网站时都下载一次静态web资源，这样不但会造成服务器的压力增大，用户的体验也一定不好
  一般来讲，我们在用户第一次访问网站时，将静态web资源发给客户，并通知客户将内容缓存起来，方便下次访问时使用

缓存的实现方式
     设置合理的缓存时间response.setDateHeader(“Expires”, 时间值);

不设置缓存的方式：
     response.setDateHeader("Expires",-1);//IE
     response.setHeader("Cache-Control","no-cache");
	 response.setHeader("Pragma","no-cache");
     @Override
     protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
         /**
          * 设置数据合理的缓存时间值，以避免浏览器频繁向服务器发送请求，提升服务器的性能
          * 这里是将数据的缓存时间设置为1天
          */
         resp.setDateHeader( "expires", System.currentTimeMillis()+1000*3600*24);
         InputStream in = getServletContext().getResourceAsStream("/web/login.html" );
         OutputStream out = resp.getOutputStream();
         int len;
         byte[] buf = new byte[1024];
         while((len=in.read(buf))>0)
             out.write(buf, 0, len);
     }

	@Override
	protected long getLastModified(HttpServletRequest req) {
   	 	// 到底该返回什么
    	// 应该返回文件的最后修改时间
    	// 1.html 如果修改了就发送，如果没修改发304让用户去拿缓存
    	String path = getServletContext().getRealPath( "/web/login.html");
    	File file = new File(path);
    	long lastModified = file.lastModified(); // 返回文件的最后修改时间
    	return lastModified;
	}
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
                      throws ServletException, IOException {
         doGet(req, resp);
    }

在HTTPServlet 的 service方法中，首先会调用getLastModified方法，判断返回值是否为-1
  从而获知子类是否重写此方法
 	 如果不是-1，说明子类重写了该方法，此时就会获取浏览器发送过来的 if-modified-since时间值，
     用此时间值和方法的返回值进行比较 判断文件是否更新，如果没有更新就发送304状态码，让用户去拿缓存
Servlet 中Servie的源代码
 protected  void service(HttpServletRequest req, HttpServletResponse resp)
                  throws ServletException, IOException
                {
                  String method = req.getMethod();

                  if (method.equals("GET" )) {
                    long lastModified = getLastModified(req);
                    if (lastModified == -1L)
                    {
                      doGet(req, resp);
                    } else {
                      long ifModifiedSince;
                      try {
                        ifModifiedSince = req.getDateHeader( "If-Modified-Since");
                      }
                      catch (IllegalArgumentException iae) {
                        ifModifiedSince = -1L;
                      }
                      if (ifModifiedSince < lastModified / 1000L * 1000L)
                      {
                        maybeSetLastModified(resp, lastModified);
                        doGet(req, resp);
                      } else {
                        resp.setStatus(304);
                      }
                    }
                  }
```

##### 设置响应头控制浏览器

```java
HttpServletRespons常见应用——设置响应头控制浏览器行为

设置Http响应头控制浏览器禁止缓存当前文档内容
response.setDateHeader("expires",-1);
response.setHeader("Cache-Control","no-cache")
response.setHeader("Pragma","no-cache");

设置Http响应头控制浏览器定时刷新网页（REFRESH）
response.setHeader("refresh","5")  设置refresh响应头控制浏览器每隔5秒钟刷新一次

通过response实现请求重定向
请求重定向指：
	一个web资源收到客户端请求后，通知客户端去访问另一个web资源，这称之为请求重定向
场景：
	用户登录，用户首先访问登录页面，登录成功后，就会跳转到某个页面，这个过程就是一个请求重定向的过程
实现方式：
	response.sendRedirect(String location)，
    即调用response对象的sendRedirect方法实现请求重电箱

sendRedirect内部实现的原理：使用response设置302状态码和设置location响应头实现重定向
```
