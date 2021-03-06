Kafka Connect Kudu
===================

A Connector and Sink to write events from Kafka to kudu.The connector takes the value from the Kafka Connect SinkRecords
and inserts a new entry to Kudu.

The Sink supports:

1. :ref:`The KCQL routing querying <kcql>` - Kafka topic payload field selection is supported, allowing you to select fields written to Kudu.
2. Topic to table routing via KCQL.
3. Auto table create with DISTRIBUTE BY partition strategy via KCQL.
4. Auto evolution of tables via KCQL.
5. Error policies for handling failures.

Prerequisites
-------------

- Confluent 3.3
- Kudu 0.8
- Java 1.8
- Scala 2.11

Setup
-----

Kudu Setup
~~~~~~~~~~

Download and check Kudu QuickStart VM starts up (VM password is demo).

.. sourcecode:: bash

    curl -s https://raw.githubusercontent.com/cloudera/kudu-examples/master/demo-vm-setup/bootstrap.sh | bash

Confluent Setup
~~~~~~~~~~~~~~~

Follow the instructions :ref:`here <install>`.

Sink Connector QuickStart
-------------------------

We you start the Confluent Platform, Kafka Connect is started in distributed mode (``confluent start``). 
In this mode a Rest Endpoint on port ``8083`` is exposed to accept connector configurations. 
We developed Command Line Interface to make interacting with the Connect Rest API easier. The CLI can be found in the Stream Reactor download under
the ``bin`` folder. Alternatively the Jar can be pulled from our GitHub
`releases <https://github.com/datamountaineer/kafka-connect-tools/releases>`__ page.

Kudu Table
~~~~~~~~~~

Lets create a table in Kudu via Impala. The Sink does support auto creation of tables but they are not sync'd yet with Impala.

.. sourcecode:: bash

    #demo/demo
    ssh demo@quickstart -t impala-shell

    CREATE TABLE default.kudu_test (id INT,random_field STRING, PRIMARY KEY(id)) 
    PARTITION BY HASH PARTITIONS 16 
    STORED AS KUDU

.. note:: 

    The Sink will fail to start if the tables matching the topics do not already exist and the KQL statement is not in auto create mode.

When creating a new Kudu table using Impala, you can create the table as an internal table or an external table.

Internal
~~~~~~~~

An internal table is managed by Impala, and when you drop it from Impala, the data and the table truly are dropped. When you create a 
new table using Impala, it is generally a internal table.

External
~~~~~~~~

An external table (created by CREATE EXTERNAL TABLE) is not managed by Impala, and dropping such a table does not drop the table from 
its source location (here, Kudu). Instead, it only removes the mapping between Impala and Kudu. This is the mode used in the syntax provided by 
Kudu for mapping an existing table to Impala.

See the Impala documentation for more information about internal and external tables. Here's an example:

.. sourcecode:: sql

    CREATE EXTERNAL TABLE default.kudu_test
    STORED AS KUDU
    TBLPROPERTIES ('kudu.table_name'='kudu_test');

Impala Databases and Kudu
~~~~~~~~~~~~~~~~~~~~~~~~~

Every Impala table is contained within a namespace called a database. The default database is called default, and users may 
create and drop additional databases as desired.

.. note::

    When a managed Kudu table is created from within Impala, the corresponding Kudu table will be named ``impala::my_database.table_name``

Tables created by the sink are not automatically visible to Impala. You must map the table in Impala. 

.. sourcecode:: sql

    CREATE EXTERNAL TABLE my_mapping_table
    STORED AS KUDU
    TBLPROPERTIES (
    'kudu.table_name' = 'my_kudu_table'
    );    

Starting the Connector (Distributed)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Download, unpack and install the Stream Reactor and Confluent. Follow the instructions :ref:`here <install>` if you haven't already done so.
All paths in the quickstart are based in the location you installed the Stream Reactor.

Once the Connect has started we can now use the kafka-connect-tools :ref:`cli <kafka-connect-cli>` to post in our distributed properties file for Kudu.
If you are using the :ref:`dockers <dockers>` you will have to set the following environment variable to for the CLI to
connect to the Rest API of Kafka Connect of your container.

.. sourcecode:: bash

   export KAFKA_CONNECT_REST="http://myserver:myport"

.. sourcecode:: bash

    ➜  bin/connect-cli create kudu-sink < conf/kudu-sink.properties
    #Connector name=kudu-sink
    connector.class=com.datamountaineer.streamreactor.connect.kudu.KuduSinkConnector
    tasks.max=1
    connect.kudu.master=quickstart
    connect.kudu.kcql = INSERT INTO impala::default.kudu_test SELECT * FROM kudu-test
    topics=kudu-test
    #task ids: 0


The ``kudu-sink.properties`` file defines:

1.  The name of the sink.
2.  The Sink class.
3.  The max number of tasks the connector is allowed to created. Should not be greater than the number of partitions in
    the Source topics otherwise tasks will be idle.
4.  The Kudu master host.
5.  :ref:`The KCQL routing querying. <kcql>`
6.  The Source kafka topics to take events from.

Use the Confluent CLI to view Connects logs.

.. sourcecode:: bash

    # Get the logs from Connect
    confluent log connect

    # Follow logs from Connect
    confluent log connect -f

We can use the CLI to check if the connector is up but you should be able to see this in logs as-well.

.. sourcecode:: bash

    #check for running connectors with the CLI
    ➜ bin/connect-cli ps
    kudu-sink

.. sourcecode:: bash

    [2016-05-08 22:00:20,823] INFO
        ____        __        __  ___                  __        _
       / __ \____ _/ /_____ _/  |/  /___  __  ______  / /_____ _(_)___  ___  ___  _____
      /  / / / / __ `/ __/ __ `/ /|_/ / __ \/ / / / __ \/ __/ __ `/ / __ \/ _ \/ _ \/ ___/
     / /_/ / /_/ / /_/ /_/ / /  / / /_/ / /_/ / / / / /_/ /_/ / / / / /  __/  __/ /
    /_____/\__,_/\__/\__,_/_/  /_/\____/\__,_/_/ /_/\__/\__,_/_/_/ /_/\___/\___/_/
           __ __          __      _____ _       __
          / //_/_  ______/ /_  __/ ___/(_)___  / /__
         / ,< / / / / __  / / / /\__ \/ / __ \/ //_/
        / /| / /_/ / /_/ / /_/ /___/ / / / / / ,<
       /_/ |_\__,_/\__,_/\__,_//____/_/_/ /_/_/|_|


    by Andrew Stevenson
           (com.datamountaineer.streamreactor.connect.kudu.KuduSinkTask:37)
    [2016-05-08 22:00:20,823] INFO KuduSinkConfig values:
        connect.kudu.master = quickstart
     (com.datamountaineer.streamreactor.connect.kudu.KuduSinkConfig:165)
    [2016-05-08 22:00:20,824] INFO Connecting to Kudu Master at quickstart (com.datamountaineer.streamreactor.connect.kudu.KuduWriter$:33)
    [2016-05-08 22:00:20,875] INFO Initialising Kudu writer (com.datamountaineer.streamreactor.connect.kudu.KuduWriter:40)
    [2016-05-08 22:00:20,892] INFO Assigned topics  (com.datamountaineer.streamreactor.connect.kudu.KuduWriter:42)
    [2016-05-08 22:00:20,904] INFO Sink task org.apache.kafka.connect.runtime.WorkerSinkTask@b60ba7b finished initialization and start (org.apache.kafka.connect.runtime.WorkerSinkTask:155)

Test Records
^^^^^^^^^^^^

Now we need to put some records it to the test_table topics. We can use the ``kafka-avro-console-producer`` to do this.

Start the producer and pass in a schema to register in the Schema Registry. The schema has a ``id`` field of type int
and a ``random_field`` of type string.

.. sourcecode:: bash

    ${CONFLUENT_HOME}/bin/kafka-avro-console-producer \
     --broker-list localhost:9092 --topic kudu-test \
     --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"id","type":"int"},
    {"name":"random_field", "type": "string"}]}'

Now the producer is waiting for input. Paste in the following:

.. sourcecode:: bash

    {"id": 999, "random_field": "foo"}
    {"id": 888, "random_field": "bar"}

Check for records in Kudu
~~~~~~~~~~~~~~~~~~~~~~~~~

Now check the logs of the connector you should see this:

.. sourcecode:: bash

    [2016-05-08 22:09:22,065] INFO
        ____        __        __  ___                  __        _
       / __ \____ _/ /_____ _/  |/  /___  __  ______  / /_____ _(_)___  ___  ___  _____
      / / / / __ `/ __/ __ `/ /|_/ / __ \/ / / / __ \/ __/ __ `/ / __ \/ _ \/ _ \/ ___/
     / /_/ / /_/ / /_/ /_/ / /  / / /_/ / /_/ / / / / /_/ /_/ / / / / /  __/  __/ /
    /_____/\__,_/\__/\__,_/_/  /_/\____/\__,_/_/ /_/\__/\__,_/_/_/ /_/\___/\___/_/
           __ __          __      _____ _       __
          / //_/_  ______/ /_  __/ ___/(_)___  / /__
         / ,< / / / / __  / / / /\__ \/ / __ \/ //_/
        / /| / /_/ / /_/ / /_/ /___/ / / / / / ,<
       /_/ |_\__,_/\__,_/\__,_//____/_/_/ /_/_/|_|


    by Andrew Stevenson
           (com.datamountaineer.streamreactor.connect.kudu.KuduSinkTask:37)
    [2016-05-08 22:09:22,065] INFO KuduSinkConfig values:
        connect.kudu.master = quickstart
     (com.datamountaineer.streamreactor.connect.kudu.KuduSinkConfig:165)
    [2016-05-08 22:09:22,066] INFO Connecting to Kudu Master at quickstart (com.datamountaineer.streamreactor.connect.kudu.KuduWriter$:33)
    [2016-05-08 22:09:22,116] INFO Initialising Kudu writer (com.datamountaineer.streamreactor.connect.kudu.KuduWriter:40)
    [2016-05-08 22:09:22,134] INFO Assigned topics kudu_test (com.datamountaineer.streamreactor.connect.kudu.KuduWriter:42)
    [2016-05-08 22:09:22,148] INFO Sink task org.apache.kafka.connect.runtime.WorkerSinkTask@68496440 finished initialization and start (org.apache.kafka.connect.runtime.WorkerSinkTask:155)
    [2016-05-08 22:09:22,476] INFO Written 2 for kudu_test (com.datamountaineer.streamreactor.connect.kudu.KuduWriter:90)


In Kudu:

.. sourcecode:: bash

    #demo/demo
    ssh demo@quickstart -t impala-shell

    SELECT * FROM kudu_test;

    Query: select * FROM kudu_test
    +-----+--------------+
    | id  | random_field |
    +-----+--------------+
    | 888 | bar          |
    | 999 | foo          |
    +-----+--------------+
    Fetched 2 row(s) in 0.14s

Now stop the connector.

Features
--------

The Kudu Sink writes records from Kafka to Kudu.

The Sink supports:

1.  Field selection - Kafka topic payload field selection is supported, allowing you to select fields written to Kudu.
2.  Topic to table routing.
3.  Auto table create with HASH partition strategy by using DISTRIBUTE BY with configurable buckets. Tables that are autocreated 
    are not immediately visible in Impala. You must map them in Impala.
4.  Auto evolution of tables.
5.  Error policies for handling failures.

Kafka Connect Query Language
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**K** afka **C** onnect **Q** uery **L** anguage found here `GitHub repo <https://github.com/datamountaineer/kafka-connector-query-language>`_
allows for routing and mapping using a SQL like syntax, consolidating typically features in to one configuration option.

.. note::

    If tables are created by Impala suffix ``impala::database_name`` to the target table.

The Kudu Sink supports the following:

.. sourcecode:: bash

    <write mode> INTO [impala]:[database].<target table> SELECT <fields> FROM <source topic> <AUTOCREATE> <AUTOEVOLVE> <DISTRIBUTEBY> <PK_FIELDS> INTO <NBR_OF_BUCKETS> BUCKETS

Example:

.. sourcecode:: sql

    #Insert mode, select all fields from topicA and write to tableA
    INSERT INTO tableA SELECT * FROM topicA

    #Insert mode, select 3 fields and rename from topicB and write to tableB
    INSERT INTO tableB SELECT x AS a, y AS b and z AS c FROM topicB

    #Upsert mode, select all fields from topicC, auto create tableC and auto evolve, use field1 and field2 as the primary keys
    UPSERT INTO tableC SELECT * FROM topicC AUTOCREATE  DISTRIBUTEBY field1, field2 INTO 10 BUCKETS AUTOEVOLVE

Error Polices
~~~~~~~~~~~~~

The Sink has three error policies that determine how failed writes to the target database are handled. The error policies
affect the behaviour of the schema evolution characteristics of the Sink. See the schema evolution section for more
information.

**Throw**

Any error on write to the target database will be propagated up and processing is stopped. This is the default
behaviour.

**Noop**

Any error on write to the target database is ignored and processing continues.

.. warning::

    This can lead to missed errors if you don't have adequate monitoring. Data is not lost as it's still in Kafka
    subject to Kafka's retention policy. The Sink currently does **not** distinguish between integrity constraint
    violations and or other expections thrown by drivers.

**Retry**

Any error on write to the target database causes the RetryIterable exception to be thrown. This causes the
Kafka connect framework to pause and replay the message. Offsets are not committed. For example, if the table is offline
it will cause a write failure, the message can be replayed. With the Retry policy the issue can be fixed without stopping
the sink.

The length of time the Sink will retry can be controlled by using the ``connect.kudu.max.retries`` and the
``connect.kudu.retry.interval``.

Auto conversion of Connect records to Kudu
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Sink automatically converts incoming Connect records to Kudu inserts or upserts.

Topic Routing
~~~~~~~~~~~~~

The Sink supports topic routing that allows mapping the messages from topics to a specific table. For example, map a
topic called "bloomberg_prices" to a table called "prices". This mapping is set in the ``connect.kudu.kcql``
option.

Example:

.. sourcecode:: sql

    //Select all
    INSERT INTO table1 SELECT * FROM topic1; INSERT INTO tableA SELECT * FROM topicC

Field Selection
~~~~~~~~~~~~~~~

The Kudu Sink supports field selection and mapping. This mapping is set in the ``connect.kudu.kcql`` option.

Examples:

.. sourcecode:: sql

    //Rename or map columns
    INSERT INTO table1 SELECT lst_price AS price, qty AS quantity FROM topicA

    //Select all
    INSERT INTO table1 SELECT * FROM topic1

.. tip:: Check you mappings to ensure the target columns exist.

.. warning::

    Field selection disables evolving the target table if the upstream schema in the Kafka topic changes. By specifying
    field mappings it is assumed the user is not interested in new upstream fields. For example they may be tapping into a
    pipeline for a Kafka stream job and not be intended as the final recipient of the stream.

    If you chose field selection you must include the primary key fields otherwise the insert will fail.

Write Modes
~~~~~~~~~~~

The Sink supports both **insert** and **upsert** modes.  This mapping is set in the ``connect.kudu.kcql`` option.

**Insert**

Insert is the default write mode of the sink.

**Insert Idempotency**

Kafka currently provides at least once delivery semantics. Therefore, this mode may produce errors if unique constraints
have been implemented on the target tables. If the error policy has been set to NOOP then the Sink will discard the error
and continue to process, however, it currently makes no attempt to distinguish violation of integrity constraints from other
exceptions such as casting issues.

**Upsert**

The Sink support Kudu upserts which replaces the existing row if a match is found on the primary keys.

**Upsert Idempotency**

Kafka currently provides at least once delivery semantics and order is a guaranteed within partitions.

This mode will, if the same record is delivered twice to the sink, result in an idempotent write. The existing record
will be updated with the values of the second which are the same.

If records are delivered with the same field or group of fields that are used as the primary key on the target table,
but different values, the existing record in the target table will be updated.

Since records are delivered in the order they were written per partition the write is idempotent on failure or restart.
Redelivery produces the same result.

Auto Create Tables
~~~~~~~~~~~~~~~~~~

The Sink supports auto creation of tables for each topic. This mapping is set in the ``connect.kudu.kcql`` option.

Primary keys are set in the ``DISTRIBUTEBY`` clause of the ``connect.kudu.kcql``.

Tables are created with the Kudu hash partition strategy. The number of buckets must be specified in the ``kcql``
statement.

.. sourcecode:: sql

    #AutoCreate the target table
    INSERT INTO table1 SELECT * FROM topic AUTOCREATE DISTRIBUTEBY field1, field2 INTO 10 BUCKETS

.. note::

    The fields specified as the primary keys (distributeby) must be in the SELECT clause or all fields must be selected

The Sink will try and create the table at start up if a schema for the topic is found in the Schema Registry. If no
schema is found the table is created when the first record is received for the topic.

.. note::

    Tables that are created are not visible to Impala. You must map them in Impala yourself.

Auto Evolve Tables
~~~~~~~~~~~~~~~~~~

The Sink supports auto evolution of tables for each topic. This mapping is set in the ``connect.kudu.kcql`` option.
When set the Sink will identify new schemas for each topic based on the schema version from the Schema registry. New columns
will be identified and an alter table DDL statement issued against Kudu.

Schema evolution can occur upstream, for example any new fields or change in data type in the schema of the topic, or
downstream DDLs on the database.

Upstream changes must follow the schema evolution rules laid out in the Schema Registry. This Sink only supports BACKWARD
and FULLY compatible schemas. If new fields are added the Sink will attempt to perform a ALTER table DDL statement against
the target table to add columns. All columns added to the target table are set as nullable.

Fields cannot be deleted upstream. Fields should be of Avro union type [null, <dataType>] with a default set. This allows
the Sink to either retrieve the default value or null. The Sink is not aware that the field has been deleted
as a value is always supplied to it.

.. warning::

    If a upstream field is removed and the topic is not following the Schema Registry's evolution rules, i.e. not full
    or backwards compatible, any errors will default to the error policy.

Downstream changes are handled by the sink. If columns are removed, the mapped fields from the topic are ignored. If
columns are added, we attempt to find a matching field by name in the topic.

Error Polices
~~~~~~~~~~~~~

The Sink has three error policies that determine how failed writes to the target database are handled. The error policies
affect the behaviour of the schema evolution characteristics of the sink. See the schema evolution section for more
information.

**Throw**

Any error on write to the target database will be propagated up and processing is stopped. This is the default
behaviour.

**Noop**

Any error on write to the target database is ignored and processing continues.

.. warning::

    This can lead to missed errors if you don't have adequate monitoring. Data is not lost as it's still in Kafka
    subject to Kafka's retention policy. The Sink currently does **not** distinguish between integrity constraint
    violations and or other expections thrown by drivers..

**Retry**

Any error on write to the target database causes the RetryIterable exception to be thrown. This causes the
Kafka connect framework to pause and replay the message. Offsets are not committed. For example, if the table is offline
it will cause a write failure, the message can be replayed. With the Retry policy the issue can be fixed without stopping
the sink.

The length of time the Sink will retry can be controlled by using the ``connect.kudu.max.retries`` and the
``connect.cassandra.retry.interval``.

Data Type Mappings
~~~~~~~~~~~~~~~~~~

+------------------+------------------+
| Connect Type     | Kudu Data Type   |
+==================+==================+
| INT8             | INT8             |
+------------------+------------------+
| INT16            | INT16            |
+------------------+------------------+
| INT32            | INT32            |
+------------------+------------------+
| INT64            | INT64            |
+------------------+------------------+
| BOOLEAN          | BOOLEAN          |
+------------------+------------------+
| FLOAT32          | FLOAT            |
+------------------+------------------+
| FLOAT64          | FLOAT            |
+------------------+------------------+
| BYTES            | BINARY           |
+------------------+------------------+

Configurations
--------------

``connect.kudu.master``

Specifies a Kudu server.

* Data type : string
* Importance: high
* Optional  : no

``connect.kudu.kcql``

Kafka connect query language expression. Allows for expressive topic to table routing, field selection and renaming.

Examples:

.. sourcecode:: sql

    INSERT INTO impala::default.TABLE1 SELECT * FROM TOPIC1;INSERT INTO TABLE2 SELECT field1, field2, field3 as renamedField FROM TOPIC2

* Data Type: string
* Importance: high
* Optional : no

``connect.kudu.write.flush.mode``

Flush mode on write.

1.  SYNC - flush each sink record. Batching is disabled.
2.  BATCH_BACKGROUND - flush batch of sink records in background thread.
3.  BATCH_SYNC - flush batch of sink records.

* Data Type: string
* Importance: medium
* Optional : yes
* Default: SYNC

``connect.kudu.mutation.buffer.space``

Kudu Session mutation buffer space.

* Data Type: int
* Importance: low
* Optional : yes
* Default: 10000

``connect.kudu.error.policy``

Specifies the action to be taken if an error occurs while inserting the data.

There are three available options, **noop**, the error is swallowed, **throw**, the error is allowed to propagate and retry.
For **retry** the Kafka message is redelivered up to a maximum number of times specified by the ``connect.kudu.max.retries``
option. The ``connect.kudu.retry.interval`` option specifies the interval between retries.

The errors will be logged automatically.

* Type: string
* Importance: high
* Optional : yes
* Default: RETRY

``connect.kudu.max.retries``

The maximum number of times a message is retried. Only valid when the ``connect.kudu.error.policy`` is set to ``retry``.

* Type: string
* Importance: medium
* Optional : yes
* Default: 10


``connect.kudu.retry.interval``

The interval, in milliseconds between retries if the Sink is using ``connect.kudu.error.policy`` set to **RETRY**.

* Type: int
* Importance: medium
* Optional : yes
* Default : 60000 (1 minute)

``connect.kudu.schema.registry.url``

The url for the Schema registry. This is used to retrieve the latest schema for table creation.

* Type : string
* Importance : high
* Optional : yes
* Default : http://localhost:8081

``connect.progress.enabled``

Enables the output for how many records have been processed.

* Type: boolean
* Importance: medium
* Optional: yes
* Default : false


Example
~~~~~~~

.. sourcecode:: bash

    name=kudu-sink
    connector.class=com.datamountaineer.streamreactor.connect.kudu.KuduSinkConnector
    tasks.max=1
    connect.kudu.master=quickstart
    connect.kudu.kcql=INSERT INTO impala::default.kudu_test SELECT * FROM kudu_test AUTOCREATE DISTRIBUTEBY id INTO 5 BUCKETS
    topics=kudu-test
    connect.kudu.schema.registry.url=http://myhost:8081

Deployment Guidelines
---------------------

Distributed Mode
~~~~~~~~~~~~~~~~

Connect, in production should be run in distributed mode. 

1.  Install the Confluent Platform on each server that will form your Connect Cluster.
2.  Create a folder on the server called ``plugins/streamreactor/libs``.
3.  Copy into the folder created in step 2 the required connector jars from the stream reactor download.
4.  Edit ``connect-avro-distributed.properties`` in the ``etc/schema-registry`` folder where you installed Confluent
    and uncomment the ``plugin.path`` option. Set it to the path you deployed the stream reactor connector jars
    in step 2.
5.  Start Connect, ``bin/connect-distributed etc/schema-registry/connect-avro-distributed.properties``

Connect Workers are long running processes so set an ``init.d`` or ``systemctl`` service accordingly.

Connector configurations can then be push to any of the workers in the Cluster via the CLI or curl, if using the CLI 
remember to set the location of the Connect worker you are pushing to as it defaults to localhost.

.. sourcecode:: bash

    export KAFKA_CONNECT_REST="http://myserver:myport"

Kubernetes
~~~~~~~~~~

Helm Charts are provided at our `repo <https://datamountaineer.github.io/helm-charts/>`__, add the repo to your Helm instance and install. We recommend using the Landscaper
to manage Helm Values since typically each Connector instance has it's own deployment.

Add the Helm charts to your Helm instance:

.. sourcecode:: bash

    helm repo add datamountaineer https://datamountaineer.github.io/helm-charts/


TroubleShooting
---------------

Please review the :ref:`FAQs <faq>` and join our `slack channel <https://slackpass.io/datamountaineers>`_.

