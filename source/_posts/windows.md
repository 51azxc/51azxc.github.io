title: "Windows系统下的部分解决办法"
date: 2016-01-13 18:02:29
tags: ["windows","virtualbox"]
categories: "其他"
---

### Windows 7 修改登录界面

> [WIN7登陆界面怎么修改](http://jingyan.baidu.com/article/9f63fb91cf169dc8410f0e7c.html)

按快捷键`Win+R`或者打开**开始**菜单，点击**运行**,然后输入`cmd`开启命令行工具。接着在命令行工具输入`regedit`开启注册表编辑器。按以下路径点开:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\LogonUI\Background
```
然后在注册表编辑器右边的面板中找到键`OEMBackground`，没有可以新建一个。
将值改成**1**,保存。
打开`C:\Windows\System32\oobe\info\backgrounds`文件夹，没有对应的文件夹就新建。
然后准备一张与当前分辨率一致的图片，命名为`backgroundDefault.jpg`,图片大小最好保持在*250KB*以下。
按`Win+L`查看效果。

<!-- more -->

----

### Windows 7 重建图标缓存

> [win7 如何重建图标缓存](http://zhidao.baidu.com/question/175795438.html)

将以下文本复制到一个新建文本中，然后重命名为`bat`文件，即可双击运行
```bash
rem 关闭Windows外壳程序explorer

taskkill /f /im explorer.exe

rem 清理系统图标缓存数据库

attrib -h -s -r "%userprofile%\AppData\Local\IconCache.db"

del /f "%userprofile%\AppData\Local\IconCache.db"

attrib /s /d -h -s -r "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\*"

del /f "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\thumbcache_32.db"
del /f "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\thumbcache_96.db"
del /f "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\thumbcache_102.db"
del /f "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\thumbcache_256.db"
del /f "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\thumbcache_1024.db"
del /f "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\thumbcache_idx.db"
del /f "%userprofile%\AppData\Local\Microsoft\Windows\Explorer\thumbcache_sr.db"

rem 清理 系统托盘记忆的图标

echo y|reg delete "HKEY_CLASSES_ROOT\Local Settings\Software\Microsoft\Windows\CurrentVersion\TrayNotify" /v IconStreams
echo y|reg delete "HKEY_CLASSES_ROOT\Local Settings\Software\Microsoft\Windows\CurrentVersion\TrayNotify" /v PastIconsStream

rem 重启Windows外壳程序explorer

start explorer
```

----

### 手动添加与删除Windows服务

> [如何手动添加和删除Windows服务](http://www.metsky.com/archives/571.html)

创建命令:
```bash
sc create [service name] [binPath= ] <option1> <option2>...
```
如:
```bash
sc create test binpath="C:\test.exe"
```

删除命令:
```bash
sc delete servicename
```

----

### 使用WinRAR解压7z分卷压缩文件

> [怎样用WinRAR解压7z.001，7z.002……格式的文件](http://blog.163.com/chenhao0528@yeah/blog/static/172439055201173010112167/)

**WinRAR**软件可以直接解压**7z**格式的文件，但是碰到7z格式的分卷压缩文件如“xx.7z.001,xx.7z.002···”这些就不能识别了，因此需要使用一个命令来解压。
开启命令行工具，定位到7z分卷压缩文件的文件目录下，然后运行`copy  /B filename.7z.*  filename.7z`命令即可将所有的7z分卷压缩文件合并成一个7z压缩文件，这样就可以使用WinRAR来解压了。

----

### virtual box 安装windows7 提示status: 0xc0000225错误

> [virtual box 安装windows7 提示status: 0xc0000225错误](http://liwpk.blog.163.com/blog/static/3632617020134153452495/)

处理此问题方法: **虚拟机 设置(setting)->系统(system)->主板(motherboard)->扩展特性**,勾选**启用 I/O APIC**

----

### Windows 7与Ubuntu双系统时卸载Ubuntu系统的方法

> [Win7与Ubuntu双系统时卸载Ubuntu的方法](http://www.linuxidc.com/Linux/2010-03/25129.htm)

1. 需要下载`MBRFix`工具,并且放置C盘中。
2. 在命令行中定位到C盘根目录。
3. 输入命令`MBRFix /drive 0 fixmbr /yes`
4. 重启之后直接进入`Win7`，然后使用系统自带的磁盘管理工具删除`Ubuntu`分区即可。
5. 如果直接在`Win7`中删除了`Ubuntu`分区，需要在**系统管理员权限命令行**中输入命令`mbrfix /dirve 0 fixmbr /yes`；重启之后直接进入`Win7`，然后使用系统自带的磁盘管理工具删除`Ubuntu`分

----

### 在WinPE下硬盘安装Win8

> [教你在winpe下硬盘全新安装win8方法](http://jujumao.org/forum.php?mod=viewthread&tid=834&highlight=win8)

1. 将`Win8`安装光盘解压到硬盘任意位置，如`D:\win8`。
2. 提取`Win8`安装光盘下的`boot`文件全部复制到`C`盘。
3. `C`盘新建文件夹`sources`。
4. 将`Win8`安装光盘下的`sources`文件夹下的`boot.win`复制到`C:\sources`下。
5. 将`Win8`安装光盘下的`bootmgr`文件复制到`C:\`下。
6. 在`WinPE`下运行命令行工具，输入命令`c:\boot\bootsect.exe /nt60 c:`;提示成功后重启计算机。
7. 重启后自动进入安装界面，选择安装语言之后，在**开始安装界面**，选择**修复安装**，然后选择**疑难问题**，点击**高级**,选择最后一项**命令提示符**，进入DOS窗口。
8. 输入命令`D:\win8\sources\setup.exe `，开始安装，**选择安装语言->自定义安装->格式化C盘->下一步**。

----

### Windows7/8.1获取文件夹最高权限

> [win7 需要来自SYSTEM的权限才能对文件更改](http://blog.sina.com.cn/s/blog_3d527ed00100zn50.html)
> [win8.1最高权限的设置方法](http://jingyan.baidu.com/article/fcb5aff7919247edaa4a713e.html)

1. 选中文件夹，右键菜单选择**属性**。
2. 在属性窗口中点击**安全**标签页，点击底部的**高级**按钮。
3. 在弹出的**高级安全设置**界面中，找到所有者，点击旁边的**更改**按钮。
4. 在弹出的**选择用户与组**界面中，在**输入要选择的对象名称**中输入需要更改的用户名，可以点击**检查名称**按钮看是否写对，如果不确定，可以点击底部的**高级..**按钮进入另一页面筛选出合适的用户。点击确认后关闭该页面。
5. 在**高级安全设置**界面中，点击**权限条目**中的`SYSTEM`主体，然后勾上**所有者**下方的**替换子内容和对象所有者**选项框，以及最下面的**使用可从此对象继承的权限项目替换所有子对象的权限项目**选项框，点击确定后关闭选项卡。