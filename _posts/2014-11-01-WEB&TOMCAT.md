---
layout: post
title: WEB&TOMCAT
categories: [tomcat, 容器]
description: web页面的启动容器tomcat
keywords: tomcat
---

<meta name="referrer" content="no-referrer"/>

​

### tomcat 的使用介绍

​

```javascript
出现这种情况的原因有下面几种
 1） 没有配置JAVA_HOME环境变量。判断方法在cmd窗口中
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635739249580-28e22e9a-e426-4369-b4da-e45fa8ef2b99.png#clientId=ua3e4fcc2-b981-4&from=paste&height=255&id=ud8040480&margin=%5Bobject%20Object%5D&name=image.png&originHeight=266&originWidth=812&originalType=binary&ratio=1&size=48152&status=done&style=none&taskId=u752968c2-4c83-4b92-85d9-dfcc73a5e3c&width=779)

```javascript
2）没有配置CATALINA_HOME，TOMCAT的环境变量
	解决方法一是配置对应的TOMCAT的环境变量
	不用配置该环境变量，修改start.bat 文件
	a) 在bat文件末尾添加pause命令，查看打印的log
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635739310525-b46c4d7b-58de-4b78-9e7b-14e3398209bd.png#clientId=ua3e4fcc2-b981-4&from=paste&height=259&id=uf9c0a1b6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=266&originWidth=811&originalType=binary&ratio=1&size=39805&status=done&style=none&taskId=ubd855df3-a9f8-422a-a250-0b83e556b25&width=789.5)

```javascript
b)当没有配置CATALINA_HOME的时候，设置为当前路径
	if not "%CATALINA_HOME%" == ""
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635739339954-a95bc74d-da84-49fb-b127-7c587161baca.png#clientId=ua3e4fcc2-b981-4&from=paste&height=260&id=u3200db8f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=268&originWidth=812&originalType=binary&ratio=1&size=42836&status=done&style=none&taskId=u3af1c5bc-91b5-4cbe-99ac-b4b536ed912&width=789)
