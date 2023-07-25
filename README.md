# Base Bigdata playground: Hadoop + Hive + Spark

Base Docker image with just essentials: Hadoop, Hive and Spark.

## Software

* [Hadoop 3.2.0](http://hadoop.apache.org/docs/r3.2.0/) in Fully Distributed (Multi-node) Mode

* [Hive 3.1.2](http://hive.apache.org/) with HiveServer2 exposed to host.

* [Spark 2.4.5](https://spark.apache.org/docs/2.4.5/) in YARN mode (Spark Scala, PySpark and SparkR)

* [Zeppelin 0.9.0](https://zeppelin.apache.org/docs/0.9.0/) 

## Usage

Hive JDBC port is exposed to host:
* URI: `jdbc:hive2://localhost:10000`
* Driver: `org.apache.hive.jdbc.HiveDriver` (org.apache.hive:hive-jdbc:3.1.2)
* User and password: unused.

```bash
cd docker-compose
docker-compose up -d
```
* **data/** directory is mounted into every container, you can use this as
a storage both for files you want to process using Hive/Spark/whatever
and results of those computations.
* **zeppelin_notebooks/** contains, quite predictably, notebook files
for Zeppelin. Thanks to that, all your notebooks persist across runs.

To shut the whole thing down, run this from the same folder:
```bash
docker-compose down
```

## Checking if everything plays well together
You can quickly check everything by opening the
[bundled Zeppelin notebook](http://localhost:8890)
and running all paragraphs.

Alternatively, to get a sense of
how it all works under the hood, follow the instructions below:

### Hadoop and YARN:

Check [YARN (Hadoop ResourceManager) Web UI
(localhost:8088)](http://localhost:8088/).
You should see 2 active nodes there.
There's also an
[alternative YARN Web UI 2 (http://localhost:8088/ui2)](http://localhost:8088/ui2).

Then, [Hadoop Name Node UI (localhost:9870)](http://localhost:9870),
Hadoop Data Node UIs at
[http://localhost:9864](http://localhost:9864) and [http://localhost:9865](http://localhost:9865):
all of those URLs should result in a page.

Open up a shell in the master node.
```bash
docker-compose exec master bash
jps
```
`jps` command outputs a list of running Java processes,
which on Hadoop Namenode/Spark Master node should include those:
<pre>
123 Jps
456 ResourceManager
789 NameNode
234 SecondaryNameNode
567 HistoryServer
890 Master
</pre>

## Version compatibility notes
* Hadoop 3.2.1 and Hive 3.1.2 are incompatible due to Guava version
mismatch (Hadoop: Guava 27.0, Hive: Guava 19.0). Hive fails with
`java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)`
* Spark 2.4.4 can not 
[use Hive higher than 1.2.2 as a SparkSQL engine](https://spark.apache.org/docs/2.4.4/sql-data-sources-hive-tables.html)
because of this bug: [Spark need to support reading data from Hive 2.0.0 metastore](https://issues.apache.org/jira/browse/SPARK-13446)
and associated issue [Dealing with TimeVars removed in Hive 2.x](https://issues.apache.org/jira/browse/SPARK-27349).
Trying to make it happen results in this exception:
`java.lang.NoSuchFieldError: HIVE_STATS_JDBC_TIMEOUT`.
When this is fixed in Spark 3.0, it will be able to use Hive as a
backend for SparkSQL. Alternatively you can try to downgrade Hive :)

## TODO
* Upgrade spark to 3.0
* When upgraded, enable Spark-Hive integration.