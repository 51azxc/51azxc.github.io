title: "Git 相关知识点收集"
date: 2015-04-22 21:52:41
tags: "git"
categories: "其他"
---

这里收集了一些关于Git的相关命令及一些Git组件的问题解决办法。

<!-- more -->

### Gitblit

> [使用Gitblit 在windows 上部署你的Git Server](http://www.cnblogs.com/keyindex/archive/2012/07/17/2594435.html)
> [Gitblit 配置](http://ttcool.blog.51cto.com/1186572/1346998)
> [官方网站](http://www.gitblit.com/)

主要修改**data/gitblit.properties**配置文件
```ini
git.repositoriesFolder #存放Git库的文件夹
server.httpPort = 10010 #http访问端口
server.httpsPort = 0 #设置为0则为默认禁用连接(https)
server.httpBindInterface = ip或域名
server.httpsBindInterface = localhost #如果禁用https，这项无需配置
```

最好使用Gitblit Go,用SSH连接方式更快，用HTTP的方式连接很慢，不知为何。

----

### Git相关命令

> [使用Git将本地代码上传到GitHub](http://blog.csdn.net/u010520912/article/details/18993001)
> [怎么修改Git remote add时使用的远程仓库](http://www.douban.com/group/topic/33666661/)
> [git怎么提交已经修改或者新增的文件？](http://www.oschina.net/question/778987_122007)
> [Git fetch和git pull的区别](http://blog.csdn.net/hudashi/article/details/7664457)
> [Git下的冲突解决](http://www.cnblogs.com/sinojelly/archive/2011/08/07/2130172.html)

#### 上传流程

**配置git版本管理工具**
打开命令行或Git Bash，输入下面的命令：
```bash
git config --global user.name “YourName” 
git config --global user.email “YouEmailAddress”
```
若省略了“--global”，则只配置当前仓库用户信息

**使用SSH密钥进行认证**
打开Git Bash（window下右击桌面菜单Git Bash选项/mac下直接使用终端），输入下面的命令：
```bash
ssh-keygen -C “YouEmailAddress” -t rsa
```
然后直接按回车使用默认路径保存密钥文件到当前用户文件夹中
密钥文件为（.ssh），接着设置密码和再次输入密码(可选)
到该目录找到.ssh文件夹（`id_rsa`为私钥文件，`id_rsa.pub`为公钥文件）
使用笔记本打开`id_rsa.pub`文件并复制文件内容,到GitHub中点击右上角的account settings然后选择左边栏中的SSH Keys添加SHH Key粘贴刚才复制的内容到Key文本框中，title文本框随意填写

**创建本地仓库并上传**
使用CMD或者Git-Bash跳转到本地仓库的目录下输入`git init`命令初始化一个仓库。
然后输入命令`git add .`意为将本目录下所有的文件添加到仓库。
接着输入`git commit -m "first commit"`将提交代码，`-m`后面跟的是提交的注释。
输入命令`git remote add origin "git@github.com:YourName/YourRepositroy.git"`为添加源到gitHub。
最后上传代码的命令为`git push -u origin master`.

下载代码则为`git pull`。

如果需要修改远程仓库链接则是`git remote set-url origin URL`
```bash
git remote set-branches [--add] <name> <branch>...
git remote set-url [--push] <name> <newurl> [<oldurl>]
git remote set-url --add <name> <newurl>
git remote set-url --delete <name> <url>
```

如果要提交新增或者修改的文件则是
```bash
git add .
git add --update .
git commit -am "add or update" 
```

#### Git冲突解决

**rebase的冲突解决**
rebase的冲突解决过程，就是解决每个应用补丁冲突的过程。
解决完一个补丁应用的冲突后，执行下面命令标记冲突已解决（也就是把修改内容加入缓存）：
```bash
git add -u
```
-u 表示把所有已track的文件的新的修改加入缓存，但不加入新的文件。
然后执行下面命令继续rebase：
```bash
git rebase --continue
```
有冲突继续解决，重复这这些步骤，直到rebase完成。
如果中间遇到某个补丁不需要应用，可以用下面命令忽略：
```bash
git rebase --skip
```
如果想回到rebase执行之前的状态，可以执行：
```bash
git rebase --abort
```
rebase之后，不需要执行commit，也不存在新的修改需要提交，都是git自动完成。

#### git fetch与git pull的区别
1. `git fetch`：相当于是从远程获取最新版本到本地，不会自动merge
```bash
git fetch origin master
git log -p master..origin/master
git merge origin/master
```
以上命令的含义：
   首先从远程的origin的master主分支下载最新的版本到origin/master分支上
   然后比较本地的master分支和origin/master分支的差别
   最后进行合并
   上述过程其实可以用以下更清晰的方式来进行：
```bash
git fetch origin master:tmp
git diff tmp 
git merge tmp
```
从远程获取最新的版本到本地的test分支上
   之后再进行比较合并
2. `git pull`：相当于是从远程获取最新版本并merge到本地
```bash
git pull origin master
```
上述命令其实相当于git fetch 和 git merge
在实际使用中，git fetch更安全一些
因为在merge前，我们可以查看更新情况，然后再决定是否合并

#### git查看分支

> [git 查看远程分支、本地分支、创建分支、把分支推到远程repository、删除本地分支](http://blog.csdn.net/arkblue/article/details/9568249)

* 查看远程分支: `git branch -a`
* 查看本地分支: `git branch`
* 创建分支： `git branch <name>`
* 切换分支: `git checkout <name>`
* 删除本地分支: `git branch -d <name>`
* 删除远程分支: `git push origin --delete <name>`

----

### Hexo 同时支持Github和Gitcafe

> [Hexo 同时支持Github和Gitcafe](http://colobu.com/2014/10/13/hexo-supports-both-github-and-gitcafe/)

Hexo支持同时发布到多个git仓库中。需要修改_config.yml。
原来的配置:
```
deploy:
type: github
repo: github: https://github.com/<username>/<username>.github.io.git
branch: master
```
改成
```
deploy:
   type: git
   repo: 
      github: https://github.com/<username>/<username>.github.io.git,master
      gitcafe: https://gitcafe.com/<username>/<username>.git,gitcafe-pages
```

首先需要你在gitcafe创建一个和用户名相同的项目，并为此项目创建一个gitcafe-pages。 静态站点发布到这个分支上。 同时需要绑定你的域名在此项目上。
这和github有点不同。 github要求创建一个<username>.github.io的项目，站点发布到master分支即可。

----

### `.gitignore`配置

> [Git的`.gitignore`配置](http://www.cnblogs.com/haiq/archive/2012/12/26/2833746.html)

建立`.gitignore`文件
```bash
touch .gitignore
```

1、配置语法：

* 以斜杠“/”开头表示目录；
* 以星号“*”通配多个字符；
* 以问号“?”通配单个字符
* 以方括号“[]”包含单个字符的匹配列表；
* 以叹号“!”表示不忽略(跟踪)匹配到的文件或目录；

  此外，git 对于 .ignore 配置文件是按行从上到下进行规则匹配的，意味着如果前面的规则匹配的范围更大，则后面的规则将不会生效；

2、示例：

（1）规则：`fd1/*`
　　 说明：忽略目录 fd1 下的全部内容；注意，不管是根目录下的 `/fd1/` 目录，还是某个子目录 `/child/fd1/` 目录，都会被忽略；

（2）规则：`/fd1/*`
　　 说明：忽略根目录下的 `/fd1/` 目录的全部内容；

（3）规则：
```ini
/*
!.gitignore
!/fw/bin/
!/fw/sf/
```
说明：忽略全部内容，但是不忽略 `.gitignore` 文件、根目录下的 `/fw/bin/` 和 `/fw/sf/` 目录；

----

### 移除submodule

> [How do I remove a submodule?](http://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule)

```bash
git submodule deinit submodule_name
git rm --cached submodule_name
rm -rf .git/modules/submodule_name
```

### submodule更新版本

如果修改了子模块里的内容，需要先到子模块目录里`git commit -am`然后成功`git push`到远端服务器上，接着在主项目目录下输入一下命令：
```bash
git submodule update --remote --merge
```
即可将子模块更新到最新版本，接着`push`主项目分支到远端服务器上即可。