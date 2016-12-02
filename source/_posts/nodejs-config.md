title: "Node.js配置"
date: 2016-01-14 16:25:26
tags: "npm"
categories: "nodejs"
---

### Windows下安装Nodejs

> [Node.js环境搭建](http://www.cnblogs.com/terrylin/archive/2013/01/26/2877699.html)
> [Windows 系统下设置Nodejs NPM全局路径](http://www.cnblogs.com/picaso/p/3848209.html)

在`Node.js`[官网](https://nodejs.org/en/)默认给出的下载安装包是直接安装的，如果想获取免安装包需要点击下载下面的**Other Downloads**链接，选择*Windows Binary (.exe)*的文件下载即可。
下载了单文件之后放入到任意文件夹中，如`D:/nodejs`,将此路径加入到系统或用户的环境变量中。
打开命令行，输入命令`node -v`，如果有显示版本号，则说明配置成功。
下载`nodejs`包管理工具`npm`的[源代码](https://github.com/npm/npm),点击**Download ZIP**按钮下载后将文件解压到任意文件夹中，如`D:/nodejs/npm`,进入命令行输入如下命令安装`npm`:
```bash
node cli.js install -gf
```
待安装完成后输入命令`npm -v`，如果有显示版本号，则说明配置成功。
默认情况下,`npm`安装包以及缓存的路径在`${APPDATA}\npm`中,打开`node_modules\npm\.npmrc`文件，修改为以下内容:
```bash
prefix = D:\nodejs
cache = D:\nodejs\npm_cache
```
其中`prefix`为npm包的安装路径,`cache`则为下载包的缓存路径。

----

### npm卸载包

> [如何删除nodejs express](http://zhidao.baidu.com/link?url=c6zMFnprbEsKBl0Oid6Mak47dZjlU3zArA8_-8NczkEIkdoM2ecd8QhnZeh2NrcwMY1epEa4mAnNPTmGmENWifCJ0HKzV9rCE8o1UGjh1kC)

使用`npm`安装包的命令为`npm install package_name`,如果需要全局安装，则加一个参数`-g`,如果需要给项目安装，则加一个参数`--save`,它会自动将安装信息加入到项目`package.json`文件中。

删除包的命令为`npm uninstall package_name`,参数与安装的一致。

----

### Mac OSX下卸载Nodejs

> [Uninstall nodejs from OSX Yosemite](https://gist.github.com/TonyMtz/d75101d9bdf764c890ef)

打开命令行，输入命令：
```bash
sudo rm -rf /usr/local/lib/node* /usr/local/lib/dtrace/node.d
sudo rm -rf /usr/local/share/man/man1/node.1
sudo rm -rf /usr/local/include/node*  /usr/local/bin/node /usr/local/bin/npm
sudo rm -rf /usr/local/bin/node_modules
sudo rm -rf ~/.npm sudo rm -rf ~/.node-gyp sudo rm -rf ~/node_modules
sudo rm -rf /var/db/receipts/org.nodejs.*
```
如果全局安装了其他的组件，需要在`/usr/local/bin`目录下把其他组件的命令删除，可以使用`ls -lsa`命令显示出链接对应的路径，删除
```bash
cd /usr/local/bin
ls -lsa
```
如果安装的组件太多，可以借助强大的命令一次性搞定：
```bash
ls -l /usr/local/bin | grep '/lib/node_modules/' | awk '{print $9}' | xargs rm
```
首先按管道符分解命令:
```bash
ls -l /usr/local/bin

total 65144
lrwxr-xr-x  1 root   wheel        66  4  5  2016 2to3 -> ../../../Library/Frameworks/Python.framework/Versions/3.5/bin/2to3
lrwxr-xr-x  1 root   wheel        70  4  5  2016 2to3-3.5 -> ../../../Library/Frameworks/Python.framework/Versions/3.5/bin/2to3-3.5
-rwxr-xr-x  1 root   wheel        80  3  5  2016 VBoxAutostart
-rwxr-xr-x  1 root   wheel        82  3  5  2016 VBoxBalloonCtrl
-rwxr-xr-x  1 root   wheel        77  3  5  2016 VBoxDTrace
-rwxr-xr-x  1 root   wheel        79  3  5  2016 VBoxHeadless
-rwxr-xr-x  1 root   wheel        77  3  5  2016 VBoxManage
-rwxr-xr-x  1 root   wheel        79  3  5  2016 VBoxVRDP
-rwxr-xr-x  1 root   wheel        77  3  5  2016 VirtualBox
-rwxr-xr-x  1 admin  admin       656  4  5  2016 brew
lrwxr-xr-x  1 admin  admin        38  5  3  2016 carthage -> ../Cellar/carthage/0.16.2/bin/carthage
lrwxrwxr-x  1 root   admin        78  4  5  2016 easy_install-3.5 -> ../../../Library/Frameworks/Python.framework/Versions/3.5/bin/easy_install-3.5
......
```
这里把所有的链接都显示出来了，还有他们的目标文件，需要查找到`node`相关的组件，就需要借助`grep`命令来搜寻：
```bash
ls -l /usr/local/bin | grep '/lib/node_modules/'

lrwxr-xr-x  1 root   admin        49 11  8 17:28 node-supervisor -> ../lib/node_modules/supervisor/lib/cli-wrapper.js
lrwxr-xr-x  1 502    staff        38 11  8 17:15 npm -> ../lib/node_modules/npm/bin/npm-cli.js
lrwxr-xr-x  1 root   admin        49 11  8 17:28 supervisor -> ../lib/node_modules/supervisor/lib/cli-wrapper.js
......
```
这样已经把我们需要的组件给找出来了，接下来我们只需要获取到组件名字，就需要借助强大的`awk`命令:
```bash
ls -l /usr/local/bin | grep '/lib/node_modules/' | awk '{print $9}'

node-supervisor
npm
supervisor
......
```
`awk`命令默认按照换行符`\n`来分割记录，使用空格或者`Tab`来分割域，这里使用`{print $9}`则是打印出第九列的数据来，刚好是对应的命令链接名。

`xargs`默认将传入的参数中的换行去除，最终显示出来的则是:
```bash
ls -l /usr/local/bin | grep '/lib/node_modules/' | awk '{print $9}' | xargs

node-supervisor npm supervisor ......
```
最后配上删除命令`rm`删除即可
