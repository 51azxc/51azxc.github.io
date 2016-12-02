title: "jdbc部分练习"
date: 2015-04-25 00:07:12
tags: "jdbc"
categories: "java"
---

#### jdbc连接oracle 11g
> [jdbc连接oracle 11g](http://blog.csdn.net/linshizhan/article/details/7760465)

ojdbc5.jar表示JDK5.0使用的jdbc包，ojdbc6.jar表示JDK6.0使用的包。建立连接的主要代码如下
```java
Class.forName("oracle.jdbc.driver.OracleDriver");  
Connection connection = DriverManager.getConnection("jdbc:oracle:thin:localhost:1521:orcl",username,password);
```

#### java图片存入oracle blob字段
> [【java】java读取图像文件存入oracle中blob字段源代码](http://duduli.iteye.com/blog/1709051)

```java
File file = new File(fileName);
Class.forName("oracle.jdbc.driver.OracleDriver");  
Connection connection = DriverManager.getConnection("jdbc:oracle:thin:localhost:1521:orcl",username,password);
conn.setAutoCommit(false);
BLOB blob = null;
PreparedStatement pstmt = conn.prepareStatement("insert into blobtest(name,content) values(?,empty_blob())");
pstmt.setString(1, fileName);
pstmt.executeUpdate();
pstmt.close();
pstmt = conn.prepareStatement("select content from blobtest where name= ? for update");
pstmt.setString(1, fileName);

ResultSet rset = pstmt.executeQuery();
if (rset.next()) blob = (BLOB) rset.getBlob(1);
FileInputStream fin = new FileInputStream(file);
pstmt = conn.prepareStatement("update blobtest set content=? where name=?");

OutputStream out = blob.getBinaryOutputStream();
byte[] data = new byte[(int) fin.available()];
fin.read(data);
out.write(data);
fin.close();
out.close();
pstmt.setBlob(1, blob);
pstmt.setString(2, fileName);
pstmt.executeUpdate();
pstmt.close();
conn.commit();
conn.close();
```
