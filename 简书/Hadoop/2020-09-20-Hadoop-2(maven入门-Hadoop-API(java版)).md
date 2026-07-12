##1.maven快速入门 (项目构建工具)

单词;toggle offline mode   离线模式开关
        show dependencies   显示依赖性

#####在maven的配置文件pom.xml标签添加

>pom.xml标签添加

```
<build>
        <plugins>
            <!--编译插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <!--添加hdfs的客户端依赖-->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>3.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>3.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-common</artifactId>
            <version>3.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
            <version>3.1.1</version>
        </dependency>
    </dependencies>
```

#####创建maven
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d45ba8f43f5a371b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>与repository同级配置settings.xml
```
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
 <localRepository>C:\maven\repository</localRepository>
  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->

  <!-- offline
   | Determines whether maven should attempt to connect to the network when executing a build.
   | This will have an effect on artifact downloads, artifact deployment, and others.
   |
   | Default: false
  <offline>false</offline>
  -->

  <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     |
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->

    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

  <!-- mirrors
   | This is a list of mirrors to be used in downloading artifacts from remote repositories.
   |
   | It works like this: a POM may declare a repository to use in resolving certain artifacts.
   | However, this repository may have problems with heavy traffic at times, so people have mirrored
   | it to several places.
   |
   | That repository definition will have a unique id, so we can create a mirror reference for that
   | repository, to be used as an alternate download site. The mirror site will be the preferred
   | server for that repository.
   |-->
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
	  <mirror>
          <id>nexus-aliyun</id>
          <mirrorOf>central</mirrorOf>
          <name>Nexus aliyun</name>
          <url>http://maven.aliyun.com/nexus/content/groups/public</url> 
      </mirror>
  </mirrors>

  <!-- profiles
   | This is a list of profiles which can be activated in a variety of ways, and which can modify
   | the build process. Profiles provided in the settings.xml are intended to provide local machine-
   | specific paths and repository locations which allow the build to work in the local environment.
   |
   | For example, if you have an integration testing plugin - like cactus - that needs to know where
   | your Tomcat instance is installed, you can provide a variable here such that the variable is
   | dereferenced during the build process to configure the cactus plugin.
   |
   | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
   | section of this document (settings.xml) - will be discussed later. Another way essentially
   | relies on the detection of a system property, either matching a particular value for the property,
   | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
   | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
   | Finally, the list of active profiles can be specified directly from the command line.
   |
   | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
   |       repositories, plugin repositories, and free-form properties to be used as configuration
   |       variables for plugins in the POM.
   |
   |-->
  <profiles>
    <!-- profile
     | Specifies a set of introductions to the build process, to be activated using one or more of the
     | mechanisms described above. For inheritance purposes, and to activate profiles via <activatedProfiles/>
     | or the command line, profiles have to have an ID that is unique.
     |
     | An encouraged best practice for profile identification is to use a consistent naming convention
     | for profiles, such as 'env-dev', 'env-test', 'env-production', 'user-jdcasey', 'user-brett', etc.
     | This will make it more intuitive to understand what the set of introduced profiles is attempting
     | to accomplish, particularly when you only have a list of profile id's for debug.
     |
     | This profile example uses the JDK version to trigger activation, and provides a JDK-specific repo.
    <profile>
      <id>jdk-1.4</id>

      <activation>
        <jdk>1.4</jdk>
      </activation>

      <repositories>
        <repository>
          <id>jdk14</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://www.myhost.com/maven/jdk14</url>
          <layout>default</layout>
          <snapshotPolicy>always</snapshotPolicy>
        </repository>
      </repositories>
    </profile>
    -->

    <!--
     | Here is another profile, activated by the system property 'target-env' with a value of 'dev',
     | which provides a specific path to the Tomcat instance. To use this, your plugin configuration
     | might hypothetically look like:
     |
     | ...
     | <plugin>
     |   <groupId>org.myco.myplugins</groupId>
     |   <artifactId>myplugin</artifactId>
     |
     |   <configuration>
     |     <tomcatLocation>${tomcatPath}</tomcatLocation>
     |   </configuration>
     | </plugin>
     | ...
     |
     | NOTE: If you just wanted to inject this configuration whenever someone set 'target-env' to
     |       anything, you could just leave off the <value/> inside the activation-property.
     |
    <profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>

  <!-- activeProfiles
   | List of profiles that are active for all builds.
   |
  <activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
  -->
</settings>
```

>修改第55行,改成与自己一样的位置信息


##idea如何设置文件头注释和方法注释

![image.png](https://upload-images.jianshu.io/upload_images/9049859-c06c1a389fb6aed0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

模板:
```
/**
 * Created with IntelliJ IDEA.
 * Description: 
 * User: ${USER}
 * Date: ${YEAR}-${MONTH}-${DAY}
 * Time: ${TIME}
 */
```



##hadoop api的使用(java版)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-9e0a57f1c9ba8c71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#####.创建client连接并封装
>创建一个配置对象
>创建一个代码分布式对象
不能直接new 一个
关于三个参数的说明:  URI  配置对象  用户名
>调用方法
>关流

```
public class Hdfs {
    static Configuration entries = null;
      //封装
     public  static   FileSystem  getFileSystem()  {
        //创建一个配置对象
        entries = new Configuration();
        //可以在这里设置储存副本的个数以及物理切片的大小
         entries.set("dfs.replication" ,"2");
         entries.set("dfs.blocksize","64M");
        //创建一个代码分布式对象
        URI uri = null;
        try {
            uri = new URI("hdfs://linux03:8020");
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
        FileSystem root = null;
        try {
            root = FileSystem.newInstance(uri, entries, "root");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return  root;

    }
    //关流
    public  static  void  closeFileSystem(FileSystem fileSystem)  {
        if (fileSystem!=null){
            try {
                fileSystem.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
##常用API
copyToLocalFile  从HDFS下载文件到本地

 copyToLocalFile(false , new Path("/a.txt") , new Path("f://"),true);
举例重载方法的说明:第三个参数:是否生成校验文件

copyFromLocalFile  上传到HDFS---重载方法  举例: 第一个参数是否删除本地文件  第二个参数是否覆盖HDFS某目录文件或者目录

testRename   修改名称或者移动文件

mkdirs          创建文件夹

delete         删除文件或者文件夹

读取数据代码如下:

```

public class Test_9_20 {
    public static void main(String[] args) throws IOException {
        FileSystem fileSystem = Hdfs.getFileSystem();
        //读取
        FSDataInputStream open = fileSystem.open(new Path("/1.txt"));
        //转化流  加强流
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(open));
        //记录长度
        long lo = 0;
        //怎么读
        String len = null;
       bufferedReader.skip(100);
        while ((len = bufferedReader.readLine()) != null) {
            lo+=len.length();
            System.out.println(len);
        }
        System.out.println(lo);
        //关流
        bufferedReader.close();
        open.close();
        Hdfs.closeFileSystem(fileSystem);
    }
}

```
**(注意skip的跳读作用  关流顺序)**

写出到本地 写出到HDFS


#####查询hdfs指定目录下的文件信息

```
  //只返回文件的信息  查询指定目录下的信息 填目录
        RemoteIterator<LocatedFileStatus> lo = fileSystem.listFiles(new Path("/aa"), false);
        //学习这个迭代器
        while (lo.hasNext()) {
            LocatedFileStatus next = lo.next();
            BlockLocation[] blockLocations = next.getBlockLocations();//块信息
            System.out.println(Arrays.toString(blockLocations) + "块信息");
            String group = next.getGroup();//组
            System.out.println( group +"组");
            long len = next.getLen();//文件长度
            System.out.println(len + "文件长度");
            Path path = next.getPath();//路径
            Path name = next.getPath().getName();//名字
            System.out.println( name + "名字");
            System.out.println( path +"路径");
            short replication = next.getReplication();//副本数量
            System.out.println( replication + "副本数量");
            long blockSize = next.getBlockSize();//块大小
            System.out.println(blockSize +"块大小");
        }
```

```
      LocatedFileStatus next = lo.next();
            String name = next.getPath().getName();
            BlockLocation[] blockLocations = next.getBlockLocations();//块信息
            for (BlockLocation blockLocation : blockLocations) {
                //偏移量
                long offset = blockLocation.getOffset();
                long length = blockLocation.getLength();
                System.out.println(name + "--" + offset +"--" + length);
                String[] hosts = blockLocation.getHosts();
                for (String host : hosts) {
                    System.out.println(host);
                }
```

![image.png](https://upload-images.jianshu.io/upload_images/9049859-3a778200f2558469.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-4cfab83ac237c7d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

listStatus  和  listFiles 区别
第一个 查询目录下的文件和文件夹信息
第二个 查询目录下的文件信息

对配置文件的修改    

>hdfs-site.xml

修改大小和节点数量

![image.png](https://upload-images.jianshu.io/upload_images/9049859-b02d91b6ec85a43d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
