https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html

https://learning.oreilly.com/library/view/apache-hadoop-3/9781788999830/05acc385-65dc-4355-8980-37b2c8933bb3.xhtml

# Install and configure dependencies

```bash
sudo apt-get install default-jre openjdk-11-jre-headless openjdk-8-jre-headless openjdk-8-jdk
```

```bash
sudo apt-get install openssh-client openssh-server
sudo apt-get install ssh pdsh
```

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

# Download and configure Hadoop

https://hadoop.apache.org/releases.html

```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xvzf hadoop-3.3.6.tar.gz
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/etc/hadoop$ ls
capacity-scheduler.xml            httpfs-env.sh               mapred-site.xml
configuration.xsl                 httpfs-log4j.properties     shellprofile.d
container-executor.cfg            httpfs-site.xml             ssl-client.xml.example
core-site.xml                     kms-acls.xml                ssl-server.xml.example
hadoop-env.cmd                    kms-env.sh                  user_ec_policies.xml.template
hadoop-env.sh                     kms-log4j.properties        workers
hadoop-metrics2.properties        kms-site.xml                yarn-env.cmd
hadoop-policy.xml                 log4j.properties            yarn-env.sh
hadoop-user-functions.sh.example  mapred-env.cmd              yarn-site.xml
hdfs-rbf-site.xml                 mapred-env.sh               yarnservice-log4j.properties
hdfs-site.xml                     mapred-queues.xml.template
```

`core-site.xml`

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

`hdfs-site.xml`

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

`mapred-site.xml`

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

`yarn-site.xml`

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export HADOOP_HOME=~/hadoop-3.3.6
export PATH=$PATH:$HADOOP_HOME/bin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_CLASSPATH=/usr/lib/jvm/java-8-openjdk-amd64/lib/tools.jar
```

```bash
hdfs namenode -format
```

```bash
start-all.sh
```

# CSV dataset

Canada 2021 Census:

https://www12.statcan.gc.ca/census-recensement/2021/dp-pd/prof/details/download-telecharger.cfm

```bash
wget --content-disposition "https://www12.statcan.gc.ca/census-recensement/2021/dp-pd/prof/details/download-telecharger/comp/GetFile.cfm?Lang=E&FILETYPE=CSV&GEONO=012"
ubuntu@LAPTOP-JBell:~$ unzip -v 98-401-X2021012_eng_CSV.zip
Archive:  98-401-X2021012_eng_CSV.zip
 Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
--------  ------  ------- ---- ---------- ----- --------  ----
2415200889  Defl:N 211587436  91% 2022-12-07 16:41 f32de755  98-401-X2021012_English_CSV_data.csv
  240208  Defl:N    44892  81% 2022-12-07 10:34 2eb21c80  98-401-X2021012_English_meta.txt
  218562  Defl:N    41420  81% 2022-12-07 10:35 e0bd4f6c  98-401-X2021012_Geo_starting_row.CSV
    1421  Defl:N      673  53% 2022-07-30 15:56 e7f8363b  README_meta.txt
--------          -------  ---                            -------
2415661080         211674421  91%                            4 files
ubuntu@LAPTOP-JBell:~$ unzip 98-401-X2021012_eng_CSV.zip
Archive:  98-401-X2021012_eng_CSV.zip
  inflating: 98-401-X2021012_English_CSV_data.csv
  inflating: 98-401-X2021012_English_meta.txt
  inflating: 98-401-X2021012_Geo_starting_row.CSV
  inflating: README_meta.txt
ubuntu@LAPTOP-JBell:~$ head 98-401-X2021012_English_CSV_data.csv
CENSUS_YEAR,DGUID,ALT_GEO_CODE,GEO_LEVEL,GEO_NAME,TNR_SF,TNR_LF,DATA_QUALITY_FLAG,CHARACTERISTIC_ID,CHARACTERISTIC_NAME,CHARACTERISTIC_NOTE,C1_COUNT_TOTAL,SYMBOL,C2_COUNT_MEN+,SYMBOL,C3_COUNT_WOMEN+,SYMBOL,C10_RATE_TOTAL,SYMBOL,C11_RATE_MEN+,SYMBOL,C12_RATE_WOMEN+,SYMBOL
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",1,"Population, 2021",1,8881,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",2,"Population, 2016",1,9334,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",3,"Population percentage change, 2016 to 2021",,-4.9,"",,"...",,"...",-4.9,"",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",4,"Total private dwellings",2,5737,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",5,"Private dwellings occupied by usual residents",3,4121,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",6,"Population density per square kilometre",,9.4,"",,"...",,"...",9.4,"",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",7,"Land area in square kilometres",,941.33,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",8,"Total - Age groups of the population - 100% data",,8880,"",4365,"",4515,"",100,"",100,"",100,""
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",9,"  0 to 14 years",,925,"",460,"",465,"",10.4,"",10.5,"",10.3,""
```

# Putting CSV into HDFS

https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html

```bash
ubuntu@LAPTOP-JBell:~$ hadoop fs -mkdir /census2021
ubuntu@LAPTOP-JBell:~$ hadoop fs -mkdir /census2021/ada
ubuntu@LAPTOP-JBell:~$ hadoop fs -put ./98-401-X2021012_English_CSV_data.csv /census2021/ada/ada.csv
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls -R /census2021
drwxr-xr-x   - ubuntu supergroup          0 2024-02-09 09:50 /census2021/ada
-rw-r--r--   1 ubuntu supergroup 2415200889 2024-02-09 09:50 /census2021/ada/ada.csv
ubuntu@LAPTOP-JBell:~$ hadoop fs -head /census2021/ada/ada.csv
CENSUS_YEAR,DGUID,ALT_GEO_CODE,GEO_LEVEL,GEO_NAME,TNR_SF,TNR_LF,DATA_QUALITY_FLAG,CHARACTERISTIC_ID,CHARACTERISTIC_NAME,CHARACTERISTIC_NOTE,C1_COUNT_TOTAL,SYMBOL,C2_COUNT_MEN+,SYMBOL,C3_COUNT_WOMEN+,SYMBOL,C10_RATE_TOTAL,SYMBOL,C11_RATE_MEN+,SYMBOL,C12_RATE_WOMEN+,SYMBOL
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",1,"Population, 2021",1,8881,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",2,"Population, 2016",1,9334,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",3,"Population percentage change, 2016 to 2021",,-4.9,"",,"...",,"...",-4.9,"",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",4,"Total private dwellings",2,5737,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","100100
ubuntu@LAPTOP-JBell:~$ hadoop fs -cat /census2021/ada/ada.csv | wc -l
14294224
```

(We remind ourselves that `hadoop fs -head` displays the first kilobyte of a file, not a fixed number of lines.)

# MapReduce code for line count

https://statinfer.com/301-2-3-map-reduce-code-for-line-count/
https://stackoverflow.com/questions/29260900/could-not-find-or-load-main-class-com-sun-tools-javac-main-hadoop-mapreduce

```bash
cat LineCount.java
```

```java
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class LineCount{
    public static class LineCntMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    Text keyEmit = new Text("Total Lines");
    private final static IntWritable one = new IntWritable(1);

    public void map(LongWritable key, Text value, Context context){
        try {
            context.write(keyEmit, one);
        }
        catch (IOException e) {
            e.printStackTrace();
            System.exit(0);
        }
        catch (InterruptedException e) {
            e.printStackTrace();
            System.exit(0);
        }
    }
}

    public static class LineCntReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    public void reduce(Text key, Iterable<IntWritable> values, Context context){
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        try {
            context.write(key, new IntWritable(sum));
        }
        catch (IOException e) {
            e.printStackTrace();
            System.exit(0);
        }
        catch (InterruptedException e) {
            e.printStackTrace();
            System.exit(0);
        }
    }
}

    public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "line count2");
    job.setJarByClass(LineCount.class);
    job.setMapperClass(LineCntMapper.class);
    job.setCombinerClass(LineCntReducer.class);
    job.setReducerClass(LineCntReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

```bash
ubuntu@LAPTOP-JBell:~$ cd hadoop-3.3.6/bin
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ ls
LineCount.java      hadoop      hdfs      mapred      oom-listener             yarn
container-executor  hadoop.cmd  hdfs.cmd  mapred.cmd  test-container-executor  yarn.cmd
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ hadoop com.sun.tools.javac.Main LineCount.java
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ jar cf lc.jar LineCount*.class
```

```bash
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ hadoop jar lc.jar LineCount /census2021/ada/ada.csv /LineCount
2024-02-09 09:53:23,449 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at /0.0.0.0:8032
2024-02-09 09:53:23,793 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2024-02-09 09:53:23,804 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/ubuntu/.staging/job_1707010689166_0003
2024-02-09 09:53:23,975 INFO input.FileInputFormat: Total input files to process : 1
2024-02-09 09:53:24,022 INFO mapreduce.JobSubmitter: number of splits:18
2024-02-09 09:53:24,127 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1707010689166_0003
2024-02-09 09:53:24,127 INFO mapreduce.JobSubmitter: Executing with tokens: []
2024-02-09 09:53:24,248 INFO conf.Configuration: resource-types.xml not found
2024-02-09 09:53:24,248 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2024-02-09 09:53:24,593 INFO impl.YarnClientImpl: Submitted application application_1707010689166_0003
2024-02-09 09:53:24,625 INFO mapreduce.Job: The url to track the job: http://LAPTOP-JBell.:8088/proxy/application_1707010689166_0003/
2024-02-09 09:53:24,626 INFO mapreduce.Job: Running job: job_1707010689166_0003
2024-02-09 09:53:29,730 INFO mapreduce.Job: Job job_1707010689166_0003 running in uber mode : false
2024-02-09 09:53:29,731 INFO mapreduce.Job:  map 0% reduce 0%
2024-02-09 09:54:17,390 INFO mapreduce.Job:  map 33% reduce 0%
2024-02-09 09:54:24,509 INFO mapreduce.Job:  map 67% reduce 0%
2024-02-09 09:54:31,629 INFO mapreduce.Job:  map 94% reduce 0%
2024-02-09 09:54:36,652 INFO mapreduce.Job:  map 100% reduce 100%
2024-02-09 09:54:37,663 INFO mapreduce.Job: Job job_1707010689166_0003 completed successfully
2024-02-09 09:54:37,776 INFO mapreduce.Job: Counters: 54
        File System Counters
                FILE: Number of bytes read=330
                FILE: Number of bytes written=5240478
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=2415272483
                HDFS: Number of bytes written=21
                HDFS: Number of read operations=59
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters
                Launched map tasks=18
                Launched reduce tasks=1
                Data-local map tasks=18
                Total time spent by all maps in occupied slots (ms)=346566
                Total time spent by all reduces in occupied slots (ms)=11939
                Total time spent by all map tasks (ms)=346566
                Total time spent by all reduce tasks (ms)=11939
                Total vcore-milliseconds taken by all map tasks=346566
                Total vcore-milliseconds taken by all reduce tasks=11939
                Total megabyte-milliseconds taken by all map tasks=354883584
                Total megabyte-milliseconds taken by all reduce tasks=12225536
        Map-Reduce Framework
                Map input records=14294224
                Map output records=14294224
                Map output bytes=228707584
                Map output materialized bytes=432
                Input split bytes=1962
                Combine input records=14294224
                Combine output records=18
                Reduce input groups=1
                Reduce shuffle bytes=432
                Reduce input records=18
                Reduce output records=1
                Spilled Records=36
                Shuffled Maps =18
                Failed Shuffles=0
                Merged Map outputs=18
                GC time elapsed (ms)=50184
                CPU time spent (ms)=88410
                Physical memory (bytes) snapshot=8216698880
                Virtual memory (bytes) snapshot=49281294336
                Total committed heap usage (bytes)=8874622976
                Peak Map Physical memory (bytes)=475123712
                Peak Map Virtual memory (bytes)=2596765696
                Peak Reduce Physical memory (bytes)=242405376
                Peak Reduce Virtual memory (bytes)=2599333888
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=2415270521
        File Output Format Counters
                Bytes Written=21
```

```bash
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls -R /LineCount
-rw-r--r--   1 ubuntu supergroup          0 2024-02-09 09:54 /LineCount/_SUCCESS
-rw-r--r--   1 ubuntu supergroup         21 2024-02-09 09:54 /LineCount/part-r-00000
ubuntu@LAPTOP-JBell:~$ hadoop fs -cat /LineCount/part-r-00000
Total Lines     14294224
```

# Spark

https://spark.apache.org/downloads.html

https://spark.apache.org/docs/latest/spark-standalone.html

https://spark.apache.org/docs/3.3.4/configuration.html

```bash
wget https://dlcdn.apache.org/spark/spark-3.3.4/spark-3.3.4-bin-hadoop3.tgz
tar -xvzf spark-3.3.4-bin-hadoop3.tgz
```

```bash
export SPARK_HOME=/home/ubuntu/spark-3.3.4-bin-hadoop3
export PATH=$SPARK_HOME/bin:$PATH
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
```

```bash
cp $SPARK_HOME/conf/spark-defaults.conf.template $SPARK_HOME/conf/spark-defaults.conf
```

```bash
spark.driver.host	localhost
```

```bash
run-example SparkPi 10
```

```bash
24/02/08 17:42:53 INFO DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 0.770038 s
Pi is roughly 3.1433631433631435
24/02/08 17:42:53 INFO SparkUI: Stopped Spark web UI at http://localhost:4040
```

```bash
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ spark-shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/ubuntu/spark-3.3.4-bin-hadoop3/jars/log4j-slf4j-impl-2.17.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/ubuntu/hadoop-3.3.6/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
24/02/09 09:58:14 WARN Utils: Your hostname, LAPTOP-JBell resolves to a loopback address: 127.0.1.1; using 172.17.194.43 instead (on interface eth0)
24/02/09 09:58:14 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
24/02/09 09:58:19 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Spark context Web UI available at http://localhost:4040
Spark context available as 'sc' (master = local[*], app id = local-1707490699967).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.3.4
      /_/

Using Scala version 2.12.15 (OpenJDK 64-Bit Server VM, Java 1.8.0_392)
Type in expressions to have them evaluated.
Type :help for more information.

scala> val df = spark.read.option("header", "true").csv("/census2021/ada/ada.csv")
df: org.apache.spark.sql.DataFrame = [CENSUS_YEAR: string, DGUID: string ... 21 more fields]

scala> val rowCount = df.count()
rowCount: Long = 14294223

scala> df.columns.foreach(println)
CENSUS_YEAR
DGUID
ALT_GEO_CODE
GEO_LEVEL
GEO_NAME
TNR_SF
TNR_LF
DATA_QUALITY_FLAG
CHARACTERISTIC_ID
CHARACTERISTIC_NAME
CHARACTERISTIC_NOTE
C1_COUNT_TOTAL
SYMBOL12
C2_COUNT_MEN+
SYMBOL14
C3_COUNT_WOMEN+
SYMBOL16
C10_RATE_TOTAL
SYMBOL18
C11_RATE_MEN+
SYMBOL20
C12_RATE_WOMEN+
SYMBOL22

scala> val df = spark.read.option("header", "true").csv("/census2021/ada/ada.csv")
df: org.apache.spark.sql.DataFrame = [CENSUS_YEAR: string, DGUID: string ... 21 more fields]

scala> val dfRenamed = df.withColumnRenamed("C2_COUNT_MEN+", "C2_COUNT_MEN").withColumnRenamed("C3_COUNT_WOMEN+", "C3_COUNT_WOMEN").withColumnRenamed("C11_RATE_MEN+", "C11_RATE_MEN").withColumnRenamed("C12_RATE_WOMEN+", "C12_RATE_WOMEN")
dfRenamed: org.apache.spark.sql.DataFrame = [CENSUS_YEAR: string, DGUID: string ... 21 more fields]

scala> dfRenamed.columns.foreach(println)
CENSUS_YEAR
DGUID
ALT_GEO_CODE
GEO_LEVEL
GEO_NAME
TNR_SF
TNR_LF
DATA_QUALITY_FLAG
CHARACTERISTIC_ID
CHARACTERISTIC_NAME
CHARACTERISTIC_NOTE
C1_COUNT_TOTAL
SYMBOL12
C2_COUNT_MEN
SYMBOL14
C3_COUNT_WOMEN
SYMBOL16
C10_RATE_TOTAL
SYMBOL18
C11_RATE_MEN
SYMBOL20
C12_RATE_WOMEN
SYMBOL22

scala> val df = spark.read.option("header", "true").csv("/census2021/ada/ada.csv")
df: org.apache.spark.sql.DataFrame = [CENSUS_YEAR: string, DGUID: string ... 21 more fields]

scala> val dfRenamed = df.withColumnRenamed("C2_COUNT_MEN+", "C2_COUNT_MEN").withColumnRenamed("C3_COUNT_WOMEN+", "C3_COUNT_WOMEN").withColumnRenamed("C11_RATE_MEN+", "C11_RATE_MEN").withColumnRenamed("C12_RATE_WOMEN+", "C12_RATE_WOMEN")
dfRenamed: org.apache.spark.sql.DataFrame = [CENSUS_YEAR: string, DGUID: string ... 21 more fields]

scala> val df_reduced = dfRenamed.select("GEO_NAME", "CHARACTERISTIC_ID", "CHARACTERISTIC_NAME", "C1_COUNT_TOTAL", "C2_COUNT_MEN", "C3_COUNT_WOMEN", "C10_RATE_TOTAL", "C11_RATE_MEN", "C12_RATE_WOMEN")
df_reduced: org.apache.spark.sql.DataFrame = [GEO_NAME: string, CHARACTERISTIC_ID: string ... 7 more fields]

scala> df_reduced.coalesce(1).write.option("header", "true").csv("/census2021/ada_reduced")
[Stage 1:>                                                          (0 + 1) / 1]
```

```bash
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls -R /census2021
drwxr-xr-x   - ubuntu supergroup          0 2024-02-09 10:29 /census2021/ada
-rw-r--r--   1 ubuntu supergroup 2415200889 2024-02-09 09:50 /census2021/ada/ada.csv
drwxr-xr-x   - ubuntu supergroup          0 2024-02-09 10:32 /census2021/ada_reduced
-rw-r--r--   1 ubuntu supergroup          0 2024-02-09 10:32 /census2021/ada_reduced/_SUCCESS
-rw-r--r--   1 ubuntu supergroup  763087510 2024-02-09 10:32 /census2021/ada_reduced/part-00000-f87a37db-597e-42df-bd05-bab7766f0c41-c000.csv
ubuntu@LAPTOP-JBell:~$ hadoop fs -head /census2021/ada_reduced/part-00000-f87a37db-597e-42df-bd05-bab7766f0c41-c000.csv
GEO_NAME,CHARACTERISTIC_ID,CHARACTERISTIC_NAME,C1_COUNT_TOTAL,C2_COUNT_MEN,C3_COUNT_WOMEN,C10_RATE_TOTAL,C11_RATE_MEN,C12_RATE_WOMEN
10010001,1,"Population, 2021",8881,,,,,
10010001,2,"Population, 2016",9334,,,,,
10010001,3,"Population percentage change, 2016 to 2021",-4.9,,,-4.9,,
10010001,4,Total private dwellings,5737,,,,,
10010001,5,Private dwellings occupied by usual residents,4121,,,,,
10010001,6,Population density per square kilometre,9.4,,,9.4,,
10010001,7,Land area in square kilometres,941.33,,,,,
10010001,8,Total - Age groups of the population - 100% data,8880,4365,4515,100,100,100
10010001,9,0 to 14 years,925,460,465,10.4,10.5,10.3
10010001,10,0 to 4 years,265,130,135,3,3,3
10010001,11,5 to 9 years,295,135,150,3.3,3.1,3.3
10010001,12,10 to 14 years,370,190,180,4.2,4.4,4
10010001,13,15 to 64 years,5010,2500,2510,56.4,57.3,55.6
10010001,14,15 to 19 years,370,185,180,4.2,4.2,4
10010001,15,20 to 24 years,280,155,120,3.2,3.6,2.7
10010001,16,25 to 29 years,250,130,120,2.8,3,2.7
10010001,17,30 to 34 years
```

(We remind ourselves that `hadoop fs -head` displays the first kilobyte of a file, not a fixed number of lines.)

```bash
scala> val df = spark.read.option("header", "true").csv("/census2021/ada_reduced/part-00000-f87a37db-597e-42df-bd05-bab7766f0c41-c000.csv")
df: org.apache.spark.sql.DataFrame = [GEO_NAME: string, CHARACTERISTIC_ID: string ... 7 more fields]

scala> val df_id = df.select($"CHARACTERISTIC_ID".cast("integer").alias("CHARACTERISTIC_ID"), $"CHARACTERISTIC_NAME").distinct().orderBy("CHARACTERISTIC_ID")
df_id: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [CHARACTERISTIC_ID: int, CHARACTERISTIC_NAME: string]

scala> df_id.coalesce(1).write.option("header", "true").csv("/census2021/ada_characteristic")
```

```bash
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls -R /census2021/ada_characteristic
-rw-r--r--   1 ubuntu supergroup          0 2024-02-09 13:31 /census2021/ada_characteristic/_SUCCESS
-rw-r--r--   1 ubuntu supergroup      78032 2024-02-09 13:31 /census2021/ada_characteristic/part-00000-3912cf78-b4fe-4986-9e70-9a62cff91ab6-c000.csv
ubuntu@LAPTOP-JBell:~$ hadoop fs -head /census2021/ada_characteristic/part-00000-3912cf78-b4fe-4986-9e70-9a62cff91ab6-c000.csv
CHARACTERISTIC_ID,CHARACTERISTIC_NAME
1,"Population, 2021"
2,"Population, 2016"
3,"Population percentage change, 2016 to 2021"
4,Total private dwellings
5,Private dwellings occupied by usual residents
6,Population density per square kilometre
7,Land area in square kilometres
8,Total - Age groups of the population - 100% data
9,0 to 14 years
10,0 to 4 years
11,5 to 9 years
12,10 to 14 years
13,15 to 64 years
14,15 to 19 years
15,20 to 24 years
16,25 to 29 years
17,30 to 34 years
18,35 to 39 years
19,40 to 44 years
20,45 to 49 years
21,50 to 54 years
22,55 to 59 years
23,60 to 64 years
24,65 years and over
25,65 to 69 years
26,70 to 74 years
27,75 to 79 years
28,80 to 84 years
29,85 years and over
30,85 to 89 years
31,90 to 94 years
32,95 to 99 years
33,100 years and over
34,Total - Distribution (%) of the population by broad age groups - 100% data
35,0 to 14 years
36,15 to 64 years
37,65 years and over
38,85 years and over
39,Average age of the population
40,Median age of the population
41,Total - Occupied pri
```
