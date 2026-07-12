##1.异步IO

>在与外部系统交互（用数据库中的数据扩充流数据）的时候，需要考虑与外部系统的通信延迟对整个流处理应用的影响。
简单地访问外部数据库的数据，比如使用 MapFunction，通常意味着同步交互： MapFunction 向数据库发送一个请求然后一直等待，直到收到响应。在许多情况下，等待占据了函数运行的大部分时间。
与数据库异步交互是指一个并行函数实例可以并发地处理多个请求和接收多个响应。这样，函数在等待的时间可以发送其他请求和接收其他响应。至少等待的时间可以被多个请求摊分。大多数情况下，异步交互可以大幅度提高流处理的吞吐量

>仅仅提高 MapFunction 的并行度（parallelism）在有些情况下也可以提升吞吐量，但是这样做通常会导致非常高的资源消耗：更多的并行 MapFunction 实例意味着更多的 Task、更多的线程、更多的 Flink 内部网络连接、 更多的与数据库的网络连接、更多的缓冲和更多程序内部协调的开销。

>https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/dev/stream/operators/asyncio.html

![image.png](https://upload-images.jianshu.io/upload_images/9049859-4051494b323890fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
public class LogBean {

    public String uid;

    public Double longitude;

    public Double latitude;

    public String province;

    public String city;

    public LogBean(){}

    public LogBean(String uid, Double longitude, Double latitude) {
        this.uid = uid;
        this.longitude = longitude;
        this.latitude = latitude;
    }

    public static LogBean of(String uid, Double longitude, Double latitude) {
        return new LogBean(uid, longitude, latitude);
    }

    @Override
    public String toString() {
        return "LogBean{" +
                "uid='" + uid + '\'' +
                ", longitude=" + longitude +
                ", latitude=" + latitude +
                ", province='" + province + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

```
public class AsyncHttpGeoQueryFunction extends RichAsyncFunction<String, LogBean> {
    private transient CloseableHttpAsyncClient httpclient; //异步请求的HttpClient
    private String url; //请求高德地图URL地址
    private String key; //请求高德地图的秘钥，注册高德地图开发者后获得
    private int maxConnTotal; //异步HTTPClient支持的最大连接
    public AsyncHttpGeoQueryFunction(String url, String key, int maxConnTotal) {
        this.url = url;
        this.key = key;
        this.maxConnTotal = maxConnTotal;
    }
    @Override
    public void open(Configuration parameters) throws Exception {
        RequestConfig requestConfig = RequestConfig.custom().build();
        httpclient = HttpAsyncClients.custom() //创建HttpAsyncClients请求连接池
                .setMaxConnTotal(maxConnTotal) //设置最大连接数
                .setDefaultRequestConfig(requestConfig).build();
        httpclient.start(); //启动异步请求httpClient
    }

    @Override
    public void asyncInvoke(String line, ResultFuture<LogBean> resultFuture) throws Exception {
        //使用fastjson将json字符串解析成json对象
        LogBean bean = JSON.parseObject(line, LogBean.class);
        double longitude = bean.longitude; //获取经度
        double latitude = bean.latitude; //获取维度
        //将经纬度和高德地图的key与请求的url进行拼接
        HttpGet httpGet = new HttpGet(url + "?location=" + longitude + "," + latitude + "&key=" + key);
        //发送异步请求，返回Future
        Future<HttpResponse> future = httpclient.execute(httpGet, null);
        CompletableFuture.supplyAsync(new Supplier<LogBean>() {
            @Override
            public LogBean get() {
                try {
                    HttpResponse response = future.get();
                    String province = null;
                    String city = null;
                    if (response.getStatusLine().getStatusCode() == 200) {
                        //解析返回的结果，获取省份、城市等信息
                        String result = EntityUtils.toString(response.getEntity());
                        JSONObject jsonObj = JSON.parseObject(result);
                        JSONObject regeocode = jsonObj.getJSONObject("regeocode");
                        if (regeocode != null && !regeocode.isEmpty()) {
                            JSONObject address = regeocode.getJSONObject("addressComponent");
                            province = address.getString("province");
                            city = address.getString("city");
                        }
                    }
                    bean.province = province; //将返回的结果给省份赋值
                    bean.city = city; //将返回的结果给城市赋值
                    return bean;
                } catch (Exception e) {
                    return null;
                }
            }
        }).thenAccept((LogBean result) -> {
            //将结果添加到resultFuture中输出（complete方法的参数只能为集合，如果只有一个元素，就返回一个单例集合）
            resultFuture.complete(Collections.singleton(result));
        });
    }
    @Override
    public void close() throws Exception {
        httpclient.close(); //关闭HttpAsyncClients请求连接池
    }
}

```

```
public class AsyncQueryFromHttpDemo1 {


    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);
        String url = "http://localhost:8080/api"; //异步IO发生REST请求的地址
        int capacity = 20; //最大异步并发请求数量
        //使用AsyncDataStream调用unorderedWait方法，并传入异步请求的Function
        DataStream<Tuple2<String, String>> result = AsyncDataStream.unorderedWait(
                lines, //输入的数据流
                new HttpAsyncFunction(url, capacity), //异步查询的Function实例
                5000, //超时时间
                TimeUnit.MILLISECONDS, //时间单位
                capacity); //异步请求队列最大的数量，不传该参数默认值为100
        result.print();
        env.execute();
    }
}

```


```
public class AsyncQueryFromHttpDemo2 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //设置job的重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);
        String url = "https://restapi.amap.com/v3/geocode/regeo"; //异步请求高德地图的地址
        String key = "4924f7ef5c86a278f5500851541cdcff"; //请求高德地图的秘钥，注册高德地图开发者后获得
        int capacity = 50; //最大异步并发请求数量
        //使用AsyncDataStream调用unorderedWait方法，并传入异步请求的Function
        SingleOutputStreamOperator<LogBean> result = AsyncDataStream.unorderedWait(
                lines, //输入的数据流
                new AsyncHttpGeoQueryFunction(url, key, capacity), //异步查询的Function实例
                3000, //超时时间
                TimeUnit.MILLISECONDS, //时间单位
                capacity);//异步请求队列最大的数量，不传该参数默认值为100
        result.print();
        env.execute();
    }
}
```

```
public class AsyncQueryFromMySQL {


    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000)); //设置job的重启策略
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);
        int capacity = 50;
        DataStream<Tuple2<String, String>> result = AsyncDataStream.orderedWait(
                lines, //输入的数据流
                new MySQLAsyncFunction(capacity), //异步查询的Function实例
                3000, //超时时间
                TimeUnit.MILLISECONDS, //时间单位
                capacity); //异步请求队列最大的数量，不传该参数默认值为100
        result.print();
        env.execute();

    }
}
```

```
public class HttpAsyncFunction extends RichAsyncFunction<String, Tuple2<String, String>> {
    private transient CloseableHttpAsyncClient httpclient; //异步请求的HttpClient
    private String url; //请求的URL地址
    private int maxConnTotal; //异步HTTPClient支持的最大连接
    public HttpAsyncFunction(String url, int maxConnTotal) {
        this.url = url;
        this.maxConnTotal = maxConnTotal;
    }
    @Override
    public void open(Configuration parameters) throws Exception {
        RequestConfig requestConfig = RequestConfig.custom().build();
        httpclient = HttpAsyncClients.custom() //创建HttpAsyncClients请求连接池
                .setMaxConnTotal(maxConnTotal) //设置最大连接数
                .setDefaultRequestConfig(requestConfig).build();
        httpclient.start(); //启动异步请求httpClient
    }
    @Override
    public void asyncInvoke(String uid, final ResultFuture<Tuple2<String, String>> resultFuture)
            throws Exception {
        HttpGet httpGet = new HttpGet(url + "/?uid=" + uid); //请求的地址和参数
        Future<HttpResponse> future = httpclient.execute(httpGet, null); //执行请求返回future
        CompletableFuture.supplyAsync(new Supplier<String>() {
            @Override
            public String get() {
                try {
                    HttpResponse response = future.get(); //调用Future的get方法获取请求的结果
                    String res = null;
                    if(response.getStatusLine().getStatusCode() == 200) {
                        res = EntityUtils.toString(response.getEntity());
                    }
                    return res;
                } catch (Exception e) {
                    return null;
                }
            }
        }).thenAccept((String result) -> {
            //将结果添加到resultFuture中输出（complete方法的参数只能为集合，如果只有一个元素，就返回一个单例集合）
            resultFuture.complete(Collections.singleton(Tuple2.of(uid, result)));
        });
    }
    @Override
    public void close() throws Exception {
        httpclient.close(); //关闭HttpAsyncClients请求连接池
    }
}
```

```
public class MySQLAsyncFunction extends RichAsyncFunction<String, Tuple2<String, String>> {
    private transient DruidDataSource dataSource; //使用alibaba的Druid数据库连接池
    private transient ExecutorService executorService; //用于提交多个异步请求的线程池
    private int maxConnTotal; //线程池最大线程数量
    public MySQLAsyncFunction(int maxConnTotal) {
        this.maxConnTotal = maxConnTotal;
    }
    @Override
    public void open(Configuration parameters) throws Exception {
        executorService = Executors.newFixedThreadPool(maxConnTotal); //创建固定的大小的线程池
        dataSource = new DruidDataSource(); //创建数据库连接池并指定对应的参数
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        dataSource.setUrl("jdbc:mysql://localhost:3306/bigdata?characterEncoding=UTF-8");
        dataSource.setMaxActive(maxConnTotal);
    }
    @Override
    public void close() throws Exception {
        dataSource.close(); //关闭数据库连接池
        executorService.shutdown(); //关闭线程池
    }
    @Override
    public void asyncInvoke(String id, ResultFuture<Tuple2<String, String>> resultFuture) throws Exception {
        //调用线程池的submit方法，将查询请求丢入到线程池中异步执行，返回Future对象
        Future<String> future = executorService.submit(() -> {
            return queryFromMySql(id); //查询数据库的方法
        });
        CompletableFuture.supplyAsync(new Supplier<String>() {
            @Override
            public String get() {
                try {
                    return future.get(); //获取查询的结果
                } catch (Exception e) {
                    return null;
                }
            }
        }).thenAccept((String result) -> {
            //将id和查询的结果用Tuple2封装，放入到ResultFuture中输出
            resultFuture.complete(Collections.singleton(Tuple2.of(id, result)));
        });
    }

    private String queryFromMySql(String param) throws SQLException {
        String sql = "SELECT id, info FROM t_data WHERE id = ?";
        String result = null;
        Connection connection = null;
        PreparedStatement stmt = null;
        ResultSet rs = null;
        try {
            connection = dataSource.getConnection();
            stmt = connection.prepareStatement(sql);
            stmt.setString(1, param); //设置查询参数
            rs = stmt.executeQuery(); //执行查询
            while (rs.next()) {
                result = rs.getString("info"); //返回查询结果
            }
        } finally {
            if (rs != null) {
                rs.close();
            }
            if (stmt != null) {
                stmt.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
        return result;
    }
}
```
