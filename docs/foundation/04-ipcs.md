# Inter Process Communication (IPC)

## Goal


TODO: How communications are secured.


### Sink API

Overview
The Sink API allows for code running within the context of either OpenNMS or Minion to forward unsolicited messages to one or more registered handlers. If running on a Minion, the messages are forwarded back to the OpenNMS instance before invoking the handlers, whereas the handlers are invoked directly when the messages are generated within the OpenNMS instance itself.
The main object in this API is the SinkModule which defines the message type, how the message is marshaled and unmarshaled, and a series of policies to control how the producers and consumers should behave. The AggregationPolicy defines how messages can be combined and provides criteria that define when they should be released.  The AsyncPolicy defines how the characteristics of the asynchronous producer including an upper limit on how many messages can be queued in memory, what to do with messages when this limit is reached, and how many background threads should be used to drain the queue.
In order to send a message, the sender must first obtain a dispatcher from the MessageDispatcherFactory while passing a reference to the SinkModule. The sender can choose to create a synchronous dispatcher, that will block until the message is successfully sent, or an asynchronous dispatcher that will return immediately.
To handle messages, you must register a MessageConsumer with the MessageConsumerManager.
JMS Implementation (Default)
The JMS implementation dispatches and consumes messages from a queue named "OpenNMS.Sink.${Sink Module Name}".
 

### RPC API

Overview
The remote producer call (RPC) API allows code running within the OpenNMS instance to invoke methods at a target location. The target location can be the current location, in which case the methods are executed locally, or a different location, in which case the calls are dispatched for remote execution.
The main object in this API is the RpcModule which defines the request and response objects, how these are marshaled and unmarshaled, and provides the execution routine.
In order to issue an RPC call, the caller must first obtain a client from the RpcClientFactory while passing a reference to the RpcModule.
In order to handle RPC requests on a Minion, the RpcModule must be exposed via the OSGi service registry. The implementation will listen for these and route the requests accordingly.
JMS Implementation
The JMS implementation performs bidirectional communication over request and reply queues.
For every registered RpcModule the Minion will create one or more consumers against a queue named "OpenNMS.${Location}.RPC.${RPC Module Name}". When executing a call at at remote location, the message will be placed on the appropriate queue, and a new temporary queue will be created, awaiting the reply.
If multiple Minions are active at a given location, they will operate in a competing consumer fashion, handling messages from the queue as quickly as they can be processed.
A high level view of the JMS implementation is as follows:

#### TTL Handling
The TTL mechanism ensures that clients get a response back within a given time-frame. This response may be the result of the call, if successful, or an exception indicating that no client has processed the request before it expired.
In certain cases it is possible to calculate an upper bound on time taken to complete a task (i.e. ping a given IP address using a timeout of 2 seconds with 2 retries), however in many other cases it is not. For example, performing an SNMP walk on some OID could return almost instantly, or take several minutes depending the number of entries in the sub-tree.
The TTL can be specified in each request by the client, or it may be calculated by the module. In cases where no TTL is specified, a (configurable) global default is used.
When a particular call occurs on a recurring schedule (i.e. invoke this monitor every 5 minutes) we can also use the interval (5 minutes in this case) as the TTL.


TODO: Using meta-data for TTL handling

#### IPC w/ Kafka


#### IPC w/ JMS

## Lab

* Add a new RPC module
* Add a new Sink module

