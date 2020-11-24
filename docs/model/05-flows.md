#  Flows

## Goal

Understand how flows work in OpenNMS

## Flows

Network flows are summaries of IP traffic that can be used to help understand bandwidth usage and patterns.

Flows come in many flavors including Netflow v5, Netflow v9, IPFIX and sFlow.

OpenNMS is able to decode these protocols and reduce the flow representations to a canonical model: [flowdocument.proto](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/features/flows/kafka-persistence/src/main/proto/flowdocument.proto#L47)

OpenNMS is able to enrich flows with inventory information, MIB-2 data, and application tags.

Application tags are computed from classification rules which can be configured to match IP addresses, port numbers or flow exporters.
A default set of rules are provided to match common and known applications.

> The classification rules allow you to define your own applications i.e. if you see traffic on this port, between these hosts, at this location, then tag the flows with the "Nightly-Backup" application.

Flows processing is built on top of telemetryd, which means that they can be supported in several different configurations based on system requirements and flow volume.

Flows are stored exclusively in Elasticsearch.

OpenNMS provides a REST API that can be used to retrieve statistics from flow records.
The REST API performs asynchronous queries to Elasticsearch and offloads the computation to the Elasticsearch engine.

Grafana and the OpenNMS Helm plugin are used for visualizing flows.

Flows records tend to have high cardinality (a lot of unique IP addresses and ports) and queries can get resource intensive for large datasets.
In order to pre-compute the statistics and reduce the data rentention requirements, the [Nephron](https://github.com/OpenNMS/nephron) stream processing pipeline can be used.

Nephron is built on [Apache Beam](https://beam.apache.org/) which allows the pipeline to be run on various engines including [Apache Flink](https://flink.apache.org/).

## How does it work?

Listener -> Parser -> Queue -> Sink API
* Look at topic
* Look at payload

Things to cover:
1. Sink API
1. Sentinel
1. Adapter
1. Proto on Kafka
1. Nephron
1. Elasticsearch
1. REST API
1. Helm

## Lab

TODO: Synthetic Flows
* Generate some known flow pattern using Riptide
* git clone https://github.com/opennms-forge/opennms-riptide

TODO: Should be a wget of a jar

TODO: Also push time series via Graphite

TODO: Be able to do this over the opennms-demo environment
