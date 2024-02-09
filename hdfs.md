https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html

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

```bash
HADOOP_CLASSPATH=/usr/lib/jvm/java-8-openjdk-amd64/lib/tools.jar
```

Ingest csv using Dataiku:

![image](https://github.com/jordanbell2357/how-to/assets/47544607/bd5b765f-c0b6-471f-b346-7beca77bfe4a)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/7b45ee6e-b65d-4fd6-bccb-c379b62a9019)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/c9a96f54-612b-4216-86ae-61b0d834c739)

https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html


```bash
ubuntu@LAPTOP-JBell:~/DATA_DIR$ hadoop fs -head /ONTARIO/ada_copy/out-s0.csv
2021    2021S051610010004       10010004        Aggregate dissemination area    10010004        2.4     2.7 00000    1964    Reformed                0.0             0.0             0.0             0.0             0.0 0.0
2021    2021S051610010004       10010004        Aggregate dissemination area    10010004        2.4     2.7 00000    1965    United Church           1075.0          530.0           545.0           10.8            11.010.6
2021    2021S051610010004       10010004        Aggregate dissemination area    10010004        2.4     2.7 00000    1966    Other Christian and Christian-related traditions                55.0            20.0        30.0             0.6             0.4             0.6
2021    2021S051610010004       10010004        Aggregate dissemination area    10010004        2.4     2.7 00000    1967    Hindu           205.0           95.0            110.0           2.1             2.0         2.1
2021    2021S051610010004       10010004        Aggregate dissemination area    10010004        2.4     2.7 00000    1968    Jewish          0.0             0.0             0.0             0.0             0.0         0.0
2021    2021S051610010004       10010004        Aggregate dissemination area    10010004        2.4     2.7 00000    1969    Muslim          230.0           115.0           110.0           2.3             2.4         2.1
2021    2021S051610010004       10010004        Aggregate dissemination area    10010004        2.4     2.7 00000    1970    Sikh            0.0             0.0             0.0             0.0             0.0         0.0
2021    2021S051610010004       10010004        Aggregate dissemination area    10
```

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



![image](https://github.com/jordanbell2357/how-to/assets/47544607/d894764b-b4c6-4f1c-b47c-6396944fbc27)



https://spark.apache.org/downloads.html

https://spark.apache.org/docs/latest/spark-standalone.html

https://spark.apache.org/docs/3.3.4/configuration.html

```bash
wget https://dlcdn.apache.org/spark/spark-3.3.4/spark-3.3.4-bin-hadoop3.tgz
tar -xvzf spark-3.3.4-bin-hadoop3.tgz
```

https://spark.apache.org/docs/latest/api/python/getting_started/install.html

```bash
pip install pyspark
```

```bash
export SPARK_HOME=/home/ubuntu/spark-3.3.4-bin-hadoop3
export PATH=$SPARK_HOME/bin:$PATH
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
```

https://kontext.tech/article/1066/install-spark-330-on-linux-or-wsl

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
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls /ONTARIO
Found 1 items
drwxr-xr-x   - ubuntu supergroup          0 2024-02-08 17:48 /ONTARIO/ada_copy
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls -R /ONTARIO
drwxr-xr-x   - ubuntu supergroup          0 2024-02-08 17:48 /ONTARIO/ada_copy
-rw-r--r--   1 ubuntu supergroup 2087945609 2024-02-08 17:50 /ONTARIO/ada_copy/out-s0.csv
```

```bash
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ hadoop jar lc.jar LineCount /ONTARIO/ada_copy/out-s0.csv /LineCount
2024-02-08 17:54:41,838 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at /0.0.0.0:8032
2024-02-08 17:54:42,213 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2024-02-08 17:54:42,227 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/ubuntu/.staging/job_1707010689166_0002
2024-02-08 17:54:42,419 INFO input.FileInputFormat: Total input files to process : 1
2024-02-08 17:54:43,288 INFO mapreduce.JobSubmitter: number of splits:16
2024-02-08 17:54:43,405 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1707010689166_0002
2024-02-08 17:54:43,405 INFO mapreduce.JobSubmitter: Executing with tokens: []
2024-02-08 17:54:43,593 INFO conf.Configuration: resource-types.xml not found
2024-02-08 17:54:43,594 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2024-02-08 17:54:44,004 INFO impl.YarnClientImpl: Submitted application application_1707010689166_0002
2024-02-08 17:54:44,047 INFO mapreduce.Job: The url to track the job: http://LAPTOP-JBell.:8088/proxy/application_1707010689166_0002/
2024-02-08 17:54:44,047 INFO mapreduce.Job: Running job: job_1707010689166_0002
2024-02-08 17:54:49,175 INFO mapreduce.Job: Job job_1707010689166_0002 running in uber mode : false
2024-02-08 17:54:49,176 INFO mapreduce.Job:  map 0% reduce 0%
2024-02-08 17:54:58,173 INFO mapreduce.Job:  map 38% reduce 0%
2024-02-08 17:55:05,307 INFO mapreduce.Job:  map 75% reduce 0%
2024-02-08 17:55:11,361 INFO mapreduce.Job:  map 81% reduce 0%
2024-02-08 17:55:12,368 INFO mapreduce.Job:  map 100% reduce 0%
2024-02-08 17:55:13,374 INFO mapreduce.Job:  map 100% reduce 100%
2024-02-08 17:55:14,386 INFO mapreduce.Job: Job job_1707010689166_0002 completed successfully
2024-02-08 17:55:14,464 INFO mapreduce.Job: Counters: 55
        File System Counters
                FILE: Number of bytes read=294
                FILE: Number of bytes written=4688923
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=2088008873
                HDFS: Number of bytes written=21
                HDFS: Number of read operations=53
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters
                Killed map tasks=1
                Launched map tasks=16
                Launched reduce tasks=1
                Data-local map tasks=16
                Total time spent by all maps in occupied slots (ms)=98159
                Total time spent by all reduces in occupied slots (ms)=6594
                Total time spent by all map tasks (ms)=98159
                Total time spent by all reduce tasks (ms)=6594
                Total vcore-milliseconds taken by all map tasks=98159
                Total vcore-milliseconds taken by all reduce tasks=6594
                Total megabyte-milliseconds taken by all map tasks=100514816
                Total megabyte-milliseconds taken by all reduce tasks=6752256
        Map-Reduce Framework
                Map input records=14294223
                Map output records=14294223
                Map output bytes=228707568
                Map output materialized bytes=384
                Input split bytes=1824
                Combine input records=14294223
                Combine output records=16
                Reduce input groups=1
                Reduce shuffle bytes=384
                Reduce input records=16
                Reduce output records=1
                Spilled Records=32
                Shuffled Maps =16
                Failed Shuffles=0
                Merged Map outputs=16
                GC time elapsed (ms)=15097
                CPU time spent (ms)=70770
                Physical memory (bytes) snapshot=7352893440
                Virtual memory (bytes) snapshot=44092829696
                Total committed heap usage (bytes)=7523008512
                Peak Map Physical memory (bytes)=489000960
                Peak Map Virtual memory (bytes)=2596409344
                Peak Reduce Physical memory (bytes)=263659520
                Peak Reduce Virtual memory (bytes)=2597912576
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=2088007049
        File Output Format Counters
                Bytes Written=21
```

```bash
spark-shell
scala> val df = spark.read.option("header", "true").csv("/ONTARIO/ada_copy/out-s0.csv")
df: org.apache.spark.sql.DataFrame = [2021      2021S051610010004       10010004        Aggregate dissemination area 10010004        2.4     2.7     00000   1964    Reformed                0.0             0.0         0.0              0.0             0.0             0.0     : string]

scala> val rowCount = df.count()
rowCount: Long = 14294222

scala> println(s"Number of rows: $rowCount")
Number of rows: 14294222

scala>
```

https://doc.dataiku.com/dss/latest/hadoop/spark.html

```bash
ubuntu@LAPTOP-JBell:~/DATA_DIR$ ./bin/dssadmin install-spark-integration
/home/ubuntu/dataiku-dss-11.3.2/scripts/_startup.inc.sh: line 230: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8): No such file or directory
[+] Saving installation log to /home/ubuntu/DATA_DIR/run/install.log
/bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
+ Using SPARK_HOME=/home/ubuntu/spark-3.3.4-bin-hadoop3
+ Found Spark version 3.3.4
+ Creating configuration file: /home/ubuntu/DATA_DIR/bin/env-spark.sh
+ Enabling DSS Spark support
/bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
/home/ubuntu/dataiku-dss-11.3.2/scripts/_startup.inc.sh: line 230: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
Loading plugin: census-us
Loading plugin: geoadmin
Loading plugin: geocoder
Loading plugin: georouter
Loading plugin: get-us-census-block
Loading plugin: kml-format
Loading plugin: sentiment-analysis
Loading plugin: timeseries-forecast-gpu-cuda100
Loading plugin: timeseries-preparation
[2024/02/08-18:06:41.586] [main] [INFO] [dku.logging]  - Loading logging settings
[2024/02/08-18:06:41.590] [main] [INFO] [dku.logging]  - Configuring additional logging settings from /home/ubuntu/dataiku-dss-11.3.2/resources/logging/dku-log4j.properties
[2024/02/08-18:06:41.604] [main] [INFO] [dku.logging]  - Configuring additional JUL logging settings from /home/ubuntu/dataiku-dss-11.3.2/resources/logging/dku-log-jul.properties
[2024/02/08-18:06:41.846] [main] [INFO] [dku.builtins]  - Loading countries geo data
[2024/02/08-18:06:42.025] [main] [INFO] [com.dataiku.dip.DKUApp]  - Reading /home/ubuntu/DATA_DIR/install.ini
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/ubuntu/dataiku-dss-11.3.2/lib/ivy/common-run/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/ubuntu/hadoop-3.3.6/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Reload4jLoggerFactory]
[2024/02/08-18:06:42.163] [main] [INFO] [dip.transactions]  - Transaction provider started (directory=/home/ubuntu/DATA_DIR/config, cache_compression=lz4, main_lock=false, temp_dir=/home/ubuntu/DATA_DIR/tmp/txn/large-files, fs_cache=v1, fs_diff=v2, git_support=true)
[2024/02/08-18:06:42.287] [main] [INFO] [dip.transactions]  - Transaction provider started (directory=/home/ubuntu/DATA_DIR/config, cache_compression=lz4, main_lock=false, temp_dir=/home/ubuntu/DATA_DIR/tmp/txn/large-files, fs_cache=v1, fs_diff=v2, git_support=true)
[2024/02/08-18:06:42.339] [main] [INFO] [x]  - **** Hadoop flavor **** {"flavor":"generic","hiveSupportsMREngine":true,"hive3":false}
[2024/02/08-18:06:42.364] [main] [INFO] [dip.commitbehavior]  - Candidate commit [GLOBAL] [no:auth] CLI: Enabled Spark (from command-line) :
 - /general-settings.json

[2024/02/08-18:06:42.448] [main] [INFO] [dip.transactions]  - Transaction committed [exec=62ms commitWait=0ms commitExec=83ms]
+ Installing toree/jupyter integration
+ Done
```

Dataiku

```scala
import com.dataiku.dss.spark._
import org.apache.spark.SparkContext
import org.apache.spark.sql.{SparkSession, Row}
import org.apache.spark.sql.types.{StructType, StructField, LongType}

val sparkConf    = DataikuSparkContext.buildSparkConf()
val sparkContext = SparkContext.getOrCreate(sparkConf)
val sparkSession = SparkSession.builder().config(sparkConf).getOrCreate()
val sqlContext   = sparkSession.sqlContext  // Get SQLContext from SparkSession
val dkuContext   = DataikuSparkContext.getContext(sparkContext)

// Recipe inputs
val ada_copy = dkuContext.getDataFrame(sqlContext, "ada_copy")

// Counting the number of rows in ada_copy
val rowCount = ada_copy.count()

// Creating a new DataFrame with the count
val schema = StructType(Array(StructField("row_count", LongType, false)))
val rowData = Seq(Row(rowCount))
val ada_linecount = sparkSession.createDataFrame(sparkContext.parallelize(rowData), schema)

// Recipe outputs
dkuContext.save("ada_linecount", ada_linecount)
```

![image](https://github.com/jordanbell2357/how-to/assets/47544607/5fd378a8-5457-41ba-bf01-8934fccc68b0)




![image](https://github.com/jordanbell2357/how-to/assets/47544607/caf9bd3a-146f-4acb-9e02-bdb394989797)

