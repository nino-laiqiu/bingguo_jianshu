>来源于我的GitHub博客中的一篇,是对大数据系统实践这本书的简单总结
## (一).新范式

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%BC%A0%E7%BB%9F%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E4%B8%8D%E8%B6%B3 "传统数据库的不足")传统数据库的不足

*   数据库更不上负载:解决方法-用队列扩展

*   数据库再次超载:解决方法-通过数据库进行分片扩展

*   处理容错问题

*   损坏问题

    **系统必须是可以容忍人为错误的**

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%B3%BB%E7%BB%9F%E5%BA%94%E6%9C%89%E7%9A%84%E5%B1%9E%E6%80%A7 "大数据系统应有的属性")大数据系统应有的属性

*   鲁棒性和容错性

*   低延迟读取和更新

*   可扩展性

*   通用性

*   延展性(目标实现大规模迁移)

*   及席查询

*   最少维护

*   可调式性

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#Lambda%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%9E%8B "Lambda架构模型")Lambda架构模型

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86 "基本原理")基本原理

**数据系统不只是记录和重现信息 你掌握的信息是真实的,只是因为它存在
数据系统的通用表达式:**
**query = function(all data)**

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#Lambda%E6%9E%B6%E6%9E%84%E6%80%BB%E4%BD%93%E7%9A%84%E5%87%BD%E6%95%B0 "Lambda架构总体的函数")Lambda架构总体的函数

**系统抽象函数:**

**batch view = function (all data)**
**realtime view = function (realtime view(实时视图) ,new data)**
**query = function (batch view ,realtime view )**

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#Lambda%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%9E%8B%E7%9A%84%E8%AF%B4%E6%98%8E "Lambda架构模型的说明")Lambda架构模型的说明

*   批处理层:先查询预先计算查询函数(批处理视图)
*   速度层:速度层只查看最近的数据,而批处理层要立即查看所有数据,速度层做增量查询.而不是像批处理层那样重写计算
*   服务层:一个专门的分布式数据库,用于加载批处理视图,并且可以对它进行随机读取

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E7%A4%BA%E4%BE%8B%E5%BA%94%E7%94%A8-SuperWebAnalytics-com "示例应用:SuperWebAnalytics.com")示例应用:SuperWebAnalytics.com

## [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%BA%8C-%E6%89%B9%E5%A4%84%E7%90%86%E5%B1%82 "(二).批处理层")(二).批处理层

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%95%B0%E6%8D%AE%E7%9A%84%E5%B1%9E%E6%80%A7 "数据的属性")数据的属性

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%95%B0%E6%8D%AE%E6%98%AF%E5%8E%9F%E5%A7%8B%E7%9A%84 "数据是原始的")数据是原始的

*   非结构化的数据比规范化的数据更原始,更多的信息并不意味更原始的数据

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%95%B0%E6%8D%AE%E6%98%AF%E4%B8%8D%E5%8F%AF%E5%8F%98%E7%9A%84 "数据是不可变的")数据是不可变的

*   容忍人为错误是数据系统的基本属性,对于可变的数据模型,一个错误会导致数据的丢失.在数据库中值会被覆盖掉;而对于不可变的数据模式,如果人为的写入了坏数据,更早一些的数据单元仍然存在.修复数据系统知识删除损坏的数据单元
*   可变数据模式数据必须以某种方式被索引,相反,不可变数据模式不需要,这是巨大的简化

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%95%B0%E6%8D%AE%E6%98%AF%E6%B0%B8%E8%BF%9C%E7%9C%9F%E5%AE%9E%E7%9A%84 "数据是永远真实的")数据是永远真实的

*   删除数据不是对数据真实性的声明,相反,它是对数据价值的声明

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%9F%BA%E4%BA%8E%E4%BA%8B%E5%AE%9E%E7%9A%84%E6%95%B0%E6%8D%AE%E8%A1%A8%E7%A4%BA%E6%A8%A1%E5%9E%8B "基于事实的数据表示模型")基于事实的数据表示模型

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%BA%8B%E5%AE%9E%E6%A8%A1%E5%9E%8B%E7%9A%84%E7%89%B9%E7%82%B9 "事实模型的特点")事实模型的特点

*   将原始数据储存为原子事实-原子性(不能再细化成有意义的组件)
*   通过时间戳保证事实的不变性和永远正确性
*   确保每个事实是可区分的,这样查询过程可以区分重复

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%9F%BA%E4%BA%8E%E4%BA%8B%E5%AE%9E%E7%9A%84%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%BC%98%E5%8A%BF "基于事实的模型的优势")基于事实的模型的优势

*   任何时刻的历史消息都是可查询的

*   容忍人为错误(删除事实)

*   只需要处理部分信息

*   拥有规范和非规范形式的优点

    **规范化-以结构化的方式存储数据-查询效率**

    **非规范化-保证数据一致性**

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%89%B9%E5%A4%84%E7%90%86%E5%B1%82%E7%9A%84%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8 "批处理层的数据存储")批处理层的数据存储

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%B8%BB%E6%95%B0%E6%8D%AE%E9%9B%86%E7%9A%84%E5%AD%98%E5%82%A8%E9%9C%80%E6%B1%82 "主数据集的存储需求")主数据集的存储需求

*   写:
    *   高效追加数据:简单高效
    *   可扩展的存储:TB或PB级别的数据,必须很容易扩展存储
*   读:
    *   支持并行处理
*   读写:
    *   可调优存储和处理成本
    *   强制不变性

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%88%86%E5%B8%83%E5%BC%8F%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%AD%98%E5%82%A8%E9%9C%80%E6%B1%82 "分布式文件系统的存储需求")分布式文件系统的存储需求

主要矛盾:处理成本和存储成本

解决方案:分布式文件系统

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%9E%82%E7%9B%B4%E5%88%86%E5%8C%BA-%E8%BF%99%E7%A7%8D%E5%AD%98%E5%82%A8%E6%A0%87%E5%87%86%E9%80%82%E7%94%A8%E5%85%A8%E6%9E%B6%E6%9E%84%E7%9A%84%E5%93%AA%E4%B8%80%E9%83%A8%E5%88%86%E7%9A%84%E6%95%B0%E6%8D%AE%E9%9B%86 "垂直分区(这种存储标准适用全架构的哪一部分的数据集)")垂直分区(这种存储标准适用全架构的哪一部分的数据集)

次要矛盾:查询效率和存储成本

解决什么问题:提高批处理查询效率

怎么实现:例如静态分区,动态分区

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%89%B9%E5%A4%84%E7%90%86%E5%B1%82 "批处理层")批处理层

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E8%A7%86%E5%9B%BE%E7%9A%84%E6%A0%87%E5%87%86 "视图的标准")视图的标准

*   批处理层上计算的一个简单策略;预先计算所以可能的查询,并将结果缓存在服务层中,但这种预先计算不能穷尽所有业务的可能性,对于给定的查询业务,要预先计算的是每一个单元

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E9%87%8D%E6%96%B0%E8%AE%A1%E7%AE%97%E7%AE%97%E6%B3%95 "重新计算算法")重新计算算法

1.  性能:需要处理整个数据集的计算工作量
2.  容忍人的错误:批处理视图不断被重建
3.  通用性:在预先处理阶段解决了算法的复杂性,由此生成了简单的批处理视图和低延迟动态处理
4.  总结:对于支持鲁棒性的数据处理系统是必不可少的

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%A2%9E%E9%87%8F%E7%AE%97%E6%B3%95-%E9%80%82%E7%94%A8%E4%BB%80%E4%B9%88%E5%9C%BA%E5%90%88%E6%88%96%E8%80%85%E8%A6%81%E6%BB%A1%E8%B6%B3%E4%BB%80%E4%B9%88%E8%A6%81%E6%B1%82 "增量算法(适用什么场合或者要满足什么要求)")增量算法(适用什么场合或者要满足什么要求)

1.  性能:需要更少的计算资源但可能产生大得多的批处理视图

2.  容忍人的错误:不容易修复批处理视图中的错误,修复是暂时的,可能要估算

3.  通用性:需要特殊定制.可能将复杂性转移到动态查询的处理中

4.  总结:提高系统效率,但只是重新计算算法的补充

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%B8%80%E7%A7%8D%E5%A4%A7%E6%95%B0%E6%8D%AE%E8%AE%A1%E7%AE%97%E7%9A%84%E8%8C%83%E5%BC%8F-MapReduce "一种大数据计算的范式:MapReduce")一种大数据计算的范式:MapReduce

*   优:可扩展性 容错性 通用性
*   缺:多步计算 join连接要手动 逻辑和物理执行耦合

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%B8%80%E7%A7%8D%E5%85%B3%E4%BA%8E%E6%89%B9%E5%A4%84%E7%90%86%E8%AE%A1%E7%AE%97%E7%9A%84%E9%AB%98%E7%BA%A7%E6%80%9D%E7%BB%B4%E6%96%B9%E5%BC%8F-%E7%AE%A1%E9%81%93%E5%9B%BE "一种关于批处理计算的高级思维方式:管道图")一种关于批处理计算的高级思维方式:管道图

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%89%B9%E5%A4%84%E7%90%86%E5%B1%82%E7%A4%BA%E4%BE%8B%E4%B8%8E%E5%AE%9E%E7%8E%B0-%E4%B8%9A%E5%8A%A1%E5%A4%84%E7%90%86 "批处理层示例与实现(业务处理)")批处理层示例与实现(业务处理)

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#URL%E8%A7%84%E8%8C%83%E5%8C%96 "URL规范化")URL规范化

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E7%94%A8%E6%88%B7%E6%A0%87%E8%AF%86%E7%AC%A6%E8%A7%84%E8%8C%83%E5%8C%96-%E8%BF%AD%E4%BB%A3%E5%9B%BE%E7%AE%97%E6%B3%95 "用户标识符规范化(迭代图算法)")用户标识符规范化(迭代图算法)

*   等效边的处理

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E9%A1%B5%E9%9D%A2%E6%B5%8F%E8%A7%88%E5%8E%BB%E9%87%8D "页面浏览去重")页面浏览去重

改日再写…………

## [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%B8%89-%E6%9C%8D%E5%8A%A1%E5%B1%82 "(三).服务层")(三).服务层

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%9C%8D%E5%8A%A1%E5%B1%82%E6%A6%82%E8%BF%B0 "服务层概述")服务层概述

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%9C%8D%E5%8A%A1%E5%B1%82%E7%9A%84%E6%80%A7%E8%83%BD%E6%8C%87%E6%A0%87 "服务层的性能指标")服务层的性能指标

起源:

*   层的索引以完全分布式的方式被创建、加载和 服务

指标:

*   延迟:响应单个查询所需 的时间
*   吞吐量:给定时间内可以服务的查询数量

性能的优化:

*   的索引促进扫描并限制磁盘寻道,从而改善了延迟和吞吐量

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%B8%80%E8%87%B4%E6%80%A7%E9%97%AE%E9%A2%98%E7%9A%84%E6%8F%8F%E8%BF%B0 "一致性问题的描述")一致性问题的描述

解决方案:lambda架构中主数据集和服务层之间是分离的,解决”一个字段的不同副本数据不一致问题”等,重新开始计算服务层

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%9C%8D%E5%8A%A1%E5%B1%82%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E9%9C%80%E6%B1%82 "服务层数据库的需求")服务层数据库的需求

*   写-服务层所用的批处理视图是从头开始产生的,当视图的一个新的版本可以用时,旧版本的视图必须能够完全用新的视图替换

*   展性-服务层数据库必须能够处理任意大小的视图

*   读取-尤对小部分视图的随机读取

*   性-分布式架构必须要容忍机器的故障

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E8%AE%BE%E8%AE%A1SuperWebAnalytics-com%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%B1%82 "设计SuperWebAnalytics.com的服务层")设计SuperWebAnalytics.com的服务层

ElephantDB数据库

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#LAMBDA%E6%9E%B6%E6%9E%84%E6%9C%8D%E5%8A%A1%E5%B1%82%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5 "LAMBDA架构服务层的基本概念")LAMBDA架构服务层的基本概念

*   视图来优化延迟和吞吐量的能力

*   持随机写的简单形式

*   处理层储存规范化数据并在服务层储存非规范化数据的能力

*   层固有的容错和纠错性.因为可以从主数据集重新计算

## [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%9B%9B-%E9%80%9F%E5%BA%A6%E5%B1%82 "(四).速度层")(四).速度层

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%AE%9E%E6%97%B6%E8%A7%86%E5%9B%BE "实时视图")实时视图

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%AD%98%E5%82%A8%E5%AE%9E%E6%97%B6%E8%A7%86%E5%9B%BE "存储实时视图")存储实时视图

*   随机读:实时视图应该支持快速回应查询,这意味着它所包含的数据必须被索引
*   随机写:为了支持增量算法,必须低延迟地修改实时视图
*   可扩展性
*   容错性

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%9C%80%E7%BB%88%E4%B8%80%E8%87%B4%E6%80%A7 "最终一致性")最终一致性

*   所有的数据最终都将表示在批处理层和服务层视图中,任何在速度层得到的近似值会被不断修正,这意味着任何近似值知识暂时的,即查询最终展示了准确性

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%A2%9E%E9%87%8F%E7%AE%97%E6%B3%95%E7%9A%84%E6%8C%91%E6%88%98 "增量算法的挑战")增量算法的挑战

*   **CAP定理的正确表述:当分布式数据系统被分区时,它可以是一致性的或者可用性的,但不能两者兼而有之.也就是说,如果选择了一致性,那么查询就会收到错误结果而不是正确答案:如果选择可用性,那么在网络分区间,读操作可能返回过时的数据,在高可用性系统中,最好的一致性属性是最终一致性**

*   **CAP定理**

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E9%98%9F%E5%88%97%E5%92%8C%E6%B5%81 "队列和流")队列和流

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E9%98%9F%E5%88%97 "队列")队列

*   单消费者队列

    *   **设计思想:从队列中读取一个事件时,该事件不会立即被删除,相反,get方法返回的记录包括一个标识符,稍后用它来确认处理事件成功还是失败.只有一个事件被确认成功删除,它才会被从队列中删除,如果事件处理失败或者超时,队列服务器将允许另一个服务器通过一个单独的get方法调用相同的事件.因此使用该方法,一个事件可能被处理多次,但是每个事件至少被处理一次是毋庸置疑的**
    *   **缺陷:消除了独立应用程序之间的任何隔离性**
    *   **完善:为每个消费者应用程序维护一个单独的队列,但是这种方法的实现大大增加服务器上的负载**
    *   **启发:队列系统所需的属性**
*   多消费者队列

    *   Apache Kafka

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%B5%81%E5%A4%84%E7%90%86 "流处理")流处理

*   storm

*   spark

*   flink

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%BE%AE%E6%89%B9%E9%87%8F%E5%A4%84%E7%90%86 "微批量处理")微批量处理

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%AE%9E%E7%8E%B0%E6%9C%89%E4%B8%94%E4%BB%85%E6%9C%89%E4%B8%80%E6%AC%A1%E7%9A%84%E8%AF%AD%E4%B9%89 "实现有且仅有一次的语义")实现有且仅有一次的语义

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%BE%AE%E6%89%B9%E9%87%8F%E6%B5%81%E5%A4%84%E7%90%86 "微批量流处理")微批量流处理

每个批次被有序处理,并且每个批次都有唯一的ID,该ID每次回放总是一样的

##### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%BE%AE%E6%89%B9%E9%87%8F%E6%B5%81%E5%A4%84%E7%90%86%E7%9A%84%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5 "微批量流处理的核心概念")微批量流处理的核心概念

*   本地批量计算
*   有状态的计算

## [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E4%BA%94-%E6%B7%B1%E5%85%A5lambda%E6%9E%B6%E6%9E%84 "(五).深入lambda架构")(五).深入lambda架构

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E5%AE%9A%E4%B9%89%E6%95%B0%E6%8D%AE%E7%B3%BB%E7%BB%9F "定义数据系统")定义数据系统

**查询要关注的属性**

*   延迟:运行一个查询的书剑

*   时效性:最新的查询如何

*   准确性:在许多情况下,为了使查询更具有较好的性能和可扩展性,必须在查询的实现中采用近似值

**延迟性和及时性的描述**

*   CAP定理表明:在网络分区的情况下,一个系统要么是一致性(查询考虑到所有以前写入的数据),要么是可用的(目前查询可以被回应)

*   一致性只是及时性的一种形式,可用性只意味查询的延迟是有界的.最终一致性的系统选择延迟而不是及时性(查询总是被回应,但可能不会考虑所有先前失败的情况下的数据)

**数据系统的基本模型**

为什么如此以及怎样:允许人为错误 易变性 唯一的办法是让核心数据保持不变

*   包含不断增长 的数据集合的主数据集

*   作为函数的查询将整个主数据集作为输入

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%89%B9%E5%A4%84%E7%90%86%E5%B1%82%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%B1%82 "批处理层和服务层")批处理层和服务层

**增量的批处理**

局限性:原始数据可能是混乱的

解决的方案:**部分重新计算**:

*   部分重新计算花费时间比基于完全重新计算的方法快
*   给予一定的能力来纠正人为错误

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E9%80%9F%E5%BA%A6%E5%B1%82 "速度层")速度层

设计的主旨:倾向于性能使用增量算法而不是重新计算方法

### [](https://nino-laiqiu.github.io/2020/09/29/Lambda-System-Construction/#%E6%9F%A5%E8%AF%A2%E5%B1%82 "查询层")查询层

目的:负责利用批处理层和实时视图来响应查询
