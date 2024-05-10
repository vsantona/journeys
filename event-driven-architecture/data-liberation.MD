# EDA Journey - Dealing with data liberation

## First of all, what is a journey?

Before being a software engineer, I consider myself a continuous learner.
Every day I try to learn something new by trying to improve my technical skills and soft skills, studying and above all learning from on-the-job activities and the expertise of my colleagues.
Sometimes, however, I try to delve deeper into a topic, reading books and articles, watching talks, and trying to do some experiments on it.
From this study, I try to select particular aspects that are not central but which perhaps could add value to the analysis of the topic.
This is where the journey was born, a series of articles relating to the same topic which analyze different aspects.
These articles do not claim to be tutorials or sources of truth that make you an expert on a specific topic.
Rather, it is a series of concepts explained with the help of examples that pragmatically introduce the topic to provide the basis for then understanding how to approach a more in-depth study.


## Ok great, but what are we talking about?

We will talk about EDA, or rather one of the aspects of EDA. The Event Drive architecture is an architectural pattern regarding the production and detection of events. Conceived in the 1970s, EDA has patiently waited for its time. EDA is taking center stage to solve modern business problems and deliver real-time user experiences with the emergence of event-driven APIs.
In this article, we will analyze one of the migration strategies for EDA, Data Liberation.
Data liberation is the identification and publication of cross-domain datasets into corresponding event streams.
When we talk about cross-domain datasets we are referring to all the data stored in a data store that is requested by other external systems.
Data liberation reinforces two key features of event-driven architecture: the single source of truth and the elimination of direct coupling between systems. The liberated event streams enable the creation of new event-driven microservices as consumers, with existing systems migrated in due time.


##  Ok, but let's give more context?

The definition of data liberation does not give us enough elements to fully understand how and when to take advantage of this approach.
Suppose we want to experiment with EDA and then introduce it into our current architecture, mainly made up of legacy monoliths, new microservices, and predominantly synchronous calls, as in the following example where three dependent services are querying the legacy system directly.


![](/event-driven-architecture/Figure-1.png "Figure-1")

This architecture is certainly suitable for the adoption of EDA since it is affected by synchronous communication problems, the coupling between services and above all the legacy monolith is central to the architecture and the new microservices depend on it.

## Ok clear, so what do we do?

With data liberation we introduce two new components to our architecture.
An event stream to propagate events containing the data to be liberated, and a Data sourcing mechanism to send data to the event stream.
The post-liberation workflow is shown below

![](/event-driven-architecture/Figure-2.png "Figure-2")

With this new architecture, the new services no longer depend directly on the legacy part, but rather use event streams to retrieve the data necessary to create their own internal aggregate of which they are the owner.
The legacy service is then isolated, increasing the possibilities for refactoring and decommissioning/rewriting.

A data set and its liberated event stream must be kept fully in sync, although this requirement is limited to eventual consistency due to the latency of event propagation.
A stream of liberated events must materialize back into an exact replica of the source table by the event-driven services, mentre  legacy systems do not rebuild their data sets from any event streams, but instead typically have their own backup and restore mechanisms and read absolutely nothing back from the liberated event stream.

It's essential to maintain complete synchronization between a dataset and its liberated event stream. However, achieving perfect consistency is challenging due to the delay in event propagation, so we aim for eventual consistency instead.
Event-driven services are responsible for reconstructing a source table from a stream of liberated events, ensuring an exact replica. In contrast, legacy systems tipically rely on their backup and restore mechanisms rather than rebuilding datasets from event streams, and they don't read data directly from the liberated event stream.

Another fundamental aspect for a correct adoption of data liberation is to have an explicit predefined event schema.

Changes made to the data definitions, such as creating new fields, altering or deleting existing ones, can result in dynamically changing data being propagated downstream to consumers. Without employing a clearly defined schema for liberated data, downstream consumers will be compelled to address any inconsistencies. This is extremely problematic for the provision of the single source of truth.


There are three main data liberation patterns that you can use to extract data from the underlying data store:

*Query-based*: extract data by querying the underlying state store. Possible on any data store

*Log-based*: extract data by following the append-only log for changes to the underlying data structures. Possible only for data stores that provides a transaction log

*Table-based*: extract data by consuming an event stream populated with data. In this approach, data is initially pushed to a table serving as an output queue. Subsequently, a separate thread or process retrieves the data from this table, sends it to the appropriate event stream, and removes the corresponding entries. This technique necessitates support for transactions and an output queue mechanism in the data store, typically implemented through a dedicated table configured for queuing purposes.


## Ok great but let's conclude with some key thoughts on the topic

Obviously dealing with data liberation means dealing with various concerns. Chief among them is Ensuring schema compatibility, with schema serialization (and therefore validation) that can also be integrated into the capture workflow before or after the event is written to the outbox table. Validating and serializing before writing ensures that data is treated as a first-class citizen and offers a guarantee that events in the output event stream are ultimately consistent with the data within the source data store. This approach safeguards the integrity of the internal data model source while maintaining isolation. It represents the highest level of assurance that a change data capture solution can provide.
Another concern is the performance overhead caused by data liberation mechanisms. The impact on business workflow performance may be non-trivial, particularly when validating schemas via serialization. Failed transactions can also prevent business operations from proceeding. The same is for performance impacts on data stores, especially when many records are written, read, and deleted from outbound mail.


Data liberation is important for providing a mature and accessible data communication layer. Legacy systems frequently contain the bulk of the core business domain models, stored within some form of centralized implementation communication structure. The liberation of this data from legacy systems is essential to empower various organization sectors to innovate and create new, independent products and services.
The success of data liberation initiatives in transitioning towards an event-driven architecture is heavily influenced by the organizational culture. Data owners must recognize the importance of generating clean and dependable event streams, and understand that data capture mechanisms are insufficient as a final destination for liberating event data.
As in many cases, the biggest obstacle to this type of innovation is cultural and not purely technical.

*That’s all for this article.*
*Thanks for reading.*

##### References
*- Building Event-Driven Microservices by  Adam Bellemare*

*- GOTO EDA Day 2023 (David Boyne, Sam Dengler, Brian Zambrano speeches)*
 
