---
title: A Lap Around Apache Spark
tutorial-id: 390
platform: hdp-2.6.0
tags: [spark]
---

# A Lap Around Apache Spark

## Introduction

This tutorial will get you started with Apache Spark and will cover:

- How to run Spark on YARN with SparkPi and WordCount examples
- How to use the Spark DataFrame & Dataset API
- How to use SparkSQL Thrift Server for JDBC/ODBC access
- How to use SparkR

You can either use Spark 1.6.x or Spark 2.x versions to run all of the examples.

Interacting with Spark will be done via the terminal (i.e. command line).

## Prerequisites

This tutorial assumes that you are running an HDP Sandbox.

Please ensure you complete the prerequisites before proceeding with this tutorial.

-   Downloaded and Installed the [Hortonworks Sandbox](https://hortonworks.com/products/sandbox/)
-   Reviewed [Learning the Ropes of the Hortonworks Sandbox](https://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)

## Outline

-   [Pi Example](#pi-example)
-   [WordCount Example](#wordcount-example)
-   [DataFrame API Example](#dataframe-api-example)
-   [DataSet API Example](#dataset-api-example)
-   [SparkSQL Thrift Server Example](#sparksql-thrift-server-example)
-   [SparkR Example](#sparkr-example)

-   [Further Reading](#further-reading)


## Pi Example

To test compute intensive tasks in Spark, the Pi example calculates pi by “throwing darts” at a circle. The example points in the unit square ((0,0) to (1,1)) and sees how many fall in the unit circle. The fraction should be pi/4, which is used to estimate Pi.

To calculate Pi with Spark in yarn-client mode.

Assuming you start as `root` user follow these steps depending on your Spark version.

**Spark 2.x**

~~~ bash
root@sandbox# export SPARK_MAJOR_VERSION=2
root@sandbox# cd /usr/hdp/current/spark2-client
root@sandbox spark2-client# su spark
spark@sandbox spark2-client$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn-client --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 examples/jars/spark-examples*.jar 10
~~~

**Spark 1.6.x**

~~~ bash
root@sandbox# export SPARK_MAJOR_VERSION=1
root@sandbox# cd /usr/hdp/current/spark-client
root@sandbox spark-client# su spark
spark@sandbox spark-client$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn-client --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 lib/spark-examples*.jar 10
~~~

**Note:** The Pi job should complete without any failure messages and produce output similar to below, note the value of Pi in the output message:

~~~ js
...
16/02/25 21:27:11 INFO YarnScheduler: Removed TaskSet 0.0, whose tasks have all completed, from pool
16/02/25 21:27:11 INFO DAGScheduler: Job 0 finished: reduce at SparkPi.scala:36, took 19.346544 s
Pi is roughly 3.143648
16/02/25 21:27:12 INFO ContextHandler: stopped o.s.j.s.ServletContextHandler{/metrics/json,null}
...
~~~


## WordCount Example

**Copy input file for Spark WordCount Example**

Upload the input file you want to use in WordCount to HDFS. You can use any text file as input. In the following example, log4j.properties is used as an example:

As user `spark`:

~~~ bash
cd /usr/hdp/current/spark2-client/
su spark
hdfs dfs -copyFromLocal /etc/hadoop/conf/log4j.properties /tmp/data.txt
~~~

#### Spark 2.x Version

Here's an example of a WordCount in Spark 2.x. (For Spark 1.6.x scroll down.)

**Run the Spark shell:**

~~~ bash
./bin/spark-shell 
~~~

Output similar to the following will be displayed, followed by a `scala>` REPL prompt:

~~~ bash
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.1.0.2.6.0.3-8
      /_/

Using Scala version 2.11.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_121)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
~~~

Read data and convert to Dataset

~~~ js
val data = spark.read.textFile("/tmp/data.txt").as[String]
~~~

Split and group by lowercased word

~~~ js
val words = data.flatMap(value => value.split("\\s+"))
val groupedWords = words.groupByKey(_.toLowerCase)
~~~

Count words

~~~ js
val counts = groupedWords.count()
~~~

Show results

~~~ js
counts.show()
~~~

You should see the following output

~~~ bash
+--------------------+--------+
|               value|count(1)|
+--------------------+--------+
|                some|       1|
|hadoop.security.l...|       1|
|log4j.rootlogger=...|       1|
|log4j.appender.nn...|       1|
|log4j.appender.tl...|       1|
|hadoop.security.l...|       1|
|            license,|       1|
|                 two|       1|
|             counter|       1|
|log4j.appender.dr...|       1|
|hdfs.audit.logger...|       1|
|yarn.ewma.maxuniq...|       1|
|log4j.appender.nm...|       1|
|              daemon|       1|
|log4j.category.se...|       1|
|log4j.appender.js...|       1|
|log4j.appender.dr...|       1|
|        blockmanager|       1|
|log4j.appender.js...|       1|
|                 set|       4|
+--------------------+--------+
only showing top 20 rows
~~~

#### Spark 1.6.x Version

Here's how to run a WordCount example in Spark 1.6.x

**Run the Spark shell:**

As user `spark`:

~~~ bash
cd /usr/hdp/current/spark-client/
./bin/spark-shell 
~~~

Output similar to the following will be displayed, followed by a `scala>` REPL prompt:

~~~ bash
Welcome to
     ____              __
    / __/__  ___ _____/ /__
   _\ \/ _ \/ _ `/ __/  '_/
  /___/ .__/\_,_/_/ /_/\_\   version 1.6.2
     /_/
Using Scala version 2.10.5 (OpenJDK 64-Bit Server VM, Java 1.7.0_101)
Type in expressions to have them evaluated.
Type :help for more information.
15/12/16 13:21:57 INFO SparkContext: Running Spark version 1.6.2
...

scala>
~~~

At the `scala` REPL prompt enter:

~~~ js
val file = sc.textFile("/tmp/data.txt")
val counts = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
~~~

Save `counts` to a file:

~~~ js
counts.saveAsTextFile("/tmp/wordcount")
~~~

**Viewing the WordCount output with Scala Shell**


To view the output, at the `scala>` prompt type:

~~~ js
counts.count()
~~~

You should see an output screen similar to:

~~~ js
...
16/02/25 23:12:20 INFO DAGScheduler: Job 1 finished: count at <console>:32, took 0.541229 s
res1: Long = 341

scala>
~~~

To print the full output of the WordCount job type:

~~~ js
counts.toArray().foreach(println)
~~~

You should see an output screen similar to:

~~~ js
...
((Hadoop,1)
(compliance,1)
(log4j.appender.RFAS.layout.ConversionPattern=%d{ISO8601},1)
(additional,1)
(default,2)

scala>
~~~

**Viewing the WordCount output with HDFS**

To read the output of WordCount using HDFS command:
Exit the Scala shell.

~~~ bash
exit
~~~

View WordCount Results:

~~~ bash
hdfs dfs -ls /tmp/wordcount
~~~

You should see an output similar to:

~~~ bash
/tmp/wordcount/_SUCCESS
/tmp/wordcount/part-00000
/tmp/wordcount/part-00001
~~~

Use the HDFS `cat` command to see the WordCount output. For example,

~~~ bash
hdfs dfs -cat /tmp/wordcount/part-00000
~~~


## DataFrame API Example

#### Spark 1.6.x Version

 DataFrame API provides easier access to data since it looks conceptually like a Table and a lot of developers from Python/R/Pandas are familiar with it.

As a `spark` user, upload people.txt and people.json files to HDFS:

~~~ bash
cd /usr/hdp/current/spark-client
su spark
hdfs dfs -copyFromLocal examples/src/main/resources/people.txt /tmp/people.txt
hdfs dfs -copyFromLocal examples/src/main/resources/people.json /tmp/people.json
~~~

Launch the Spark Shell:

As `spark` user:

~~~ bash
./bin/spark-shell
~~~

At a `scala>` REPL prompt, type the following:

~~~ js
val df = sqlContext.jsonFile("/tmp/people.json")
~~~

Using `df.show`, display the contents of the DataFrame:

~~~ js
df.show
~~~

You should see an output similar to:

~~~ js
...
15/12/16 13:28:15 INFO YarnScheduler: Removed TaskSet 2.0, whose tasks have all completed, from pool
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+

scala>
~~~

**Additional DataFrame API examples**

Continuing at the `scala>` prompt, type the following to import sql functions:

~~~ js
import org.apache.spark.sql.functions._ 
~~~

and select "name" and "age" columns and increment the "age" column by 1:

~~~ js
df.select(df("name"), df("age") + 1).show()
~~~

This will produce an output similar to the following:

~~~ bash
...
+-------+---------+
|   name|(age + 1)|
+-------+---------+
|Michael|     null|
|   Andy|       31|
| Justin|       20|
+-------+---------+

scala>
~~~

To return people older than 21, use the filter() function:

~~~ js
df.filter(df("age") > 21).show()
~~~

This will produce an output similar to the following:

~~~ bash
...
+---+----+
|age|name|
+---+----+
| 30|Andy|
+---+----+

scala>
~~~

Next, to count the number of people of specific age, use groupBy() & count() functions:

~~~ js
df.groupBy("age").count().show()
~~~

This will produce an output similar to the following:

~~~ bash
...
+----+-----+
| age|count|
+----+-----+
|null|    1|
|  19|    1|
|  30|    1|
+----+-----+

scala>
~~~

**Programmatically Specifying Schema**

~~~ js
import org.apache.spark.sql._ 

// Create sql context from an existing SparkContext (sc)
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

// Create people RDD
val people = sc.textFile("/tmp/people.txt")

// Encode schema in a string
val schemaString = "name age"

// Import Spark SQL data types and Row
import org.apache.spark.sql.types.{StructType,StructField,StringType} 

// Generate the schema based on the string of schema
val schema = 
  StructType(
    schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, true)))

// Convert records of people RDD to Rows
val rowRDD = people.map(_.split(",")).map(p => Row(p(0), p(1).trim))

// Apply the schema to the RDD
val peopleSchemaRDD = sqlContext.createDataFrame(rowRDD, schema)

// Register the SchemaRDD as a table
peopleSchemaRDD.registerTempTable("people")

// Execute a SQL statement on the 'people' table
val results = sqlContext.sql("SELECT name FROM people")

// The results of SQL queries are SchemaRDDs and support all the normal RDD operations.
// The columns of a row in the result can be accessed by ordinal
results.map(t => "Name: " + t(0)).collect().foreach(println)
~~~

This will produce an output similar to the following:

~~~ js
15/12/16 13:29:19 INFO DAGScheduler: Job 9 finished: collect at :39, took 0.251161 s
15/12/16 13:29:19 INFO YarnHistoryService: About to POST entity application_1450213405513_0012 with 10 events to timeline service http://green3:8188/ws/v1/timeline/
Name: Michael
Name: Andy
Name: Justin

scala>
~~~


## DataSet API Example

If you haven't done so already in previous sections, make sure to upload people data sets (people.txt and people.json) to HDFS:

~~~ bash
cd /usr/hdp/current/spark-client
su spark
hdfs dfs -copyFromLocal examples/src/main/resources/people.txt /tmp/people.txt
hdfs dfs -copyFromLocal examples/src/main/resources/people.json /tmp/people.json
~~~

#### Spark 1.6.x Version

The Spark Dataset API brings the best of RDD and Data Frames together, for type safety and user functions that run directly on existing JVM types.

**Launch Spark**

As `spark` user, launch the Spark Shell:

~~~ bash
cd /usr/hdp/current/spark-client
./bin/spark-shell
~~~

At the `scala>` prompt, copy & paste the following:

~~~ js
val ds = Seq(1, 2, 3).toDS()
ds.map(_ + 1).collect() // Returns: Array(2, 3, 4)

// Encoders are also created for case classes.
case class Person(name: String, age: Long)
val ds = Seq(Person("Andy", 32)).toDS()

// DataFrames can be converted to a Dataset by providing a class. Mapping will be done by name.
val path = "/tmp/people.json"
val people = sqlContext.read.json(path).as[Person]
~~~
To view contents of people type:

~~~ js
people.show()
~~~

You should see an output similar to the following:

~~~
...
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+

scala>
~~~

To exit type `exit`.


## SparkSQL Thrift Server Example

SparkSQL’s thrift server provides JDBC access to SparkSQL.

**Create logs directory**

Change ownership of `logs` directory from `root` to `spark` user:

~~~ bash
cd /usr/hdp/current/spark-client
chown spark:hadoop logs
~~~

**Start Thrift Server**

~~~ bash
su spark
./sbin/start-thriftserver.sh --master yarn-client --executor-memory 512m --hiveconf hive.server2.thrift.port=10015
~~~

**Connect to the Thrift Server over Beeline**

Launch Beeline:

~~~ bash
./bin/beeline
~~~

**Connect to Thrift Server and issue SQL commands**

On the `beeline>` prompt type:

~~~ sql
!connect jdbc:hive2://localhost:10015
~~~

***Notes***
- This example does not have security enabled, so any username-password combination should work.
- The connection may take a few seconds to be available in a Sandbox environment.

You should see an output similar to the following:

~~~
beeline> !connect jdbc:hive2://localhost:10015
Connecting to jdbc:hive2://localhost:10015
Enter username for jdbc:hive2://localhost:10015:
Enter password for jdbc:hive2://localhost:10015:
...
Connected to: Spark SQL (version 1.6.2)
Driver: Spark Project Core (version 1.6.1.2.5.0.0-817)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://localhost:10015>
~~~

Once connected, try `show tables`:

~~~ sql
show tables;
~~~

You should see an output similar to the following:

~~~ bash
+------------+--------------+--+
| tableName  | isTemporary  |
+------------+--------------+--+
| sample_07  | false        |
| sample_08  | false        |
| testtable  | false        |
+------------+--------------+--+
3 rows selected (2.399 seconds)
0: jdbc:hive2://localhost:10015>
~~~

Type `Ctrl+C` to exit beeline.

**Stop Thrift Server**

~~~ bash
./sbin/stop-thriftserver.sh
~~~


## SparkR Example

#### Spark 1.6.x Version

**Prerequisites**

Before you can run SparkR, you need to install R linux distribution by following these steps as a `root` user:

~~~ bash
cd /usr/hdp/current/spark-client
su -c 'rpm -Uvh http://download.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm'
sudo yum update
sudo yum install R
~~~

**Upload data set**

If you haven't done so already in previous sections, make sure to upload people.json to HDFS:

~~~ bash
cd /usr/hdp/current/spark-client
su spark
hdfs dfs -copyFromLocal examples/src/main/resources/people.json /tmp/people.json
~~~

**Launch SparkR**

As `spark` user:

~~~ bash
./bin/sparkR
~~~

You should see an output similar to the following:

~~~ js
...
Welcome to
   ____              __
  / __/__  ___ _____/ /__
 _\ \/ _ \/ _ `/ __/  '_/
/___/ .__/\_,_/_/ /_/\_\   version  1.6.2
   /_/


Spark context is available as sc, SQL context is available as sqlContext
>
~~~

Initialize SQL context:

~~~ js
sqlContext <- sparkRSQL.init(sc)
~~~

Create a DataFrame:

~~~ js
df <- createDataFrame(sqlContext, faithful)
~~~

List the first few lines:

~~~ js
head(df)
~~~

You should see an output similar to the following:

~~~ js
...
eruptions waiting
1     3.600      79
2     1.800      54
3     3.333      74
4     2.283      62
5     4.533      85
6     2.883      55
>
~~~

Create people DataFrame from 'people.json':

~~~ js
people <- read.df(sqlContext, "/tmp/people.json", "json")
~~~

List the first few lines:

~~~ js
head(people)
~~~

You should see an output similar to the following:

~~~ js
...
age    name
1  NA Michael
2  30    Andy
3  19  Justin
~~~

For additional SparkR examples, see https://spark.apache.org/docs/latest/sparkr.html.

To exit SparkR type:

~~~ js
quit()
~~~


## Further Reading

For more tutorials on Spark, visit: [https://hortonworks.com/tutorials](https://hortonworks.com/tutorials).