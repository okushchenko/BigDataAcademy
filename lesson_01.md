Big Data Academy lesson 1

```sh
docker run --hostname=quickstart.cloudera --privileged=true -t -i -p 8888:8888 -p 7180:7180 -p 80:80 -p 60010:60010 -p 10002:10002 -p 8088:8088 -p 19888:19888 4239cd2958c6 /usr/bin/docker-quickstart
```

```sh
sudo su cloudera
```

```sh
dd if=/dev/zero of=1m bs=1M count=1
dd if=/dev/zero of=128m bs=128M count=1
dd if=/dev/zero of=129m bs=129M count=1
dd if=/dev/zero of=1m bs=128M count=1
mkfile -n 1m /path/to/file
mkfile -n 128m /path/to/file
mkfile -n 129m /path/to/file
mkfile -n 1g /path/to/file
```

HDFS
---

```sh
mkdir /home/bda/
cd /home/bda/
hadoop fsck /user/cloudera/1m -files -blocks
hadoop fs -stat "Blocks: %b Group: %g Name: %n Block Size: %o Replication : %r username: %u" /user/cloudera/generated.csv
hadoop fs -get /user/cloudera/1m ./
```

ORC, Parquet:
File Formats

Flume
---
https://flume.apache.org/FlumeUserGuide.html

```sh
mkdir flumeSpool
flume-ng agent -n agent_name -c conf -f flume.conf
less /usr/lib/flume-ng/conf/flume.conf
flume-ng agent --conf ./conf/ -f /usr/lib/flume-ng/conf/flume.conf -Dflume.root.logger=DEBUG,console -n a1
```

```
a1.sources = r1
a1.sinks = k1
a1.channels = c1

a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

a1.sinks.k1.type = logger

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

```sh
telnet localhost 44444
```

```
tier1.sources  = source1
tier1.channels = channel1
tier1.sinks    = sink1

tier1.sources.source1.type = spooldir
tier1.sources.source1.channels = channel1
tier1.sources.source1.spoolDir = /home/bda/flumeSpool
tier1.sources.source1.fileHeader = true

tier1.channels.channel1.type   = memory
tier1.channels.channel1.capacity = 100

tier1.sinks.sink1.type         = logger
tier1.sinks.sink1.channel      = channel1
```

```sh
less /var/log/flume-ng/flume-cmf-flume-AGENT-quickstart.cloudera.log
```

ONLY UTF-8 for parsing

```
tier1.sources  = source1
tier1.channels = channel1
tier1.sinks    = sink1

tier1.sources.source1.type = spooldir
tier1.sources.source1.channels = channel1
tier1.sources.source1.spoolDir = /home/bda/flumeSpool
tier1.sources.source1.fileHeader = true

tier1.channels.channel1.type   = memory
tier1.channels.channel1.capacity = 100

tier1.sinks.sink1.type = hdfs
tier1.sinks.sink1.channel = channel1
tier1.sinks.sink1.hdfs.path = /user/cloudera/flume/events/%y-%m-%d/%H%M
tier1.sinks.sink1.hdfs.rollInterval = 0
tier1.sinks.sink1.hdfs.rollSize = 0
tier1.sinks.sink1.hdfs.rollCount = 1000000
tier1.sinks.sink1.hdfs.useLocalTimeStamp = true
```

Hive
---

Agent Environment Advanced Configuration Snippet:
```sh
export HCAT_HOME=/usr/lib/hive-hcatalog
export HIVE_HOME=/usr/lib/hive
hive
```

```
create table social(
  userid int,
  notify string,
  targetid int,
  `timestamp` timestamp
  )
  clustered by (userid) into 5 buckets
  stored as orc
```

```
tier1.sources  = source1
tier1.channels = channel1
tier1.sinks    = sink1

tier1.sources.source1.type = spooldir
tier1.sources.source1.channels = channel1
tier1.sources.source1.spoolDir = /home/bda/flumeSpool
tier1.sources.source1.fileHeader = true

tier1.channels.channel1.type   = memory
tier1.channels.channel1.capacity = 100

tier1.sinks.sink1.type = hive
tier1.sinks.sink1.channel = channel1
tier1.sinks.sink1.hive.metastore = thrift://127.0.0.1:9083
tier1.sinks.sink1.hive.database = default
tier1.sinks.sink1.hive.table = social
tier1.sinks.sink1.useLocalTimeStamp = false
tier1.sinks.sink1.round = true
tier1.sinks.sink1.roundValue = 10
tier1.sinks.sink1.roundUnit = minute
tier1.sinks.sink1.serializer = DELIMITED
tier1.sinks.sink1.serializer.delimiter = "\t"
tier1.sinks.sink1.serializer.serdeSeparator = '\t'
tier1.sinks.sink1.serializer.fieldnames =userid,notify,targetid,timestamp
```

```sh
less /var/log/flume-ng/flume-cmf-flume-AGENT-quickstart.cloudera.log
```

Will have this error:
```
org.apache.flume.EventDeliveryException: org.apache.flume.ChannelException: Take list for MemoryTransaction, capacity 100 full, consider committing more frequently, increasing capacity, or increasing thread count
```

```
tier1.sources  = source1
tier1.channels = channel1
tier1.sinks    = sink1

tier1.sources.source1.type = spooldir
tier1.sources.source1.channels = channel1
tier1.sources.source1.spoolDir = /home/bda/flumeSpool
tier1.sources.source1.fileHeader = true

tier1.channels.channel1.type = file
tier1.channels.channel1.transactionCapacity = 1000000
tier1.channels.channel1.checkpointDir = /tmp/flume/checkpoint
tier1.channels.channel1.dataDirs = /tmp/flume/data

tier1.sinks.sink1.type = hive
tier1.sinks.sink1.channel = channel1
tier1.sinks.sink1.hive.metastore = thrift://127.0.0.1:9083
tier1.sinks.sink1.hive.database = default
tier1.sinks.sink1.hive.table = social
tier1.sinks.sink1.useLocalTimeStamp = false
tier1.sinks.sink1.serializer = DELIMITED
tier1.sinks.sink1.serializer.fieldnames =userid,notify,targetid,timestamp
```

```sql
SELECT COUNT(*) FROM social;
LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE table;
LOAD DATA LOCAL INPATH './examples/files/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
```

```sql
CREATE TABLE u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
```
