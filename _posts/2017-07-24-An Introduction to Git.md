---
layout:     post
title:      An Introduction to Git
subtitle:   Git使用指南
date:       2017-07-24
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Git
    - Gitolite
    - Ubuntu
    - An Introduction to
---

>本教程介绍Git的基本操作、基于Gitolite插件的权限管理，以及使用Git进行版本控制时的开发流程。


# 前言

#### 版本控制

版本控制系统是一套软件，可以追踪你对文件和目录所做的改变，主要用于源代码管理。以下场合你需要版本控制系统：
1. 你希望记录文件的变化，以便将来查阅或恢复到历史版本
2. 你需要与其他人协同工作，共享对文件和目录所做的改变

最知名的版本控制系统包括SVN和Git。
1. SVN是一个C/S架构的集中式版本控制系统，每个用户都与Server端的中心版本仓库进行交互。
2. 而Git作为一个分布式版本控制系统，每个用户都有自己的版本仓库，都保存着一份完整的拷贝。
3. 传统上，我们也会为Git指定一个主仓库，并且将所有对文件和目录的更新都合并到这一仓库中，这个主仓库就是我们后面将要介绍的Git服务器。

#### 准备工作

Git官网：http://git-scm.com   //这里有使用Git所需的一切

Git安装：apt install git       //执行git --version检验是否安装成功

Git配置：在使用git之前必须进行一项基本配置，即告诉Git你是谁(姓名和邮件)，这项工作由git config命令完成：
```Bash
git config --global user.name "xxx"       //--glabal选项表示用户级的配置
git config --global user.email xxx@zydz.com
git config --global core.editor gedit     //用gedit代替默认的nano编辑器
git config --alias.st status              //为status命令设置别名st
git config --list                         //列出所有配置
```

注1：此处一般都使用用户级的配置，此外还有系统级配置及项目级配置，如下所示

配置的级别|配置选项|配置文件
---------|--------|--------
系统级	  |--system |	/etc/gitconfig
用户级	  |--global	|~/.gitconfig
项目级	  |默认	    |.git/config

注2：只有user.name和user.email是必须配置的，配置core.editor只是为了方便，因为nano比较难用，editor主要用于编辑较长的提交说明（见2.1节）。

注3：Git原生环境是命令行界面，但是也有一些GUI工具可供使用，如Git自带的gitk和git gui、Windows下的tortoiseGit以及Linux下的GitKraken。

# Git本地仓库

获得一个Git仓库有两种方法，一种是使用git init命令将某个本地目录初始化为一个仓库，另一种是将远程仓库克隆到本地，本章介绍本地仓库相关操作，远程仓库则在下一章介绍。

#### 新建仓库

使用Git进行本地版本控制时，文件可能处于工作区(working directory)，暂存区(staging area)和本地仓库(reposotory)三个区域，对工作区所做的改变（change set）要用git add命令添加到暂存区，然后再用git commit提交到版本仓库。

下面将新建一个名为git_test的仓库并进行初次提交：
```Bash
#将某个目录初始化为Git的工作目录/[工作区]
mkdir git_test    //新建目录，目录名标识了版本仓库
cd git_test
git init          //此时会生成.git目录，删除该目录即可移除版本控制

#新建README文件并将其添加到[暂存区]，让Git开始追踪这个文件
touch README      //Git仓库一般应该有README、LICENSE、.gitignore文件
git add README    //此时.git目录下生成一个index文件，它就代表暂存区
                  //常用[git add .]递归地将当前目录所有改变添加到暂存区

#将暂存区的改变提交到[本地仓库]，以SHA-1哈希值的形式保存到.git/objects目录
git commit -m "Initial commit"   //-m选项用于指定提交说明(message)，指出该次提交做了什么
                                 //如果没有使用-m选项，Git会启动ditor供你输入提交说明
```

#### 基本操作

1)	git status  
用于查看工作区的状态，命令的输出常常包含以下关键词：
1. Untracked files：表示工作区有未被追踪的文件，可使用git add将其添加到暂存
2. Changes not statged for commit：表示工作区存在未添加到暂存区的改变，可使用git add将更改添加到暂存区
3. Changes to be commit：表示暂存区存在未被提交的改变，可使用git commit提交到本地仓库

对文件的改变有四种类型：new file, modified, deleted和renamed。new file处于未追踪状态，其余三种类型处于已追踪状态（已被纳入版本控制），
```Bash
#对于已追踪的文件所做的改变，可通过–am选项合并add和commit两个步骤
git commit –am "this commit does what"。
```

2)	git log  
用于查看提交日志，命令的输出包括commit id，Author <email>，Date以及message，其中commit id是通过SHA-1哈希算法产生的一个40字符的16进制SHA-1值，它标识了每一次提交。

```Bash
#产生简洁的日志输出
git log --oneline
#当log很多时，可以通过一些选项进行筛选
git log –5         				  //显示最近的5条log
git log --since="1 hour ago"
git log --until="2017-07-20 09:30"   //通过时间进行筛选
git log --author=somebody			//通过作者进行筛选
git log --grep="bug"				 //通过正则表达式筛选
git log –p                     //显示每次提交的内容差异
```

3)	git rm/git mv vs. rm/mv  
删除和重命名也是常见的操作，在命令行中执行删除和重命名操作时，使用git rm和git mv与直接使用linux自带的rm和mv的区别在于：
1. 使用git命令执行的操作同时作用于工作区和暂存区
2. 而rm/mv属于工作区的操作，因而还要通过git add添加这些改变到暂存区。
3. 带有--statged选项的git rm，它仅从暂存区删除某文件而在工作区保留该文件。

4)	git diff & git checkout  
git diff用于比较不同区域的文件，git checkout用于恢复工作区文件。
```Bash
git diff              //查看工作区相对于暂存区的改变
git diff –staged       //查看暂存区相对于本地仓库的改变，staged也写作cached
git checkout <path>    //撤销工作区更改，i.e.将暂存区的内容恢复到工作区
git checkout commit_id <path>   //将文件从特定版本恢复到暂存区待提交

#顺便说一下撤销暂存和修补提交
git reset HEAD <path>        //将暂存区的内容恢复到未暂存的状态(unstage)
                             //常用于重新组织更改形成一个有关联的commit
git commit –amend <file>     //修补最后一次提交，常用于添加改变或修改message
```

5)	git clean  
递归地删除工作区所有未被追踪的文件，通常需要结合-n和-f选项使用。

6)	git help  
查看帮助，如git help add查看命令git add的用法，这与man git-add/ git add –help的作用相同。

# Git远程仓库

更多的时候，我们不仅要对本地文件进行版本控制，还要通过远程仓库协同工作，共享我们对文件和目录所做的改变，这就涉及和远程仓库通信。Git支持SSH, http/https, Git等多种数据传输协议，其中SSH协议被广泛采用。

```Bash
#SSH初体验
ssh user@host pwd && ls   //先要有user和host,别傻傻运行这个例子
```

#### 克隆仓库

引入远程仓库后，使用以下命令我们可以获得一个初始仓库，示例如下：

```Bash
git clone ssh://user@host/path/to/repo  //host是Git服务器，user是host上的用户
git clone user@host:/path/to/repo      //scp格式的url写法，无ssh://，有分号
#以下四种写法作用相同
#它试图以user用户的身份，去克隆远程主机host上r用户家目录下的git_test仓库
#回车后会要求输入user用户的密码，输入确认后会在当前目录克隆一个git_test仓库
git clone ssh://user@host/home/user/git_test
git clone user@host:~user/git_test
git clone user@host:git_test
git clone user@host:git_test.git
#将仓库克隆到repo目录而不是git_test
git clone user123@192.168.1.2:git_test repo
```

#### 基本操作



# 权限管理

#### 访问规则

Gitolite采用gitolite.conf配置文件来设置访问权限，其中有很多类似类似于以下所示的访问控制列表：

```
repo 903-1-hwa-android                           //repo行
		RW+				  =	fye yifyang    //规则1
		-  	master		=	@all	       //规则2
		RW				   =	cliu           //规则3
		R					=	@all	       //规则4
```

访问控制列表由repo行和多条访问规则构成，这些访问规则依序检查，放在前面的被优先匹配，具体解释如下:

1. repo行：服务器端将自动创建一个名为903-1-hwa-android的仓库
2. 规则1：fye和yifyang对这个仓库有强制(+)读写权限，参见man git-push中关于+号的解释，fye和yifyang代表用户，实际上是用户.pub文件的名字
3. 规则2：禁止所有人推送到master分支，由于规则依序检查，这条规则并不影响fye和yifyang推送到master分支，@all为gitolite内建的组，在此代表所有用户
4. 规则3：cliu对这个仓库有读写权限，但是不能推送到master分支
5. 规则4：所有人都有读取权限

#### Gitolite命令

使用SSH协议登陆远程主机可以直接执行Linux命令，如ssh user@host pwd && ls。然而，当远程主机上安装了Gitolite之后，将只能执行Gitlolite命令。
```Bash
#查看Gitolite支持的命令
ssh git@host help
```

你将看到，Gitolite默认支持的命令是很少的，仅有7个，它们将在后文介绍。现在让我们关注一下图中红色下划线标注的内容：Gitolite以SSH公钥的名字来标识用户，输出“hello cliu”意味着当前用户公钥的名字为cliu.pub。
下面介绍常用命令：

1)	ssh git@host info  
显示Git服务器上有哪些仓库，以及当前用户的访问权限，你会看到类似如下的结果：

可以看到，服务器上一共有10个仓库，用户cliu对一些用户有读写权限（RW），对另一些仓库只有读权限（R），需要注意的是强制读写权限（RW+）中的+号并不会标出来。再一次，让我们关注一些红色下划线的内容，字母C代表create (新建)，表示用户可以在服务器上创建仓库。

2)	ssh git@host create  

3)	ssh git@host perms dev/alice/repo + WRITERS dave to add a user  

4)	ssh git@host perms dev/alice/repo - WRITERS dave to remove a user  

5)	ssh git@host perms -l dev/alice/repo to list current user lists  

6)	ssh git@host D unlock dev/alice/my-new-repo  

7)	ssh git@host D rm dev/alice/my-new-repo  


# 开发流程
