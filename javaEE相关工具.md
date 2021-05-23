# Maven

## 概念

​		maven：是一个**跨平台的项目管理工具**，主要服务于Java平台的**项目构建、依赖管理和项目信息管理**。

* 项目构建：通过插件帮你完成项目的清理、编译、测试、打包、部署。
* 依赖管理：通过坐标从maven仓库导入java类库（jar文件）
* 项目信息管理：项目描述、开发者列表、版本控制系统地址、许可证、缺陷管理系统地址等。



## 配置文件

maven里面有三个种配置文件：

* 全局配置文件：安装目录下 conf/settings.xml文件。安装目录可以在终端输入mvn -v获取。此settings.xml文件用于maven的所有project项目，它作为maven的全局配置。
* 用户配置文件：~/.m2/settings.xml文件。如需要个性配置，则需要在用户配置中设置。
* 项目配置文件：pom.xml文件是所在项目的局部配置。



Global Settings是默认包含的，User Settings 与pom.xml 可以在工程中自己指定。

​		配置优先级从高到低：pom.xml> User Settings > Global Settings 如果这些文件同时存在，在应用配置时，会合并它们的内容，如果有重复的配置，优先级高的配置会覆盖优先级低的。



## 生命周期

Maven中存在三种生命周期：clean、default、site，分别用于清理项目、构建项目、生成项目站点，而在一个生命周期中通常又会包含若干个阶段，下面是default构建生命周期的阶段：

![截屏2021-05-05 14.05.29](/Users/shiwen/Desktop/截屏2021-05-05 14.05.29.png)





# Git

## 常见命令

* 变更分支名：git branch -m name 
* 拷贝仓库代码：git clone ssh://git@git.sankuai.com/xy/xy-xg-marketing-portal.git



### 查看/提交

* 查看状态（还没有提交的更改）：git status
* 查看变更的地方：git diff  xxx
* 把文件添加到暂存区：git add xxx
* 把暂存区全部的文件提交：git commit  -m "你的评论，以便看出做了哪些修改"



* 查看当下的版本（操作）情况：git log  或者 git log --pretty=oneline    //从当前分支开始往旧看，不会往新看。且只能看到比当前分支小的分支，不能看到比当前分支大或者相等的其他分支
* 查看所有操作：git reflog



注意：同一个文件的多次修改也就是多次add到暂存区后，将会被合并为一个修改，commit将会把这个合并了的修改提交到本地版本库里



### 版本进退

* 版本后退一步：git reset --hard  head^
* 版本后退两步：git reset --hard  head^^
* 版本后退n步：git reset --hard  head～n
* 版本前进：git reset --hard  版本号     //注意：版本号可以通过git reflog查看所有变更获取，版本可以截取开头部分能唯一确定一个版本后即可



注意：移动版本时，如果前面的修改没有提交，将会被丢弃。

​           移动版本是版本库整体移动的，而非是对里面某个文件移动，是时间线的前进后退。



### 修改撤销

* 撤销暂存区（add之后）的修改，放回工作区：git reset head xxx 
* 撤销工作区的修改：git  checkout  --xxx  //回到最近的 add 或者 commit（add不存在时） 的状态




### 删除文件

* 删除工作区文件并放入暂存区：git rm xxx  



### 重命名文件

* 重命名工作区文件，并放入暂存区：git mv [file-original] [file-renamed] 



### 本地仓库与远程仓库的关联与取关

* 本地仓库关联远程仓库：git remote add origin git@github.com:Mr-ShiWen/learngit.git  这是通过ssh关联，如果不行可以通过https关联
* 本地仓库取关远程仓库：git remote rm origin
* 从远程仓库克隆：git clone 你的仓库地址（参考github仓库的code选项）//除了把远程仓库下载到当前外，还关联了二者



注意：仓库虽然关联了，但是如果需要push或者pull，还需要设置分支的关联



​    

### 分支创建/切换/关联

* 创建分支：git branch xxx      //创建名为xxx的分支
* 切换分支：git switch xxx   或者  git  checkout xxx   //切换到xxx分支
* 创建并切换分支：git switch -c xxx   或者  git checkout  -b xx   //创建名为xxx分支，并切换到xxx分支。相当于上面两条命令
* 创建一个跟踪远程xxx分支的同名本地分支：git switch --track remotes/origin/xxx
* 查看分支：git branch      //列出所有分支，当前分支前面会标一个 * 号
* 查看远程分支：git branch -a
* 关联远程分支：git branch --set-upstream-to=origin/远程分支名  本地分支名



### 分支合并/冲突/删除

* 合并当前分支到指定分支：git merge xxx     //把当前分支合并到指定分支，即让当前分支与指定分支一样，这只能是旧分支向新分支合并，不能反过来。所谓合并分支，是把另一个分支的最新版本的内容与当前分支的最新版本内容合并，迭代出当前分支的下一个版本。
* 删除分支：git branch -d xxx     //删除xxx分支
* 强行删除分支：git branch -D  xxx   //一般当分支未进行合并时，需要删除只能用强行删除
* 解决分支冲突：手动编辑为我们希望的内容，提交后删除一个分支或再合并



注意：

​		因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在`master`分支上工作效果是一样的，但过程更安全。

​		只有在两个分支都修改了，且内容不一致才会引起分支合并冲突。如果只是修改一个分支，虽然两分支内容不同但还是可以顺利合并。



### 分支现场保存/恢复

* 保存分支现场：git stash
* 查看分支现场：git stash list
* 恢复分支现场：【 git stash pop】  或者【 git stash apply 与 git stash drop】



### 分支推送

* 本地仓库向远程仓库推送：git push -u origin main  //这是第一次，需要加上 -u 

  ​											  git push  origin main   //后续，不需要加上 -u

  

* 推送命令格式：git push <远程主机名>  <本地分支名>:<远程分支名>

  ​						   git push <远程主机名> <本地分支名>     //不写远程分支名，默认推送到远程同名分支

  ​                		   git push    //当前分支推送到其关联的远程分支（需要同名）

* 拉取命令：git pull <远程主机名> <远程分支名>:<本地分支名>

   				  git pull <远程主机名> <远程分支名>  //不写本地分支名，默认拉取到当前分支

  ​				   git pull <远程主机名>   //   当前分支关联的远程分支拉取到当前分支  

  

  注意：上面提到的远程主机名一般是指 `origin` ，一般来说，本地分支与远程分支满足 **同名且关联** 的条件下，直接使用 	`git push / git pull` 即可，较为方便 



## 注意

本地分支的合并是把一个分支的内容合并进另一个分支，相同的内容丢弃，不同的内容保存并标示出来

![Git](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/Git.png)



push提交时，需要先 pull 拉取最新版本下来后处理冲突完后再提交，否则出错。



# vim

## 删除

* 要一次删除多行，请在dd命令前添加要删除的行数，例如，要删除五行，请执行以下操作：

  ​	1、按Esc键进入正常模式。

  ​	2、将光标放在要删除的第一行上。

  ​	3、键入5dd并按Enter键以删除接下来的五行。



## 移动

* 快速移到行首部，正常模式下：fn+左方向
* 快速移到行尾部，正常模式下：fn+右方向
* 快速移到首行：g+g
* 快速移到尾行：shift+g



## 查找

参考：https://harttle.land/2016/08/08/vim-search-in-file.html

* 查找：正常模式下  `/字符串`   	// 如  /foo 表示查找foo字符串
* 大小写敏感查找： /字符串\C
* 大小写不敏感查找： /字符串\c

