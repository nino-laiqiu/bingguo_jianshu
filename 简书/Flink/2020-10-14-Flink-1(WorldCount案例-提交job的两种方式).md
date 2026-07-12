##1.有状态的流式处理
![image.png](https://upload-images.jianshu.io/upload_images/9049859-dcbd88f4edba7deb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>  pom.xml的依赖(flink书籍上的依赖)
```
    <dependencies>
        <!-- Apache Flink dependencies -->
        <!-- These dependencies are provided, because they should not be packaged into the JAR file. -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- Scala Library, provided by Flink as well. -->
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- runtime-web dependency is need to start web UI from IDE -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-runtime-web_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- queryable-state dependencies are needed for respective examples -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-queryable-state-runtime_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-queryable-state-client-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!--
        Derby is used for a sink connector example.
        Example only works in local mode, i.e, it is not possible to submit it to a running cluster.
        The dependency is set to provided to reduce the size of the JAR file.
        -->
        <dependency>
            <groupId>org.apache.derby</groupId>
            <artifactId>derby</artifactId>
            <version>10.13.1.1</version>
            <scope>provided</scope>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- We use the maven-shade plugin to create a fat jar that contains all necessary dependencies. -->
            <!-- Change the value of <mainClass>...</mainClass> if your program entry point changes. -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <!-- Run shade goal on package phase -->
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <artifactSet>
                                <excludes>
                                    <exclude>org.apache.flink:force-shading</exclude>
                                    <exlude>org.apache.flink:flink-shaded-netty</exlude>
                                    <exlude>org.apache.flink:flink-shaded-guava</exlude>
                                    <exclude>com.google.code.findbugs:jsr305</exclude>
                                    <exclude>org.slf4j:*</exclude>
                                    <exclude>log4j:*</exclude>
                                </excludes>
                            </artifactSet>
                            <filters>
                                <filter>
                                    <!-- Do not copy the signatures in the META-INF folder.
                                    Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>io.github.streamingwithflink.StreamingJob</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- Java Compiler -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <!-- Scala Compiler -->
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- Eclipse Scala Integration -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-eclipse-plugin</artifactId>
                <version>2.8</version>
                <configuration>
                    <downloadSources>true</downloadSources>
                    <projectnatures>
                        <projectnature>org.scala-ide.sdt.core.scalanature</projectnature>
                        <projectnature>org.eclipse.jdt.core.javanature</projectnature>
                    </projectnatures>
                    <buildcommands>
                        <buildcommand>org.scala-ide.sdt.core.scalabuilder</buildcommand>
                    </buildcommands>
                    <classpathContainers>
                        <classpathContainer>org.scala-ide.sdt.launching.SCALA_CONTAINER</classpathContainer>
                        <classpathContainer>org.eclipse.jdt.launching.JRE_CONTAINER</classpathContainer>
                    </classpathContainers>
                    <excludes>
                        <exclude>org.scala-lang:scala-library</exclude>
                        <exclude>org.scala-lang:scala-compiler</exclude>
                    </excludes>
                    <sourceIncludes>
                        <sourceInclude>**/*.scala</sourceInclude>
                        <sourceInclude>**/*.java</sourceInclude>
                    </sourceIncludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>build-helper-maven-plugin</artifactId>
                <version>1.7</version>
                <executions>
                    <!-- Add src/main/scala to eclipse build path -->
                    <execution>
                        <id>add-source</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>add-source</goal>
                        </goals>
                        <configuration>
                            <sources>
                                <source>src/main/scala</source>
                            </sources>
                        </configuration>
                    </execution>
                    <!-- Add src/test/scala to eclipse build path -->
                    <execution>
                        <id>add-test-source</id>
                        <phase>generate-test-sources</phase>
                        <goals>
                            <goal>add-test-source</goal>
                        </goals>
                        <configuration>
                            <sources>
                                <source>src/test/scala</source>
                            </sources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <!-- This profile helps to make things run out of the box in IntelliJ -->
    <!-- Its adds Flink's core classes to the runtime class path. -->
    <!-- Otherwise they are missing in IntelliJ, because the dependency is 'provided' -->
    <profiles>
        <profile>
            <id>add-dependencies-for-IDEA</id>

            <activation>
                <property>
                    <name>idea.version</name>
                </property>
            </activation>

            <dependencies>
                <dependency>
                    <groupId>org.apache.flink</groupId>
                    <artifactId>flink-scala_${scala.binary.version}</artifactId>
                    <version>${flink.version}</version>
                    <scope>compile</scope>
                </dependency>
                <dependency>
                    <groupId>org.apache.flink</groupId>
                    <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
                    <version>${flink.version}</version>
                    <scope>compile</scope>
                </dependency>
                <dependency>
                    <groupId>org.scala-lang</groupId>
                    <artifactId>scala-library</artifactId>
                    <version>${scala.version}</version>
                    <scope>compile</scope>
                </dependency>
                <dependency>
                    <groupId>org.apache.flink</groupId>
                    <artifactId>flink-runtime-web_${scala.binary.version}</artifactId>
                    <version>${flink.version}</version>
                    <scope>compile</scope>
                </dependency>
                <dependency>
                    <groupId>org.apache.derby</groupId>
                    <artifactId>derby</artifactId>
                    <version>10.13.1.1</version>
                    <scope>compile</scope>
                </dependency>
            </dependencies>
        </profile>
    </profiles>
</project>
```
##2.批处理
```
import org.apache.flink.api.scala.ExecutionEnvironment
import org.apache.flink.api.scala._
//批处理
object WordCount {
  def main(args: Array[String]): Unit = {
    //创建执行环境
    var eve: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
    //读取的路径
    var path: String = "C:\\Users\\hp\\IdeaProjects\\Flink_Scala\\src\\main\\resources\\count.txt";
    var inputData: DataSet[String] = eve.readTextFile(path)
    var reseltData: DataSet[(String, Int)] = inputData
      .flatMap(_.split(" "))
      .map((_, 1))
      .groupBy(0)
      .sum(1)
    reseltData.print()

  }
}
```
##3.流处理
```
import org.apache.flink.api.java.utils.ParameterTool
import org.apache.flink.streaming.api.scala._

object WorldCount1 {
  def main(args: Array[String]): Unit = {
    //创建执行环境
    var eve: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    //设置线程
    // eve.setParallelism(8)
    //从外部命令获取host,port
    val tool = ParameterTool.fromArgs(args)
    var host = tool.get("host")
    var post = tool.getInt("post")
    var inputDataStream: DataStream[String] = eve.socketTextStream(host, post)
    var resultDataresult = inputDataStream
      //流映射
      .flatMap(_.split(" "))
      //元素映射
      .map((_, 1))
      .keyBy(0)
      .sum(1)
    resultDataresult.print()
    eve.execute()
  }
}
```

#####web提交
![image.png](https://upload-images.jianshu.io/upload_images/9049859-6f0a37bdea2809a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####本地提交

> flink run -c 主类( -p 2) 本地路径

>flink cancel jobID(提交之后)

##两个问题:
start-scala-shell.sh .......启动不了
scala jar包在集群上找不到类..........

