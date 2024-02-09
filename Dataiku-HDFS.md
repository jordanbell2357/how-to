Ingest csv using Dataiku:

![image](https://github.com/jordanbell2357/how-to/assets/47544607/7b45ee6e-b65d-4fd6-bccb-c379b62a9019)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/bd5b765f-c0b6-471f-b346-7beca77bfe4a)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/c9a96f54-612b-4216-86ae-61b0d834c739)

Access directly loaded HDFS file using Dataiku:

![image](https://github.com/jordanbell2357/how-to/assets/47544607/403cfbcb-0ddd-4bc3-91c9-834b754bfbea)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/8ef1939c-3a74-4c95-8f24-37460ec953c2)

![image](https://github.com/jordanbell2357/how-to/assets/47544607/2b27723e-8bf9-4790-81e4-987c614d2a47)

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

View using Dataiku:

![image](https://github.com/jordanbell2357/how-to/assets/47544607/d894764b-b4c6-4f1c-b47c-6396944fbc27)



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

Calculate line count using Spark Scala in Dataiku:

![image](https://github.com/jordanbell2357/how-to/assets/47544607/5fd378a8-5457-41ba-bf01-8934fccc68b0)

Line count output in Dataiku:

![image](https://github.com/jordanbell2357/how-to/assets/47544607/caf9bd3a-146f-4acb-9e02-bdb394989797)
