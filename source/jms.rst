Kafka Connect JMS Sink
======================

The JMS Sink connector allows you to extract entries from a Kafka topic with the CQL driver and pass them to a JMS topic/queue.
The connector allows you to specify the payload type sent to the JMS target:

1. JSON
2. AVRO
3. MAP
4. OBJECT

The Sink supports:

1. :ref:`The KCQL routing querying <kcql>`. Kafka topic payload field selection is supported, allowing you to select fields written to the queue or topic in JMS.
2. Topic to topic routing via KCQL.
3. Payload format selection via KCQL.
4. Error policies for handling failures.
5. Payload support for Schema.Struct and payload Struct, Schema.String and Json payload and Json payload with no schema.

The Sink supports three Kafka payloads type for TextMessage (Format JSON) only:

.. note::

    Only support with used with KCQL format type JSON. This sends messages at TextMessages to the JMS destination.

**Connect entry with Schema.Struct and payload Struct.** If you follow the best practice while producing the events, each
message should carry its schema information. Best option is to send Avro. Your connect configurations should be set to
``value.converter=io.confluent.connect.avro.AvroConverter``.
You can find an example `here <https://github.com/confluentinc/kafka-connect-blog/blob/master/etc/connect-avro-standalone.properties>`__.
To see how easy is to have your producer serialize to Avro have a look at
`this <http://docs.confluent.io/3.0.1/schema-registry/docs/serializer-formatter.html?highlight=kafkaavroserializer>`__.
This requires the SchemaRegistry which is open source thanks to Confluent! Alternatively you can send Json + Schema.
In this case your connect configuration should be set to ``value.converter=org.apache.kafka.connect.json.JsonConverter``. This doesn't
require the SchemaRegistry.

**Connect entry with Schema.String and payload json String.** Sometimes the producer would find it easier, despite sending
Avro to produce a GenericRecord, to just send a message with Schema.String and the json string.

**Connect entry without a schema and the payload json String.** There are many existing systems which are publishing json
over Kafka and bringing them in line with best practices is quite a challenge. Hence we added the support.

Prerequisites
-------------
- Confluent 3.3
- Java 1.8
- Scala 2.11
- A JMS framework (ActiveMQ for example)

Setup
-----

Before we can do anything, including the QuickStart we need to install the Confluent platform.
For ActiveMQ follow http://activemq.apache.org/getting-started.html for the instruction of setting
it up.


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

Starting the Connector (Distributed)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Download, unpack and install the Stream Reactor and Confluent. Follow the instructions :ref:`here <install>` if you haven't already done so.
All paths in the quickstart are based in the location you installed the Stream Reactor.

Once the Connect has started we can now use the kafka-connect-tools :ref:`cli <kafka-connect-cli>` to post in our distributed properties file for JMS.
If you are using the :ref:`dockers <dockers>` you will have to set the following environment variable to for the CLI to
connect to the Rest API of Kafka Connect of your container.

.. sourcecode:: bash

   export KAFKA_CONNECT_REST="http://myserver:myport"

.. sourcecode:: bash

    ➜  bin/connect-cli create jms-sink < conf/jms-sink.properties

The ``jms-sink.properties`` file defines:

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
    jms-sink


Test Records
^^^^^^^^^^^^

Now we need to put some records it to the test_table topics. We can use the ``kafka-avro-console-producer`` to do this.

Start the producer and pass in a schema to register in the Schema Registry. The schema has a ``id`` field of type int
and a ``random_field`` of type string.

.. sourcecode:: bash

    ${CONFLUENT_HOME}/bin/kafka-avro-console-producer \
     --broker-list localhost:9092 --topic jms_test \
     --property value.schema='{"type":"record","name":"User",
    "fields":[{"name":"firstName","type":"string"},{"name":"lastName","type":"string"},{"name":"age","type":"int"},{"name":"salary","type":"double"}]}'

Now the producer is waiting for input. Paste in the following:

.. sourcecode:: bash

    {"firstName": "John", "lastName": "Smith", "age":30, "salary": 4830}
    {"firstName": "Anna", "lastName": "Jones", "age":28, "salary": 5430}

Now check for records in ActiveMQ.

Now stop the connector.

Features
--------

The Sink supports:

1. Field selection - Kafka topic payload field selection is supported, allowing you to select fields written to the queue or topic in JMS.
2. Topic to JMS Destination routing.
3. Payload format selection.
4. Error policies for handling failures.
5. Payload support for Schema.Struct and payload Struct, Schema.String and Json payload and Json payload with no schema. Only supported when storing as JSON

Kafka Connect Query Language
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**K** afka **C** onnect **Q** uery **L** anguage found here `GitHub repo <https://github.com/datamountaineer/kafka-connector-query-language>`_
allows for routing and mapping using a SQL like syntax, consolidating typically features in to one configuration option.

The JMS Sink supports the following:

.. sourcecode:: bash

    INSERT INTO <jms target> SELECT <fields> FROM <source topic> STOREAS <AVRO|JSON|MAP|OBJECT> WITHTYPE <TOPIC|QUEUE>

Example:

.. sourcecode:: sql

    #select all fields from topicA and write to jmsA queue
    INSERT INTO jmsA SELECT * FROM topicA WITHTYPE QUEUE

    #select 3 fields and rename from topicB and write to jmsB topic as JSON in a TextMessage
    INSERT INTO jmsB SELECT x AS a, y AS b and z AS c FROM topicB STOREAS JSON WITHTYPE TOPIC


JMS Payload
~~~~~~~~~~~

When a message is sent to a JMS target it can be one of the following:

1.  JSON -   Send a TextMessage;
2.  AVRO -   Send a BytesMessage;
3.  MAP -    Send a MapMessage;
4.  OBJECT - Send an ObjectMessage

Topic Routing
~~~~~~~~~~~~~

The Sink supports topic routing that allows mapping the messages from topics to a specific jms target. For example, map a
topic called "bloomberg_prices" to a jms target named "prices". This mapping is set in the ``connect.jms.kcql``
option.

Example:

.. sourcecode:: sql

    //Select all
    INSERT INTO prices SELECT * FROM bloomberg_prices; INSERT INTO jms3 SELECT * FROM topic2

Configurations
--------------

``connect.jms.url``

Provides the JMS broker url

* Data Type: string
* Importance: high
* Optional : no

``connect.jms.username``

Provides the user for the JMS connection.

* Data Type: string
* Importance: high
* Optional : no

``connect.jms.password``

Provides the password for the JMS connection.

* Data Type: string
* Importance: high
* Optional : no

``connect.jms.initial.context.factory``

* Data Type: string
* Importance: high
* Optional: no

Initial Context Factory, e.g: org.apache.activemq.jndi.ActiveMQInitialContextFactory.

``connect.jms.connection.factory``

The ConnectionFactory implementation to use.

* Data Type: string
* Importance: high
* Optional : no

``connect.jms.destination.selector``

* Data Type: String
* Importance: high
* Optional: no
* Default: CDI

Selector to use for destination lookup. Either CDI or JNDI.

``connect.jms.initial.context.extra.params``

* Data Type: String
* Importance: high
* Optional: yes

List (comma separated) of extra properties as key/value pairs with a colon delimiter to supply to the initial context e.g. SOLACE_JMS_VPN:my_solace_vp.

``connect.jms.kcql``

KCQL expression describing field selection and routes. The kcql expression also handles setting the JMS destination type, i.e. TOPIC or
QUEUE via the ``withtype`` keyword.

* Data Type: string
* Importance: high
* Optional : no

``connect.jms.error.policy``

Specifies the action to be taken if an error occurs while inserting the data.

There are three available options, **noop**, the error is swallowed, **throw**, the error is allowed to propagate and retry.
For **retry** the Kafka message is redelivered up to a maximum number of times specified by the ``connect.jms.max.retries``
option. The ``connect.jms.retry.interval`` option specifies the interval between retries.

The errors will be logged automatically.

* Type: string
* Importance: medium
* Optional: yes
* Default: RETRY

``connect.jms.max.retries``

The maximum number of times a message is retried. Only valid when the ``connect.jms.error.policy`` is set to ``retry``.

* Type: string
* Importance: medium
* Optional: yes
* Default: 10

``connect.jms.retry.interval``

The interval, in milliseconds between retries if the Sink is using ``connect.jms.error.policy`` set to **RETRY**.

* Type: int
* Importance: medium
* Optional: yes
* Default : 60000 (1 minute)

``connect.progress.enabled``

Enables the output for how many records have been processed.

* Type: boolean
* Importance: medium
* Optional: yes
* Default : false

Schema Evolution
----------------

Not applicable.

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


