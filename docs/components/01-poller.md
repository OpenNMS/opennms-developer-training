# Poller

## Goal

Understand how the poller daemon works.

## Overview

The poller daemon is responsible for scheduling, executing and tracking the results of monitors.
Monitors are used to test, or evaluate, services and determine whether or not they are available.
When services are unavailable, outage records are created and events are sent to trigger alarms.

## Startup

The poller daemon starts by [initializing the LegacyScheduler](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/opennms-services/src/main/java/org/opennms/netmgt/poller/Poller.java#L338) using the configured number of threads.

> The `LegacyScheduler` is used by many daemons to schedule and run tasks synchronously against a thread pool.
Tasks are split into different queues based on their execution delay and a main thread iterates over these and executes as necessary.
Tasks are not automatically added back to the queues once executed, it is up to the tasks to reschedule themselves.

The poller daemon then proceeds to close outages for any services that are no longer managed (configuration may have been changed since the last start) and then schedules the polls for all managed services.
Services are initially scheduled with a interval/timeout of 0, meaning they are candidates for immediate execution - see [Schedule.java#L130](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/core/daemon/src/main/java/org/opennms/netmgt/scheduler/Schedule.java#L130).
The service is subsequently scheduled at the configured interval.

## A Network

The poller daemon use a concept of a "network". The network contains nodes. Nodes contain interfaces and interfaces container services.

These classes are maintained in the `org.opennms.netmgt.poller.pollables` package, for reference see:
 * [PollableNetwork](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/opennms-services/src/main/java/org/opennms/netmgt/poller/pollables/PollableNetwork.java)
 * [PollableNode](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/opennms-services/src/main/java/org/opennms/netmgt/poller/pollables/PollableNode.java)
 * [PollableInterface](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/opennms-services/src/main/java/org/opennms/netmgt/poller/pollables/PollableInterface.java)
 * [PollableService](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/opennms-services/src/main/java/org/opennms/netmgt/poller/pollables/PollableService.java)

These objects are modeled in this hierarchical fashion to help implement "node outage processing" and the related "critical service" functionality. See [Service Assurance: Critical Service](https://docs.opennms.org/opennms/releases/27.1.1/guide-admin/guide-admin.html#ga-service-assurance-critical-service) for details.

## Polling

When a service is ready to be polled, the `PollableService` will attempt to acquire a lock on the node - see [withTreeLock](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/opennms-services/src/main/java/org/opennms/netmgt/poller/pollables/PollableService.java#L405).
When acquired, this lock is held for the duration of the poll (until the monitor is done executing and the results have been returned) and no other services on the node can be polled.
This means that only a single service can be polled on a given at any given time.
If the service is unable to obtain the lock before the delay elapses, the service is rescheduled in the future after some short random delay.

> Locks are disabled and no longer required when "node outage processing" is disabled.

Once the lock has been acquired, the service executes the poll via the `LocationAwarePollerClient` - see [PollableServiceConfig.java#L127](https://github.com/OpenNMS/opennms/blob/opennms-26.2.2-1/opennms-services/src/main/java/org/opennms/netmgt/poller/pollables/PollableServiceConfig.java#L127).
The `LocationAwarePollerClient` returns a future while the monitor is executed asynchronously, either on a Minion via messaging, or on another thread pool.
Due to the original synchronous design and nature of the poller, the thread is blocked while waiting for the result of the poll.

In the event that the monitor is not succesfully executed, due to interruption, TTL timeout, or some other error, then the existing state of the service is kept.
As a result, if all of the Minions at a particular location go offline, then we expect the poller related threads for monitored services at this location to be blocked pending timeouts and for no outages to be reported.

> See [TTL Handling](../foundation/04-ipcs.md#ttl-handling) for additional details.
