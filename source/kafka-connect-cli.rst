.. _cli:

.. image:: https://img.shields.io/badge/latest%20release-v0.4-blue.svg?label=maven%20latest%20release
    :target: http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22kafka-connect-cli%22
.. image:: ../images/git.png
    :target: https://github.com/datamountaineer/kafka-connect-tools/releases/tag/v0.5

Kafka Connect CLI
=================

This is a tiny command line interface (CLI) around the `Kafka Connect REST Interface
<http://docs.confluent.io/3.0.1/connect/userguide.html#rest-interface>`__
to manage connectors. It is used in a git like fashion where the first program argument indicates the command: it can be one of
``[ps|get|rm|create|run]``.

The CLI is meant to behave as a good unix citizen: input from ``stdin``; output to ``stdout``; out of band info to ``stderr`` and non-zero exit
status on error. Commands dealing with configuration expect or producedata in .properties style: ``key=value`` lines and comments start with a
``#``.

.. sourcecode:: bash

    kafka-connect-cli 0.5
    Usage: kafka-connect-cli [ps|get|rm|create|run|status] [options] [<connector-name>]

      --help
            prints this usage text
      -e <value> | --endpoint <value>
            Kafka Connect REST URL, default is http://localhost:8083/

    Command: ps
    list active connectors names.

    Command: get
    get the configuration of the specified connector.

    Command: rm
    remove the specified connector.

    Command: create
    create the specified connector with the .properties from stdin; the connector cannot already exist.

    Command: run
    create or update the specified connector with the .properties from stdin.

    Command: status
    get connector and it's task(s) state(s).

      <connector-name>
            connector name

You can override the default endpoint by setting an environment variable `KAFKA_CONNECT_REST` i.e.

    export KAFKA_CONNECT_REST="http://myserver:myport"

Requirements
------------

-  Java 1.8

To Build
--------

.. sourcecode:: bash

    gradle fatJar

Usage
-----

Clone this repository, do a ``mvn package`` and run the jar in a way you
prefer, for example with the provided ``cli`` shell script. The CLI can
be used as follows.

Get Active Connectors
~~~~~~~~~~~~~~~~~~~~~

Command: ``ps``

Example:

.. sourcecode:: bash

    $ ./cli ps
    twitter-source

Get Connector Information
~~~~~~~~~~~~~~~~~~~~~~~~~

Command: ``get``

Example:

.. sourcecode:: bash

    $ ./cli get twitter-source
    #Connector `twitter-source`:
    name=twitter-source
    tasks.max=1

    (snip)

    track.terms=test
    #task ids: 0

Delete a Connector
~~~~~~~~~~~~~~~~~~

Command: ``rm``

Example:

.. sourcecode:: bash

    $ ./cli rm twitter-source

Create a New Connector
~~~~~~~~~~~~~~~~~~~~~~

The connector cannot already exist.

Command: ``create``

Example:

.. sourcecode:: bash

    $ ./cli create twitter-source <twitter.properties
    #Connector `twitter-source`:
    name=twitter-source
    tasks.max=1

    (snip)

    track.terms=test
    #task ids: 0

Create or Update a Connector
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Either starts a new connector if it did not exist, or update an existing
connector.

Command: ``run``

Example:

.. sourcecode:: bash

    $ ./cli run twitter-source <twitter.properties
    #Connector `twitter-source`:
    name=twitter-source
    tasks.max=1

    (snip)

    track.terms=test
    #task ids: 0


