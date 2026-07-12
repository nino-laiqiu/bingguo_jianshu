#####1.本地库和远程库的交互
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9685f6f3f92a7df7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-2c04db482ceb9865.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2,本地初始化
在某个目录执行初始化

>git init (.git是隐藏目录 ls -lA)

设置签名(此签名与托管中心无任何关系,项目级别优先级大于系统优先级)
>git config  user.name   ....
>git config user.global   ...
(保存在.git.config)

>git config --global  user.name ...
>git config --global  user.global ...
(保存在./.git.config)

#####3.提交文件与查看状态

>git status (查看工作区 暂缓区)

>git add 文件名 (添加 从工作区到暂缓区)

>git commit 文件名(提交 暂缓区到本地库)  只有要到vim 命令添加本次操作的名称
或者 
>git commit -m "操作.....取名" 文件名

>git rm -- cached 还原撤回提交


#####4.前进与后退 删除文件找回

>git regfog  查看所有日志
>git log 查看日志()
>git log--omline 漂亮的查看日志

前进后退
索引   
HEAD^
HEAD~n

>git reset --hard ....

reset参数的说明:
-soft  仅在本地库移动head指针
-mixed 在本地库,及重置暂缓区
-hard 本地库 暂缓区 工作区

删除文件及找回:
前提删除前文件存在提交到本地库
操作依赖于 git reset --hard ...


比较
>git diff 文件名
比较:工作区的文件同其他区的计较 不带文件名比较多个文件


#####5.分支
git branch -v 查看
git branch  分支名  创建
getcheckout 分支名  切换

合并分支:
第一步:切换到被合并分支支名(追加新内容)
第二部:git merge 另一分支名

冲突的解决
![image.png](https://upload-images.jianshu.io/upload_images/9049859-cb0f8a53e590124e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####6.Git连接GitHub

>git remote -v 查看别名

>git remote add [别名]  [GitHub地址]  创建别名

>git push [别名] master 上传本地库

pull = fetch +merge 下载到分支
git fetch [远程库地址别名] [远程分支名]


#####7.ssh免密连接GitHub

>cd ~  删除掉从前建立的ssh

>ssh-keygen -t rsa -C 邮箱(邮箱????)

>cd ~/.ssh  (cat id_rsa.pub 复制公钥到GitHub的ssh窗口)-----这一步什么意思

>git remote add [别名] ssh地址(GitHub)

>git push [别名] master

##8.idea与Git

vcs ->创建本地库
git add gitgnore (取消添加到暂存区)
git add (增添暂存区)
git commit (提交到本地库)

切换版本 copy revision number (获取版本的哈希值)
git repository resetHEAD

创建分支 git repository branches(创建)
合并分支  git repository mergerchange(合并分支)

解决冲突(建议第三个)

克隆 vcs git clone URL(某个库)

#####9.如何删除GitHub上的文件目录

>git rm  -r --cached filename

>git commit -m "hehe"

>git push origin


##10.GitHub的简单命令

>关键字 in:name,readme,description(名字 文件 描述)

>关键字 stars:>90 forks:>90..100
