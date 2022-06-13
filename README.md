# azure-eventhub
Azure Event Hub

-------
Video
-------

https://www.youtube.com/watch?v=Dc3P27BsK3E

-------
Info
-------

https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-about

Azure Event Hubs is a big data streaming platform and event ingestion service. It can receive and process millions of events per second. Data sent to an event hub can be transformed and stored by using any real-time analytics provider or batching/storage adapters.

The following scenarios are some of the scenarios where you can use Event Hubs:

* Anomaly detection (fraud/outliers)
* Application logging
* Analytics pipelines, such as clickstreams
* Live dashboards
* Archiving data
* Transaction processing
* User telemetry processing
* Device telemetry streaming

![image](https://user-images.githubusercontent.com/38502893/169323052-d9db73a0-88ae-4dea-8404-813d502cd83a.png)

-------
Namespace
-------

An Event Hubs namespace is a management container for event hubs (or topics, in Kafka parlance). It provides DNS-integrated network endpoints and a range of access control and network integration management features such as IP filtering, virtual network service endpoint, and Private Link.

-------
Event publishers
-------

Any entity that sends data to an event hub is an event publisher (synonymously used with event producer). Event publishers can publish events using HTTPS or AMQP 1.0 or the Kafka protocol. Event publishers use Azure Active Directory based authorization with OAuth2-issued JWT tokens or an Event Hub-specific Shared Access Signature (SAS) token gain publishing access.

-------
Event retention
-------

Published events are removed from an event hub based on a configurable, timed-based retention policy. Here are a few important points:

The default value and shortest possible retention period is 1 day (24 hours).
For Event Hubs Standard, the maximum retention period is 7 days.
For Event Hubs Premium and Dedicated, the maximum retention period is 90 days.
If you change the retention period, it applies to all messages including messages that are already in the event hub.

-------
Capture
-------

Event Hubs Capture enables you to automatically capture the streaming data in Event Hubs and save it to your choice of either a Blob storage account, or an Azure Data Lake Storage account. You can enable capture from the Azure portal, and specify a minimum size and time window to perform the capture. Using Event Hubs Capture, you specify your own Azure Blob Storage account and container, or Azure Data Lake Storage account, one of which is used to store the captured data. Captured data is written in the Apache Avro format.

-------
Partitions
-------

Event Hubs organizes sequences of events sent to an event hub into one or more partitions. As newer events arrive, they're added to the end of this sequence.

A partition can be thought of as a "commit log". Partitions hold event data that contains body of the event, a user-defined property bag describing the event, metadata such as its offset in the partition, its number in the stream sequence, and service-side timestamp at which it was accepted.

-------
Event consumers
-------

Any entity that reads event data from an event hub is an event consumer. All Event Hubs consumers connect via the AMQP 1.0 session and events are delivered through the session as they become available. The client does not need to poll for data availability.

Event Hubs organizes sequences of events sent to an event hub into one or more partitions. As newer events arrive, they're added to the end of this sequence.

-------
Consumer groups
-------

The publish/subscribe mechanism of Event Hubs is enabled through consumer groups. A consumer group is a view (state, position, or offset) of an entire event hub. Consumer groups enable multiple consuming applications to each have a separate view of the event stream, and to read the stream independently at their own pace and with their own offsets.

In a stream processing architecture, each downstream application equates to a consumer group. If you want to write event data to long-term storage, then that storage writer application is a consumer group. Complex event processing can then be performed by another, separate consumer group. You can only access partitions through a consumer group. There's always a default consumer group in an event hub, and you can create up to the maximum number of consumer groups for the corresponding pricing tier.

There can be at most 5 concurrent readers on a partition per consumer group; however it's recommended that there's only one active receiver on a partition per consumer group. Within a single partition, each reader receives all of the messages. If you have multiple readers on the same partition, then you process duplicate messages. You need to handle this in your code, which may not be trivial. However, it's a valid approach in some scenarios.

Some clients offered by the Azure SDKs are intelligent consumer agents that automatically manage the details of ensuring that each partition has a single reader and that all partitions for an event hub are being read from. This allows your code to focus on processing the events being read from the event hub so it can ignore many of the details of the partitions. For more information, see Connect to a partition.

-------
Stream offsets
-------

An offset is the position of an event within a partition. You can think of an offset as a client-side cursor. The offset is a byte numbering of the event. This offset enables an event consumer (reader) to specify a point in the event stream from which they want to begin reading events. You can specify the offset as a timestamp or as an offset value. Consumers are responsible for storing their own offset values outside of the Event Hubs service. Within a partition, each event includes an offset.

-------
Checkpointing
-------

Checkpointing is a process by which readers mark or commit their position within a partition event sequence. Checkpointing is the responsibility of the consumer and occurs on a per-partition basis within a consumer group. This responsibility means that for each consumer group, each partition reader must keep track of its current position in the event stream, and can inform the service when it considers the data stream complete.

If a reader disconnects from a partition, when it reconnects it begins reading at the checkpoint that was previously submitted by the last reader of that partition in that consumer group. When the reader connects, it passes the offset to the event hub to specify the location at which to start reading. In this way, you can use checkpointing to both mark events as "complete" by downstream applications, and to provide resiliency if a failover between readers running on different machines occurs. It's possible to return to older data by specifying a lower offset from this checkpointing process. Through this mechanism, checkpointing enables both failover resiliency and event stream replay.

-------
Duplicates
-------

https://docs.microsoft.com/en-us/azure/architecture/serverless/event-hubs-functions/resilient-design#idempotency

Deduplication techniques

Designing your functions for identical input should be the default approach taken when paired with the Event Hub trigger binding. You should consider the following techniques:

Looking for duplicates: Before processing, take the necessary steps to validate that the event should be processed. In some cases, this will require an investigation to confirm that it is still valid. It could also be possible that handling the event is no longer necessary due to data freshness or logic that invalidates the event.

Design events for idempotency: By providing additional information within the payload of the event, it may be possible to ensure that processing it multiple times will not have any detrimental effects. Take the example of an event that includes an amount to withdrawal from a bank account. If not handled responsibly, it is possible that it could decrement the balance of an account multiple times. However, if the same event includes the updated balance to the account, it could be used to perform an upsert operation to the bank account balance. This event-carried state transfer approach occasionally requires coordination between producers and consumers and should be used when it makes sense to participating services.

