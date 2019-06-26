# KSQL Query Sysmon Process Create Events

## Requirements

* Ubuntu Bionic 18.04+

## Execute

Install Kafkacat

```
sudo apt install kafkacat

kafkacat -h

Usage: kafkacat <options> [file1 file2 .. | topic1 topic2 ..]]
kafkacat - Apache Kafka producer and consumer tool
https://github.com/edenhill/kafkacat
Copyright (c) 2014-2015, Magnus Edenhill
Version 1.3.1 (JSON) (librdkafka 0.11.6 builtin.features=gzip,snappy,ssl,sasl,regex,lz4,sasl_gssapi,sasl_plain,sasl_scram,plugins)


General options:
  -C | -P | -L | -Q  Mode: Consume, Produce, Metadata List, Query mode
  -G <group-id>      Mode: High-level KafkaConsumer (Kafka 0.9 balanced consumer groups)
                     Expects a list of topics to subscribe to
  -t <topic>         Topic to consume from, produce to, or list
  -p <partition>     Partition
  -b <brokers,..>    Bootstrap broker(s) (host[:port])
  -D <delim>         Message delimiter character:
                     a-z.. | \r | \n | \t | \xNN
                     Default: \n
..
...
```

if you have not cloned the demo repo yet, do it now

```
git clone https://github.com/Cyb3rWard0g/demos
cd demos/KSQL
```

Innstall docker and docker-compose

```
sudo ./docker_install
```

Run/Build KSQL Demo containers. Make sure you set the `ADVERTISED_LISTENER` to your Ubuntu's IP address

```
export ADVERTISED_LISTENER=192.168.64.138
sudo -E docker-compose -f helk-kafka-ksql.yml up --build -d

Creating network "ksql_helk" with driver "bridge"
Pulling helk-zookeeper (cyb3rward0g/helk-zookeeper:2.2.0)...
2.2.0: Pulling from cyb3rward0g/helk-zookeeper
c64513b74145: Pull complete
01b8b12bad90: Pull complete
c5d85cf7a05f: Pull complete
b6b268720157: Pull complete
..
...
Creating helk-zookeeper ... done
Creating helk-kafka-broker ... done
Creating helk-ksql-server  ... done
Creating helk-ksql-cli     ... done
```


Check if containers are up and runnning:

```
sudo docker ps

CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                          NAMES
a2114ad16b2f        confluentinc/cp-ksql-cli:5.1.3        "/bin/sh"                About a minute ago   Up About a minute                                  helk-ksql-cli
a34e66a2bc3f        confluentinc/cp-ksql-server:5.1.3     "/etc/confluent/dock…"   About a minute ago   Up About a minute   0.0.0.0:8088->8088/tcp         helk-ksql-server
7b32cefcf372        cyb3rward0g/helk-kafka-broker:2.2.0   "./kafka-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:9092->9092/tcp         helk-kafka-broker
4d93c663a597        cyb3rward0g/helk-zookeeper:2.2.0      "./zookeeper-entrypo…"   About a minute ago   Up About a minute   2181/tcp, 2888/tcp, 3888/tcp   helk-zookeeper
```

Check if KSQL Server is ready:

```
sudo docker logs --follow helk-ksql-server

===> ENV Variables ...
COMPONENT=ksql-server
CUB_CLASSPATH="/usr/share/java/cp-base-new/*"
HOME=/root
HOSTNAME=a34e66a2bc3f
KSQL_BOOTSTRAP_SERVERS=helk-kafka-broker:9092
KSQL_CLASSPATH=/usr/share/java/ksql-server/*
KSQL_CUB_KAFKA_TIMEOUT=300
KSQL_HEAP_OPTS=-Xmx1g
KSQL_KSQL_CACHE_MAX_BYTES_BUFFERING=10000000
KSQL_KSQL_COMMIT_INTERVAL_MS=2000
KSQL_KSQL_SERVICE_ID=wardog
KSQL_KSQL_STREAMS_AUTO_OFFSET_RESET=earliest
KSQL_LISTENERS=http://0.0.0.0:8088
LANG=C.UTF-8
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
SHLVL=1
_=/usr/bin/env
===> User
uid=0(root) gid=0(root) groups=0(root)
===> Configuring ...
===> Running preflight checks ... 
===> Check if Kafka is healthy ...
[main] INFO org.apache.kafka.clients.admin.AdminClientConfig - AdminClientConfig values: 
	bootstrap.servers = [helk-kafka-broker:9092]
	client.dns.lookup = default
	client.id = 
	connections.max.idle.ms = 300000
	metadata.max.age.ms = 300000
..
...
```

Launch KSQL CLI interface

```
sudo docker exec -ti helk-ksql-cli ksql http://helk-ksql-server:8088

                  ===========================================
                  =        _  __ _____  ____  _             =
                  =       | |/ // ____|/ __ \| |            =
                  =       | ' /| (___ | |  | | |            =
                  =       |  <  \___ \| |  | | |            =
                  =       | . \ ____) | |__| | |____        =
                  =       |_|\_\_____/ \___\_\______|       =
                  =                                         =
                  =  Streaming SQL Engine for Apache Kafka® =
                  ===========================================

Copyright 2017-2018 Confluent Inc.

CLI v5.1.3, Server v5.1.3 located at http://helk-ksql-server:8088

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql>
```

Show KSQL properties

```
ksql> SHOW PROPERTIES;

 Property                                               | Default override | Effective Value                                               
-------------------------------------------------------------------------------------------------------------------------------------------
 ksql.avro.maps.named                                   |                  | false                                                         
 ksql.extension.dir                                     |                  | ext                                                           
 ksql.functions.substring.legacy.args                   |                  | false                                                         
 ksql.output.topic.name.prefix                          |                  |                                                               
 ksql.persistent.prefix                                 |                  | query_                                                        
 ksql.schema.registry.url                               |                  | http://localhost:8081                                         
 ksql.service.id                                        | SERVER           | wardog                                                        
 ksql.sink.partitions                                   |                  | 4                                                             
 ksql.sink.replicas                                     |                  | 1                                                             
 ksql.sink.window.change.log.additional.retention       |                  | 1000000                                                       
 ksql.statestore.suffix                                 |                  | _ksql_statestore                                              
 ksql.streams.application.id                            | SERVER           | KSQL_REST_SERVER_DEFAULT_APP_ID                               
 ksql.streams.auto.offset.reset                         | SERVER           | earliest                                                      
 ksql.streams.bootstrap.servers                         | SERVER           | helk-kafka-broker:9092                                        
 ksql.streams.cache.max.bytes.buffering                 | SERVER           | 10000000                                                      
 ksql.streams.commit.interval.ms                        | SERVER           | 2000                                                          
 ksql.streams.default.deserialization.exception.handler | SERVER           | io.confluent.ksql.errors.LogMetricAndContinueExceptionHandler 
 ksql.streams.num.stream.threads                        | SERVER           | 4                                                             
 ksql.transient.prefix                                  |                  | transient_                                                    
 ksql.udf.collect.metrics                               |                  | false                                                         
 ksql.udf.enable.security.manager                       |                  | true                                                          
 ksql.udfs.enabled                                      |                  | true                                                          
-------------------------------------------------------------------------------------------------------------------------------------------
ksql>
```

List Topics

```
ksql> SHOW TOPICS;

 Kafka Topic | Registered | Partitions | Partition Replicas | Consumers | ConsumerGroups 
-----------------------------------------------------------------------------------------
 filebeat    | false      | 1          | 1                  | 0         | 0              
 SYSMON_JOIN | false      | 1          | 1                  | 0         | 0              
 winlogbeat  | true       | 1          | 1                  | 0         | 0              
-----------------------------------------------------------------------------------------
ksql>
```

Create Stream

```
ksql> CREATE STREAM WINLOGBEAT_STREAM (source_name VARCHAR, type VARCHAR, task VARCHAR, log_name VARCHAR, computer_name VARCHAR, event_data STRUCT< UtcTime VARCHAR, ProcessGuid VARCHAR, ProcessId INTEGER, Image VARCHAR, FileVersion VARCHAR, Description VARCHAR, Product VARCHAR, Company VARCHAR, CommandLine VARCHAR, CurrentDirectory VARCHAR, User VARCHAR, LogonGuid VARCHAR, LogonId VARCHAR, TerminalSessionId INTEGER, IntegrityLevel VARCHAR, Hashes VARCHAR, ParentProcessGuid VARCHAR, ParentProcessId INTEGER, ParentImage VARCHAR, ParentCommandLine VARCHAR>, event_id INTEGER) WITH (KAFKA_TOPIC='winlogbeat', VALUE_FORMAT='JSON');

 Message        
----------------
 Stream created 
----------------
ksql> 
```

Show Stream

```
ksql> SHOW STREAMS;

 Stream Name       | Kafka Topic | Format 
------------------------------------------
 WINLOGBEAT_STREAM | winlogbeat  | JSON   
------------------------------------------
ksql> 
```

Query Stream for Sysmon event 1 (ProcessCreate) and wait..

```
SELECT S.COMPUTER_NAME, S.EVENT_DATA->PROCESSGUID , S.EVENT_DATA->IMAGE FROM WINLOGBEAT_STREAM S WHERE S.SOURCE_NAME='Microsoft-Windows-Sysmon' AND S.EVENT_ID=1 LIMIT 5;
..
...
```

In a differennt terminal, run kafkacat to publish `empire_rubeus_asktgt` mordor dataset to the Kafka broker

```
kafkacat -b 192.168.64.138:9092 -t winlogbeat -P -l empire_rubeus_asktgt_ptt_createnetonly_2019-03-19151006.json
```

Go back to your KSQL terminal and you will get events

```
ksql> SELECT S.COMPUTER_NAME, S.EVENT_DATA->PROCESSGUID , S.EVENT_DATA->IMAGE FROM WINLOGBEAT_STREAM S WHERE S.SOURCE_NAME='Microsoft-Windows-Sysmon' AND S.EVENT_ID=1 LIMIT 5;
IT001.shire.com | {aa6b4a20-3f53-5c91-0000-0010ae393b05} | C:\Windows\System32\Rubeus.net4.5.exe
IT001.shire.com | {aa6b4a20-3f53-5c91-0000-0010e43d3b05} | C:\Windows\System32\cmd.exe
IT001.shire.com | {aa6b4a20-3f53-5c91-0000-001038413b05} | C:\Windows\System32\conhost.exe
IT001.shire.com | {aa6b4a20-3f5c-5c91-0000-0010eb513b05} | C:\Windows\System32\backgroundTaskHost.exe
IT001.shire.com | {aa6b4a20-3f5d-5c91-0000-0010eb5e3b05} | C:\Windows\System32\RuntimeBroker.exe
Limit Reached
Query terminated
ksql>
```

Fin

