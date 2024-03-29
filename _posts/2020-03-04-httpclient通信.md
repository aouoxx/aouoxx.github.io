---
layout: post
title: httpclient原理
categories: [网络通信, httpclient]
description: netty原理
keywords: 网络通信,httpclient
---

 <meta name="referrer" content="no-referrer"/>

## httpclient 简述

httpclient 是 java 开发中非常常见的一种访问网络资源的方式。可以读取网页(http/https)内容,以 GET 或 POST 方式向网页提交参数,处理页面重定向,模拟输入用户名和口令进行登录,提交 xml 格式参数,通过 HTTP 上传文件,当问启动认证的页面以及 httpclient 在多线程下的使用。

## httpclient 连接池使用

```java
多线程模式下使用httpclient连接池的注意事项
	org.apache.http.impl.conn.PoolingClientConnectionManager
 使用这个类就可以使用httpclient连接池的功能了,其可以设置最大的连接数和最大路由连接数。
 	public final static int MAX_TOTAL_CONNECTIONS=400;
	public final static int MAX_ROUTE_CONNECTIONS=200;
	cm = new PoolingClientManager();
	cm.setMaxTotal(MAX_TOTAIL_CONNECTIONS);
	cm.setDefaultMaxPerRoute(MAX_ROUTE_CONNECTIONS);
最大连接数就是连接池允许的最大连接数,最大路由连接数就是每个路由站点的最大连接数。
	HttpHost google = new HttpHost("www.google.com",8080);
	HttpHost baidu = new HttpHost("www.baidu.com",8090);
	google.setMaxPerRoute(new HttpRoute(google),30);
	baidu.setMaxPerRoute(new HttpRoute(baidu),50);
并且可以设置httpclient连接等待时间, 请求等待时间,响应时间等。
```

##

## 注意点

```java
1) 首先配置最大连接数和最大路由连接数,如果你要连接的URL只有一个,两个必须配置成一样,否则只会取最小值。
	(这是个坑,默认最大连接数是20, 默认路由最大连接数是2)
2) 配置httpclient连接等待时间和响应时间。否则会一直等待。
	httpParams = new BasicHttpParams();
	httpParams.setParameter(CoreConnectionPNames.CONNECTION_TIMEOUT,CONNECT_TIMEOUT);
	httpParams.setParameter(CoreConnectionPNames.SO_TIMEOUT,READ_TIMEOUT);
3) httpclient 必须releaseconnection,不是abort。因为releaseConnection是归还连接到连接池,而abort是直接抛弃这个连接而且占用连接池的数目(<<注意>>)
     HttpGet httpGet = new HttpGet(searchurl);
     httpGet.releaseConnection();
4) 一定要注意httpclient设置的最大连接数绝对不能超过tomcat设置的最大连接数,否则tomcat的连接就会被httpclient连接池一直占用,直到系统挂掉。

5) 可以通过tomcat的长连接和httpclient连接池合理使用来增加系统响应速度。
```

## httpclient 连接池

```java
连接池技术作为创建和管理连接的缓冲池技术,目前广泛的用于诸如数据库连接等长连接的维护和管理中,能够有效的减少系统响应时间,节省服务器资源开销。其优势主要有两个:
	1) 减少创建连接的资源开销
    2) 资源的访问控制。
连接池管理的对象是长连接,对于HTTP连接是否适用。
```

### 长连接&短连接

```java
长连接是指客户端与服务器端一旦建立连接之后,可以进行多次数据传输而不需要重新建立连接。
短连接则每次数据传输都需要客户端和服务器端建立一次连接。

长连接的优势在于省去了每次数据传输连接建立的时间开销,能够大幅度提高数据传输的速度,对于P2P应用十分合适,但是对于诸如WEB网站之类的B2C应用,并发请求量大,每一个用户又不需频繁的操作的场景下,维护大量的长连接对服务器无疑是一个巨大的考验。而此时,短连接可能更加适用。但是短连接每次数据传输都需要建立连接,我们知道HTTP协议的传输层是TCP协议,TCP连接的建立和释放分别需要进行3次握手和4次握手,频繁的建立连接即增加了时间开销,同时频繁的创建和销毁Socket同样是对服务器端资源的浪费。所以需要频繁的发送HTTP请求应用,需要在客户端使用HTTP长连接。


HTTP连接是无状态,这样容易给我们造成HTTP连接是短连接的错觉,实际上HTTP1.1默认即是持久连接,HTTP1.0也可以通过在请求头中设置Connection:keep-alive使得连接为长连接。既然HTTP协议支持长连接,我们就有理由相信HTTP连接同样需要连接池技术来管理和维护连接建立和销毁。
HTTPClient4.0的ThreadSafeClientConnManager实现了HTTP连接的池化管理,其管理连接的基本单位是Route(路由),每个路由上都会维护一定数量的HTTP连接。这里的Route的概念可以理解为客户端机器的目标机器的一条线路。例如使用HttpClient的实现来分别请求www.163.com的资源和www.baidu.com的资源就会产生两个route.

缺省情况下,对于每个route,http仅维护了2个连接,总数不超过20个连接,显然对于大多数应用来讲,都是不够用的,可以通过设置HTTP参数进行调整。

HttpParams params = new BasicHttpParams();
//将每个路由的最大连接数增加到200
ConnManagerParams.setMaxTotalConnections(params,200);
//将每个路由的默认连接数设置为20
ConnPerRouteBean connPerRoute=new ConnPerRouteBean(20);
// 设置某一个IP的最大连接数
HttpHost baiduHost = new HttpHost("www.baidu.com",8080);
connPerRoute.setMaxForRoute(new HttpRoute(baiduHost),50);
```

### 连接的维护

```java
连接的有效性检测是所有连接池都面临的一个通用问题,大部分HTTP服务器为了控制资源开销,并不会永久的维护一个长连接,而是一段时间就会关闭该连接。
放回连接池的连接,如果在服务器端已经关闭,客户端是无法检测到这个状态变化而即使关闭的socket的。这就造成了线程从连接池获取的连接不一定是有效的。这个问题的一个解决方法就是在每次请求之前检查该连接是否已经存在了过长时间,可能已过期。
但是这个方法会使得每次请求都增加额外的开销。HTTPclient4.0的ThreadSafeClientConnManager提供了closeExpiredConnections()方法和closeIdleConnections()方法来解决这个问题。前一个是清楚连接池中所有过期的连接,至于连接什么时候过期可以设置,设置方法将在下面提高。后一个方法则是关闭一定时间空闲的连接,可以使用一个单独的线程完成这个工作。
```

## keepAlive

客户端可以设置连接的过期时间,可以通过 httpclient 的 setKeepAliveStrategy 方法设置连接的过期时间,这样就可以配合 closeExpiredConnections()方法解决连接池中连接的失效问题

```java
http协议1.1默认都是开启keep-alive的,除非显示关闭。
http1.0的默认关闭,在请求头添加keep-alive参数。目前adx会优先解析response中的keep-alive时间长度,否则默认包活4.5s
```

```java
public class KeepAliveDemo {
    public static void main(String[] args) {
        DefaultHttpClient defaultHttpClient = new DefaultHttpClient();
        defaultHttpClient.setKeepAliveStrategy(
                new ConnectionKeepAliveStrategy() {
                    @Override
                    public long getKeepAliveDuration(HttpResponse httpResponse, HttpContext httpContext) {
                        HeaderElementIterator it = new BasicHeaderElementIterator(httpResponse.headerIterator(HTTP.CONN_KEEP_ALIVE));
                        while (it.hasNext()){
                            HeaderElement he = it.nextElement();
                            String param = he.getName();
                            String value =he.getValue();
                            if(value!=null&&param.equalsIgnoreCase("timeout")){
                                try{
                                    return Long.valueOf(value)*1000;
                                } catch (Exception e){
                                    e.printStackTrace();
                                }
                            }
                        }
                        HttpHost target = (HttpHost) httpContext.getAttribute(ExecutionContext.HTTP_TARGET_HOST);
                        if("www.baidu.com".equalsIgnoreCase(target.getHostName())){
                            // 对于163这个路由的连接,保持5秒
                            return 5*1000L;
                        }else{
                            // 其他路由保持30秒
                            return 30*1000L;
                        }
                    }
                }
        );
    }
}
```

## http 连接池

```
 PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
        // 将最大连接数增加到200
        cm.setMaxTotal(200);
        // 将每个路由的基础连接增加到20
        cm.setDefaultMaxPerRoute(20); // 这路由啥意思 ?
        HttpHost localhost = new HttpHost("www.netease.com",50);
        cm.setMaxPerRoute(new HttpRoute(localhost),50);
```

### 路由和连接数

```java
PoolingHttpClientConnectionManager 连接池管理, MaxTotal是池子能保存最大连接数。
DefaultMaxPerRoute: 指一个访问地址,按照上面的配置,连接池里面可以保持<<长连接>>到www.netease.com地址的最大数是50个,如果请求量很大,把50个Http连接都占用完了,那新的请求过来就需要等到其他使用连接离线到这个地址的http连接释放了才行。

路由: 是指A到B这条路径,上面代码中的意思是:
    1) 总共保持200个连接
    2) 每个网站的默认连接最多20个,比如同时抓取百度,腾讯。那么百度20个,腾讯20个,但累计不能超过200个。
    3) 最后两行www.netease.com这个网站可以特殊点,特别处理一下,让它最大保持50个
```

### httpclient 实例

```java
public class HttpClientTest {

    public static String cookie = "xxx";

    public static void main(String[] args) throws IOException {
        HttpClientTest.testGet();
    }
    public static CloseableHttpClient init(){
        // 创建连接池管理器
        PoolingHttpClientConnectionManager manager = new PoolingHttpClientConnectionManager(60000, TimeUnit.SECONDS);
        manager.setMaxTotal(200);
        manager.setDefaultMaxPerRoute(20);
        // 创建HttpClient对象
        CloseableHttpClient httpClient = HttpClients.custom()
                                .setConnectionManager(manager)
                                .disableAutomaticRetries()
                                .build();
        return httpClient;
    }

    public static void testGet() throws IOException {
        CloseableHttpClient httpClient = init();
        CloseableHttpResponse response = null;
        try{
            //创建请求对象
            HttpGet httpGet = new HttpGet("https://adnet.qq.com/adFilter/industry/firstlevel/list");
            httpGet.setHeader("cookie",cookie);
            httpGet.setHeader("user-agent","Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36");
            httpGet.setHeader("Content-Type",CONTENT_TYPE_TEXT_JSON);
            RequestConfig.Builder builder = RequestConfig.custom();
            builder.setSocketTimeout(3000)  //设置客户端等待服务端返回数据的超时时间
                    .setConnectTimeout(1000) // 设置客户端发起TCP连接请求的超时时间
                    .setConnectionRequestTimeout(3000); // 设置客户端从连接池获取链接的超时时间
            httpGet.setConfig(builder.build());
            // 发起请求
            response = httpClient.execute(httpGet);
            // 解析结果
            if(200==response.getStatusLine().getStatusCode()){
                String strResult = EntityUtils.toString(response.getEntity());
                JSONObject jsonObject = JSON.parseObject(strResult);
                System.out.println(jsonObject);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            response.close();
        }
    }
}

```

### buildRequestConfig

```java
public void requestConfig() {
        //新建一个RequestConfig
        RequestConfig defaultConfig = RequestConfig.custom()
                // 连接目标服务器超时时间 ConnectTimeout表示:连接一个URL的连接等到时间
                .setConnectTimeout(5000)
                // 读取目标服务器超时时间 SocketTimeout表示:连接上一个url,获取response的返回等待时间
                .setSocketTimeout(5000)
                // 从连接池获取连接的超时时间 ConnectionRequestTimeout
                .setConnectionRequestTimeout(5000)
                .build();
        // 超时可以设置为客户端级别,作为所有请求的默认值。
        CloseableHttpClient httpclient = HttpClients.custom()
                .setDefaultRequestConfig(defaultConfig)
                .build();
        // 执行的之后可以让httppost直接使用httpclient中的默认设置。
        httpclient.execute(httppost);

        // httpget可以单独的使用新的RequestConfig请求配置,不会对别的request请求产生影响
        HttpGet httpGet = new HttpGet("www.baidu.com");
        RequestConfig getConfig = RequestConfig.copy(defaultConfig)
                .setProxy(new HttpHost("myproxxy", 8080))
                .setConnectTimeout(2000)
                .build();
        httpGet.setConfig(getConfig)
    }
```
