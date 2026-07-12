需求的背景--需要借助api来监控kylin-cube的运行状态,这样做的目的是及时知道cube的运行状态,这里简单的介绍一些常用的api在kylin监控中的应用.以及使用AWK,grep正则匹配需要的字段
关于kylin-restful-api的使用在https://kylin.apache.org/docs/howto/howto_use_restapi.html官网上有详细的说明

##KYLIN的监控
```
job_list=curl -X GET -u $kylin_user:$kylin_pass -H 'Content-Type: application/json' http://$host:$port/kylin/api/jobs?cubeName=$cube_name&projectName=$project #根据job_list拿到jobId 
jobid=echo $job_list |awk -F',' '{print $1}' |grep 'uuid' |awk -F':' '{print $2}' |xargs 
#根据jobId获取任务信息 
result=curl -X GET -u $kylin_user:$kylin_pass -H 'Content-Type: application/json' http://$host:$port/kylin/api/jobs/${jobid} 
#截取最终任务状态 
job_status=echo $result |grep -oE job_status.* |grep 'job_status' |awk -F':' '{print $2}' |awk -F',' '{print $1}'|xargs 
#截取最终任务构建时长，并转换为分秒
关于此处的| xargs 表示的是多行转为一行输出
```
主要的难点就是正则匹配获取字段,上面的可以优化一下

此处是关于kylin-api的一些简单使用
##工作清单
例如:获取全部的状态(最主要的用法)
```
GET /kylin/api/jobs
cubeName -optional string多维数据集名称。
projectName -required string项目名称。
status -optional int作业状态，例如（NEW: 0, PENDING: 1, RUNNING: 2, STOPPED: 32, FINISHED: 4, ERROR: 8, DISCARDED: 16）
offset -required int分页使用的偏移量。
限制 -required int每页作业数。
timeFilter - required int，例如（最后一天：0，最后一周：1，最后一个月：2，最后一年：3，所有：4）
```

GET: /kylin/api/jobs?cubeName=kylin_sales_cube&limit=15&offset=0&projectName=learn_kylin&timeFilter=1

响应
```
[
  { 
    "uuid": "9eb7bccf-4448-4578-9c29-552658b5a2ca", 
    "last_modified": 1490957579843, 
    "version": "2.0.0", 
    "name": "Sample_Cube - 19700101000000_20150101000000 - BUILD - GMT+08:00 2017-03-31 18:36:08", 
    "type": "BUILD", 
    "duration": 936, 
    "related_cube": "Sample_Cube", 
    "related_segment": "53a5d7f7-7e06-4ea1-b3ee-b7f30343c723", 
    "exec_start_time": 1490956581743, 
    "exec_end_time": 1490957518131, 
    "mr_waiting": 0, 
    "steps": [
      { 
        "interruptCmd": null, 
        "id": "9eb7bccf-4448-4578-9c29-552658b5a2ca-00", 
        "name": "Create Intermediate Flat Hive Table", 
        "sequence_id": 0, 
        "exec_cmd": null, 
        "interrupt_cmd": null, 
        "exec_start_time": 1490957508721, 
        "exec_end_time": 1490957518102, 
        "exec_wait_time": 0, 
        "step_status": "DISCARDED", 
        "cmd_type": "SHELL_CMD_HADOOP", 
        "info": { "endTime": "1490957518102", "startTime": "1490957508721" }, 
        "run_async": false 
      }, 
      { 
        "interruptCmd": null, 
        "id": "9eb7bccf-4448-4578-9c29-552658b5a2ca-01", 
        "name": "Redistribute Flat Hive Table", 
        "sequence_id": 1, 
        "exec_cmd": null, 
        "interrupt_cmd": null, 
        "exec_start_time": 0, 
        "exec_end_time": 0, 
        "exec_wait_time": 0, 
        "step_status": "DISCARDED", 
        "cmd_type": "SHELL_CMD_HADOOP", 
        "info": {}, 
        "run_async": false 
      }
    ],
    "submitter": "ADMIN", 
    "job_status": "FINISHED", 
    "progress": 100.0 
  }
]
```

##获取工作状态概览
GET /kylin/api/jobs/overview

请求变量
（同“获取工作清单”）

响应样本
```
{
    "DISCARDED": 0,
    "NEW": 0,
    "STOPPED": 0,
    "PENDING": 0,
    "RUNNING": 0,
    "FINISHED": 1,
    "ERROR": 0
}
```
正则表达式
https://blog.csdn.net/u010003835/article/details/80763526
