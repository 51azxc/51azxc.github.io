title: "Java 生成验证码"
date: 2015-04-24 17:43:15
tags: ["servlet", "jsp"]
categories: "java"
---

> [java jsp实现登录验证码](http://a455360448201209214217.iteye.com/blog/1953785)
> [Java生成图片验证码](http://wuzhengfei.iteye.com/blog/647754)
> [struts2的验证码及利用jquery发送ajax请求并利用json做数据交换](http://blog.csdn.net/confirmaname/article/details/9254611)
> [struts2 实现图片验证码](http://tmq.iteye.com/blog/286022)
> [Struts2 验证码图片实例](http://www.cnblogs.com/dongliyang/archive/2012/08/24/2654431.html)

生成验证码的类
```java
public class ImageCode {
	//图片宽度
	private int width = 100;
	//图片高度
	private int height = 30;

	private Random random = new Random();
	
	//生成随机颜色 fc-前景色  bc-后景色
	public Color getColor(int fc,int bc){
		if(fc > 255) fc = 255;
		if(bc > 255) bc = 255;
		int r = fc + random.nextInt(bc - fc);
		int g = fc + random.nextInt(bc - fc);
		int b = fc + random.nextInt(bc - fc);
		return new Color(r, g, b);
	}
	//生成干扰线
	public void drawLine(Graphics2D g,int num){
		g.setColor(this.getColor(100, 200));
		for(int i=0;i<num;i++){
			int x1 = random.nextInt(width);
			int y1 = random.nextInt(height);
			int x2 = random.nextInt(10);
			int y2 = random.nextInt(10);
			g.drawLine(x1, y1, x2, y2);
		}
	}
	//生成验证码
	public String drawString(Graphics2D g,int num){
		StringBuffer sb = new StringBuffer();
		String str = "";
		int no = 0;
		for(int i=0;i<num;i++){
			switch(random.nextInt(3)){
			case 1:
				no = random.nextInt(26)+65;	 //A-Z
				str = String.valueOf((char)no);
				break;
			case 2:
				no = random.nextInt(26)+97;  //a-z
				str = String.valueOf((char)no);
				break;
			default:
				no = random.nextInt(10)+48;
				str = String.valueOf((char)no);
				break;
			}
			Color color = new Color(20+random.nextInt(20) , 20+random.nextInt(20) ,20+random.nextInt(20) );
            g.setColor(color);
            //想文字旋转一定的角度  
            AffineTransform trans = new AffineTransform();
            trans.rotate(random.nextInt(45)*3.14/180, 15*i+8, 7);
            //缩放文字  
            float scaleSize = random.nextFloat() + 0.8f;
            if(scaleSize>1f)  
                scaleSize = 1f ;  
            trans.scale(scaleSize, scaleSize);
            g.setTransform(trans);
            g.drawString(str, 15*i+18, 14);
            
            sb.append(str);
		}
		g.dispose();
		return sb.toString();
	}
    //getter / setter ···
}
```
发送图片的servlet：
```java
@WebServlet("/CodeServlet")
public class CodeServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
    public CodeServlet() {
        super();
    }

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//设置不缓存图片
		response.setHeader("Pragma", "No-cache");
		response.setHeader("Cache-Control", "No-cache");
		response.setDateHeader("Expires", 0);
		response.setContentType("image/ipeg");
		ImageCode ic = new ImageCode();
		BufferedImage bi = new BufferedImage(ic.getWidth(), ic.getHeight(), BufferedImage.TYPE_INT_RGB);
		Graphics2D g = bi.createGraphics();
		//定义字体
		Font f = new Font("Times New Roman", Font.BOLD, 18);
		g.setFont(f);
		g.setColor(ic.getColor(123, 222));
		//绘制背景
		g.fillRect(0, 0, ic.getWidth(), ic.getHeight());
		g.setColor(ic.getColor(11, 111));
		ic.drawLine(g, 24);
		String code = ic.drawString(g, 4);
		System.out.println("code: "+code);
		//将验证码存入session中
		HttpSession session = request.getSession();
		session.setAttribute("code", code);
		g.dispose();
		ImageIO.write(bi, "JPEG", response.getOutputStream());
		response.getOutputStream().flush();    
		response.getOutputStream().close();    
		response.flushBuffer();
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		String code = request.getParameter("code");
		String checkCode = (String) request.getSession().getAttribute("code");
		if(code.equals(checkCode)){
			System.out.println("same");
		}
		request.getRequestDispatcher("index.jsp").forward(request, response);
	}

}
```
jsp页面部分:
```html
<form action="<%=request.getContextPath()%>/CodeServlet" method="post">
<input type="text" id="code" name="code">
<img id="image" src="<%=request.getContextPath()%>/CodeServlet" title="点击更换图片" onclick="refresh()">
<input type="submit" id="btn" value="submit">
</form>
<script type="text/javascript">
  function refresh(){
	  //添加日期用于刷新图片
	  document.getElementById("image").src="<%=request.getContextPath()%>/CodeServlet?"+new Date();
  }
</script>
```

