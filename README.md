# OpenNMS Developer Training

These guides were built to give you the knowledge you need to set up a development environment and start developing new features with OpenNMS’ APIs.

## Work in progress

This documentation is actively being worked on.
It is intended to be used as support material for instructor led training sessions.

## Target Audience

You are familiar with Java, Docker and have some existing experience using OpenNMS.

You would like to integrate with OpenNMS and extend it to provide functionality that meets your specific business requirements.

## OpenNMS Details

* Horizon 26.2.2 (or Meridian 2020.1.0)
* Java 8 compile time, Java 11 runtime
* PostgreSQL 12

## Modules

Some more are complete, whereas others contain nothing but notes:
1. :sunny: - ready
1. :fire: - work in progress
1. :cloud: - brainstorming

### Foundation

Base concepts & developments

1. [System Architecture](docs/foundation/01-system-architecture.md) :sunny:
1. [Local development](docs/foundation/02-local-development.md) :sunny:
1. [JVM Architecture](docs/foundation/03-jvm-architecture.md) :sunny:
1. [Inter Process Communication (IPC)](docs/foundation/04-ipcs.md) :cloud:

### APIs

Interfacing with OpenNMS via supported APIs

1. [REST API](docs/apis/01-rest-api.md) :cloud:
1. [OpenNMS Integration API (OIA) Plugins](docs/apis/02-integration-api.md) :cloud:
1. [Kafka Northbound & Southbound](docs/apis/03-kafka-api.md) :cloud:
1. [OpenNMS.js](docs/apis/04-opennms-js.md) :cloud:

### Model & Lifecycle

Data models and internals

1. [Inventory](docs/model/01-inventory.md) :cloud:
1. [Events](docs/model/02-events.md) :cloud: 
1. [Alarms](docs/model/03-alarms.md) :cloud:
1. [Time Series](docs/model/04-timeseries.md) :cloud:
1. [Flows](docs/model/05-flows.md) :fire:
1. [Links](docs/model/06-links.md) :cloud:

### Use Cases

Extensions to the platform

1. [SDN integration w/ OpenDaylight](docs/usecases/01-opendaylight.md) :cloud:
1. [Correlation with ALEC](docs/usecases/02-alec.md) :cloud:
1. [Notification w/ PagerDuty](docs/usecases/03-pagerduty.md)  :cloud:
1. [Time Series w/ Cortex](docs/usecases/04-cortex.md) :cloud:
