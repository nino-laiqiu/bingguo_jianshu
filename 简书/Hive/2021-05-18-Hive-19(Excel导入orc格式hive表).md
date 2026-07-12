Excel文件不支持导入到orc压缩格式的hive表中,需要借助中间表来完成操作

步骤
1. 将Excel文件更改为csv文件,改变编码格式为UTF-8
2. 上传至hadoop,获取地址和文件名
3. 建一张orc的中间表mid_,中间表为TEXTFILE格式
```
    row format delimited fields terminated by '\t'
    STORED AS TEXTFILE
```
4. 将数据导入到mid表中
```
load data local inpath '......' into table ......;
```
5. 将mid表中的数据导入到orc表中
```
INSERT INTO TABLE tmp_orc_table SELECT * FROM mid_.....;
```

#####其他格式
其他存储格式如 SEQUENCEFILE、PARQUET 等，都可以按照 ORC 格式那样，先导入到 TEXTFILE 表，再进行两个表之间的数据转移即可
