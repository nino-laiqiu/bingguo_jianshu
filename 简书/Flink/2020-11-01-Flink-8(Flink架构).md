##1.系统架构
Flink是一个用于状态化并行流处理的分布式系统,它的搭建涉及多个进程,这些进程通常会分布在多台机子上.分布式系统需要应对的常见挑战包括分配和管理集群资源,进程协调,持久且高可用的数据存储以及故障恢复
#####搭建Flink所需组件
JobManager:是**主进程(master process)** ,控制着单个应用程序的执行,换句话说就是每个应用都由不同的JobMannger掌控.JobManager可以接受需要执行的应用,该应用包含一个所谓的**JobGraph(逻辑图)**,以及包含一个打包了全部所需类,库以及其他资源的jar文件.JobManager从ResourceManager申请执行任务的必要资源(TaskManager处理槽).一旦它收集到执行任务的必要资源处理槽(slot),就会把ExecutionGraph中的**任务分发给TaskManager来执行**

ResourceManager:针对不同的环境和资源提供者,Flink提供了不同的ResourceManager,ResourceManager负责管理Flink的处理资源单元-TaskManager处理槽.当JobManager申请处理槽时,ResourceManager会指示一个拥有空闲处理槽的taskmanager处理槽将其处理槽提供给jobmanager,如果ResourceManager的处理槽数无法满足jobmanager的请求,则ResourceManager可以**和资源提供者通信**,让其提供容器来启动更多的taskmanager进程,同时ResourceManager还**负责终止空闲taskmanager以释放计算资源**

TaskManager:是flink的工作进程(worker process ) ,通常flink在搭建时会启动多个taskmanager.每个TaskManager提供一定数量的处理槽,处理槽的数目限制了一个TaskManager可执行的任务数,TaskManager启动后,会**向ResourceManager注册它的处理槽**,当接收到ResourceManager的指示时,TaskManager会**向jobmanager提供一个或者多个处理槽**,之后JobManager就可以向处理槽中分配任务来执行了,在执行期间,运行同一个应用不同任务的TaskManager之间会产生**数据交换**

Dispatcher会跨多个作业运行,它提供了一个REST接口来让我们提交需要执行的应用

交互过程
1. 提交应用
2. 启动,提交任务
3. 申请处理槽
4. 启动
5. 注册处理槽
6. 指示提供处理槽
7. 提供处理槽
8. 将任务提交到处理槽执行
9. 数据交换
#####应用部署
框架模式:flink应用会打包成一个jar文件,通过客户端提交到运行的服务上
库模式:flink应用绑定到一个特定应用的容器镜像
#####任务执行
一个taskmanager允许同时执行多个任务,这些任务可以属于同一个算子(数据并行),也可以是不同算子(任务并行),甚至可以来自于不同的应用(作业并行).taskmanager通过提供固定数量的处理槽来控制可以并行执行的任务数

将任务以切片的形式调度至处理槽中的一个好处是taskmanager中的多个任务可以在同一个进程内高效地执行数据交换而无需访问网络,然而任务过于集中会使taskmanager负载变高,继而可能导致性能下降

taskmanager会在同一个JVM进程内以多线程的方式执行任务,和独立进程相比,线程更加轻量并且通信开销更低,但无法严格将任务彼此隔离,因此只要有一个任务运行异常,就有可能杀死整个taskmanager进程.通常在taskmanager内部采用线程并行以及在每个主机上部署多个taskmanager

#####高可用性设置
taskmanager故障:
无法申请到足够的槽,jobmanager就无法重启.直到有足够的可用槽,应用的重启策略决定了jobmanager以何种频率重启应用以及重启的尝试之间的等待间隔

jobmanager故障
jobmanager发生故障更为棘手.它用于控制流式应用执行以及保存该过程中元数据(如已完成检查点的存储路径).如果负责管理的jobmanager进程消失,流式应用将无法继续处理数据,这就导致了jobmanager成为flink应用中的一个单点失效组件,为了解决该问题,flink提供了高可用模式,支持在原来jobmanager消失的情况下将作业的管理职责以及元数据迁移到另一个jobmanager上
