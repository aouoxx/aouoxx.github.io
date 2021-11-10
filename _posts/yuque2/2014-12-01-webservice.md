### JAVA 项目中发布 WebService

```java
与Web服务相关的类都位于Javax.jws.*包中
     @Webservice——此注解用在类上将此类发布成一个webservice
     EndPoint——此类为端点服务类，其中publish方法用于将一个已经添加了@WebService注解对象绑定到一个地址端口上，用于发布

```

```java
package Webservice.demo;

import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebService;
import javax.xml.ws.Endpoint;
@WebService
public class HelloWebService {
  /**
       * 参数@WebParam(name="word")只是说明在 wsdl中指明参数的名字是word
       * @param word
       */
       public String Hello(@WebParam (name="word")String word){
            System. out.println("*********Hello world***********" );
             return "Hello: " +word;
      }

       /**
       * 添加exclude=true后方法say()不会被发布
       */
       @WebMethod(exclude=true)
       public String say(){
            System. out.println("************Say null*************" );
             return "ssgao" ;
      }

       public static void main(String[] args) {
             /**
             * 参数1：服务的发布地址
             * 参数2：服务的实现者
             */
            Endpoint. publish("http://127.0.0.1/hello", new HelloWebService());
            System. out.println("***webservice has already send****" );
      }
     }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635740105741-5a5bd1dc-38b2-4b42-9faf-defb2bc5ca27.png#clientId=ua9c0864e-3bf4-4&from=paste&height=106&id=u6f7eb1e2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=211&originWidth=1056&originalType=binary&ratio=1&size=27189&status=done&style=none&taskId=udaebf723-8898-4b2a-9458-22cbdc626c8&width=528)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635740115450-9920422a-2f0e-4f91-8d04-cabd4f0dd697.png#clientId=ua9c0864e-3bf4-4&from=paste&height=280&id=u08f2bd00&margin=%5Bobject%20Object%5D&name=image.png&originHeight=560&originWidth=1058&originalType=binary&ratio=1&size=61217&status=done&style=none&taskId=ub4c6cfb0-2457-489a-8739-948c9c245c8&width=529)

```java
总结发布一个webservice的流程：
     1）在类上添加@WebService注解
     2）通过EndPoint(端点服务)发布一个webservice（注：EndPoint是一个专门用于发布服务的类，该类的publish方法接受两个参数，一个是本地的服务地址，一个是提供服务的类）
     3）类上添加注解@WebService，类中所有的非静态方法都会被发布
     静态方法和final方法不能被发布
     方法上加上@WebMethod(exclude=true)后该方法也不能被发布
```
