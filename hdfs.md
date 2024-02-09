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

Canada 2021 Census:

https://www12.statcan.gc.ca/census-recensement/2021/dp-pd/prof/details/download-telecharger.cfm

```bash
wget --content-disposition "https://www12.statcan.gc.ca/census-recensement/2021/dp-pd/prof/details/download-telecharger/comp/GetFile.cfm?Lang=E&FILETYPE=CSV&GEONO=012"
unzip 98-401-X2021012_eng_CSV.zip
98-401-X2021012_English_CSV_data.csv
98-401-X2021012_English_meta.txt
98-401-X2021012_Geo_starting_row.CSV
```

```bash
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

```bash
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls /ONTARIO
Found 2 items
drwxr-xr-x   - ubuntu supergroup          0 2024-02-08 17:48 /ONTARIO/ada_copy
drwxr-xr-x   - ubuntu supergroup          0 2024-02-08 18:20 /ONTARIO/ada_linecount
ubuntu@LAPTOP-JBell:~$ hadoop fs -mkdir /ONTARIO/ada
ubuntu@LAPTOP-JBell:~$ hadoop fs -put 98-401-X2021012_English_CSV_data.csv /ONTARIO/ada
ubuntu@LAPTOP-JBell:~$ hadoop fs -ls /ONTARIO/ada
Found 1 items
-rw-r--r--   1 ubuntu supergroup 2415200889 2024-02-08 21:12 /ONTARIO/ada/98-401-X2021012_English_CSV_data.csv
ubuntu@LAPTOP-JBell:~$ hadoop fs -head /ONTARIO/ada/98-401-X2021012_English_CSV_data.csv
CENSUS_YEAR,DGUID,ALT_GEO_CODE,GEO_LEVEL,GEO_NAME,TNR_SF,TNR_LF,DATA_QUALITY_FLAG,CHARACTERISTIC_ID,CHARACTERISTIC_NAME,CHARACTERISTIC_NOTE,C1_COUNT_TOTAL,SYMBOL,C2_COUNT_MEN+,SYMBOL,C3_COUNT_WOMEN+,SYMBOL,C10_RATE_TOTAL,SYMBOL,C11_RATE_MEN+,SYMBOL,C12_RATE_WOMEN+,SYMBOL
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",1,"Population, 2021",1,8881,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",2,"Population, 2016",1,9334,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",3,"Population percentage change, 2016 to 2021",,-4.9,"",,"...",,"...",-4.9,"",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","10010001",3.1,4.6,"00000",4,"Total private dwellings",2,5737,"",,"...",,"...",,"...",,"...",,"..."
2021,"2021S051610010001","10010001","Aggregate dissemination area","100100
```





https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html




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
```bash
ubuntu@LAPTOP-JBell:~$ hadoop fs -rm /ONTARIO/ada/98-401-X2021012_English_CSV_data.csv
Deleted /ONTARIO/ada/98-401-X2021012_English_CSV_data.csv
ubuntu@LAPTOP-JBell:~$ hadoop fs -rmdir /ONTARIO/ada
ubuntu@LAPTOP-JBell:~$
```
