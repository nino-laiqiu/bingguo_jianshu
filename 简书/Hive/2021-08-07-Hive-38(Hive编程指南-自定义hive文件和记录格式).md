##1. 文件和记录格式
Hive中文件格式间具有明显的差异,例如文件中记录的编码方式、记录格式以及记录中字节流的编码方式都是不同的。
##2. 阐述 create table 句式
##3. 文件格式
1. SequenceFile
SequenceFile文件是含有键-值对的二进制文件。当Hive将查询转换成MapReduce job时,对于指定的记录,其取决使用哪些合适的键-值对。SequenceFile可以,在块级别和记录级别进行压缩,这对于优化磁盘利用率和10来说非常有意义。同时仍然可以支持按照块级别的文件分割,以方便并行处理。
2. RCfile


##4. 记录格式: SerDe
##5. CSV和TSV SerDe
##6. XML UDF
XML天生就是非结构化的,这使得Hive成为XML处理的一个强大的数据库平台。众多原因之一就是大型XML文档解析和处理所需要的复杂性和资源消耗使得Hadoop非常适合作为一个XML数据库平台,因为Hadoop可以并行地处理XML文档, Hive也成为了解决XML相关数据的完美工具。此外, HiveQL原生就支持访问XML的嵌套元素和值,然后进一步地,可以允许对嵌套的字段、值和属性进行连接操作。
##7. JSON SerDe
WITH SERDEPROPERTIES的用法
##8. Avro Hive SerDe
Avro是一个序列化系统,其主要特点是它是一个进化的模式驱动的二进制数据存储模式。
#####使用表属性信息定义Avro Schema
##9. 二进制输出

>主要的知识点:
储存格式区别和优势
