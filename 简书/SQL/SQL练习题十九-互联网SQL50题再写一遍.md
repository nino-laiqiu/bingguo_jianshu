>很多地方都有这些题目,这些题目1年前也写过一遍,这里回顾一下,使SQL练习这个主题成为一个完整的主题,这些题目也会较好的巩固子查询/where联合查询/group by维度/自连接 等基本的用法,再写一遍希望有所收获

>花了一个早上,和一年前对比一下,以前写还是有点吃力,现在写发现,哈哈哈,都是简单题,但还是发现如下问题:
1.对自连接的处理还是不够好,理解性较差
2.对多条件问题的处理SQL的性能优化问题
3.维度一致性问题,group by 与select中的维度要一致
4.mysql中的日期函数方言,date_format date_sub,date_add等/year.week等
一些注意事项;
193多条件比较分别求,join 
196方法二,max的应用
212自连接对dense_rank()的处理  对自连接的左表的每一行做出判断是否符合过滤的条件
213自连接对rank()的处理
自连接怎么实现row_number()
注意group by 的维度要与聚和以后要求的select的维度一致
case when 的处理 ;例如 sum(case when )  case when then when then ....的适用,if的多条件判断
如何选取:子查询 group by 自连接 join/or/and 窗口
```
create table Student (SId varchar(10), Sname varchar(10), Sage datetime, Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-12-20' , '男');
insert into Student values('04' , '李云' , '1990-12-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-01-01' , '女');
insert into Student values('07' , '郑竹' , '1989-01-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2012-06-06' , '女');
insert into Student values('12' , '赵六' , '2013-06-13' , '女');
insert into Student values('13' , '孙七' , '2014-06-01' , '女');


create table Course(CId varchar(10),Cname nvarchar(10),TId varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');


create table Teacher(TId varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');


create table SC(SId varchar(10),CId varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```
![表之间的关联](https://upload-images.jianshu.io/upload_images/9049859-e75cce0207c06ea1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##191.查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数
1.子查询 
2.group by  
3.连接
```
--子查询
select
s0.*,s3.CId,s3.score
from student s0
join (
    select t1.SId
    from (
             select sid,
                    score
             from sc
             where CId = '01'
         ) t1
             join (
        select sid,
               score
        from sc
        where CId = '02'
    ) t2
                  on t1.SId = t2.SId
    where t1.score > t2.score
) s1 on s0.SId = s1.SId
join sc s3 on s0.SId = s3.SId;
```
```
--求取01 02的分数就行了不用求全部的
select  t1.*,t3.CId,t3.score
from student t1 join (
    select SId,
           max(if(CId = '01', score, 0)) as q1,
           max(if(CId = '02', score, 0)) as q2
    from sc
    group by SId
) t2 on t1.SId = t2.SId
  join sc t3 on t1.SId = t3.SId
where t2.q1 > t2.q2 and t2.q2 !=0 and t2.q1!= 0;
```
```
SELECT
	s.*,sc1.score,sc2.score
FROM
	sc AS sc1,sc AS sc2,student AS s -- 三表相连
WHERE sc1.SId = sc2.SId
  AND sc1.CId = '01' 
  AND sc2.CId ='02' 
  AND sc1.score > sc2.score 
  AND s.SId = sc1.SId  
```
##192. 查询同时存在" 01 "课程和" 02 "课程的情况
1.子查询
2.group by
3.连接

**小心注意子查询的嵌套层数**,直接过滤出*也行,但符合事实上的答案
```
select
*
from sc where SId in (
    select SId
    from sc
    where SId in (select SId from sc where CId = '01')
      and CId = '02'
);
```
```
select
SId
from sc
group by SId
having  sum(if(CId='01',1,0)) >0 and sum(if(CId='02',1,0))>0;
```
```
SELECT * 
FROM
	(SELECT * FROM sc WHERE sc.CId = '01') sc1, 
	(SELECT * FROM sc WHERE sc.CId = '02') sc2 
WHERE sc1.sid = sc2.sid
```

##193.查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
同上一题
```
select
*
from sc where  SId in (
    select SId
    from sc
    where SId in (select SId from sc where CId = '01')
      and SId not in (select SId from sc where CId = '02')
);
```
看了下,别人写的题目中有个可能,写错了,直接过滤出01 02 left join即可
```
SELECT *
FROM
	(SELECT * FROM sc WHERE sc.CId = '01') sc1
LEFT JOIN
	(SELECT * FROM sc WHERE sc.CId = '02') sc2
ON sc1.sid = sc2.sid;
```
以后碰到这个多条件的比较直接分别查询来join,比较清晰
##194.查询不存在" 01 "课程但存在" 02 "课程的情况
right join就行了
```
SELECT *
FROM
	(SELECT * FROM sc WHERE sc.CId = '01') sc1
right join
	(SELECT * FROM sc WHERE sc.CId = '02') sc2
on  sc1.sid = sc2.sid
where  sc1.SId is not  null ;
```
##195.查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
1.join
2.直接关联
```
select
t1.SId,Sname,score
from student t1
join (
    select SId, avg(score) as score
    from sc
    group by SId
    having avg(score) >= 60
) t2 on t1.SId = t2.SId;
```
```
SELECT s.sid,s.Sname,avg(sc.score) avg_s
FROM sc,student s
WHERE sc.sid = s.sid 
GROUP BY sid,Sname
HAVING avg_s >= 60;
```
##196. 查询所有课程成绩小于60分学生的学号、姓名
```
select
SId,Sname
from student where SId in (
    select sid
    from sc
    group by sid
    having sum(if(score < 60, 0, 1)) = 0
);
```
方法二:如果最大的score小于60,则所有的科目成绩就都小于60
```
select
SId,Sname
from student where SId in (
    select sid
    from sc
    group by sid
    having max(score) < 60
);
```
##197.查询在 SC 表存在成绩的学生信息
```
select
distinct  student.*
from student join sc on student.SId = sc.SId;
```
方法二:先获取在成绩表中有的sid信息,子查询where获取
##198.查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )
```
select
t1.SId,t1.Sname,count(CId) as count_c ,sum(score) as score
from student t1 left join sc t2 on t1.sid = t2.SId
group by  t1.SId,t1.Sname;
```
count不存在时显示为0 ,而sum不存在则显示null

##199.查有成绩的学生信息
```
select * from student where SId in (
    select distinct sid
    from sc
    where score is not null
);
```
##200.查询「李」姓老师的数量
```
select
count(1)
from teacher
where Tname like '李%';
```
##201.查询学过「张三」老师授课的同学的信息
两种思路,方法一:过滤出张三教的课程信息,多个子查询,方法二:多表join之后where
```
select
distinct t1.*
from student t1 join sc t2 on t1.SId = t2.sid
join course t3 on t2.cid = t3.CId
join teacher t4 on t3.tid = t4.TId
where t4.Tname = '张三';
```
##202.查询没有学全所有课程的同学的信息
```
select
*
from student where SId not in (
    select sid
    from sc
    group by SId
    having count(1) = (select count(1) from course)
);
```
##203.查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
方法一:先查询存在与01学号相同课程的sid,方法二:查询02学号所有的cid,in 如果其他学号也有就是了
```
select * from student where SId in (
    select distinct sc.sid
    from (
             select CId
             from sc
             where SId = '01'
         ) t1
             join sc on t1.CId = sc.CId
)
and SId != '01';
```
##204.查询和" 01 "号的同学学习的课程 完全相同的其他同学的信息
比较麻烦,有没有什么简单方法???
```
select
*
from student where  SId in (
    select SId
    from (
             select CId
             from sc
             where SId = '01'
         ) t1
             join sc on t1.CId = sc.CId
    group by sc.SId
    having count(sc.SId) = (select count(1) from sc where SId = '01')
)
and SId != '01';
```
##205.查询没学过"张三"老师讲授的任一门课程的学生姓名
```
select sid from student where SId not in (
    select distinct SId
    from teacher t1
             join course t2 on t1.TId = t2.TId
             join sc t3 on t2.CId = t3.CId
    where t1.Tname = '张三'
);
```
##206.查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
```
select
t1.SId,t1.Sname,avg(score)
from student t1 join sc t2 on t1.SId = t2.SId
group by  t1.SId,t1.Sname
having  sum(if(score<60,1,0)) >= 2 ;
```
##207.检索" 01 "课程分数小于 60，按分数降序排列的学生信息
```
select
t1.*
from student t1 join  sc t2 on t1.SId = t2.SId
where  CId = '01' and score <60
order by score desc;
```
##208.按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
```
select
SId,
score,
avg(score) over (partition by SId ) as avg_score
from sc
order by avg_score desc ;
```
方法二,先求平均值,join
##209.查询各科成绩最高分、最低分和平均分
```
select
CId,max(score),min(score),avg(score),
       concat(sum(if(score<60,1,0))/count(1),'%'),
       concat(sum(if(score<=80 and score >=60,1,0))/count(1),'%'),
       concat(sum(if(score>80,1,0))/count(1),'%')
from sc
group by CId;
```
##210.输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
```
select
CId,count(distinct SId) as count_nm
from sc
group by CId
order by count_nm desc,CId;
```
##211.按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
```
select
CId,SId,score,rank() over (partition by CId order by score desc) rn
from sc;
```
自连接,哇,有点绕
```
select
t1.CId,t1.SId,t1.score,count(t2.score) + 1 as rn
from sc t1 left join sc t2 on t1.CId =t2.CId
and  t1.score < t2.score
group by t1.CId,t1.SId,t1.score
order by t1.CId,rn asc ;
```

##212.按各科成绩进行排序，并显示排名， Score 重复时合并名次
自连接
```
SELECT a.*,COUNT( distinct b.score) AS `rank`
FROM SC AS a
LEFT JOIN SC AS b
ON a.score <= b.score
AND a.cid = b.cid
GROUP BY a.cid, a.sid, a.score
ORDER BY a.cid, `rank` ASC;
```
```
SELECT sc.*,
       dense_rank() over (PARTITION BY cid ORDER BY score DESC) 'rank'
FROM sc
```
##213.查询学生的总成绩，并进行排名，总分重复时保留名次空缺
```
select
sid, sum(score),
 rank() over (order by sum(score) desc)
from sc
group by SId;
```
```
select
t1.SId,t1.s1,count(t2.s2)+1 as rn
from (
         select sid,
                sum(score) as s1
         from sc
         group by SId
     ) t1  left join (
         select sid,
                sum(score) as s2
         from sc
         group by SId
    ) t2
on  t1.s1 < t2.s2
group by t1.SId,t1.s1
order by  t1.s1 desc,rn;
```
##214.统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
```
select
t2.SId,
t1.Cname,
concat(sum(if(score<=100 and score >=85,1,0)) / count(1),'%') as '[100-85]',
sum(if(score<=85 and score >= 70,1,0)) / count(1),
sum(if(score<=70 and score >= 60,1,0)) / count(1)  ,
sum(if(score<=60,1,0)) /count(1)
from course t1 join sc t2
group by t1.Cname,t2.SId;
```
##215.查询各科成绩前三名的记录
--自连接有点绕啊,写了半小时,题目说是前三序列函数应该是rank,所以比较时应该是<没有等于,并且使用left join ,而对于dense_rank应该用<=且去重
```
select
t1.CId,t1.sid,t1.score,count(t2.score) + 1
from sc t1 left join sc t2 on t1.CId = t2.CId and t1.score <  t2.score
group by  t1.CId,t1.sid,t1.score
having count(t2.score) + 1 <= 3
order by t1.CId;
```

##216.查询每门课程被选修的学生数
```
select
cid ,count(1) as count_n
from sc
group by CId;
```
##217.查询出只选修两门课程的学生学号和姓名
```
select SId,Sname from student where SId in (
    select sid
    from sc
    group by SId
    having count(1) = 2
)
```
##218.查询男生、女生人数
```
select Ssex,count(1) from student group by Ssex;
```
```
select
sum(if(Ssex='男',1,0)) as '男生',
sum(if(Ssex='男',0,1)) as '女生'
from student;
```
##219.查询名字中含有「风」字的学生信息
```
select *
from student
where Sname like '%风%';
```
##220.查询同名同性学生名单，并统计同名人数
```
select
Sname
from student
group by Sname
having count(1) >1;
```
##221.查询 1990 年出生的学生名单
```
select * from student where year(Sage) ='1990';
```
##222.查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
```
select
cid ,avg(score) as avg_score
from sc
group by CId
order by  avg_score desc ,CId;
```
##223.查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
```
select
t1.SId,Sname,avg(score) as avg_score
from student t1 join sc t2 on t1.SId = t2.SId
group by  t1.SId,Sname
having avg_score >= 85;
```
##224.查询课程名称为「数学」，且分数低于 60 的学生姓名和分数
```
select
Sname,score
from student t1 join sc t2 on t1.SId = t2.SId
join course t3 on  t2.CId =t3.CId
where t3.Cname = '数学' and score < 60;
```
##225.查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
```
select
*
from student t1 left join  sc t2 on t1.SId = t2.SId;
```
##226.查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
having 条件过滤,如果是说有一门可在70/所有的课都在70分
```
select Cname,Sname,score
from student t1 join sc t2 on  t1.SId = t2.SId
join  course t3 on t2.CId = t3.CId
where t2.score >= 70;
```
都在70分
```
select Cname,Sname,score
from student t1 join sc t2 on  t1.SId = t2.SId
join  course t3 on t2.CId = t3.CId
where  t1.SId in (
    select SId
    from sc
    group by SId
    having sum(if(score >= 70, 0, 1)) = 0
);
```
##227.查询不及格的课程
```
select SId,Cname
from course t1 join sc t2 on t1.CId = t2.CId
where t2.score < 60;
```
##228.查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
```
select *
from student  where  SId in (
    select SId
    from sc
    where CId = '01'
      and score >= 80
) ;
```
##229.求每门课程的学生人数
```
SELECT COUNT(*),cid
FROM sc
GROUP BY cid
```
##230.成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
```
select
t4.SId,t4.Sname,t4.Sage,t4.Ssex,max(score) as max_n
from teacher t1 join  course t2 on t1.TId = t2.TId
join  sc t3 on  t2.CId = t3.CId
join student t4 on t3.SId = t4.SId
where t1.Tname ='张三'
group by t4.SId,t4.Sname,t4.Sage,t4.Ssex
order by  max_n desc
limit 1;
```
方法二:分解步骤,子查询获取最高的成绩,然后join
##231.成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
```
select s0.*,CId
from student s0
join (
    select SId,
           t3.CId,
           rank() over (order by score desc ) as rn
    from teacher t1
             join course t2 on t1.TId = t2.TId
             join sc t3 on t2.CId = t3.CId
    where t1.Tname = '张三'
) s1 on s1.SId = s0.SId
where rn =1;
```
方法二,同上面的获取最高的成绩,子查询获取
##232.查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
```
select distinct t1.sid, t1.cid, t1.score
from sc t1 join sc t2 on t1.score = t2.score and t1.CId <> t2.CId;
```
方法二:先获取有重复成绩的成绩分数
```
SELECT DISTINCT Sid,
       Cid,
       score
FROM SC WHERE score IN (
	SELECT score
	FROM SC
	GROUP BY score
	HAVING COUNT(DISTINCT Cid) > 1);
```
##233.查询每门功成绩最好的前两名
有且只有两个人rank函数
```
select
t1.CId,t1.SId,t1.score,count(t2.score) + 1
from sc t1 left join sc t2 on t1.CId = t2.CId and t1.score < t2.score
group by t1.CId,t1.SId,t1.score
having count(t2.score) <= 1
order by  t1.CId;
```
##234.统计每门课程的学生选修人数（超过 5 人的课程才统计）
```
SELECT COUNT(SId),cid
FROM sc
GROUP BY cid
HAVING COUNT(sid) > 5;
```
##235.检索至少选修两门课程的学生学号
having过滤
```
select SId
from sc
group by SId
having count(1)>=2;
```
##236.查询选修了全部课程的学生信息
不存在同一门课选修多次的情况??这题好像做过了..
```
select *
from student
where SId in (
    select SId
    from sc
    group by SId
    having count(1) = (select count(1) from course)
);
```
##237.查询各学生的年龄，只按年份来算
```
select *,year(now())-year(Sage) +1 as `year`
from student;
```
##238.按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
```
select *,year(now())-year(Sage) +
    if((date_format(now(),'%m%d')-date_format(Sage,'%m%d'))<0,1,0)
    as `year`
from student;
```
##239.查询本周过生日的学生
```
select * from student where  week(Sage) =
       week(now());
```
##240.查询下月过生日的学生
```
SELECT *
FROM Student
WHERE MONTH(Sage) = MONTH(CURDATE()) + 1;
```
