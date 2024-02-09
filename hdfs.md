<https://ubuntu.com/server/docs/service-openssh>

https://kontext.tech/article/448/install-hadoop-330-on-linux

https://linuxconfig.org/ubuntu-20-04-hadoop

```bash
sudo apt-get install openssh-client openssh-server
sudo apt-get install ssh pdsh
```

https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html

```bash
sudo apt-get install default-jre openjdk-11-jre-headless openjdk-8-jre-headless openjdk-8-jdk
```

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html

![image](https://github.com/jordanbell2357/how-to/assets/47544607/c9a96f54-612b-4216-86ae-61b0d834c739)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/7659a8b7-2f68-45b8-b41d-547032987f5d)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/bb177d40-078a-4765-af7a-2756d951c2ab)

```bash
HADOOP_CLASSPATH=/usr/lib/jvm/java-8-openjdk-amd64/lib/tools.jar
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

```bash
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ hadoop jar lc.jar LineCount /ONTARIO/ada_copy/out-s0.csv /LineCount
2024-02-07 16:54:12,280 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at /0.0.0.0:8032
2024-02-07 16:54:13,093 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2024-02-07 16:54:13,097 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/ubuntu/.staging/job_1707010689166_0001
2024-02-07 16:54:13,373 INFO input.FileInputFormat: Total input files to process : 1
2024-02-07 16:54:13,841 INFO mapreduce.JobSubmitter: number of splits:16
2024-02-07 16:54:13,986 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1707010689166_0001
2024-02-07 16:54:13,986 INFO mapreduce.JobSubmitter: Executing with tokens: []
2024-02-07 16:54:14,122 INFO conf.Configuration: resource-types.xml not found
2024-02-07 16:54:14,122 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2024-02-07 16:54:14,931 INFO impl.YarnClientImpl: Submitted application application_1707010689166_0001
2024-02-07 16:54:14,966 INFO mapreduce.Job: The url to track the job: http://LAPTOP-JBell.:8088/proxy/application_1707010689166_0001/
2024-02-07 16:54:14,966 INFO mapreduce.Job: Running job: job_1707010689166_0001
2024-02-07 16:54:23,219 INFO mapreduce.Job: Job job_1707010689166_0001 running in uber mode : false
2024-02-07 16:54:23,220 INFO mapreduce.Job:  map 0% reduce 0%
2024-02-07 16:54:55,181 INFO mapreduce.Job:  map 38% reduce 0%
2024-02-07 16:55:03,390 INFO mapreduce.Job:  map 50% reduce 0%
2024-02-07 16:55:04,403 INFO mapreduce.Job:  map 75% reduce 0%
2024-02-07 16:55:08,453 INFO mapreduce.Job:  map 81% reduce 0%
2024-02-07 16:55:09,463 INFO mapreduce.Job:  map 94% reduce 0%
2024-02-07 16:55:10,469 INFO mapreduce.Job:  map 100% reduce 100%
2024-02-07 16:55:12,490 INFO mapreduce.Job: Job job_1707010689166_0001 completed successfully
2024-02-07 16:55:12,573 INFO mapreduce.Job: Counters: 55
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
                Total time spent by all maps in occupied slots (ms)=240133
                Total time spent by all reduces in occupied slots (ms)=6381
                Total time spent by all map tasks (ms)=240133
                Total time spent by all reduce tasks (ms)=6381
                Total vcore-milliseconds taken by all map tasks=240133
                Total vcore-milliseconds taken by all reduce tasks=6381
                Total megabyte-milliseconds taken by all map tasks=245896192
                Total megabyte-milliseconds taken by all reduce tasks=6534144
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
                GC time elapsed (ms)=99665
                CPU time spent (ms)=79840
                Physical memory (bytes) snapshot=7707107328
                Virtual memory (bytes) snapshot=44080840704
                Total committed heap usage (bytes)=8267497472
                Peak Map Physical memory (bytes)=488071168
                Peak Map Virtual memory (bytes)=2596761600
                Peak Reduce Physical memory (bytes)=258457600
                Peak Reduce Virtual memory (bytes)=2595168256
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
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ hadoop fs -ls /LineCount
Found 2 items
-rw-r--r--   1 ubuntu supergroup          0 2024-02-07 16:55 /LineCount/_SUCCESS
-rw-r--r--   1 ubuntu supergroup         21 2024-02-07 16:55 /LineCount/part-r-00000
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ hadoop fs -ls /LineCount/part-r-00000
-rw-r--r--   1 ubuntu supergroup         21 2024-02-07 16:55 /LineCount/part-r-00000
ubuntu@LAPTOP-JBell:~/hadoop-3.3.6/bin$ hadoop fs -cat /LineCount/part-r-00000
Total Lines     14294223
```

Note: it is unfruitful to suggest running hadoop fs -cat and piping to wc. This loses the thread of working in Hadoop.

![image](https://github.com/jordanbell2357/how-to/assets/47544607/d894764b-b4c6-4f1c-b47c-6396944fbc27)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/bd5b765f-c0b6-471f-b346-7beca77bfe4a)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/7b45ee6e-b65d-4fd6-bccb-c379b62a9019)


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
