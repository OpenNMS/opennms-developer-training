# System Architecture

## Goal

Understand the function of the different services in the OpenNMS platform and how they can be composed.

## Components

The following comprehensive architecture diagram shows all of the components deployed in a common enterprise environment:

![comprehensive](images/arch-comprehensive.png)

### OpenNMS Services

#### OpenNMS Core

The primary OpenNMS instance (or core) manages configuration, scheduling, the event bus, state management, maintenance, etc.

Most of the magic happens here.

#### OpenNMS Web

An instance of OpenNMS configured to help offload web traffic and API calls from the core.

The web application and REST APIs are limited to read-only calls.

> It must be configured in a special way to play nicely alongside the core, and there are several limitations in the UI.

#### Minion

A Minion is an instance of the [Karaf OSGi runtime](https://karaf.apache.org/) that enables OpenNMS to monitor devices and services in locations that OpenNMS cannot reach.
Minions communicate with these remote devices while OpenNMS performs coordination and task delegation.

Can be used to scale processing of inbound messages.

For active checks where the Minion must initiate the connection, the scheduling is done on the core.
Checks are triggered on the Minion via a remote procedure call (RPC).

#### Sentinel

Used to scale and offload processing of specific workloads from the core.

Currently supports processing streaming telemetry, thresholding on streaming telemetry, and flows.

> Unlike Minion, we assume that Sentinel has access to the database and other services used by the core.

### Supporting Services - Storage

#### PostgreSQL

PostgreSQL is the primary data store used for inventory, topology, events, alarms, etc.

The schema is managed using [Liquibase changelogs](https://github.com/OpenNMS/opennms/tree/opennms-26.2.1-1/core/schema/src/main/liquibase) which are applied when running `$OPENNMS_HOME/bin/install -dis`.

![db schema](images/arch-db-schema.png)

We depend specifically on PostgreSQL and are not compatible with other relational databases.

Horizon 26.2.2 requires PostgreSQL 10.x - 12.x.

#### Elasticsearch

Elasticsearch stores flows and archives both events & alarms.

The following templates are used:
1. [Flows](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/features/flows/elastic/src/main/resources/netflow-template.json)
1. [Events](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/features/opennms-es-rest/src/main/resources/eventsIndexTemplate.es7.json)
1. [Alarms](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/features/alarms/history/elastic/src/main/resources/alarm-template.es7.json)

For flows, we maintain an aggregation plugin called the [opennms-drift-plugin](https://github.com/OpenNMS/elasticsearch-drift-plugin) that extends Elasticsearch's aggregation functionality and provides more accurate calculations for flow records.
With this plugin, flow records are treated as ranges, with a start and end time, as opposed to a single point in time.
The values are distributed evenly over the range, assuming a fixed rate over time.

#### ScyllaDB or Cassandra

ScyllaDB or Cassandra are used when storing time series data with [Newts](https://opennms.github.io/newts/).

The following schemas are used:
1. [Samples](https://github.com/OpenNMS/newts/blob/1.5.2/cassandra/storage/src/main/resources/samples_schema.cql)
1. [Search/Indexing](https://github.com/OpenNMS/newts/blob/1.5.2/cassandra/search/src/main/resources/search_schema.cql)

See [Sizing of Cassandra for Newts](https://opennms.discourse.group/t/sizing-cassandra-for-newts/771) for details on sizing, tuning, and schema recommendations.

### Supporting Services - Messaging

#### Kafka

Kafka can be used as a messaging platform to facilitate communication between OpenNMS and Minion.

OpenNMS and Minion both perform simultaneously as producers and consumers on different topics to send and receive messages.
Details about inter-process communication (IPC) is covered in the IPC module.

#### ActiveMQ

ActiveMQ can be used as a message bus between OpenNMS and Minion.

An instance of ActiveMQ is embedded in the OpenNMS JVM to facilitate setup on small installs.
We generally recommend an external ActiveMQ server/cluster or an alternative such as Kafka for large installs.

### Supporting Services - Other

#### Grafana

The [OpenNMS Helm Plugin](https://github.com/OpenNMS/opennms-helm) provides data sources and panels for Grafana to visualize performance, fault, and flow data from OpenNMS.

Grafana can also be used to aggregate data from distinct OpenNMS systems and provide a single pane of glass.

#### Nginx

Nginx is commonly used as a reverse proxy in front of OpenNMS to perform TLS termination and SSO validation.

#### Flink

Flink is used as a distributed processing engine to deploy [Nephron](https://github.com/OpenNMS/nephron) in environments that process tens or hundreds of thousands of flows per second.

Nephron performs aggregation of raw flow records and generates summary statistics which greatly reduce the storage and compute requirements of the Elasticsearch cluster.

## Reference Architectures

### Simple

In the simplest form OpenNMS only requires PostgreSQL:

![simple arch](images/arch-simple.png)

### Minion w/ Kafka

We can introduce Minion and Kafka if we want to monitor remote networks:

![minion kafka arch](images/arch-minion-kafka.png)

### Comprehensive

Here's a comprehensive reference architecture that includes all of the components:

![comprehensive arch](images/arch-comprehensive.png)

### ALEC Distributed

Here's another architecture view for a distributed deployment of [ALEC](https://alec.opennms.com/) (Architecture for Learning Enabled Correlation), a framework that uses artificial intelligence and machine learning to group related alarms: 

![alec](images/arch-alec-distributed.png)

> ALEC was originally called OCE.

## Lab

> Here we show how to stand up a stack with some Minions and sample data

Start services:
```
$ git clone -b H26.2.2 https://github.com/opennms-forge/opennms-demo.git
$ cd opennms-demo
$ docker-compose up -d
```

Initialize:
```
$ cd setup
$ ./configure.sh
```

Access Web UI via: http://localhost:8980

Access Grafana via: http://localhost:3000

Tear down:
```
$ docker-compose down
```
