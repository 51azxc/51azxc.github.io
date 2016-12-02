title: 使用Cocoapods来进行项目依赖管理
date: 2015-10-26 17:06:33
tags: "Cocoapods"
categories: "iOS"
---

### 安装，使用Cocoapods

> [CocoaPods详解之----使用篇](http://blog.csdn.net/wzzvictory/article/details/18737437)
> [用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)
> [CocoaPods 安装 使用](http://www.jianshu.com/p/071d30a3af02)
> [CocoaPods报错：The dependency `AFNetworking ` is not used in any concrete target](http://blog.csdn.net/sjl_leaf/article/details/50506057)

#### 安装
在安装之前，先做一些准备工作。首先需要替换掉`Ruby`的默认源:
```bash
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/ 
```
然后可以输入命令`gem sources -l`验证是否替换成功。
若是`gem`版本低，可以使用`update`命令更新:
```bash
sudo gem update -n /usr/local/bin --system
```
接下来则是安装`Cocoapods`:
```bash
sudo gem install -n /usr/local/bin cocoapods
```

#### 使用
可以使用`search`命令搜寻需要的第三方库:
```bash
pod search 第三方库名称
```
在已经建立好的`Xcode`项目中加入**Podfile**文件，或者在终端中*cd*到项目路径中，然后运行`touch`命令建立该文件
```bash
touch Podfile
```
然后编辑**Podfile**文件
```bash
platform :ios, '9.0' #支持的系统版本
target 'MyApp' do
  pod 'AFNetworking', '~> 3.0'
end
```
编辑完保存之后在终端运行命令
```bash
pod install
```
待其构建好项目之后打开`MyApp.xcworkspace`文件即可。

----

#### 卸载Cocoapods

> [如何从电脑中卸载cocoapods](http://blog.csdn.net/qq_18670721/article/details/50432892)

首先使用命令查找`pod`的安装路径
```bash
which pod
```
然后删除这它:
```bash
sudo rm -rf /usr/local/bin/pod
```
这里的路径则是通过`which`命令找出来的。然后通过`gem list`命令来查找`gem`中的`Cocoapods`包
```bash
gem list
```
接下来将所有与`Cocoapods`有关的包移除掉
```bash
sudo gem uninstall cocoapods
```
如果有多个版本同时存在，终端会提示需要删除哪一个版本，按对应的数字即可。
这样`Cocoapods`就算是成功卸载了。
