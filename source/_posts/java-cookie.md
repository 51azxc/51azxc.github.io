title: "Java Cookie使用"
date: 2015-04-24 16:01:48
tags: ["cookie", "servlet"]
categories: "java"
---

> [JSP Cookie 使用完全详解](http://blog.csdn.net/xiaoyycq/article/details/3891266)

**cookies**是一种WEB服务器通过 浏览器在访问者的硬盘上存储信息的手段,当用户再次访问某个站点时，服务端将要求浏览器查找并返回先前发送的Cookie信息，来识别这个用户。cookie中的名字和值都不能包含空白字符以及下列字符：`@ : ;? , " / [ ] ( ) = `

**cookie使用方法**
```java
//获取cookie
Cookie cs[] = request.getCookies();
for(int i=0,len=cs.length-1;i<len;i++){
	Cookie cc = cs[i];
	System.out.println(cc.getName()+":"+cc.getValue());
}
//存入cookie
Cookie c = new Cookie("userName", "a");
c.setMaxAge(-1);  //设置存活时间0为删除，负数为浏览器关闭就删除,单位为秒
response.addCookie(c);
```
