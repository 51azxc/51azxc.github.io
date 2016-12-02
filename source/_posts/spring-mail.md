title: "spring 发送邮件"
date: 2015-05-13 18:06:39
tags: "mail"
categories: ["java", "spring"]
---

> [邮件发送](http://liudong-1985.iteye.com/blog/780714)
> [java 利用spring JavaMailSenderImpl发送邮件，支持普通文本、附件、html、velocity模板](http://trinea.iteye.com/blog/1278334)
> [ Java Mail(二)：JavaMail介绍及发送一封简单邮件](http://blog.csdn.net/ghsau/article/details/17839983)

需要的jar包
```gradle
'javax.mail:mail:1.4.7',
'org.springframework:spring-context-support:4.1.5.RELEASE'
```
发送邮件
```java
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;

public void sendEmail() throws MessagingException {
	JavaMailSenderImpl mailSender = new JavaMailSenderImpl();
	MimeMessage mimeMessage = mailSender.createMimeMessage();
	
	FileSystemResource attach = new FileSystemResource(new File(EMAIL_PATH+fileName));
	MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true);
	
	//发送邮件地址
	mailSender.setHost(EMAIL_HOST);
	//发送人用户名
	mailSender.setUsername(EMAIL_USERNAME);
	//发送人密码
	mailSender.setPassword(EMAIL_PASSWORD);
	mailSender.setDefaultEncoding("GB2312");
	Properties properties = new Properties();
	// 如果为true,邮件服务器会去验证用户名和密码
    properties.put("mail.smtp.auth", "true");
    properties.put("mail.smtp.timeout", "25000");  
    mailSender.setJavaMailProperties(properties);
    
    //接收人
    helper.setTo(EMAIL_TO);
    //来自
    helper.setFrom(EMAIL_USERNAME);
    //邮件标题
    helper.setSubject("title");
    //邮件正文，第二个参数指定是否以html为脚本编辑邮件正文
    helper.setText("test", false);
    //添加附件
    helper.addAttachment(fileName, attach);
    发送邮件
	mailSender.send(mimeMessage);
}
```
