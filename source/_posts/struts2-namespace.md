title: "struts2中不同namespace的重定向用法"
date: 2015-04-21 00:01:14
tags:
categories: ["java", "struts2"]
---

> [struts2中不同namespace的重定向用法](http://hi.baidu.com/wanglshen1/item/ab0c599a12b1a236326eeb50)

简单的同一个包或者namespace中：
```xml
<result name="success" type="redirectAction">xxx?a=1</result>
```

xxx 就是指你要重定向的action，?a=1 这个是参数的传递，你若是有参数的话可以这样传递
不同包或者不同namespace的
```xml
<result name="success" type="redirectAction">
  <param name="actionName">xxx</param>
  <param name="namespace">/xx</param>
</result>
```

xxx ：重定向的action
/xx: xxx所在的namespace
