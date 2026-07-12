##1. 使用git之前需要做的最小配置
#####配置user.name 和 user.email
```
git config --global user.name 'name'
git config --global user.email 'email'
```
#####config的三个作用域
```
缺省值等于 local
git config  --local  只对某个仓库有效
git config  --global  对当前用户所有仓库有效
git config  --system  对系统所有登陆的用户有效


显示config的配置 ,加 --list
例如: git config --list --local
```
local的优先级比global的优先级要高的

##2. 初始化仓库
```
git init
```

##3. 提交
工作目录  -->  暂存区 --> 版本历史
```
touch index.html
git status -- 查看仓库当前的状态，显示有变更的文件
git add index.html --添加文件到暂存区
git status 
git commit -m 'add index.html'  -- 提交暂存区到本地仓库
git log -- 查看历史提交记录

echo 'abc' >> index.html 
git diff 比较文件的不同，即暂存区和工作区的差异
git status
git add u
git commit -m 're index.html' 
```

##4. 给文件重命名
1. 危险的操作
```
mv index.html index.md
git add index.md
git add status
git rm index.html
```
2.git mv
```
git mv index.html index.md
git status 
git commit -m'rename index.html index.md'
```
##5. log版本历史
```
git log 无参数
git log --pretty=oneline  漂亮的查看
git log -p -1  展开显示每次提交的内容差异, 用 -1 则仅显示最近的两次更新
git log --stat  显示简要的增改行数统计
```
![git log常用参数](https://upload-images.jianshu.io/upload_images/9049859-897e182d1871225a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

贴个链接挺全的,学习一下:[git log 的使用](https://www.jianshu.com/p/0805b5d5d893)

##6. .git目录有什么

[Git目录为什么这么大](https://cloud.tencent.com/developer/article/1870732)


##7. 数数tree的个数
```
git init git_doc
mkdir doc
touch doc/readme
echo 'abc' >> doc/readme
git add doc
git status 
git commit -m 'add doc'
find .git/objects/ -type f  查看有什么
git cat-file -t e69de29  查看类型
git cat-file -p e69de29  查看内容
```
一共两个数,一个doc,一个readme ,然后四条记录,其余两条分别是 commit和readme的内容blob

[BLOB、Commit和Tree组件 ](https://www.jianshu.com/p/8659c9ae00cb)

##8. 分离头指针
HEAD指针不再指向分支，而是直接指向某个commit
[Git操作 — 67.分离头指针状态](https://www.jianshu.com/p/de099617455b)

##9.进一步理解HEAD和branch
[HEAD、master 与 branch ](https://www.jianshu.com/p/4219b6f62ce3)
HEAD指的就是 `.git/HEAD` 文件，它存储着当前working directory所处的某次commit

```
 git branch 查看有哪些分支
 git checkout -b newmaster master  创建一个分支,并切换到分支
 cat .git/HEAD 查看HEAD指向哪里

 git diff HEAD HEAD^1 比较HEAD和HEAD的父节点
```

##10. 怎么删除不需要的分支
HEAD 指向的 branch 不能删除。如果要删除 HEAD 指向的 branch，需要先用 checkout 把 HEAD 指向其他地方
```
git branch -av 查看分支
git checkout -b git rebase
```
##11. 修改最近一次的commit的message
```
git commit --amend
git log
```

##12. 修改老旧的commit的message(开发不用)
```
git rebase i  名称
```
##13. 怎么把连续的commit整合
[Git 合并多个 commit，保持历史简洁](https://cloud.tencent.com/developer/article/1690638)
```
git rebase i  名称
```

##14. 怎么把间隔的commit整合
##15. 怎么比较暂存区和HEAD所指commit的差异
```
vi readme.md
git add readme.md
git diff --cached 
```
##16. 怎么比较工作区和暂存区
[Git：git diff 命令详解 ](https://www.jianshu.com/p/80542dc3164e)
##17. 如何把暂存区恢复成和HEAD一样的内容
```
git reset HEAD
git diff --cached 
```
##18. 如何让工作区的文件恢复为暂存区(开发不用)
```
git checkout file
```
##19. 如何把暂存区部分文件恢复成和HEAD一样的内容
```
 git reset  HEAD  file
```
##20. 消除最近几次commit(开发不用)
```
 git reset --hard  commitid
```
##21. 看不同提交的指定文件的差异
```
git diff commitid commitid
git diff  temp  master
git diff commitid commitid -- <file>
```
##22. 正确删除文件
```
 git rm file
```
##23. 开发中临时加塞了紧急任务
```
git stash 先把已经修改的文件存起来
git stash pop  将代码追加到最新的提交之后
```
[git stash详解](https://blog.csdn.net/stone_yw/article/details/80795669)
[一次git stash pop引发的血案](https://blog.csdn.net/cteng/article/details/78538118?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-78538118-blog-80795669.pc_relevant_antiscanv2&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-78538118-blog-80795669.pc_relevant_antiscanv2)
##24. 如何指定不需要git管理的文件
添加.gitignore文件
##25. git的备份
##26. GitHub关键概念
1. 提交(commit)
无论何时你将一个或多个文件修改保存到Git的历史记录,你都会创建一个新的提交
2. 提交消息(commit message)
每次做出提交时,你需要提供一个消息,描述为什么要进行这种改动,当以后试图理解为什么实现特定的修改时,提交的这一消息是非常有用的
3. 分支(branch)
就是存放在一侧的独立的系列提交,你可以使用它来进行一个实验或者创建一个新的功能
4. 主分支(master branch)
无论你什么时候创建一个新的Git项目,都会创建一个默认的分支,称为主分支,这个分支一旦准备发布,你的工作则应完全停止
5. 功能分支(feature branch)
无论何时构建一个新的功能,都将创建一个分支,称为功能分支
6. 发布分支(release branch)
如果你有一个手动QA(质量管理)流程,或者为满足客户需求而必须支持旧版本的软件.你需要一个发布分支以存放必要的补丁或者更新记录.功能分支和发布分支没有任何技术差别,但是在和团队讨论项目时,区别两者时有用的
7. 合并(merge)
合并是将一个分支完成的全部工作归并到另一个分支,通常情况下是将一个功能分支合并到主分支
8. 标签(tag)
引用一个特定历史的提交.最常用于记录发布版本,据此你可以知道发布的是哪个版本的代码以及何时生成
9. 查看(check out)
找到一个不同版本的项目历史记录,以及时查看该时间点的文件
10. 拉请求(pull request)
最初,拉请求是用来请求别人复查已经完成的分支工作,并将它合并到主分支,现在,拉请求常用于在一个流程的早期阶段,用于讨论可能的功能
11. 提出问题(issue)
GitHub中有一个称为提出问题的功能,可以用来讨论功能,跟踪缺陷,或者两者兼备
12. 维基(wiki)
一个轻量级的web页面创建方式
13. 克隆(clone)
通常你要从GitHub下载一个项目的副本,这样你可以在本地工作
14. 分叉(fork)
有时候那你不具备改变一个项目的必要许可,如果你想对这样的一个项目提交修改,首先在GitHub上用你的用户账号复制这个项目.然后你可以克隆,修改,并使用拉请求将其提交回最初的项目
##27. 配置GitHub的公钥私钥
[git 之 配置github公钥私钥 ](https://www.jianshu.com/p/9746fc72bca7)

##28. 把本地仓库同步到GitHub
[git学习-如何将本地项目上传（同步）到github远程仓库](https://blog.csdn.net/vir_lee/article/details/80464408)

##29. 查看GitHub
1. 项目页面介绍
2. README.md文件
该文件的内容将显示在项目主页上文件夹和文件列表的正下方,该文件提供了项目说明和其他额外信息
3. 提交历史
提交历史是一个了解最新工作单元的很好的方法,这些小工作单元已经在任何给定的分支上完成
4. 拉请求
拉请求可使得你了解当前正在进展的工作
5. 问题
查看问题,可以使得你更广泛的了解一个项目任需要做的优秀工作
6. 脉冲
近期活动
7. 图表
