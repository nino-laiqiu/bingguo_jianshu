MapReduce
#####1. reduce side join
Map端的主要工作：为来自不同表（文件）的key/value对打标签以区别不同来源的记录。然后用连接字段作为key，其余部分和新加的标志作为value，最后进行输出。

reduce端的主要工作：在reduce端以连接字段作为key的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录（在map阶段已经打标志）分开，最后进行笛卡尔

这种方式的缺点很明显就是会造成map和reduce端也就是shuffle阶段出现大量的数据传输，效率很低。
```
public class ReduceSideJoin_LeftOuterJoin extends Configured implements Tool{
    private static final Logger logger = LoggerFactory.getLogger(ReduceSideJoin_LeftOuterJoin.class);
    public static class LeftOutJoinMapper extends Mapper<Object, Text, Text, CombineValues> {
        private CombineValues combineValues = new CombineValues();
        private Text flag = new Text();
        private Text joinKey = new Text();
        private Text secondPart = new Text();
        @Override
        protected void map(Object key, Text value, Context context)
                throws IOException, InterruptedException {
            //获得文件输入路径
            String pathName = ((FileSplit) context.getInputSplit()).getPath().toString();
            //数据来自tb_dim_city.dat文件,标志即为"0"
            if(pathName.endsWith("tb_dim_city.dat")){
                String[] valueItems = value.toString().split("\\|");
                //过滤格式错误的记录
                if(valueItems.length != 5){
                    return;
                }
                flag.set("0");
                joinKey.set(valueItems[0]);
                secondPart.set(valueItems[1]+"\t"+valueItems[2]+"\t"+valueItems[3]+"\t"+valueItems[4]);
                combineValues.setFlag(flag);
                combineValues.setJoinKey(joinKey);
                combineValues.setSecondPart(secondPart);
                context.write(combineValues.getJoinKey(), combineValues);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
            }//数据来自于tb_user_profiles.dat，标志即为"1"
            else if(pathName.endsWith("tb_user_profiles.dat")){
                String[] valueItems = value.toString().split("\\|");
                //过滤格式错误的记录
                if(valueItems.length != 4){
                    return;
                }
                flag.set("1");
                joinKey.set(valueItems[3]);
                secondPart.set(valueItems[0]+"\t"+valueItems[1]+"\t"+valueItems[2]);
                combineValues.setFlag(flag);
                combineValues.setJoinKey(joinKey);
                combineValues.setSecondPart(secondPart);
                context.write(combineValues.getJoinKey(), combineValues);
            }
        }
    }
    public static class LeftOutJoinReducer extends Reducer<Text, CombineValues, Text, Text> {
        //存储一个分组中的左表信息
        private ArrayList<Text> leftTable = new ArrayList<Text>();
        //存储一个分组中的右表信息
        private ArrayList<Text> rightTable = new ArrayList<Text>();
        private Text secondPar = null;
        private Text output = new Text();
        /**
         * 一个分组调用一次reduce函数
         */
        @Override
        protected void reduce(Text key, Iterable<CombineValues> value, Context context)
                throws IOException, InterruptedException {
            leftTable.clear();
            rightTable.clear();
            /**
             * 将分组中的元素按照文件分别进行存放
             * 这种方法要注意的问题：
             * 如果一个分组内的元素太多的话，可能会导致在reduce阶段出现OOM，
             * 在处理分布式问题之前最好先了解数据的分布情况，根据不同的分布采取最
             * 适当的处理方法，这样可以有效的防止导致OOM和数据过度倾斜问题。
             */
            for(CombineValues cv : value){
                secondPar = new Text(cv.getSecondPart().toString());
                //左表tb_dim_city
                if("0".equals(cv.getFlag().toString().trim())){
                    leftTable.add(secondPar);
                }
                //右表tb_user_profiles
                else if("1".equals(cv.getFlag().toString().trim())){
                    rightTable.add(secondPar);
                }
            }
            logger.info("tb_dim_city:"+leftTable.toString());
            logger.info("tb_user_profiles:"+rightTable.toString());
            for(Text leftPart : leftTable){
                for(Text rightPart : rightTable){
                    output.set(leftPart+ "\t" + rightPart);
                    context.write(key, output);
                }
            }
        }
    }
    @Override
    public int run(String[] args) throws Exception {
          Configuration conf=getConf(); //获得配置文件对象
            Job job=new Job(conf,"LeftOutJoinMR");
            job.setJarByClass(ReduceSideJoin_LeftOuterJoin.class);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
            FileInputFormat.addInputPath(job, new Path(args[0])); //设置map输入文件路径
            FileOutputFormat.setOutputPath(job, new Path(args[1])); //设置reduce输出文件路径
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                
            job.setMapperClass(LeftOutJoinMapper.class);
            job.setReducerClass(LeftOutJoinReducer.class);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          
            job.setInputFormatClass(TextInputFormat.class); //设置文件输入格式
            job.setOutputFormatClass(TextOutputFormat.class);//使用默认的output格式
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
            //设置map的输出key和value类型
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(CombineValues.class);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
            //设置reduce的输出key和value类型
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(Text.class);
            job.waitForCompletion(true);
            return job.isSuccessful()?0:1;
    }
    public static void main(String[] args) throws IOException,
            ClassNotFoundException, InterruptedException {
        try {
            int returnCode =  ToolRunner.run(new ReduceSideJoin_LeftOuterJoin(),args);
            System.exit(returnCode);
        } catch (Exception e) {
            // TODO Auto-generated catch block
            logger.error(e.getMessage());
        }
    }
}
```
#####2. map side join
使用场景：一张表十分小、一张表很大。

用法:在提交作业的时候先将小表文件放到该作业的DistributedCache中，然后从DistributeCache中取出该小表进行join key / value解释分割放到内存中（可以放大Hash Map等等容器中）。然后扫描大表，看大表中的每条记录的join key /value值是否能够在内存中找到相同join key的记录，如果有则直接输出结果。

另外还有一种比较变态的Map Join方式，就是结合HBase来做Map Join操作。这种方式完全可以突破内存的控制，使你毫无忌惮的使用Map Join，而且效率也非常不错
#####3. Semi Join
将小表中参与join的key单独抽出来通过DistributedCach分发到相关节点，然后将其取出放到内存中（可以放到HashSet中），在map阶段扫描连接表，将join key不在内存HashSet中的记录过滤掉，让那些参与join的记录通过shuffle传输到reduce端进行join操作，其他的和reduce join都是一样的。

SemiJoin也是有一定的适用范围的，其抽取出来进行join的key是要放到内存中的，所以不能够太大，容易在Map端造成OOM。
```
public class SemiJoin extends Configured implements Tool{
    private static final Logger logger = LoggerFactory.getLogger(SemiJoin.class);
    public static class SemiJoinMapper extends Mapper<Object, Text, Text, CombineValues> {
        private CombineValues combineValues = new CombineValues();
        private HashSet<String> joinKeySet = new HashSet<String>();
        private Text flag = new Text();
        private Text joinKey = new Text();
        private Text secondPart = new Text();
        /**
         * 将参加join的key从DistributedCache取出放到内存中，以便在map端将要参加join的key过滤出来。b
         */
        @Override
        protected void setup(Context context)
                throws IOException, InterruptedException {
            BufferedReader br = null;
            //获得当前作业的DistributedCache相关文件
            Path[] distributePaths = DistributedCache.getLocalCacheFiles(context.getConfiguration());
            String joinKeyStr = null;
            for(Path p : distributePaths){
                if(p.toString().endsWith("joinKey.dat")){
                    //读缓存文件，并放到mem中
                    br = new BufferedReader(new FileReader(p.toString()));
                    while(null!=(joinKeyStr=br.readLine())){
                        joinKeySet.add(joinKeyStr);
                    }
                }
            }
        }
        @Override
        protected void map(Object key, Text value, Context context)
                throws IOException, InterruptedException {
            //获得文件输入路径
            String pathName = ((FileSplit) context.getInputSplit()).getPath().toString();
            //数据来自tb_dim_city.dat文件,标志即为"0"
            if(pathName.endsWith("tb_dim_city.dat")){
                String[] valueItems = value.toString().split("\\|");
                //过滤格式错误的记录
                if(valueItems.length != 5){
                    return;
                }
                //过滤掉不需要参加join的记录
                if(joinKeySet.contains(valueItems[0])){
                    flag.set("0");
                    joinKey.set(valueItems[0]);
                    secondPart.set(valueItems[1]+"\t"+valueItems[2]+"\t"+valueItems[3]+"\t"+valueItems[4]);
                    combineValues.setFlag(flag);
                    combineValues.setJoinKey(joinKey);
                    combineValues.setSecondPart(secondPart);
                    context.write(combineValues.getJoinKey(), combineValues);
                }else{
                    return ;
                }
            }//数据来自于tb_user_profiles.dat，标志即为"1"
            else if(pathName.endsWith("tb_user_profiles.dat")){
                String[] valueItems = value.toString().split("\\|");
                //过滤格式错误的记录
                if(valueItems.length != 4){
                    return;
                }
                //过滤掉不需要参加join的记录
                if(joinKeySet.contains(valueItems[3])){
                    flag.set("1");
                    joinKey.set(valueItems[3]);
                    secondPart.set(valueItems[0]+"\t"+valueItems[1]+"\t"+valueItems[2]);
                    combineValues.setFlag(flag);
                    combineValues.setJoinKey(joinKey);
                    combineValues.setSecondPart(secondPart);
                    context.write(combineValues.getJoinKey(), combineValues);
                }else{
                    return ;
                }
            }
        }
    }
    public static class SemiJoinReducer extends Reducer<Text, CombineValues, Text, Text> {
        //存储一个分组中的左表信息
        private ArrayList<Text> leftTable = new ArrayList<Text>();
        //存储一个分组中的右表信息
        private ArrayList<Text> rightTable = new ArrayList<Text>();
        private Text secondPar = null;
        private Text output = new Text();
        /**
         * 一个分组调用一次reduce函数
         */
        @Override
        protected void reduce(Text key, Iterable<CombineValues> value, Context context)
                throws IOException, InterruptedException {
            leftTable.clear();
            rightTable.clear();
            /**
             * 将分组中的元素按照文件分别进行存放
             * 这种方法要注意的问题：
             * 如果一个分组内的元素太多的话，可能会导致在reduce阶段出现OOM，
             * 在处理分布式问题之前最好先了解数据的分布情况，根据不同的分布采取最
             * 适当的处理方法，这样可以有效的防止导致OOM和数据过度倾斜问题。
             */
            for(CombineValues cv : value){
                secondPar = new Text(cv.getSecondPart().toString());
                //左表tb_dim_city
                if("0".equals(cv.getFlag().toString().trim())){
                    leftTable.add(secondPar);
                }
                //右表tb_user_profiles
                else if("1".equals(cv.getFlag().toString().trim())){
                    rightTable.add(secondPar);
                }
            }
            logger.info("tb_dim_city:"+leftTable.toString());
            logger.info("tb_user_profiles:"+rightTable.toString());
            for(Text leftPart : leftTable){
                for(Text rightPart : rightTable){
                    output.set(leftPart+ "\t" + rightPart);
                    context.write(key, output);
                }
            }
        }
    }
    @Override
    public int run(String[] args) throws Exception {
            Configuration conf=getConf(); //获得配置文件对象
            DistributedCache.addCacheFile(new Path(args[2]).toUri(), conf);
                                                                                                                                                                                                                                          
            Job job=new Job(conf,"LeftOutJoinMR");
            job.setJarByClass(SemiJoin.class);
                                                                                                                                                                                                                                          
            FileInputFormat.addInputPath(job, new Path(args[0])); //设置map输入文件路径
            FileOutputFormat.setOutputPath(job, new Path(args[1])); //设置reduce输出文件路径
                                                                                                                                                                                                                                                                                                                                                                               
            job.setMapperClass(SemiJoinMapper.class);
            job.setReducerClass(SemiJoinReducer.class);
                                                                                                                                                                                                                                         
            job.setInputFormatClass(TextInputFormat.class); //设置文件输入格式
            job.setOutputFormatClass(TextOutputFormat.class);//使用默认的output格式
                                                                                                                                                                                                                                          
            //设置map的输出key和value类型
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(CombineValues.class);
                                                                                                                                                                                                                                          
            //设置reduce的输出key和value类型
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(Text.class);
            job.waitForCompletion(true);
            return job.isSuccessful()?0:1;
    }
    public static void main(String[] args) throws IOException,
            ClassNotFoundException, InterruptedException {
        try {
            int returnCode =  ToolRunner.run(new SemiJoin(),args);
            System.exit(returnCode);
        } catch (Exception e) {
            logger.error(e.getMessage());
        }
    }
}
```
>https://wenku.baidu.com/view/ae7442db7f1922791688e877.html
#####4. reduce side join + BloomFilter
在某些情况下，SemiJoin抽取出来的小表的key集合在内存中仍然存放不下，这时候可以使用BloomFiler以节省空间。
>https://blog.csdn.net/jiaomeng/article/details/1495500
#####5. 二次排序
1. 在reduce()函数中，将某个key对应的所有value保存下来，然后进行排序。 这种方法最大的缺点是：可能会造成out of memory。
2. 实现Paritioner，以便只按照key进行数据划分

之前写的MapReduce api关于重写分区器(划分分区的方法),分组器(value 排序的依据)
>https://nino-laiqiu.github.io/2020/08/23/MapReduce-API/



