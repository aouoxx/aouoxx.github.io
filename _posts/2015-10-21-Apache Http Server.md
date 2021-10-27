---
layout: post
title: Apache Http Server
categories: [apache, http]
description: Apache Http Server
keywords: apache, http
---

```java
设置Apache HTTP Server的文件根目录（DocumentRoot）
     安装Apache 时，系统会给定一个缺省的文件根目录
     如果你觉得这个网页存在这个缺省目录不方便，需要另外设个Apache 文件根目录，
     你可以修改Apache的配置文件httpd.conf里有关根目录的设置。

Apache HTTP Server的缺省文件根目录（DocumentRoot）是：
      DocumentRoot "C:\Program Files\Apache Software Foundation\Apache2.2\htdocs"

修改Apache 文件根目录（DocumentRoot）的操作如下：
   1）  为了避免修改失误，请先备份你的Apach配置文件httpd.conf，该配置文件的路径是
   2）  打开http.conf文件找到DocumentRoot为开头的那一行，将
          DocumentRoot "C:/Program Files/Apache Software Foundation/Apache2.2/htdocs"
          改成新的 DocumentRoot 路径，比如你新的路径为 C:\htdocs，就改成
          DocumentRoot "C:/htdocs"
   3）然后找到httpd.conf 文件中的如下内容：
               # This should be changed to whatever you set DocumentRoot to.
               #
               <Directory "C:/Program Files/Apache Software Foundation/Apache2.2/htdocs">
               将 Diectory 中的路径改成你新设的文件根目录，比如：
               <Directory "C:/htdocs">
    4）保存配置文件httpd.conf ，重启Apache Service

如果需要修改端口
 找到Listener 80 （--->修改为自己的端口即可）
```
