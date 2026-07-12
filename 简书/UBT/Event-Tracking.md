
##一. 埋点需求分析和设计
#####什么是埋点(Event Tracking)
埋点是一种常用的数据采集方法，是收集并记录用户行为数据的过程。
通过埋点收集用户行为的有效信息，用作统计页面加载和事件行为的数据支撑，比如访问量、点击率、跳出率等，同时埋点为数据运营提供基础，为未来的业务发展提供有力支持。
事件一般包含:WHO(用户ID,设备ID),WHEN(事件时间),WHERE(IP.地理位置,注册),WHAT(是什么,首页,二级页面),HOW(滑动,曝光,点击)
#####埋点的作用
1. 驱动决策：ABtest、漏斗优化、用户增长、bug修复、精准营销、流失用户预警
2. 驱动产品智能：智能推荐(千人千面)、场景化提示(私人助理)等
3. 驱动安全：风险识别

#####埋点的流程
梳理业务逻辑 --> 核心业务流程  --> 事件设计 --> 数据采集 --> 构建指标系统 --> 数据分析

#####设计数据埋点方案
[埋点方式](https://zhuanlan.zhihu.com/p/92348445)
[埋点系列](https://www.jianshu.com/p/250001a9e57b)
这篇博客案例非常值得学习,我也是从我们的数仓老师那找到的这个作者的博客
[数据驱动：从方法到实践](https://book.douban.com/subject/30168661/)
[如何做好数据埋点 ](https://www.jianshu.com/p/b0051361ecfa)
[当我们在谈论数据埋点时，我们在谈论些什么？](http://www.woshipm.com/data-analysis/4269149.html)
##二. 输出埋点需求文档
[如何科学地输出一份的埋点需求文档？](http://www.woshipm.com/data-analysis/4271491.html)

记得我们老板讲过trace系统是非常重要的,连接了大前端,数仓,算法的一个平台,甚至更多的参与方,trace系统要解决埋点混乱的问题,帮助不同角色的人准确的找到他们要的埋点,帮助业务准备的埋点.

#####什么是埋点需求文档
埋点文档是一个对我们之前埋点需求分析结果的方案落地，上一章里，我们知道埋点分为前端埋点和后端埋点，我们常说的埋点文档是指前端埋点文档，通常由产品经理进行梳理。

首先，埋点文档没有一个固定结构。不同平台，不同渠道，不同公司对于业务需求的不同，所产出的埋点方案都不同。但是通常需要包含：

● 应用标识：指当前应用的唯一标识，用来区分各数据来源于哪个的应用（站点）。
● 页面名称：即当前埋点所属的页面，有了这个页面位置，才能知道这个埋点属于哪个页面的数据。通常我们埋点文档中是根据页面来划分的。
● 埋点位置：即需要添加埋点的位置信息，比如一个收藏按钮，一个分享按钮，某一指定位的曝光区域等等。
● 埋点标识：每个埋点的位置有一个埋点标识，用来标记这个位置，有且仅有一个，不能出现标识重复的情况，不然统计到的数据一定有重复。
● 埋点参数：参数就是除了点击这个位置带来的点击量，你还想看到哪些信息。比如点击支付按钮时，需要附带支付方式，支付金额，支付流号等信息

#####如何写埋点文档
[如何写一份高质量的埋点文档 ](http://www.woshipm.com/pmd/1865530.html)
[埋点需求文档怎么写？](https://www.zhihu.com/question/40502836)
#####埋点文档实例
之前做过梳理SPM[超级位置模型SPM](https://www.jianshu.com/p/fd21e78d70be),一遍遍的点击查看,也不知道做的是否是准确的,哈哈哈,后来也不知道怎么样了.....但是现在的步骤还是人工的梳理埋点,具体的埋点文档我也没怎么看,看了作者写的示例....有两个点更新和维护怎么搞???,按照作者的梳理方式还是很麻烦啊,这里先整个文档以后再系统学习一下
##三. 埋点的框架设计及其准确性
#####数据采集流程
#####埋点上传机制
#####数据存储过程
#####常见埋点问题及原因
1. 平常数据稳定，突然过高
2. 平常数据稳定，报表数据过低
3. 数据异常
#####如何提升埋点质量
1. 关键行为的采集，使用后端埋点
2. 建立完善的埋点规范
[如何建立埋点规范？](http://www.woshipm.com/data-analysis/4297452.html)


##四. 从埋点系统搭建到数据可视化落地
![数据分析平台](https://upload-images.jianshu.io/upload_images/9049859-f9b0082bf7936e00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[如何设计出一个实用高效的埋点管理系统？](http://www.woshipm.com/data-analysis/5107308.html)
[从埋点系统搭建到数据可视化落地](https://www.jianshu.com/p/2b43c823d059)
#####为什么要构建埋点管理系统
#####埋点管理系统功能设计
常见第三方数据分析平台参考：

>growing io：[https://www.growingio.com/](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.growingio.com%2F)
神策：[https://www.sensorsdata.cn/](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.sensorsdata.cn%2F)
诸葛IO： [https://zhugeio.com/](https://links.jianshu.com/go?to=https%3A%2F%2Fzhugeio.com%2F)
talking data:[http://www.talkingdata.com/](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.talkingdata.com%2F)
友盟：[https://developer.umeng.com](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.umeng.com)
百度统计： [https://mtj.baidu.com/web/welcome/login](https://links.jianshu.com/go?to=https%3A%2F%2Fmtj.baidu.com%2Fweb%2Fwelcome%2Flogin)
>Google Analytics: [https://analytics.google.com](https://links.jianshu.com/go?to=https%3A%2F%2Fanalytics.google.com)
数猎天下DataHunter：[https://www.datahunter.cn/](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.datahunter.cn%2F)



