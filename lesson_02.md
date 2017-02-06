Big Data Academy lesson 2

Sqoop
---

```sh
mysql -uroot -pcloudera
```

```sql
show databases;
use retail_db;
show tables;
```

```sh
sqoop list-databases --connect jdbc:mysql://localhost --username root --password cloudera
sqoop import --connect jdbc:mysql://localhost:3306/retail_db --table departments --username root --password cloudera
```
Launched map tasks=4

```sh
hadoop fs -ls /user/root/departments
```

Outputs:
```
	Found 5 items
	-rw-r--r--   1 root supergroup          0 2017-01-24 18:36 /user/root/departments/_SUCCESS
	-rw-r--r--   1 root supergroup         21 2017-01-24 18:36 /user/root/departments/part-m-00000
	-rw-r--r--   1 root supergroup         10 2017-01-24 18:36 /user/root/departments/part-m-00001
	-rw-r--r--   1 root supergroup          7 2017-01-24 18:36 /user/root/departments/part-m-00002
	-rw-r--r--   1 root supergroup         22 2017-01-24 18:36 /user/root/departments/part-m-00003
```

```sh
sqoop import --connect jdbc:mysql://localhost:3306/retail_db --table customers --username root --password cloudera --target-dir /items --fields-terminated-by '\t' -m1
```
Launched map tasks=1

```sh
hadoop fs -ls /items
```

Outputs:
```
	Found 2 items
	-rw-r--r--   1 root supergroup          0 2017-01-24 18:38 /items/_SUCCESS
	-rw-r--r--   1 root supergroup     953525 2017-01-24 18:38 /items/part-m-00000
```

```sh
hadoop fs -cat /user/root/departments/part*
hadoop fs -cat /items/part*
```

```sh
sqoop import --connect jdbc:mysql://localhost:3306/retail_db --table departments --username root --password cloudera --target-dir /departments_2 --fields-terminated-by '\t' -m1 --where department_id=2
```

```sh
hadoop fs -cat /departments_2/part*
```

Outputs
```
	2	Fitness
```

```sh
sqoop import-all-tables --connect jdbc:mysql://localhost:3306/retail_db --username root --password cloudera
```

```sh
sqoop import --connect jdbc:mysql://localhost:3306/retail_db --table departments --username root --password cloudera -as-sequencefile --target-dir /departments_seq/
```

```sh
hadoop fs -cat /departments_seq/part*
```

```sh
sqoop import --connect jdbc:mysql://localhost:3306/retail_db --table departments --username root --password cloudera -as-avrodatafile --target-dir /departments_avro/
```

```sh
hadoop fs -cat /departments_avro/part*
```

Import to Hive: `--hive-import`
Import to HBase: `--hbase-table table_name --column-family column_family --hbase-row-key row_key --hbase-create-table`

Back to SQL:

```sh
mysql -uroot -pcloudera retail_db
```

```sql
create table departments_data like departments;
```

```sh
sqoop export --connect jdbc:mysql://localhost:3306/retail_db --table departments_data --username root --password cloudera --export-dir departments
```

```sh
mysql -uroot -pcloudera retail_db
```

```sql
select count(*) from departments_data;
update departments_data SET department_name = '';
```

```sh
sqoop export --connect jdbc:mysql://localhost:3306/retail_db --table departments_data --username root --password cloudera --export-dir departments --update-key department_id
```

```sh
mysql -uroot -pcloudera retail_db
```

```sql
select count(*) from departments_data;
```

Job creating:
BE CAREFUL !!! We need SPACE between `--` and `import` !!!

```sh
sqoop job --create jobname -- import --connect jdbc:mysql://localhost:3306/retail_db --table departments --username root --password cloudera -as-avrodatafile --target-dir /departments_avro/
hadoop fs -rmr /departments_avro/
sqoop job --list
sqoop job --exec jobname -> password: cloudera
sqoop job --show jobname
sqoop job --delete jobname
```

remove password promting: !!!!

```sh
mcedit /usr/lib/sqoop/conf/sqoop-site.xml
sqoop job --delete jobname
```

Options file
https://sqoop.apache.org/docs/1.4.3/SqoopUserGuide.html#_using_options_files_to_pass_arguments

Autoincrements:
`--check-column(col)` -> check column with --last-value param
`--incremental(mode)` -> append / lastmodified. Defaul - append. Lastmodified for columns which could be changed. Each change should update column with current timestamp
`--last-value(value)` -> start from values > to "value"

Sqoop will ask you to save job and handle status change automatically (in logs)

Pig
---

https://pig.apache.org/docs/r0.7.0/piglatin_ref2.html

```sh
hadoop fs -mkdir /user/cloudera/pig
hadoop fs -put ./generated.5.csv /user/cloudera/pig
pig
```

```sh
ls /
```

Outputs:
```
	hdfs://quickstart.cloudera:8020/benchmarks	<dir>
	hdfs://quickstart.cloudera:8020/flume	<dir>
	hdfs://quickstart.cloudera:8020/hbase	<dir>
	hdfs://quickstart.cloudera:8020/tmp	<dir>
	hdfs://quickstart.cloudera:8020/user	<dir>
	hdfs://quickstart.cloudera:8020/var	<dir>
```

```sh
ls /user/cloudera/pig/
```

Outputs
```
	hdfs://quickstart.cloudera:8020/user/cloudera/pig/generated.5.csv<r 1>	19268
```

```sh
cd /user/cloudera/pig
pwd
copyFromLocal /home/bda/generated.4.csv generated.4.csv
ls
```

```sql
csv = LOAD 'generated.5.csv' using PigStorage(',') as (userid: int, notify: chararray, targetid: int, timestamp: int);
DUMP csv;
```

Outputs
```
	(1,a,43394,1594357)
	(97,d,299610,845596)
	(26,d,410195,2552280)
```

```sql
csv_limit = LIMIT csv 3;
DUMP csv_limit;
csv_filtered = FILTER csv BY userid == 26;
DUMP csv_filtered;
```

Outputs
```
	(26,f,117096,351237)
	(26,b,198687,135918)
	(26,f,246531,205754)
	(26,c,318628,1261149)
	(26,c,424167,1665551)
	(26,b,17652,1475624)
	(26,e,466421,2343187)
	(26,f,449813,524981)
	(26,c,364205,722843)
	(26,d,410195,2552280)
```

```sql
ILLUSTRATE csv_filtered;
csv_grouped = GROUP csv BY userid;
ILLUSTRATE csv_grouped;
```

Outputs
```
	{(20, ..., 2587099), (20, ..., 2454892)}
```

```sql
csv_count = FOREACH csv_grouped GENERATE group as userid, COUNT(csv);
ILLUSTRATE csv_count;
```

Ouputs
```
	----------------------------------------------
	| csv_count     | userid:int     | :long     |
	----------------------------------------------
	|               | 20             | 2         |
	----------------------------------------------
```

```sql
STORE csv_count INTO 'csv_count_result' USING PigStorage(',');
```

```sh
ls
```

Ouputs
```
	hdfs://quickstart.cloudera:8020/user/cloudera/pig/csv_count_result	<dir>
	hdfs://quickstart.cloudera:8020/user/cloudera/pig/generated.4.csv<r 1>	21285
	hdfs://quickstart.cloudera:8020/user/cloudera/pig/generated.5.csv<r 1>	19268
```

```sh
cd csv_count_result
ls
```

Outputs
```
	hdfs://quickstart.cloudera:8020/user/cloudera/pig/csv_count_result/_SUCCESS<r 1>	0
	hdfs://quickstart.cloudera:8020/user/cloudera/pig/csv_count_result/part-r-00000<r 1>	537
```

```sh
cat part-r-00000
```

Hive
---

Metadata -> in MySQL
Data -> HDFS

TYPES:
```
primitive_type +
array_type
  : ARRAY < data_type >
map_type
  : MAP < primitive_type, data_type >
struct_type
  : STRUCT < col_name : data_type [COMMENT col_comment], ...>
union_type
   : UNIONTYPE < data_type, data_type, ... >
```

```sql
create table social_users(
  userid int,
  notify string
  )
  row format delimited
  fields terminated by '\t'
  stored as textfile;
```

```sql
INSERT INTO social_users VALUES(35,'1');
INSERT INTO social_users VALUES(69,'2'),(24,'3'),(23,'4'),(9,'5');
```

```sh
hdfs dfs -put /tmp/generated.1.csv /
```

```sql
LOAD DATA INPATH '/generated.1.csv' INTO TABLE social_new;
```

`LOAD DATA` will MOVE (not copy !!!) file to `/apps/hive/warehouse/social` on HDFS

```sql
create table social_new(
  userid int,
  notify string,
  targetid int,
  `timestamp` int
  )
  stored as orc;
```

```sql
LOAD DATA INPATH '/generated.5.csv' INTO TABLE social_new;
select count(*) from social_new;
```

Outputs:
```
Caused by: java.io.IOException: Malformed ORC file hdfs://quickstart.cloudera:8020/user/hive/warehouse/social_new/generated.5.csv. Invalid postscript.
```

```sql
drop table social_new;
```

```sh
hadoop fs -put generated.5.csv /generated.5.csv
```

```sql
create table social_new(
  userid int,
  notify string,
  targetid int,
  `timestamp` int
)
row format delimited
fields terminated by ','
stored as textfile;
```

```sql
LOAD DATA INPATH '/generated.5.csv' INTO TABLE social_new;
select count(*) from social_new;
```

Outputs:
```
	Total MapReduce CPU Time Spent: 2 seconds 610 msec
	OK
	1000
```

Joins:
```sql
SELECT * FROM social_new INNER JOIN social_users ON social_new.userid = social_users.userid;
```

Immidiatly add data:
```sh
hadoop fs -put /path/to/file/file.csv /user/hive/warehouse/social/
```

Hive support inputs from:
- SerDe
- AVRO
- XML
- JSON

Recommended format: ORC (STORED AS ORC)

Cost Based Optimization:
```
set hive.cbo.enable=true;
set hive.compute.query.using.stats=true;
set hive.stats.fetch.column.stats=true;
set hive.stats.fetch.partition.stats=true;
```

```
analyze table TableName compute statistics for columns;
```

// Memory Joins:
Slides.

// Sort-Merge-Bucket JOIN
```
set hive.auto.convert.sortmerge.join=true;
set hive.optimize.bucketmapjoin = true;
set hive.optimize.bucketmapjoin.sortedmerge = true;
set hive.auto.convert.sortmerge.join.noconditionaltask = true;
```

```sql
CREATE TABLE table1 (id INT, field STRING) CLUSTERED BY (id) SORTED BY (id) INTO 10 BUCKETS;
CREATE TABLE table2 (id INT, id_1 INT, field_2 STRING) CLUSTERED BY (id_1) SORTED BY (id_1) INTO 10 BUCKETS;
```

Show Slide.
No reduce task (!)

Skewed problem:

	Suppose we have table A with a key column, "id" which has values 1, 2, 3 and 4, and table B with a similar column, which has values 1, 2 and 3.
	We want to do a join corresponding to the following query:

```sql
select A.id from A join B on A.id = B.id
```

	A set of Mappers read the tables and gives them to Reducers based on the keys. e.g., rows with key 1 go to Reducer R1, rows with key 2 go to Reducer R2 and so on. These Reducers do a cross product of the values from A and B, and write the output. The Reducer R4 gets rows from A, but will not produce any results.
	Now let's assume that A was highly skewed in favor of id = 1. Reducers R2 and R3 will complete quickly but R1 will continue for a long time, thus becoming the bottleneck. If the user has information about the skew, the bottleneck can be avoided manually as follows:

```sql
select A.id from A join B on A.id = B.id where A.id <> 1;
select A.id from A join B on A.id = B.id where A.id = 1 and B.id = 1;
```

https://cwiki.apache.org/confluence/display/Hive/ListBucketing#ListBucketing-SkewedTablevs.ListBucketingTable

Compression:

```
SET hive.exec.compress.output=true;
SET io.seqfile.compression.type=BLOCK;
```

LZO compression supported. See: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LZO

//GROUP by
```
set hive.map.aggr=true;
```

Spark
---

```sh
ln -s /usr/lib/hive/conf/hive-site.xml /usr/lib/spark/conf/hive-site.xml
```

```sh
spark-shell
```

```scala
import org.apache.spark.sql.hive.HiveContext
import sqlContext.implicits._
val hiveObj = new HiveContext(sc)
hiveObj.refreshTable("social_new")
sqlContext.sql("SELECT * FROM social_users").collect().toList
```

Outputs:
```
res10: List[org.apache.spark.sql.Row] = List([35,1], [69,2], [24,3], [23,4], [9,5], [69,2], [24,3], [23,4], [9,5])
```

```scala
sqlContext.sql("SELECT * FROM social_users").collect().toSet
```

Outputs:
```
res11: scala.collection.immutable.Set[org.apache.spark.sql.Row] = Set([35,1], [9,5], [69,2], [24,3], [23,4])
```
