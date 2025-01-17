# Kafka

```{toctree}
:maxdepth: 1
```

## Streaming data to Kafka from IBM i
There are several approaches to streaming data from Db2 transactions to Kafka, including but not limited to:
1. [Db2 triggers and Apache Camel](KAFKA_CAMEL_DTAQ.md): stream events in real-time
1. [Kafka Connect JDBC Source connector](KAFKA_CONNECT_JDBC.md): Simple, polling-based technique for importing Db2 data into Kafka
1. [Db2 WebQuery](https://techchannel.com/SMB/02/2023/IBM%E2%80%99s-DataMigrator-for-IBM-i) (external link): (**commercial offering**) Has a DataMigrator ETL extension that includes the ability to perform both bulk loads and incremental updates of data to Kafka.
1. [InfoSphere Data Replication and the CDC Replication Engine for Kafka](https://www.ibm.com/docs/en/idr/11.4.0?topic=replication-cdc-engine-kafka) (external link): (**commercial offering**) a CDC-based approach that may be a good options for current IBM CDC customers. Currently, this approach is unverified
1. [Native ILE Kafka client (unsupported)](https://github.com/AlexeiBaranov/librdkafka/blob/port-os400/packaging/os400/README.md) (external link): call Kafka functions directly from ILE programs.
1. The [Confluent Platform](https://docs.confluent.io/platform/current/platform.html) provides a number of tools for working with Kafka. To install,
follow the steps for "using only Confluent Community components"
[here](https://docs.confluent.io/platform/current/installation/installing_cp/zip-tar.html#prod-kafka-cli-install). The most notable components for IBM i
integration are:
    - [ksqlDB](https://docs.confluent.io/platform/current/platform.html#ksqldb), which provides an SQL interface for doing Kafka streaming operations using one
 of the ksqlDB clients, such as the provided Java client.
    - [Kafka REST APIs](https://docs.confluent.io/platform/current/kafka-rest/index.html), which provides a REST interface for doing Kafka streaming operations. The REST interfaces can then be called from anywhere, including from Db2 triggers by way of [SQL HTTP functions in Db2](https://techchannel.com/Trends/09/2021/sql-http-part-2) (example below).

## Kafka REST example with SQL

If using the Kafka REST API gateway, you can call use SQL to post data to the Kafka streams. For example:

```sql
VALUES QSYS2.HTTP_POST('http://myrestserver:8082/topics/fromrest',
                       '{"records":[{"key":"my_key","value":"Just some data to stream to Kafka"}]}',
                       '{"header":"Content-Type,application/vnd.kafka.json.v2+json" ,
                         "header":"Accept,application/vnd.kafka.v2+json" 
                         }');
```

This can also be embedded as needed in a Db2 trigger.

## Deploying Kafka on IBM i

These steps walk you through installing Kafka 3.0.1 (built with Scala 2.13) and deploying
on an IBM i system using OpenJDK 
[Early Access Builds](https://ibmi-oss-docs.readthedocs.io/en/latest/java11/JAVA11_EARLY_ACCESS.html).

This assumes you are using an SSH terminal and that you have
[your PATH set up properly](https://ibmi-oss-docs.readthedocs.io/en/latest/troubleshooting/SETTING_PATH.html),
for instance:
```
PATH=/QOpenSys/pkgs/bin:$PATH
export PATH
```
and you are hopefully, but optionally, [using bash](https://ibmi-oss-docs.readthedocs.io/en/latest/troubleshooting/SETTING_BASH.html)

Also, note that this deploys with the default "out of the box" settings for Zookeeper
and Kafka. Please refer to the Zookeeper and Kafka documentation to learn about customizing
these appropriately for a production deployment as needed. 

#### 1. Download requisite software
```
yum install wget ca-certificates-mozilla gzip tar-gnu openjdk-11 coreutils-gnu sed-gnu grep-gnu
```

#### 2. Change to your installation directory
```
cd /home/myusr/mydir
```

#### 3. Download kafka
```
wget https://dlcdn.apache.org/kafka/3.0.1/kafka_2.13-3.0.1.tgz
```
(you may need to update the version number on this and subsequent steps based on [the latest version](https://kafka.apache.org/downloads).

#### 4. extract Kafka
```
tar xzvf kafka_2.13-3.0.1.tgz
```

#### 5. Set up environment to use OpenJDK
```
JAVA_HOME=/QOpenSys/pkgs/lib/jvm/openjdk-11
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
```

#### 6. Start a Zookeeper server
```
cd kafka_2.13-3.0.1/config
../bin/zookeeper-server-start.sh zookeeper.properties
```

#### 7. Open a new session and change to your installation directory from earlier
```
cd /home/myusr/mydir
```

#### 8. Set up environment to use OpenJDK and start a Kafka server
```
JAVA_HOME=/QOpenSys/pkgs/lib/jvm/openjdk-11
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
cd kafka_2.13-3.0.1/config
../bin/kafka-server-start.sh server.properties
```

## Starting a Kafka visualizer

The Kafka visualizer used for the interactive workshop is an open source, Java-based visualizer that is accessed through
a web browser. The [project page](https://github.com/manasb-uoe/kafka-visualizer) has more information. It can be installed
with docker via
```
docker pull kbhargava/kafka-visuals
```
and run like this (substitute host names and port numbers as appropriate). 
```
docker run -p 8080:8080 --rm kbhargava/kafka-visuals idevphp.idevcloud.com:2181 idevphp.idevcloud.com:9092 DEV
```

If you want to run this visualizer on IBM i, install via yum:
```
yum install ca-certificates-mozilla
yum install https://github.com/ThePrez/kafka-visualizer/releases/download/v1.0.0/kafka-visualizer-1.0.0-0.ibmi7.3.ppc64.rpm
```
... and run using the following command and  navigate to `localhost:8080` on your browser:

```
/opt/kafka-visualizer/bin/kafka-visualizer
```


You can opt to use any kafka visualizer you'd like. Kafka even comes with a single-topic visualizer that can run in your SSH terminal, for instance

```
JAVA_HOME=/QOpenSys/pkgs/lib/jvm/openjdk-11
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
cd kafka_2.13-3.0.1/config
../bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mytopic
```
