---
layout:     post
title:      An Introduction to Regular Expression
subtitle:   正则表达式指南
date:       2016-11-14
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Java
	- Regular Expression
    - An Introduction to
---

# 前言

#### 版本控制

版本控制系统是一套软件，可以追踪你对文件和目录所做的改变，主要用于源代码管理。以下场合你需要版本控制系统：
1. 你希望记录文件的变化，以便将来查阅或恢复到历史版本
2. 你需要与其他人协同工作，共享对文件和目录所做的改变

最知名的版本控制系统包括SVN和Git：
1. SVN是一个C/S架构的集中式版本控制系统，每个用户都与Server端的中心版本仓库进行交互。
2. 而Git作为一个分布式版本控制系统，每个用户都有自己的版本仓库，都保存着一份完整的拷贝。
3. 传统上，我们也会为Git指定一个主仓库，并且将所有对文件和目录的更新都合并到这一仓库中，这个主仓库就是我们后面将要介绍的Git服务器。

#### 准备工作

Git官网：<http://git-scm.com>   //这里有使用Git所需的一切

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

注2：只有user.name和user.email是必须配置的，配置core.editor只是为了方便，因为nano比较难用，editor主要用于编辑较长的提交说明。

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

对文件的改变有四种类型：new file, modified, deleted和renamed。new file处于未追踪状态，其余三种类型处于已追踪状态（已被纳入版本控制）。
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
#回车后会要求输入用户密码passwd，输入确认后会在当前目录克隆一个git_test仓库
git clone ssh://user@host/home/user/git_test
git clone user@host:~user/git_test
git clone user@host:git_test
git clone user@host:git_test.git
#将仓库克隆到repo目录而不是git_test
git clone user@host:git_test repo
```

#### 基本操作

1)	git remote & git remote -v  
通过git clone得到的仓库，除了工作区、暂存区和本地仓库，会多出[远程仓库]这样一个区域。对于通过git init创建的本地仓库，也可以通过git remote add命令将它与一个远程仓库关联起来。远程的信息存储在.git/config文件中。
```Bash
git remote             //查看关联的远程仓库，Git服务器默认的名字为origin
git remote –v          //查看远程仓库及其URL
git remote show origin   //查看更详细的信息，尤其是远程分支的信息
git remote add origin user@host:/path/to/repo   //添加远程仓库，相应地还有rename/rm
```

2)	git pull & git push  
远程仓库的存在主要是为了协同工作、共享改变，git pull和git push命令就是用来和远程仓库保持同步的命令。你对工作区的改变提交到本地仓库后，可以进一步使用git push推送到远程仓库；而别人推送到远程仓库的更新，你也可以通过git pull获取，示例如下：
```Bash
echo "Hello World" >> README
git add .
git commit –m "Edit file README"
git pull origin master     //每次push之前先执行pull是必要的
                      //否则，当服务器存在本地没有的更新时，push会失败
git push origin master   // origin是远程仓库的代号/别名，master是主分支的默认名称

#命令的一般格式
#":<destination>"省略时，push推送到同名分支（没有则新建），pull拉取到当前分支
git push/pull <remote-host> <source>:<destination>
```
push和pull的过程可能会产生冲突（conflict），一般需要手动编辑来解决。
Git push命令中+和-f的含义：
1. \+ means allowing non-fast-forward updates,
2. \-f 强制push，server端将丢失commits
3. 联想git merge的--no-ff选项要求不采用快进方式（试图直接移动HEAD指针）

3)	git branch & git merge  
Git的分支就是一个指向commit id的指针，新建仓库时都会默认创建master分支，可用git branch -a查看所有分支(本地&远程)，形如origin/master的分支称为远程追踪分支，它存在于本地并引用/指向远程分支（位于Git服务器），不混淆的情况下也直接称origin/master为远程分支。git fetch origin会建立/更新所有远程跟踪分支，然后作为起始点新建分支。
常见分支操作如下：
```Bash
git branch          //查看本地分支，-r查看远程分支，-a查看所有分支
git branch xxx       //新建分支，一般不在master分支开发，新建xxx分支进行开发
git pull origin xxx    //获取分支的另一种方法是从服务器拉取
git checkout xxx     //由master分支切换到xxx分支
git checkout -b dev   //一步到位，新建xxx分支并切换到xxx分支
---------------------- 在xxx分支上进行发ing ----------------------
git push origin xxx    //将xxx分支推送到远程仓库，如果有必要的话
git checkout master   //切换回master分支，会恢复工作区到master分支的内容
git diff master xxx    //查看xxx分支相对于master分支的改变
git branch --merged   //当前分支包含哪些分支
git merge xxx        //将xxx分支的改变合并到master分支
git merge --abort      //合并冲突的消极处理，即撤销本次合并，应手动编辑解决
git branch -d xxx     //开发全部完成后，可以删除dev分支
git branch -D xxx     //强制删除，-D相当于-df
git branch -m xxx ooo    //移动/重命名分支
git push origin --delete xxx  //从服务器上删除xxx分支
```
每个分支都有独立的工作区和本地仓库，但是他们共享暂存区，所以从分支branch1切换到branch2时，branch1中未提交的更改对banch2也可见，而这通常不是我们所期望的，解决方法包括切换分支前提交所有更改，或者通过git stash解决。

4)	git stash  
将当前工作区和暂存区状态存储起来，并将工作区和暂存区恢复到上一次提交的状态（相当于执行了git reset hard HEAD）。
```Bash
git stash [save] "message"
git stash list
git stash show –p|--patch
git stash drop
git stash clear
git stash pop|apply stash@{0}
```

5)	git tag  
对某一个时间点上的版本打一个标签来标识版本号，发布某个版本时经常这样做。
```Bash
git tag                 //列出所有标签
git tag –l '1.0.*'      //列出特定标签
git tag v1.4-lw         //新建轻量级标签，它仅是一个指向特定提交的引用
git tag –a v1.4 –m "version 1.4 for release"  //新建带附注annotated的标签，它是一个Git对象
git show v1.4       //查看标签
git tag –a v1.2 commit_id   //后期加注标签
git push origin v1.2        //分享标签v1.2，--tags选项分享所有标签
```

6)	git revert & git reset  
Git维持着一个HEAD指针，它指向当前分支的最新位置。文件.git/HEAD中存储着当前分支(ref: refs/heads/current_branch)，文件.git/refs/heads/current_branch存储着最新位置(最后一次提交的commit id)。版本是由commit id标识的，因而版本[回退]也要根据commit id(或HEAD指针)来操作：
```Bash
git revert HEAD   //回撤最后一次提交，会保留提交日志，并新增一条回退日志
git revert HEAD^  //回撤最后两次提交
git revert HEAD~3  //回撤最后三次提交
git revert HEAD~4  //回撤最后四次提交
……
```
版本回退离不开git log [--oneline]命令，每次回退都应该查看提交日志！

与git revert类似，git reset用于[重置]到之前的某个版本，也就是将HEAD指针移动到之前的某次提交，而那次之后的所有提交（包括提交日志）都会被丢弃(并不会立刻消失，在产生新的提交之前你还可以用reset指回去，前提是你提前备份了commit id)。git reset不会影响远程仓库，虽然带-f选项的git push会强制覆盖远程仓库，但显然是不推荐的。
```Bash
git reset HEAD     //重置HEAD指针，指向最后一次提交
git reset commit_id  //重置HEAD指针，指向commit id
git reset HEAD^    //重置HEAD指针，指向倒数第二次提交
git reset --soft HEAD~3  //--soft选项重置HEAD指针，除此之外不做任何事
git reset [--mix] HEAD~3  //--mix是默认选项，重置并同时改变暂存区以匹配仓库
git reset --hard HEAD~3  //--hard选项重置并同时改变暂存区和工作区以匹配仓库
```

注1：git_test && git_test.git  
在本章开头的示例中，我们提到git_test和git_test.git都可作为仓库名。实际上远程仓库git_test是一个裸仓库，裸仓库指的是没有工作区的仓库，可用git init --bare初始化一个裸仓库。按照Git的惯例，裸仓库命名常以.git结尾，但这不是必须的。

只有裸仓库才能接受git push推送，因此Git服务器上的仓库都应该是裸仓库。在Git命令中用到裸仓库时，扩展名.git是可选的，因此例子中用的都是git_test，也可以写成git_test.git。

注2：SSH协议的认证方式  
在本章开头的示例中，我们以用户名(user)+密码(passwd)的方式访问远程主机，这时如果要实现访问控制，就需要远程主机为每个客户新建一个账号。实际上，SSH协议还提供了一种更加方便的认证方式，即公钥(xxx.pub)+私钥(xxx)的方式。

用户可使用ssh-keygen命令生成自己的公钥和私钥，默认会在~/.ssh目录下生成公钥id_rsa.pub和私钥id_rsa。只需将公钥发送给远程主机，由远程主机管理员将公钥内容附加到~user/.ssh/authorized_keys文件中，下次访问即无需密码。

注3：虚拟机作为远程仓库  
当Git服务器不是一台实际的电脑，而是一台虚拟机时。要使虚拟机成为局域网中的一员，能被局域网用户访问，比较方便的做法是：在[虚拟机—>设置—>网络适配器]中将网络连接设置成[桥接模式]，然后开启虚拟机并正确配置其IP地址。

注4：tracking branch && upstream branch  
设置本地分支追踪某个远程分支，就可以直接拉取pull（=抓取fetch+合并merge）
1. git clone的仓库会自动创建tracking branch
2. 第一次push加-u可创建tracking branch
3. git branch –u origin/branch1修改上游分支
