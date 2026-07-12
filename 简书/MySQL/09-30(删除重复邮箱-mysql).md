
![image.png](https://upload-images.jianshu.io/upload_images/9049859-8bca2304651df916.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
select t2.Id Id
from Weather t1
join Weather t2
on DATEDIFF(t2.RecordDate, t1.RecordDate) = 1
where t2.Temperature > t1.Temperature
```

![image.png](https://upload-images.jianshu.io/upload_images/9049859-fc955e9250cbbfa6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
delete p1 from (person as p1 left join person as p2 on p1.email = p2.email) where p1.id > p2.id ;
```
