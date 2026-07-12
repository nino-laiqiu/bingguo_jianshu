发现问题:
1.hashmap的位置不同对结果有重大影响 比如第四题
2.不能再add或者put还没有结束的途中 排序
3.代码命名的规范
4.equals的重写,自然比较
5.关于索引??第6小题还没搞清楚,为什么会索引越界,大概率有脏数据,导致get(0)为空;
经过试验发现切实如此
```
     movies1.retainAll(value);
          // value.removeAll(Collections.singleton(null));
           // ArrayList<Object> objects = new ArrayList<>();
          //  objects.add("");
          //  movies1.removeAll(objects);
           // movies1.remove("");
           if (movies1.size()==0){
               continue;
             // System.out.println(movies1);
           }
```
**说明这些❀//的都不行,null是去不掉的,采取跳过**//可以在这里比较时间戳.......
6.比较器要比尽所有情况
7.关于hashmap的分类功能调用函数 -getOrDefault()..最后要add添加
8.第7题关于%的函数..获取前几位:
BigDecimal bigDecimal1 = new BigDecimal(SSIM).setScale(2, RoundingMode.UP);
关于第六题,retainAll 返回的仍然是自选的那个用户与遍历用户的公共的部分,关于公共部分的具体内容是自选用户的那一部分.
```
9.注意map的嵌套,第四套改进#写法,获取get(0)
package Dayt9_3;

import com.alibaba.druid.sql.visitor.SQLASTOutputVisitorUtils;
import com.alibaba.fastjson.JSON;
import sun.awt.shell.ShellFolder;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.*;


/**
 * 1.最火热的TOP10电影（评论最多，如评论数量一样则以平均评分做评判标准，越高越优先）
 * 2.最活跃的TOP10用户（评论最多，如评论数量一样则以平均评分做评判标准，越高越优先）
 * 3.最差劲的TOP10电影（评分最低，如评分一样则以评论条数做评判标准，越多越优先）
 * 4.每个用户评价最高的10部电影（如评分相同则以评论早晚做评判标准，越早评分越优先）
 * 5.每个电影评分最高的10个用户（如评分相同则以评论早晚做评判标准，越早评分越优先）
 * 6.查询与指定某个用户观影爱好最相近的10个用户（相同影片量越多，推荐度越高，若相同影片数量一
 * 致，则以最后活跃时间做评判标准，活跃越近的优先度越高）
 * 7.查询某影片的平均分、最高分（时间最早）、最低分（时间最早）、评论量、
 * 相似度最高的影片Top10（相同评论用户量越多，推荐度越高，分数越相近，推荐度越高）
 */
public class MovieTest {

    static List<Movie> fasitjionList = new ArrayList<>();
    //建立存放用户和电影名的map
    static Map<String, List<Movie>> movieHash = new HashMap<>();
    static Map<String, List<Movie>> uidHash = new HashMap<>();


    //测试方法
    public static void main(String[] args) throws IOException {
        // MovieTest movieTest = new MovieTest();
        jions();
        movieAndUid();
        //1.
        //hotMovie();
        //4.
        // userMovieLke();
        //4##
        //userMovieLking();
        //5.
        // ratingMovie();
        //6.
        // usersRoreCommend("4193");
        // test();
        //  userhot10liking();
        //userhot10likinging();
        //6
        theSameFilm();
        //7.
        //similarityMovie();
       // userhot10liking();
    }

    //把数据导入到fasitjionList
    public static void jions() throws IOException {

        Map<String, String> obj = new HashMap<>();
        BufferedReader bu = new BufferedReader(new FileReader(new File("C:\\Users\\hp\\IdeaProjects\\BookCollectio" +
                "n\\com.java.configuration\\rating (1).json")));
        String st = null;
        //这里的剔除的数据与老师的不一致,??????
        while ((st = bu.readLine()) != null) {
            try {
                Movie movie = JSON.parseObject(st, Movie.class);
                if (movie.getMovie() == null || movie.getMovie().equals("") || movie.getUid().equals("")|| movie.getTimeStamp() == 0||movie.getRate() == 0.0){
                    System.err.println(st);
                    continue;
                }
                fasitjionList.add(movie);
            } catch (Exception e) {
                System.err.println(st);
                continue;
            }
        }
    }


    //获取不重复的电影,用户,及其条数用list存储
    public static void movieAndUid() {
        for (Movie movie : fasitjionList) {
            //注意这里采取for循环是错误的
            /*String movieKey = fasitjionList.get(i).getMovie();
            String uidKey = fasitjionList.get(i).getUid();*/
            String movieKey = movie.getMovie();
            String uidKey = movie.getUid();

            List<Movie> movieValue = movieHash.getOrDefault(movieKey, new ArrayList<Movie>());
            //注意这里要添加不然为空.........
            movieValue.add(movie);
            List<Movie> uidValue = uidHash.getOrDefault(uidKey, new ArrayList<Movie>());
            uidValue.add(movie);
            movieHash.put(movieKey, movieValue);
            uidHash.put(uidKey, uidValue);

        }
    }

    // 1.最火热的TOP10电影（评论最多，如评论数量一样则以平均评分做评判标准，越高越优先
    public static void hotMovie() {
        //对moviehash集合操作
        //使用键值对容易操作,这个集合中存放了键值对
        List<Map.Entry<String, List<Movie>>> keyAndValueJianZhiDui = new ArrayList<>(movieHash.entrySet());
        //对这个集合进行排序
        keyAndValueJianZhiDui.sort(new Comparator<Map.Entry<String, List<Movie>>>() {
            @Override
            public int compare(Map.Entry<String, List<Movie>> stringListEntry, Map.Entry<String, List<Movie>> t1) {
                if (t1.getValue().size() > stringListEntry.getValue().size()) {
                    return 1;
                }
                //要获取平均值则要遍历

                //这里可以采用提取方法
                double sum = 0;
                //获取了总值
                for (Movie movie : t1.getValue()) {
                    sum += movie.getRate();
                }
                double sumt1 = sum / t1.getValue().size();
                //重复这个过程
                double sum2 = 0;
                //获取了总值
                for (Movie movie : stringListEntry.getValue()) {
                    sum2 += movie.getRate();
                }
                double sumt3 = sum2 / stringListEntry.getValue().size();
                if (t1.getValue().size() == stringListEntry.getValue().size()) {
                    if (sum2 > sumt1) {
                        return 1;
                    }
                }
                return -1;
            }
        });
        //获取前十个list
        if (keyAndValueJianZhiDui.size() > 10) {
            //不足10,不输入
            for (int i = 0; i < 10; i++) {
                //获取键值对关于movie 和对应的数量
                Map.Entry<String, List<Movie>> listHotMovies = keyAndValueJianZhiDui.get(i);
                //获取数量和对应名称
                System.out.println("最火的电影排名第" + (i + 1) + "位" + "   " + listHotMovies.getKey() +
                        "数量是" + listHotMovies.getValue().size());
            }

        }


    }

    //* 4.每个用户评价最高的10部电影（如评分相同则以评论早晚做评判标准，越早评分越优先）
    public static void userMovieLke() {
        //遍历原来的集合获取每个UID对应的list评论
        List<Map.Entry<String, List<Movie>>> entries = new ArrayList<>(uidHash.entrySet());
        //获取的键值对,要点是吧键对应的value值一一放到list中,对这个list继续排序
        //循环获取,或者如下增强for获取每一个键值对
        for (Map.Entry<String, List<Movie>> entry : entries) {
            //这是获取了用户
            String key = entry.getKey();
            //获取value值,放在list继续自定义排序
            //这是一个list集合,我们可以判断这个集合中元素
            List<Movie> value = entry.getValue();
            //这里的问题是报错了///////?????
            value.sort(new Comparator<Movie>() {
                @Override
                public int compare(Movie movie, Movie t1) {

                    return (t1.getRate() > movie.getRate()) ? 1 : (t1.getRate() < movie.getRate()) ?
                            -1 : (t1.getTimeStamp() == movie.getTimeStamp()) ? 0 : (t1.getTimeStamp() > movie.getTimeStamp()) ? 1 : -1;

                }
            });
            //获取用户前十的电影
            // 进行打印输出
            String st = " ";
            if (value.size() >= 10) {
                for (int i = 0; i < 10; i++) {
                    //问题是这么一次输出10部电影
                    st += value.get(i).getMovie() + " ";
                }
            }
            System.out.println("用户" + key + "  " + "最喜欢的10本书为" + st);

        }
    }

    public static void test() {
        Set<Map.Entry<String, List<Movie>>> entries = uidHash.entrySet();
        for (Map.Entry<String, List<Movie>> entry : entries) {
            String key = entry.getKey();
            List<Movie> value = entry.getValue();
            for (Movie movie : value) {
                System.out.println(key + "   " + movie);
            }
        }
    }

    //4#改进版,假设用户对同一电影有多次评论,则进行分组,获取最大的值(排序的标准先比较分数,然后比较时间戳)
    public static void userMovieLking() {
        //存储方式:电影名字,对应评论数
        Map<String, Map<String, List<Movie>>> objectObjectHashMap = new HashMap<>();
        //1.存储电影名字,2.存储那个电影的评论数量.......
        // List<Movie>  orMovie = new ArrayList<>();
        Map<String, List<Movie>> movieHashMap = new HashMap<>();
        for (Map.Entry<String, List<Movie>> entry : uidHash.entrySet()) {
            //获取uid
            String uid = entry.getKey();
            //System.out.println(uid);
            //获取电影按电影名字分类,获取了单个用户的电影,进行排序
            List<Movie> value = entry.getValue();
            //new 一个map存储,对这个用户的movie进行遍历
            for (Movie movie : value) {
                String movieName = movie.getMovie();
                //获取这个list的时间戳,拿不到提升变量
                List<Movie> orMovie = movieHashMap.getOrDefault(movieName, new ArrayList<Movie>());
                //注意一定要添加..........
                orMovie.add(movie);
                System.out.println(orMovie);
                // System.out.println(orMovie);

                //把这个去重过后的list存入到map集合中,如下这个map存储,////对这个map里的movie进行评分排序
                movieHashMap.put(movieName, orMovie);
            }
            //把外层放UID,里层的放(电影name,和对应的评论)
            objectObjectHashMap.put(uid, movieHashMap);
        }
       /* orMovie.sort(new Comparator<Movie>() {
            @Override
            public int compare(Movie movie, Movie t1) {
                //直接返回不对,三目运算符
                return t1.getTimeStamp()>movie.getTimeStamp() ? 1:-1;
            }
        });*/

/*public static  void userBestRateMovieTop10() {
    Set<Map.Entry<String, List<Movie>>> entries = uidHash.entrySet();
    List<Map.Entry<String, List<Movie>>> entries1 = new ArrayList<>(entries);
    for (Map.Entry<String, List<Movie>> integerListEntry : entries1) {
        String uid = integerListEntry.getKey();
        List<Movie> userAllMoviesList = integerListEntry.getValue();
        for (Movie movie : userAllMoviesList) {
            String movie1 = movie.getMovie();
            Map<String, ArrayList<Movie>> stringMovieHashMap = new HashMap<>();
            List<Movie> orDefault = stringMovieHashMap.getOrDefault(movie1, new ArrayList<Movie>());
            orDefault.add(movie);
            orDefault.sort(new Comparator<Movie>() {
                @Override
                public int compare(Movie o1, Movie o2) {
                    if (o2.getTimeStamp() > o1.getTimeStamp()) {
                        return 1;
                    } else if (o2.getTimeStamp() == o1.getTimeStamp()) {
                        return 0;
                    } else {
                        return -1;
                    }

                }
            });
            Map<String, Movie> orDefault1 = userMovieMap.getOrDefault(uid, new HashMap<String, Movie>());
            orDefault1.put(movie1, orDefault.get(0));
            userMovieMap.put(uid, orDefault1);
        }

    }
    Set<Map.Entry<Integer, Map<String, Movie>>> entries2 = userMovieMap.entrySet();
    ArrayList<Map.Entry<Integer, Map<String, Movie>>> entries3 = new ArrayList<>(entries2);
    for (Map.Entry<Integer, Map<String, Movie>> integerMapEntry : entries3) {
        Integer uid = integerMapEntry.getKey();
        System.out.println(uid + "喜欢的电影为");
        Map<String, Movie> movie = integerMapEntry.getValue();
        Set<Map.Entry<String, Movie>> entries4 = movie.entrySet();
        List<Map.Entry<String, Movie>> entries5 = new ArrayList<>(entries4);
    }
}*/

        //进行排序,获取每个用户最高的10部电影
        //获取每个电影的最近距离的电影是....
        //发现问题....
        Set<Map.Entry<String, Map<String, List<Movie>>>> entries2 = objectObjectHashMap.entrySet();
        for (Map.Entry<String, Map<String, List<Movie>>> stringMapEntry : entries2) {
            String key = stringMapEntry.getKey();
            // System.out.println(key+"==========");
            Map<String, List<Movie>> value = stringMapEntry.getValue();
            Set<Map.Entry<String, List<Movie>>> entries1 = value.entrySet();
            for (Map.Entry<String, List<Movie>> stringListEntry : entries1) {
                List<Movie> value1 = stringListEntry.getValue();
                for (Movie movie : value1) {
                    // System.out.println(movie);
                }
            }
        }
        Set<Map.Entry<String, Map<String, List<Movie>>>> entries4 = objectObjectHashMap.entrySet();
        for (Map.Entry<String, Map<String, List<Movie>>> stringMapEntry : entries4) {
            //获取名字
            String key = stringMapEntry.getKey();
            System.out.println("用户" + key);
            //对于一个电影名只取一个get(0),把所有的存放在集合中
            ArrayList<Movie> objects = new ArrayList<>();


            Set<Map.Entry<String, List<Movie>>> entries1 = stringMapEntry.getValue().entrySet();
            for (Map.Entry<String, List<Movie>> stringListEntry : entries1) {
                objects.add(stringListEntry.getValue().get(0));
            }
            objects.sort(new Comparator<Movie>() {
                @Override
                public int compare(Movie movie, Movie t1) {
                    return (t1.getRate() > movie.getRate()) ? 1 : (t1.getRate() < movie.getRate()) ?
                            -1 : (t1.getTimeStamp() == movie.getTimeStamp()) ? 0 :
                            (t1.getTimeStamp() > movie.getTimeStamp()) ? 1 : -1;
                }
            });

            //在里边添加打印结果
            if (objects.size() < 10) {
                for (Movie object : objects) {
                    //  System.out.println("最好评的书籍" + object);
                }
            } else {
                for (int i = 0; i < 10; i++) {
                    Movie movie = objects.get(i);
                    //System.out.println( "最好评的书籍" + movie);
                }
            }
        }
        //获取电影时间戳排名第一位
        //要对电影的评分进行排序????我是对这个新生成的list进行排序的?????
        /* List<Map.Entry<String, List<Movie>>> entries1 = new ArrayList<>(movieHashMap.entrySet());*/
           /*Collections.sort(entries1, new Comparator<Map.Entry<String, List<Movie>>>() {
               @Override
               public int compare(Map.Entry<String, List<Movie>> stringListEntry, Map.Entry<String, List<Movie>> t1) {
                     //对这个list进行排序
                   return   (t1.getValue().get(0).getRate()>stringListEntry.getValue().get(0).getRate()) ? 1:
                           (t1.getValue().get(0).getRate()>stringListEntry.getValue().get(0).getRate()) ? -1:
                             (t1.getValue().get(0).getRate()==stringListEntry.getValue().get(0).getRate()) ? 0:


               }
           });*/
        //这是一个排序完成后的list,或者怎么找UID???
        //取出对于UID
       /* Set<Map.Entry<String, Map<String, List<Movie>>>> entries2 = objectObjectHashMap.entrySet();
        for (Map.Entry<String, Map<String, List<Movie>>> stringMapEntry : entries2) {
            //这个是UID
            String uid = stringMapEntry.getKey();
            //这是里边的map
            Set<Map.Entry<String, List<Movie>>> entries3 = stringMapEntry.getValue().entrySet();
            for (Map.Entry<String, List<Movie>> stringListEntry : entries3) {
                //这是电影名字
                String key = stringListEntry.getKey();
                //这是电影的评论,在这里进行排序把
                List<Movie> value = stringListEntry.getValue();

            }
        }*/

        // 把这个新生成的list放到map中


    }

    //4###重写一遍:假设用户对同一电影有多次评论,则进行分组,获取最大的值(排序的标准先比较分数,然后比较时间戳)
    public static void userhot10liking() {

        // HashMap<String, List<Movie>> hashMap = new HashMap<>();
        //里层为上面那个集合....
        HashMap<String, Map<String, List<Movie>>> ultimatelyMap = new HashMap<>();
        //获取键值对
        for (Map.Entry<String, List<Movie>> stringListEntry : uidHash.entrySet()) {//stringListEntry 键值对
            //,,,,,,,,,,,,,,,,,,这个错误找半天了,应该放在for里面
            //建一个map集合存储,分组标准为movieName
            HashMap<String, List<Movie>> standardWithMovieName = new HashMap<>();
            //考虑建立一个list存储评论(不重复电影)
            List<Movie> notRepetitionMovie = new ArrayList<>();
            String uidName = stringListEntry.getKey();// uidName 用户名字
            List<Movie> uidMovieU = stringListEntry.getValue();// uidMovieU 用户为分组标准,电影评论集合
            for (Movie movie : uidMovieU) {
                String movieName = movie.getMovie();
                //建一个map集合存储,分组标准为movieName
                List<Movie> orDefault = standardWithMovieName.getOrDefault(movieName, new ArrayList<Movie>());
                orDefault.add(movie);
                // System.out.println(orDefault);
                standardWithMovieName.put(movieName, orDefault);//以电影名字为标准的评论分组


            }

            //考虑需求,用户对同一电影有多次评论,获取最近的那一条
            //在add之后再排序.....
            for (Map.Entry<String, List<Movie>> listEntry : standardWithMovieName.entrySet()) {
                List<Movie> value = listEntry.getValue();
                //排序
                value.sort(new Comparator<Movie>() {
                    @Override
                    public int compare(Movie movie, Movie t1) {
                        return t1.getTimeStamp() > movie.getTimeStamp() ? 1 : -1;

                    }
                });
                //获取第一条
                Movie movie = value.get(0);
                //System.out.println(movie.getTimeStamp());
                //考虑建立一个list存储评论(不重复电影的电影评论)
                notRepetitionMovie.add(movie);
            }
            //排序repetition
            notRepetitionMovie.sort(new Comparator<Movie>() {
                @Override
                public int compare(Movie movie, Movie t1) {
                    return (t1.getRate() > movie.getRate()) ? 1 : (t1.getRate() < movie.getRate()) ?
                            -1 : (t1.getTimeStamp() == movie.getTimeStamp()) ? 0 :
                            (t1.getTimeStamp() > movie.getTimeStamp()) ? 1 : -1;
                }
            });
            // hashMap.put(,notRepetitionMovie)
            //考虑建立一个map集合存储uid....如下操作没用到

            ultimatelyMap.put(uidName, standardWithMovieName);
            //进行遍历
            if (notRepetitionMovie.size() < 10) {
                for (int i = 0; i < notRepetitionMovie.size(); i++) {
                   // System.out.println(uidName + " " + notRepetitionMovie.get(i));
                }
            } else {
                for (int i = 0; i < 10; i++) {
                  //  System.out.println(uidName + "  " + notRepetitionMovie.get(i));
                }
            }
        }
    }

    //4###重写一遍2:假设用户对同一电影有多次评论,则进行分组,获取最大的值(排序的标准先比较分数,然后比较时间戳)
   public  static void  userhot10likinging() {
        // HashMap<String, List<Movie>> hashMap = new HashMap<>();
        //里层为上面那个集合....
        HashMap<String, Map<String, Movie>> ultimatelyMap = new HashMap<>();
        //获取键值对
        for (Map.Entry<String, List<Movie>> stringListEntry : uidHash.entrySet()) {//stringListEntry 键值对
            //考虑建立一个list存储评论(不重复电影)
            List<Movie> notRepetitionMovie = new ArrayList<>();
            String uidName = stringListEntry.getKey();// uidName 用户名字
            List<Movie> uidMovieU = stringListEntry.getValue();// uidMovieU 用户为分组标准,电影评论集合
            for (Movie movie : uidMovieU) {
                String movieName = movie.getMovie();
                //建一个map集合存储,分组标准为movieName
                HashMap<String, List<Movie>> standardWithMovieName = new HashMap<>();
                List<Movie> orDefault = standardWithMovieName.getOrDefault(movieName, new ArrayList<Movie>());
                orDefault.sort(new Comparator<Movie>() {
                    @Override
                    public int compare(Movie o1, Movie o2) {
                        if (o2.getTimeStamp() > o1.getTimeStamp()) {
                            return 1;
                        } else if (o2.getTimeStamp() == o1.getTimeStamp()) {
                            return 0;
                        } else {
                            return -1;
                        }
                    }
                });

                System.out.println(orDefault);
                standardWithMovieName.put(movieName, orDefault);//以电影名字为标准的评论分组*//*

                Map<String, Movie> orDefault1 = ultimatelyMap.getOrDefault(uidName, new HashMap<String, Movie>());
                orDefault1.put(movieName, orDefault.get(0));
                ultimatelyMap.put(uidName, orDefault1);

                //考虑需求
                Set<Map.Entry<String, Map<String, Movie>>> entries2 = ultimatelyMap.entrySet();
                ArrayList<Map.Entry<String, Map<String, Movie>>> entries3 = new ArrayList<>(entries2);
                for (Map.Entry<String, Map<String, Movie>> integerMapEntry : entries3) {
                    String uid = integerMapEntry.getKey();
                    System.out.println(uid + "喜欢的电影为");
                    Map<String, Movie> movie1 = integerMapEntry.getValue();
                    Set<Map.Entry<String, Movie>> entries4 = movie1.entrySet();
                    List<Map.Entry<String, Movie>> entries5 = new ArrayList<>(entries4);
                    entries5.sort(new Comparator<Map.Entry<String, Movie>>() {
                        @Override
                        public int compare(Map.Entry<String, Movie> movie, Map.Entry<String, Movie> t1) {
                            return (t1.getValue().getRate() > movie.getValue().getRate()) ? 1 : (t1.getValue().getRate()
                                    < movie.getValue().getRate()) ? -1 : (t1.getValue().getTimeStamp() == movie.getValue()
                                    .getTimeStamp()) ? 0 : (t1.getValue().getTimeStamp() > movie.getValue().getTimeStamp()
                            ) ? 1 : -1;
                        }
                    });
                    if (entries5.size() < 10) {
                        for (int i = 0; i < entries5.size(); i++) {
                            System.out.println(entries5.get(i));
                        }
                    } else {
                        for (int i = 0; i < 10; i++) {
                            System.out.println(entries5.get(i));
                        }
                    }
                }
            }
        }
    }
    //  5.每个电影评分最高的10个用户（如评分相同则以评论早晚做评判标准，越早评分越优先）
    public static void ratingMovie() {
        //获取电影
        List<Map.Entry<String, List<Movie>>> entries = new ArrayList<>(movieHash.entrySet());
        //获取键值对,获取对应的list中评分,排序
        for (Map.Entry<String, List<Movie>> entry : entries) {
            //获取一个键值对
            //这是电影名称
            String key = entry.getKey();
            //这是电影的品论
            List<Movie> value = entry.getValue();
            //排序
    /*         value.sort(new Comparator<Movie>() {
                 @Override
                 public int compare(Movie movie, Movie t1) {
                     if (t1.getRate()-movie.getRate()>0.0) {
                          return 1;
                     }
                     else if (t1.getRate()-movie.getRate()==0.0){
                         if (t1.getTimeStamp()> t1.getTimeStamp()){
                              return  1;
                         }
                         else {return  -1;}
                     }
                     return -1;
                 }
             });*/
            Collections.sort(value, new Comparator<Movie>() {
                @Override
                public int compare(Movie movie, Movie t1) {
                    System.setProperty("java.util.Arrays.useLegacyMergeSort", "true");

       /*   if (t1.getRate()-movie.getRate()>0.0) {
                return 1;
            }
            else if (t1.getRate()-movie.getRate()==0.0){
                if (t1.getTimeStamp()> t1.getTimeStamp()){
                    return  1;
                }

            }*/

                    return (t1.getRate() > movie.getRate()) ? 1 : (t1.getRate() < movie.getRate()) ?
                            -1 : (t1.getTimeStamp() == movie.getTimeStamp()) ? 0 : (t1.getTimeStamp() > movie.getTimeStamp()) ? 1 : -1;
                }

            });
            //排序完成获取用户,加入不足10则按size
            if (value.size() < 10) {
                //输出size位
                String st = " ";
                for (int i = 0; i < value.size(); i++) {
                    //问题是这么一次输出10部电影
                    st += value.get(i).getMovie() + " ";
                }
                System.out.println("电影" + key + "  " + "评分最高的10位" + st);
            } else {
                String st = " ";
                for (int i = 0; i < 10; i++) {
                    //问题是这么一次输出10部电影
                    st += value.get(i).getMovie() + " ";
                }
                System.out.println("电影" + key + "  " + "评分最高的10位" + st);
            }

        }
    }

    /* 6.查询与指定某个用户观影爱好最相近的10个用户（相同影片量越多，推荐度越高，若相同影片数量一
     * 致，则以最后活跃时间做评判标准，活跃越近的优先度越高）*/
    public static void usersRoreCommend(String uid) {
        //使用retainAll方法获取相同的元素,把用户所有的电影数量放在一个集合中继续遍历,对于一致的我们比较时间戳
        //先判断是否包含该用户
        if (!uidHash.containsKey(uid)) {
            System.err.println("没有该用户");
        } else {
            //获取该用户的value值
            //返回了评论的集合
            List<Movie> movies = uidHash.get(uid);
            //获取这个电影的名称,放到集合中
            List<String> movieName = new ArrayList<>();
            //遍历,放到集合中
            for (Movie movie : movies) {
                movieName.add(movie.getMovie());
            }
            //对hashmap继续遍历获取用户,电影的集合
            Set<Map.Entry<String, List<Movie>>> entries = uidHash.entrySet();
            //map,存放UID 和 用户电影的名字
            Map<String, List> obje = new HashMap<>();
            for (Map.Entry<String, List<Movie>> entry : entries) {
                //获取名字
                String name = entry.getKey();
                //评论的个数集合形式
                List<Movie> value = entry.getValue();
                //获取其中的电影名字
                // 这里可以优化
                List<Object> objects = new ArrayList<>();
                for (Movie movie : value) {
                    String movie1 = movie.getMovie();
                    //每次获取一个就添加进map中的list中
                    objects.add(movie1);

                }
                //添加到map集合中
                //储存了用户名字和它的电影数量
                obje.put(name, objects);
            }
            //进行比较,把比较结果存放到集合中
            //建立一个副本继续操作
            Map<String, Integer> objec = new HashMap<>();//用来存放int书进行排序
            Set<Map.Entry<String, List>> entries1 = obje.entrySet();
            //对该set集合进行遍历获取用户,和数量
            for (Map.Entry<String, List> stringListEntry : entries1) {
                //这个是姓名
                String key = stringListEntry.getKey();
                //获取的是电影名字的集合
                List value = stringListEntry.getValue();

                List<String> uidList1 = new ArrayList<>(movieName);
                //进行比较赋给uidList1
                boolean b = uidList1.removeAll(value);
                //int 把这个int 存入到集合中
                int size = uidList1.size();
                objec.put(key, size);

            }
            //对获取的集合获取
            //转换成键值对
            List<Map.Entry<String, Integer>> entries2 = new ArrayList<>(objec.entrySet());
            //定制排序
            Collections.sort(entries2, new Comparator<Map.Entry<String, Integer>>() {
                @Override
                public int compare(Map.Entry<String, Integer> stringIntegerEntry, Map.Entry<String, Integer> t1) {
                    return t1.getValue() > stringIntegerEntry.getValue() ? 1 : -1;

                }
            });
            if (entries2.size() >= 10) {
                for (int i = 0; i < 10; i++) {
                    //这是姓名
                    String key = entries2.get(i).getKey();
                    //这是数量
                    Integer value = entries2.get(1).getValue();
                    System.out.println("相似与" + key + "相似个数为" + value);
                }
            }
        }
    }

    /* 6改进版.查询与指定某个用户观影爱好最相近的10个用户（相同影片量越多，推荐度越高，若相同影片数量一
     * 致，则以最后活跃时间做评判标准，活跃越近的优先度越高）*/
    //注意要点重写equals方法,只比较第一个moviename,s使用new BigDecimal()方法获取百分前几位
    public static void theSameFilm() {
        //重写equals方法
        //例如某用户为1667
        List<Movie> movies = uidHash.get("1667");//返回了这个用户moie集合
        movies.sort(new Comparator<Movie>() {
            @Override
            public int compare(Movie movie, Movie t1) {

                if (t1.getTimeStamp()>movie.getTimeStamp()){
                    return  1;
                }
                else if (t1.getTimeStamp()==movie.getTimeStamp()){
                    return 0;
                }
                else {return -1;}
            }
        });
        //获取第一个 这是这个用户1667用户的最大时间戳
        long timeStamp = movies.get(0).getTimeStamp();
        Map<String, List<Movie>> obj = new HashMap<>();//考虑建立一个map集合存储
        for (Map.Entry<String, List<Movie>> stringListEntry : uidHash.entrySet()) {
            String key = stringListEntry.getKey();//这是用户的名字
            List<Movie> value = stringListEntry.getValue();//这是用户的电影评论数集合
            //建立一个副本进行比较
            List<Movie> movies1 = new ArrayList<>(value);

            //movies 和 movies 进行比较相同电影数,因为重写了equals方法只比较电影数
            movies1.retainAll(movies);
            //考虑建立一个map集合存储..这是一个很重要的问题这个map集合放在循环外面
            obj.put(key, movies1);
        }
        //
        //在循环外面进行排序规则
        List<Map.Entry<String, List<Movie>>> entries = new ArrayList<>(obj.entrySet());
            for (Map.Entry<String, List<Movie>> stringListEntry : entries) {
                List<Movie> value = stringListEntry.getValue();
                value.sort(new Comparator<Movie>() {
                    @Override
                    public int compare(Movie movie, Movie t1) {
                        if (t1.getTimeStamp() > movie.getTimeStamp()) {
                            return 1;
                        } else if (t1.getTimeStamp() == movie.getTimeStamp()) {
                            return 0;
                        } else {
                            return -1;
                        }
                    }
                });

                    long timeStamp1 = value.get(0).getTimeStamp();
                    System.out.println(timeStamp1);
            }
            entries.sort(new Comparator<Map.Entry<String, List<Movie>>>() {
                @Override
                public int compare(Map.Entry<String, List<Movie>> t2, Map.Entry<String, List<Movie>> t1) {
                        if (t1.getValue().size()>t2.getValue().size()){
                            return 1;
                        }
                        else if (t1.getValue().size()==t2.getValue().size()){
                            if (t1.getValue().get(0).getTimeStamp()>t2.getValue().get(0).getTimeStamp()){
                                return 1;
                            }
                            else if (t1.getValue().get(0).getTimeStamp()==t2.getValue().get(0).getTimeStamp()){
                                return 0;
                            }
                            else {return -1;}
                        }
                    return -1;
                }
            });


        //排序完成,获取1667电影评论的长度
        int size = movies.size();
        for (int i = 1; i <= 10; i++) {
            String key = entries.get(i).getKey();//这是用户
            int size1 = entries.get(i).getValue().size();//这是这个用户与某用户电影评论相似数
           // long timeStamp1 = entries.get(i).getValue().get(0).getTimeStamp();
           // long l = timeStamp- timeStamp1;
            //学习一下......
            double SSIM = size1 / (size + 0.0) * 100;
            //获取前几位
            BigDecimal bigDecimal = new BigDecimal(SSIM).setScale(2, RoundingMode.UP);
            System.out.println("1667用户与" + key + "用户的相似度为" + bigDecimal + "%" + " " + "相同影片数量为" + size1 );

        }

    }
    /*  * 7.查询某影片的平均分、最高分（时间最早）、最低分（时间最早）、评论量、
     * 相似度最高的影片Top10（相同评论用户量越多，推荐度越高，分数越相近，推荐度越高）*/

    public static void similarityMovie() {
        //就平均分进行比较
        //获取电影1998为 的平均分
        List<Movie> movies = movieHash.get("1998");
        //进行排序,就降序
        movies.sort(new Comparator<Movie>() {
            @Override
            public int compare(Movie movie, Movie t1) {

                if (t1.getRate() >movie.getRate()) {
                     return 1;
                }
                else if (t1.getRate() ==movie.getRate()){
                    return 0;
                }
               return -1;
            }
        });
        //定义一个final接收平均分
        //遍历获取平均分
        double fianlRateMovie = 0.0;
        for (Movie movie : movies) {
            double rate = movie.getRate();
             fianlRateMovie+=rate;
        }
        //平均分如下
         fianlRateMovie = fianlRateMovie/movies.size();


        //对moviehash进行遍历,获取相同的电影评论
       Map<String, List<Movie>> obj = new HashMap<>();//key电影名 value电影评论条数集合
        ArrayList<Map.Entry<String, List<Movie>>> entries = new ArrayList<>(movieHash.entrySet());
        for (Map.Entry<String, List<Movie>> entry : entries) {
            String key = entry.getKey();
            List<Movie> value = entry.getValue();
            List<Movie> movies1 = new ArrayList<>(value);
            movies1.retainAll(movies);
            //把如上添加到map中
             obj.put(key,movies1);
        }
        //完成之后进行排序 排序的规则是先按评论条数,如果相同则按平均分来排序
        ArrayList<Map.Entry<String, List<Movie>>> entries1 = new ArrayList<>(obj.entrySet());
          //获取某一个用户的平均分,防止模糊不清
        double finalFianlRateMovie = fianlRateMovie;
       Collections.sort(entries1,new Comparator<Map.Entry<String, List<Movie>>>() {
            @Override
            public int compare(Map.Entry<String, List<Movie>> t2, Map.Entry<String, List<Movie>> t1) {

                  //获取平均分的步骤
                double s1 =0.0;
                double s2 =0.0;
                List<Movie> value1 = t1.getValue();
                List<Movie> value2 = t2.getValue();
                for (Movie movie : value1) {
                    double rate = movie.getRate();
                    s1+=rate;
                }
                for (Movie movie : value2) {
                    double rate = movie.getRate();
                    s2+=rate;
                }
                //获取平均分
                s1 =s1/value1.size();
                s2=s2/value2.size();
                //判断规则
                if (t1.getValue().size()>t2.getValue().size()){
                    return 1;
                }
                else if (t1.getValue().size()==t2.getValue().size()){
                    if ((s1-finalFianlRateMovie)>(s2-finalFianlRateMovie)){
                        return 1;
                    }
                    else if ((s1-finalFianlRateMovie)<(s2-finalFianlRateMovie)){
                        return -1;
                    }
                    else {return -1;}
                }
                else {return -1;}
            }
        });

        //获取某影片的需求 查询某影片的平均分、最高分（时间最早）、最低分（时间最早）、评论量、
        BigDecimal bigDecimal = new BigDecimal(finalFianlRateMovie).setScale(2, RoundingMode.UP);
        System.out.println("影片1998" + "的平均分是" + bigDecimal +" " +
                "最高分是" + movies.get(0).getRate() + "最低分是" + movies.get(movies.size()-1).getRate() + "评论量是" +movies.size());

        //相似度比较,与上一题类似   相似度最高的影片Top10（相同评论用户量越多，推荐度越高，分数越相近，推荐度越高）

        //注意要重写equals方法
        for (int i = 1; i <= 10; i++) {
            String key = entries1.get(i).getKey();//这是影片
            List<Movie> value = entries1.get(i).getValue();//这是影片数量集合
            double SSIM = value.size()/(movies.size()+0.0)*100;
            BigDecimal bigDecimal1 = new BigDecimal(SSIM).setScale(2, RoundingMode.UP);
            System.out.println("影片1998与电影" + key + "的相似度为" + bigDecimal1 + "%");
        }
    }
}

```
