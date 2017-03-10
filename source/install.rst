.. _install:

Install
=======

The Stream Reactor components are built around The Confluent Platform. They rely on the Kafka Brokers, Zookeepers and
optionally the Schema Registry provided by this distribution.

The following releases are available:

-  `0.2.4 <https://github.com/datamountaineer/stream-reactor/releases/tag/v0.2.4>`__
-  `0.2.3 <https://github.com/datamountaineer/stream-reactor/releases/tag/v0.2.3>`__
-  `0.2.2 <https://github.com/datamountaineer/stream-reactor/releases/tag/v0.2.2>`__

+------------------------+------------------------+------------------------+
| Kafka Version          | Confluent Version      | Stream reactor version |
+========================+========================+========================+
| 0.10.0.1               | 3.1                    | 0.2.4                  |
+------------------------+------------------------+------------------------+
| 0.10.0.1               | 3.0.1                  | 0.2.3                  |
+------------------------+------------------------+------------------------+
| 0.10.0.1               | 3.0.1                  | 0.2.2                  |
+------------------------+------------------------+------------------------+

+------------------------+------------------------+
| Connector              | Versions               |
+========================+========================+
| BlockChain             | Not applicable         |
+------------------------+------------------------+
| Bloomberg              | blpapi-3.8.8-2         |
+------------------------+------------------------+
| Cassandra              | Driver 3.0.0,          |
|                        | Server 3.0.9           |
+------------------------+------------------------+
| CoAP                   | Californium 2.0.0-M2   |
+------------------------+------------------------+
| Druid                  | Tranquility 0.7.4      |
+------------------------+------------------------+
| Elastic                | Elastic 2.2.0,         |
|                        | Elastic4s 2.3.0        |
+------------------------+------------------------+
| FTP                    | commons-net 0.5        |
+------------------------+------------------------+
| HazelCast              | HazelCast 3.6.0        |
+------------------------+------------------------+
| HBase                  | HBase Server 1.2.0,    |
|                        | HBase Client 1.2.0     |
+------------------------+------------------------+
| InfluxDB               | InfluxDB 2.3           |
+------------------------+------------------------+
| JMS                    | javax.jms 1.1-rev-1,   |
|                        | active-mq-code 1.26.0  |
+------------------------+------------------------+
| Kudu                   | Kudu Client 0.9.0      |
+------------------------+------------------------+
| MongoDB                | MongoDB 3.3.0          |
+------------------------+------------------------+
| MQTT                   | Mqtt 1.1.0             |
+------------------------+------------------------+
| Redis                  | Redis 2.8.1            |
+------------------------+------------------------+
| ReThinkDB              | ReThinkDB 2.3.3        |
+------------------------+------------------------+
| VoltDB                 | VoltDB 6.4             |
+------------------------+------------------------+
| Yahoo                  | yahoofinance-api 1.3.0 |
+------------------------+------------------------+

Install Confluent
~~~~~~~~~~~~~~~~~

Confluent can be downloaded for `here <http://www.confluent.io/download/>`__

.. sourcecode:: bash

    #make confluent home folder
    ➜  mkdir confluent

    #download confluent
    ➜  wget http://packages.confluent.io/archive/3.1/confluent-3.1.1-2.11.tar.gz

    #extract archive to confluent folder
    ➜  tar -xvf confluent-3.1.1-2.11.tar.gz -C confluent

    #setup variables
    ➜  export CONFLUENT_HOME=~/confluent/confluent-3.1.1

Start the Confluent platform.

.. sourcecode:: bash

    #Start the confluent platform, we need kafka, zookeeper and the schema registry
    bin/zookeeper-server-start etc/kafka/zookeeper.properties &
    sleep 10 && bin/kafka-server-start etc/kafka/server.properties &
    sleep 10 && bin/schema-registry-start etc/schema-registry/schema-registry.properties &

Stream Reactor Install
~~~~~~~~~~~~~~~~~~~~~~

Download the latest release from `here <https://github.com/datamountaineer/stream-reactor/releases>`__.

Unpack the archive:

.. sourcecode:: bash

    #Stream reactor release 0.2.4
    mkdir stream-reactor
    tar xvf stream-reactor-0.2.4-3.1.1.tar.gz -C stream-reactor

Within the unpacked directory you will find the following structure:

.. sourcecode:: bash

    stream-reactor-0.2.4-3.1.1
    |-- LICENSE
    |-- README.md
    |-- bin
    |   |-- cli.sh
    |   |-- install-ui.sh
    |   |-- sr-cli-linux
    |   |-- sr-cli-osx
    |   |-- start-connect.sh
    |   `-- start-ui.sh
    |-- conf
    |   |-- blockchain-source.properties
    |   |-- bloomberg-source.properties
    |   |-- cassandra-sink.properties
    |   |-- cassandra-source.properties
    |   |-- coap-hazelcast-sink.properties
    |   |-- coap-hazelcast-source.properties
    |   |-- coap-sink.properties
    |   |-- coap-source.properties
    |   |-- druid-sink.properties
    |   |-- elastic-sink.properties
    |   |-- ftp-source.properties
    |   |-- hazelcast-sink.properties
    |   |-- hbase-sink.properties
    |   |-- influxdb-sink.properties
    |   |-- jms-sink.properties
    |   |-- kudu-sink.properties
    |   |-- mongodb-sink.properties
    |   |-- mqtt-source.properties
    |   |-- redis-sink.properties
    |   |-- rethink-sink.properties
    |   |-- rethink-source.properties
    |   |-- voltdb-sink.properties
    |   `-- yahoo-source.properties
    |-- cql
    |-- libs
    |   |-- kafka-connect-blockchain-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-bloomberg-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-cassandra-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-cli-0.9-all.jar
    |   |-- kafka-connect-coap-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-druid-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-elastic-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-ftp-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-hazelcast-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-hbase-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-influxdb-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-jms-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-kudu-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-mongodb-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-mqtt-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-redis-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-rethink-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-voltdb-0.2.4-3.1.1-all.jar
    |   |-- kafka-connect-yahoo-0.2.4-3.1.1-all.jar
    |   `-- kafka-socket-streamer-0.2.4-3.1.1-all.jar

The ``libs`` folder contains all the Stream Reactor Connector jars.

The ``bin`` folder contains the ``start-connect.sh`` script. This loads all the Stream Reactors jars onto the CLASSPATH and starts
Kafka Connect in distributed mode. The Confluent Platform, Zookeeper, Kafka and the Schema Registry must be started first.

.. _dockers:

Docker Install
~~~~~~~~~~~~~~

All the Stream Reactor Connectors, Confluent and UI's for Connect, Schema Registry and topic browsing are available in Dockers.
The Docker images are available in `DockerHub <https://hub.docker.com/>`__ and maintained by our partner `Landoop <https://www.landoop.com/>`__

Pull the latest images:

.. sourcecode:: bash

    docker pull landoop/fast-data-dev
    docker pull landoop/fast-data-dev-connect-cluster

    #UI's
    docker pull landoop/kafka-topics-ui
    docker pull landoop/schema-registry-ui

Release Notes
-------------

0.2.4 (26 Jan 2017)
~~~~~~~~~~~~~~~~~~~

*   Added FTP and HTTP Source.
*   Added InfluxDB tag support. KCQL: INSERT INTO targetdimension SELECT * FROM influx-topic WITHTIMESTAMP sys_time() WITHTAG(field1, CONSTANT_KEY1=CONSTANT_VALUE1, field2,CONSTANT_KEY2=CONSTANT_VALUE1)
*   Added InfluxDb consistency level. Default is ALL. Use connect.influx.consistency.level to set it to ONE/QUORUM/ALL/ANY
*   InfluxDb connect.influx.sink.route.query was renamed to connect.influx.sink.kcql
*   Added support for multiple contact points in Cassandra

0.2.3 (5 Jan 2017)
~~~~~~~~~~~~~~~~~~

*   Added CoAP Source and Sink.
*   Added MongoDB Sink.
*   Added MQTT Source.
*   Hazelcast support for ring buffers, maps, sets, lists and cache.
*   Redis support for Sorted Sets.
*   Added start scripts.
*   Added Kafka Connect and Schema Registry CLI.
*   Kafka Connect CLI now supports pause/restart/resume; checking connectors on the classpath and validating configuration of connectors.
*   Support for Struct, Schema.STRING and Json with schema in the Cassandra, ReThinkDB, InfluxDB and MongoDB sinks.
*   Rename export.query.route to sink.kcql.
*   Rename import.query.route to source.kcql.
*   Upgrade to KCQL 0.9.5 - Add support for STOREAS so specify target sink types, e.g. Redis Sorted Sets, Hazelcast map, queues, ringbuffers.

Fast Data Dev
-------------

This is Docker image for development.

If you need

1.  Kafka Broker
2.  ZooKeeper
3.  Schema Registry
4.  Kafka REST Proxy
5.  Kafka Connect Distributed
6.  Certified DataMountaineer Connectors (ElasticSearch, Cassandra, Redis ..)
7.  Landoop's Fast Data Web UIs : schema-registry , kafka-topics , kafka-connect and
8.  Embedded integration tests with examples

Run with:

.. sourcecode:: bash

    docker run --rm -it --net=host landoop/fast-data-dev

On Mac OSX run:

.. sourcecode:: bash

    docker run --rm -it \
           -p 2181:2181 -p 3030:3030 -p 8081:8081 \
           -p 8082:8082 -p 8083:8083 -p 9092:9092 \
           -e ADV_HOST=127.0.0.1 \
           landoop/fast-data-dev

That's it. Your Broker is at localhost:9092, your Kafka REST Proxy at localhost:8082, your Schema Registry at
localhost:8081, your Connect Distributed at localhost:8083, your ZooKeeper at localhost:2181 and at
`<http://localhost:3030>`__ you will find Landoop's Web UIs for Kafka Topics and Schema Registry, as well as a Coyote test report.

.. figure:: ../images/landoop-docker.png
    :alt:

Fast Data Dev Connect
---------------------

This docker is targeted to more advanced users and is a special case since it doesn't set-up a Kafka cluster,
instead it expects to find a Kafka Cluster with Schema Registry up and running.

The developer can then use this docker image to setup a connect-distributed cluster by just spawning a couple containers.

.. sourcecode:: bash

    docker run -d --net=host \
           -e ID=01 \
           -e BS=broker1:9092,broker2:9092 \
           -e ZK=zk1:2181,zk2:2181 \
           -e SC=http://schema-registry:8081 \
           -e HOST=<IP OR FQDN> \
           landoop/fast-data-dev-connect-cluster


Things to look out for in configuration options:

1. It is important to give a full URL (including schema —http://) for schema registry.

2. ID should be unique to the Connect cluster you setup, for current and old instances. This is because Connect stores
data in Brokers and Schema Registry. Thus even if you destroyed a Connect cluster, its data remain in your Kafka setup.

3.  HOST should be set to an IP address or domain name that other connect instances and clients can use to reach the
current instance. We chose not to try to autodetect this IP because such a feat would fail more often than not.
Good choices are your local network ip (e.g 10.240.0.2) if you work inside a local network, your public ip (if you have
one and want to use it) or a domain name that is resolvable by all the hosts you will use to talk to Connect.

If you don't want to run with --net=host you have to expose Connect's port which at default settings is 8083.
There a PORT option, that allows you to set Connect's port explicitly if you can't use the default 8083. Please remember
that it is important to expose Connect's port on the same port at the host. This is a choice we had to make for simplicity's sake.


.. sourcecode:: bash

    docker run -d \
           -e ID=01 \
           -e BS=broker1:9092,broker2:9092 \
           -e ZK=zk1:2181,zk2:2181 \
           -e SC=http://schema-registry:8081 \
           -e HOST=<IP OR FQDN> \
           -e PORT=8085 \
           -p 8085:8085 \
           landoop/fast-data-dev-connect-cluster

Advanced
^^^^^^^^

The container does not exit with CTRL+C. This is because we chose to pass control directly to Connect, so you check your logs via docker logs.
You can stop it or kill it from another terminal.

Whilst the PORT variable sets the rest.port, the HOST variable sets the advertised host. This is the hostname that
Connect will send to other Connect instances. By default Connect listens to all interfaces, so you don't have to worry
as long as other instances can reach each instance via the advertised host.

Latest Test Results
-------------------

To see the latest tests for the Connectors, in a docker, please vist Landoop's test github `here <https://github.com/Landoop/kafka-connectors-tests>`__
Test results can be found `here <https://coyote.landoop.com/connect/>`__.

An example for BlockChain is:

.. figure:: ../images/blockchain-coyote-top.png
    :alt:

.. figure:: ../images/blockchain-coyote-bottom.png
    :alt:

