---
title: Event capabilities
description: Description of the event-driven architecture for Altinn Apps and Altinn Platform.
toc: true
tags: [architecture, events]
aliases:
  - /teknologi/altinnstudio/architecture/capabilities/runtime/integration/eventcapabilties
---

{{%notice warning%}}
This is work-in-progress. The event-driven architecture is still in analysis.
{{% /notice%}}

The new generation of Altinn is moving to an [event-driven architecture](https://en.wikipedia.org/wiki/Event-driven_architecture). 
This means that the Altinn Platform solution and applications running in Altinn Apps will publish events that
application owners (agencies) and parties(citizens and businesses) can subscribe to and react to.

Most components inside the Altinn Platform and Altinn Apps solutions will communicate throug syncronus http calls.  

## Overall Concept

In Altinn there will over time be thousands of different digital services deployed to Altinn Apps.
Those digital services will be accessed by the citizens and the businesses in Norway. 

They will recieve and submitt data to/from the entity that is responsible for the digital service and others using the platform. 
![Event concept](concept.svg "Event concept")

The event architecture would make it possible to get notified when there is events in the platform related to data that the different actors has intereset in.
It could be anything from the digital service (app) owner beeing notified that a citizen has completed a form, to that the citizen is informed that there is a new form he need to fill out.


### Events

Events would be a combination of standard events defined by the platform and
custom events added in an application by application developers.

The events will typical only contains information about that an event has happened with a reference
to some data that was changed because of that event. Typical the  

**Standard events could be**

- An instance is created
- An instance changes state (from one process task to another, example: data -> signing)
- An instance is completed

**Custom events could be**

- A user has asked for a deduction in a form
- A specific validation of data failed

**Event attributes**

Events would typically have some attributes used for filtering. 

- [org] - The application owner / service owner that owns the source (app) of the event. Example: SKD, NAV
- [app] - The app the event is created for. Example: taxreport
- [instanceid] - The instanceid: 
- [eventtype] - The type of event. Created, completed ++. Free text not locked to a schema.

An event will contain a limited set of information. To get the full details for an event the consumer would need to get all details using APIs.

#### Event Schema

The Altinn 3 will use the defined [CloudEvents](https://cloudevents.io/) specification to describe events in Altinn Apps and Altinn Platform.

The reason for choosing cloud events are

- It is a standardized and open format as prefered by our architecture principles
- It backed by many and The specification is now under the [Cloud Native Computing Foundation](https://www.cncf.io/)
- It is flexible so it would support scenarious in the future


Below you find a offical example. [See full JSON Schema](https://raw.githubusercontent.com/cloudevents/spec/master/spec.json)

```json
{
    "specversion" : "1.x-wip",
    "type" : "com.github.pull.create",
    "source" : "https://github.com/cloudevents/spec/pull",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "text/xml",
    "data" : "<much wow=\"xml\"/>"
}
```

- `specversion`: The version of the CloudEvents specification which the event uses. This enables the interpretation of the context.
  Compliant event producers MUST use a value of 1.x-wip when referring to this version of the specification.
- `type`: This is the event type. Examples: instance.created, instance.process.paymentcompleted, instance.process.completed
- `source`: Describes what the event is related to. Will be used to filter event types. For an app it would typical be /{org}/{app}/{partyId}/{instanceGuid}.
  This would be used for consumers to look up a given instance.
- `subject`: The party the event is related to. PartyID is used.
- `id`: Unique id for a given event.
- `time`: The time the event was triggered. Set by the publisher.
- `datacontenttype`: Optional. Content type of data value.
  This attribute enables data to carry any type of content, whereby format and encoding might differ from that of the chosen event format
- `data`: Optional. Can contain a structure of data specific for an event type.


##### Example 1

A instance has been created for a given party. It is not possible from the event itself to know who did it.

```json {hl_lines=[3]}
[{
  "source":  "skd/skattemelding/234234422/2acb1253-07b3-4463-9ff5-60dc82fd59f8",
  "subject": "party/234234422",
  "type": "instance.created",
  "time": "2020-02-20T08:00:06.4014168Z",
  "id": "91f2388f-bd8c-4647-8684-fd9f68af5b14"
}]
```


##### Example 2

A user has completed the confirmation1 task in the process.

```json {hl_lines=[4]}
[{
  "source":  "skd/skattemelding/234234422/2acb1253-07b3-4463-9ff5-60dc82fd59f8",
  "subject": "party/234234422",
  "type": "instance.process.taskcompleted.confirmation1",
  "time": "2020-03-16T10:23:46.6443563Z",
  "id": "91f2388f-bd8c-4647-8684-fd9f68af5b14"
}]
```

##### Example 3

A user/system has completed the process for an instance.

```json {hl_lines=[4]}
[{
  "source":  "skd/skattemelding/234234422/2acb1253-07b3-4463-9ff5-60dc82fd59f8",
  "subject": "party/234234422",
  "type": "instance.process.completed",
  "time":  "2020-02-20T09:06:50.3736712Z",
  "id": "91f2388f-bd8c-4647-8684-fd9f68af5b14"
}]
```

### Event Producers

This is a start. Going forward other event sources can be added, for example from Altinn 2.

#### Altinn Platform

Storage is probably the one component that would create the most standard events.

This could be events for the creation of instances when instances state is updated and so on.
We would need to define what kind of standard events storage should create.

The assumption is that all process change events logged to instance events in storage would be published to the event architecture with limited information.

#### Applications

Applications hosted in the Altinn Apps infrastructure would be able to create events.

The application template will contain API so logic in applications can publish events based on rules defined by the developer.

These app events could be anything, and could also be triggered by other external systems through custom APIs in the app.

### Event consumers

#### Orgs (application owners)

Orgs will need to know about events happening in their apps in Altinn.

For some orgs there is a need for subscribing to events for a specific app, others might want to subscribe to all events, or maybe a specific type of event.

#### Parties (persons and organizations)

Parties submitting and receiving data in Altinn would benefit from knowing about events. This could be feedback has been added to a form, or that a new message has been received.

In many cases, parties use professionals to handle their data in Altinn. These professionals typically have many hundred or thousands of clients.

## Requirments

The following requirements are identified for the new event architecture in Altinn 3.

{{%notice warning%}}
TODO: Verify requirements
{{% /notice%}}

- It should be possible to subscribe to a specific type of event. (Example alls `instance.process.completed` events for a given app)
- It should be possible to go at least 3 months back in history.
- The consumer will keep track of which events the consumer has processed.
- The architecture should be able to list feeds for 5 000 000 users and 1 000 000 businesses.
- The architecture should support more than 10 000 publishers.
- The architecture should support more than 1 000 000 consumers.
- The architecture should support more than 500 000 000 events a year.
- Access to events should be authorized. Accessing a event for a party requires that the consumer has the correct role 

See also [Referansearkitektur for datautveksling](https://doc.difi.no/nasjonal-arkitektur/nab_referanse_arkitekturer_datautveksling/#overskrift-grunnleggende-publisering)


## Event Principles and pattern

During the analysis the following principles and pattern has been applied

### Expose events through REST-API

- Use of REST-API ensures low complexity for consuming events
- REST-API URLS and parameters are uses for filtering

### Small events
- The events will only contain a small amount of data. If more information is needed this is available from the resource itself
- Every event links to the resource affected by the event. 
- We use CloudEvent as format. 

### Consumers keep track of their status
- Consumers will 

### Time is used for sequencing

- The published time is the parameters used for sorting events

### Events does not change

- Events for a resource are never changed. A new event can revert an earlier event
  
 ### Events are stored for a limited time

 - Events will be available for 3 months.  (todo: needs to be verified)


## Proposed Event Architecture

To reduce complexity for clients and reduce lock-in to a specific product the proposed solutions is to build
a new Events component in Altinn Platform and not expose products like Kafka or Azure Event Hub directly to external systems.

The Events component will expose clean and simple REST APIs.

### API Structure

The API's will be structured so the URLs are filtered queries into the events storage.

{{%notice warning%}}
TODO: Verify proposed API structure
{{% /notice%}}

#### Instances events for Org

##### Endpoint

```http
GET {platformurl}/events/instanceevents/{org}/{app}?from={lastchanged}
```

##### Usage

This will be used by application owners to identify changes on instances for their applications.

##### Authorization

We will use scopes from Maskinporten to authorize access. In this way, it should also be possible for an org to delegate access to
events for a given org/app.

The full detail for this API is described in this [issue](https://github.com/Altinn/altinn-studio/issues/4551). 


#### Party events

A very common scenario is that a party needs to know about events for the party or other party. 

##### Endpoint

```http
POST {platformurl}/events/instanceeventsforparty/
```

This returns the events for a given party identified with a person number or organisation number.

```json {hl_lines=[4]}
{
    "appId" : "{org}/{app}",
    "party": {
        "personNumber": "12247918309",
        "organisationNumber": null
    },
    "fromtime": "2019-06-01T12:00:00Z",
    "type": null
}
```

##### Usage

This is used by end users to see events for a given party.
This will list all changes for a given party.

##### Authorization

Access to events needs to be authorized. To be able to read events, you need to have the read right for the given app for the given party.

The topic and subject would be used to identify the correct XACML-policy to use. 

The operation would be read and the proccess task will be set to `null`.
This way there would be no need to verify the current state of an instance.

From the examples above we have this example

```json {hl_lines=[4]}
[{
  "source":  "skd/skattemelding/234234422/2acb1253-07b3-4463-9ff5-60dc82fd59f8",
  "subject": "party/234234422",
  "type": "instance.process.completed",
  "time":  "2020-02-20T09:06:50.3736712Z",
  "id": "91f2388f-bd8c-4647-8684-fd9f68af5b14"
}]
```

To be able to read this event, the authenticated party is required to have the rights for SKD/Skattmelding for the party 234234422

This is something it gets throug roles for that specific application.

The full detail for this API is described in this [issue](https://github.com/Altinn/altinn-studio/issues/4552). 


### Adding events

#### Endpoint

```http
POST {platformurl}/events/
```

The full detail for this API is described in this [issue](https://github.com/Altinn/altinn-studio/issues/4550). 


### Event components

The below diagram shows the different components in the proposed Event Architecture for Altinn 3.

![Event architecture diagram](event_architecture_custom.svg "Altinn Event Architecture")

#### Publishers

Both different applications and components will publish events to the Event component in Altinn Platform.  
Publishers will use a REST API to post new events to the event API.

#### Event Component

The event components expose REST-APIs for publishing and subscribing to events.

When a publish request is received it will push the event document to the event storage.
When a request is received it will query the events stored in the event storage.

### Storage technology

Choosing the technology to physical store the events will affect what kind of capabilities the
event component can expose and what kind of scalability and performance the event architecture will have.

{{%notice warning%}}
TODO: Verify proposed storage technology.
{{% /notice%}}

### Cosmos DB

Using CosmosDB gives the possibility to have "endless" number of topics/feeds based on queries.

Based on filters on the db query you could get an endless amount of feeds containg events with specific criteria.

#### Partition key

Currently, the limitations on Cosmos DB are that one logical partition can [maximum be 20GB](https://docs.microsoft.com/en-us/azure/cosmos-db/concepts-limits).

If we assume events in average on 350 Bytes (the examples above are around 300 Bytes).
This would hold approx 57 million events inside a logiical partition.

If we assume 5 events on average on each instance that would be around 11 million instances per partition key.
Looking at the biggest digital services in the current platform (Altinn 2) this indicates that using a partition key 
based on {org}/{app} is not possible because many of them have many more elements.  

The suggestion is to use the subject as partition key. 57 million on a given subject should be more than enough.
And if that limit is reached the limitations probably have increased (it was raised from 10GB to 20GB in may 2020).

#### Event sequencing

Events will be sequnced by eventTime. This is a datetime where precision is 7 digits, with an accuracy of 100 nanoseconds.
The eventTime would be set by a [stored procedure](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-write-stored-procedures-triggers-udfs) when inserting document.

It will overwrite any eventTime set by the publisher.

### Indexing

The default indexing policy for newly created containers indexes every property of every item, 
enforcing range indexes for any string or number, and spatial indexes for any GeoJSON object of type Point. 
This allows you to get high query performance without having to think about indexing and index management upfront.

For the topic and eventTime a composite index should be investigated to see if that performs better.

[See Cosmos DB indexing for details](https://docs.microsoft.com/en-us/azure/cosmos-db/index-overview).

## Delegating access to events

There are serveral user scenarios when there is a need to delegate access to the events for a given party to another user/organisation.

### Delegating Org access

For orgs (application owners) there might be some scenarios where they want to give access to events for a given applications.
This delegation is done through Maskinporten.

### Delegating party event access

In general, access to events for a given party will be authorized based on roles the requesting organization/user
have for the subject of the event.

## Detailed Scenarios

### Org waiting on ProcessComplete for a given app

In this scenario, an org is waiting on end users to complete one given app.

1. System (consumer) authenticates using Maskinporten and requests scope `altinn:events/{org}/{app}`
2. System exchanges Maskinporten token to an Altinn token. Scopes is included in new token.
3. System calls 

```http
GET {platformurl}/events/instanceevents/{org}/{app}?from={lastchanged}&eventType=instance.process.completed
```
4. Event component verifies that scope matches request
5. Event components searches Cosmos DB for events that matches search criteria
6. Event component returns the filtered and possible capped response ordered by sequence
7. Consumer process the received events and call other API to download related data (instances, files, etc)


### User needing to know if there is anything new for a party

In this scenario, a user wants to see if there are any changes for a client or the user itself.

1. System authenticates end user with ID-porten
2. System exchanges token with Altinn
3. System calls event API


```http
POST {platformurl}/events/instanceeventsforparty/
```

4. Event component query events in database 
5. Event components authorized the event and filter away events where user is not authorized
6. Events are returned
7. Consumer process events
8. Consumer gets relevant data


### Organization needing to know if there are anything new for a party

In this scenario a professional organization wants to see if there are any changes for a client or the organization itself.

1. System authenticates end user with Maskinporten
2. System exchanges token with Altinn
3. System calls event api

```http
POST {platformurl}/events/instanceeventsforparty/
```

4. Event component query events in database 
5. Event components authorized the event and filter away events where user is not authorized
6. Events are returned
7. Consumer process events
8. Consumer gets relevant data


### Approved organization needs to know about changes for all reportees

This is a scenario where some organizations like banks need to be informed when somebody dies and a "deathestate" is created for that user.

### Anonym access to a given instances events.

In this scenario the end user has used a system to submit data, and the system needs to follow up if any feedback is given to
the instance without the user needing to log in.

1. System calls event api

```http
POST {platformurl}/events/instanceeventsforinstance/{instanceId}
```

2. Event component query events in database
3. Events are returned
4. The Consumer process events
5. The Consumer gets relevant data

## Push Events

To reduce the amount of request from consumer we should look in to supporting push of events.

This has not been detailed yet but the solution could contain:

- Consumer can set up URL webhook that would receive all or a filtered list of events. A new component will send events to the different consummers.
- User can set up a notification SMS number to get a notification about events.
- There can be a mobile app that can listen to push notifcations.


## Other technologies considered

### Azure Event Hub

Azure Event Hub is an Event ingestion service.  It can receive and process millions of events per second.

Each Subscription can have 100 Event Hub Namespaces
Each Namespace can have 10 Event Hubs

Meaning for each Subscription we can have 1000 Event Hubs,

![Event Hubs diagram](https://user-images.githubusercontent.com/13309071/75520779-a43d3600-5a06-11ea-94d7-c2b0fe856e8a.png "Azure Event Hubs")

Subscribers to Hubs use [AMQP 1.0](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) protocol to read the event feed.

#### Event format data

The events can be any JSON format or text

#### Event filtering

There is no way to filter events before they are read at the subscriber.

#### Subscribers

Subscribers need to have a SAS key for accessing the event hub.

The client needs to use AMQB 1.0 standard. 
There exist client library for [Java](https://github.com/Azure/azure-event-hubs-java)
and [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs)


#### Pro

- Most popular event streaming platform.
- Can store events indefently

#### Cons

- Cost of HDInsight cluster
- Requires more admin compared to other platforms in Azure
- Not able to support topic per party.
- Need to build a custom REST-API as a proxy to Kafka.


### Apache Kafka in Azure HDInsight

Apache Kafka is an open-source distributed streaming platform that can be used to build real-time streaming data pipelines and applications.
Kafka also provides message broker functionality similar to a message queue, where you can publish and subscribe to named data streams.

Apache Kafka kan be deployed in to a HDInsight cluster. [See details](https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-introduction)

Producers would send events to Kafka Brokers. In a HD-insight cluster each worker nod is a Kafka broker.

![Kafka diagram](kafka.png "Kafka")

#### Quotas

| Resource                 | Limit           |
| ------------------------ | --------------- |
| Topics per kafka cluster | 2000-3000       |
| Retention time           | up to unlimited |

#### Pro

- Most popular event streaming platform.
- Can store events indefently

#### Cons

- Cost of HDInsight cluster
- Requires more admin compared to other platforms in Azure
- Not able to support topic per party.
- Need to build a custom REST-API as a proxy to Kafka.


## Event Analytics

With a new event architecture it is possible imagine that we can run analytics on the events to give
important insight in to the data in the platform. 

Example
- How many instances is created for the different applications
- How long time does each task take to complete
- Is there any relationship between apps. 

This is analyzed in the following [issue](https://github.com/Altinn/altinn-studio/issues/4555)

## Open Clarification

- Is partyID ok for the consumer to be returned? (need to call service to map to orgnr/ssn)
- Would it be ok to cap the response from feed for the latest second or two to reduce the change for loosing events because of
timing an paralell events.

## Other event concepts in the platform

Events are used in different scenarios in the platform.

- **Instance Events** - Events that happen on a given instance.
  It could be created, saved, ++ This is stored to cosmos DB. The number of details in these events is higher than we would put on an event feed. 
- **Application logic events** - These are events where app developers could implement logic to get a specific behavior.
  Calculation, validation ++ This type of event is probably not relevant to push to the event feed.  