title: "Java EE实现定时任务"
date: 2015-04-24 13:31:06
tags: "timer"
categories: "java"
---


### 通过Listener实现定时任务

> [J2EE实现计划任务](http://xml.iteye.com/blog/341311)
> [使用JAVA在TOMCAT下实现计划任务监听器](http://blog.csdn.net/rongdajian/article/details/3296772)

首先定义一个`TimerTask`的子类，利用其中`run`方法运行相关的任务
```java
public class MyTask extends TimerTask {
	@Override
	public void run() {
		System.out.println("Hello World");
	}
}
```

然后在`ServletContextListener`中运行定义的`TimerTask`子类
```java
@WebListener
public class BaseListener implements ServletContextListener {

	private Timer timer = null;
	private MyTask myTask = null;
	
    public BaseListener() {
    }
    public void contextDestroyed(ServletContextEvent arg0)  { 
    	myTask.cancel();	//终止定时器
    }
    public void contextInitialized(ServletContextEvent arg0)  { 
    	if(myTask==null){
    		Calendar cal = Calendar.getInstance();
    		cal.set(Calendar.HOUR, 12);
    		myTask = new MyTask();
    		timer = new Timer();
    		timer.schedule(myTask, cal.getTime());
    	}
    }
}
```

----

### `Timer`的`schedule`方法与`scheduleAtFixedRate`方法的区别

> [schedule和scheduleAtFixedRate区别](http://blog.sina.com.cn/s/blog_69c5534d01010eho.html)

`schedule`和`scheduleAtFixedRate`的区别在于，如果指定开始执行的时间在当前系统运行时间之前，`scheduleAtFixedRate`会把已经过去的时间也作为周期执行，而`schedule`不会把过去的时间算上。
需要注意的是`scheduleAtFixedRate`和`schedule`在参数完全相同的情况下，执行效果是不同的。如果任务比较复杂，或者由于任何原因（如垃圾回收或其他后台活动）而延迟了某次执行，则`scheduleAtFixedRate`方法将快速连续地出现两次或更多的执行，从而使后续执行能够“追赶上来”；而`schedule`方法的后续执行也将被延迟。所以，在长期运行中，`scheduleAtFixedRate`执行的频率将正好是指定周期的倒数，`schedule` 执行的频率一般要稍慢于指定周期的倒数。

`schedule`和`scheduleAtFixedRate` 区别：
（1） 2个参数的`schedule`在制定任务计划时， 如果指定的计划执行时间`scheduledExecutionTime<= systemCurrentTime`，则task会被立即执行。`scheduledExecutionTime`不会因为某一个`task`的过度执行而改变。
（2） 3个参数的`schedule`在制定反复执行一个task的计划时，每一次执行这个task的计划执行时间随着前一次的实际执行时间而变，也就是 `scheduledExecutionTime(第n+1次)=realExecutionTime(第n次)+periodTime`。也就是说如果第n 次执行`task`时，由于某种原因这次执行时间过长，执行完后的`systemCurrentTime>= scheduledExecutionTime(第n+1次)`，则此时不做时隔等待，立即执行`第n+1次task`，而接下来的`第n+2次task的 scheduledExecutionTime(第n+2次)`就随着变成了`realExecutionTime(第n+1次)+periodTime`。这个方法更注重保持间隔时间的稳定。
（3）3个参数的`scheduleAtFixedRate`在制定反复执行一个`task`的计划时，每一次 执行这个`tas`k的计划执行时间在最初就被定下来了，也就是`scheduledExecutionTime(第n次)=firstExecuteTime +n*periodTime`；如果`第n次执行task`时，由于某种原因这次执行时间过长，执行完后的`systemCurrentTime>= scheduledExecutionTime(第n+1次)`，则此时不做`period`间隔等待，立即执`行第n+1次task`，而接下来的`第n+2次的 task的scheduledExecutionTime(第n+2次)`依然还是`firstExecuteTime+（n+2)*periodTime`这 在第一次执行task就定下来了。这个方法更注重保持执行频率的稳定。
