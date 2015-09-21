## What's in this package

This repo contains Spark examples of applications built using RecordService client APIs.

- `Query1`/`Query2`: simple examples demonstrate how to use `RecordServiceRDD` to query
  a Parquet table.

- `WordCount`: a Spark application that counts the words in a directory. This demonstrates
  how to use the `SparkContext.recordServiceTextfile`.

- `SchemaRDDExample`: a example demonstrate how to use `RecordServiceSchemaRDD` to query tables.

- `TeraChecksum`: a terasort example that ported to Spark. It can be run with or without
  RecordService.

- `TpcdsBenchmark`: driver that can be used to run [TPC-DS](http://www.tpc.org/tpcds/) benchmark
  queries. It shows how to use SparkSQL and RecordService to execute the queries.

- `DataFrameExample`: a simple example demonstrate how to use DataFrame with RecordService by
  setting the data source.


## How to use RecordService with Spark shell

To use Spark shell with RecordService, first you'll need to start up `spark-shell`
with RecordService jar:

```bash
spark-shell --jars /path/to/recordservice-spark.jar
```

Then follow the examples to use Spark shell with RecordService in different ways.
F clarity most of the log outputs are omitted from the output, also the output may
vary depending on your environment, Spark version, etc.

- Example on using `RecordServiceRDD`

```scala
scala> import com.cloudera.recordservice.spark._
import com.cloudera.recordservice.spark._

scala> val data = sc.recordServiceRecords("select * from tpch.nation")
data: org.apache.spark.rdd.RDD[Array[org.apache.hadoop.io.Writable]] = RecordServiceRDD[0] at RDD at RecordServiceRDDBase.scala:57

scala> data.count()
res0: Long = 25
```

- Example on using DataFrame with RecordService

```scala
scala> import com.cloudera.recordservice.spark._
import com.cloudera.recordservice.spark._

scala> val context = new org.apache.spark.sql.SQLContext(sc)
context: org.apache.spark.sql.SQLContext = org.apache.spark.sql.SQLContext@706ef676

scala> val df = context.load("tpch.nation", "com.cloudera.recordservice.spark")
df: org.apache.spark.sql.DataFrame = [n_nationkey: int, n_name: string, n_regionkey: int, n_comment: string]

scala> val results = df.groupBy("n_regionkey").count().orderBy("n_regionkey").collect()
results: Array[org.apache.spark.sql.Row] = Array([0,5], [1,5], [2,5], [3,5], [4,5])
```

- Example on using SparkSQL with RecordService

```scala
scala> val context = new org.apache.spark.sql.SQLContext(sc)
context: org.apache.spark.sql.SQLContext = org.apache.spark.sql.SQLContext@53bfb38e

scala> context.sql(s"""
     | CREATE TEMPORARY TABLE nationTbl
     | USING com.cloudera.recordservice.spark
     | OPTIONS (
     |   RecordServiceTable 'tpch.nation'
     | )
     | """)
res0: org.apache.spark.sql.DataFrame = []

scala> val result = context.sql("SELECT count(*) FROM nationTbl").collect()(0).getLong(0)
result: Long = 25
```