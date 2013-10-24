http://blog.gopivotal.com/products/in-memory-data-grid-hadoop-integrated-real-time-big-data-platform-previewed-at-springone-2gx-2013


#  In-Memory Data Grid + Hadoop: Integrated Real-Time Big Data Platform Previewed at SpringOne 2GX 2013

Apache Hadoop is gaining tremendous momentum, as it is becoming the ubiquitous answer to managing massive amounts of data from disparate sources at a very low cost.  The core Hadoop (HDFS) design center is batch and sequential in nature. However, since it  scales so well, there are a growing number of projects and products emerging that are trying to make it applicable to real time applications as well.

The magic  of Hadoop primarily lies in how its core, distributed file system (HDFS) is designed to work with large blocks of data (64MB at a minimum) as the smallest unit of management and not permitting these blocks, once created, to be mutated. This design principle paves the way for its meta-data management to be simpler, allowing the platform to dramatically scale from a volume perspective.

Today, by and large, Hadoop is used for batch processing where the data is written once and multiple batch programs walk through the entire set sequentially. HDFS isn’t suitable for real time or online applications that require random access and support thousands of concurrent users at predictably low latencies. HBase, a data store for real time, falls short for a number of reasons—things like transaction support and SQL querying, to name a few. One could also argue that its dependence on HDFS for each write (Synchronous writes to Logs—WAL) makes it challenging to offer low latencies.

Hadoop is  also slow in detecting failures. It can take several minutes and results in huge application pauses.  Hadoop couples failure detection and recovery with overload handling into a conservative design with conservative parameter choices. As a result, not only is Hadoop usually slow in reacting to failures, it also exhibits large variations in response time under failure (as explained in this Rice University paper).

## In-memory Data Grid Can Bottleneck on the Database



In-memory data grids (IMDG) offer advantages when working with Hadoop. IMDGs have evolved and matured over the last decade to help enterprises scale with very low latencies for transactional applications. Several IMDG products use sophisticated failure detection algorithms to ascertain if a lack of response is indeed a failure, quickly. An IMDG pools memory and scales by spreading the load across the cluster’s memory or even allowing the cluster to dynamically expand or contract with changing demand. These solutions are fundamentally designed to work with relational databases (RDB) and offer advanced techniques like “parallel async write behind”—the ability to reliably enqueue all writes in parallel across the grid and carry out batch writes to the RDB.

The most common pattern to use a traditional RDB as the backend exposes the design to a few challenges:

(1)  __Throttling due to sustained writes__: When the write rate is high, the shock absorption capability of the queues is limited due to the fact that ultimately all writes have to make it to the relational database. When the queues fill up, the application could ultimately be required to slow down.

(2)  __Handling large volumes__: RDBs cannot handle very large volumes well, or the cost associated can get prohibitively high. In addition, the ability to recover the data set back into IMDG memory is important, particularly when multiple machines fail and the cluster has to be bounced. Massive parallel loading is fraught with many challenges.

These factors can pose challenges for RDBs when developing a data platform and architecture with Hadoop.

## Using HDFS as the storage backend



We believe there is a new paradigm for using HDFS as the backend store for an IMDG. Our model provides a new data store for real time applications where a large quantity of operational data can be managed in-memory and “all data” (i.e. including massive historical) can be managed within Hadoop. This architecture avoids any contention points given the parallel nature of Hadoop and scales to extreme volumes. It also brings up an interesting question. How do you turn the ‘write once, read many’ file system into one that is suitable for real-time, ‘write many, read many’ while giving low latency access to data?

If the records have to be read back with no sequential walk through, then, the records have to be stored in a sorted manner with sufficient indexing information in some proprietary manner.

Once the data arrives in Hadoop, how can the various tools in the Hadoop ecosystem such as Hive or MapReduce run batch processing jobs? We would like to do this without having to go through the in-memory data grid tier, which could be busy, serving online applications. Could we plugin a ‘InputFormatter’ in Hadoop that can turn these records into a form that is easily consumable?

How can the batch processing output be absorbed into the memory tier and made available to applications quickly? Could there be a similar ‘OutputFormatter’ can push these records directly into the in-memory tier doing any necessary data format conversions transparently?

The graphic below illustrates this high level architecture.

![](http://blog.gopivotal.com/wp-content/uploads/2013/09/high_level_architecture.png)

With an integrated architecture as shown above, the following types of design patterns are enabled:

1. __Streaming Ingest__. This capability allows you to consume, process and store unbounded event streams. You may decide to write directly to the IMDG or drive the whole streaming workflow with SpringXD (which promotes the idea of a DSL to define the entire flow). The IMDG now allows the events to be consumed reliably (in-memory copies or on local disk) and stream these to HDFS in batches for further analytics. For instance, a trading application could maintain the latest prices for all securities in memory and also retain all time series data for price changes in HDFS. Other examples could be ingesting click streams, capturing an audit trail of interactions for compliance, etc.The more interesting pattern is the ability to now capture raw events into memory and then process them (a transform), perhaps drive some actionable insights and store the derived records to HDFS.

2. __A high performance__, operational database that scales. Increasingly, we are finding applications that want to manage data as time-series or even bi-temporal in nature. Users need a way to ask, ”what was the precise state of my database on some past date?” For instance, this is becoming an imperative in financial services due to regulatory requirements—where an app might need to access the history in a random manner, even if there is some performance hit. Archiving the state of data implies snapshots are taken for every change, causing data to grow significantly. For this type of storage, we need something like Hadoop. Of course, current data, typically the most requested, can be retained in-memory.

## Closing the Loop Between Real-time and Batch Analytics with an HDFS Glue

At Pivotal, we are actively working on a project aimed at integrating the in-memory data grid capabilities in GemFire and SQLFire and Pivotal Hadoop. It presents a novel approach by providing a parallel two-way integration with Hadoop. All writes from the real-time tier make it into Hadoop and output of analytics inside Hadoop can emerge in the in-memory “operational” tier and distributed across data centers.

![](http://blog.gopivotal.com/wp-content/uploads/2013/09/quote.png)
![](http://blog.gopivotal.com/wp-content/uploads/2013/09/SQL.png)

The idea is to leverage distributed memory across a large farm of commodity servers to offer very low latency SQL queries and transactional updates. The memory tier seamless integrates with PivotalHD’s Hadoop file system. HDFS can be configured to be the destination for all writes captured in the distributed in-memory tier (distributed, high speed ingest) or can be configured as the underlying read-write storage tier for the data cached in memory. HDFS provides reliable storage and efficient parallel recovery during restarts. Historical data that cannot fit in main memory can automatically be faulted in from HDFS in O(1) lookup times.

Besides being a storage mechanism, the data in HDFS is formatted in a manner suitable for consumption from any tool within the Hadoop eco-system. Essentially, we offer a new “closed loop” architecture between real-time and batch analytics using HDFS (and the Hadoop eco-system) as the glue layer.

We natively store the records in HDFS in an indexed fashion but offer a native Input/OutputFormat system within Hadoop so any Hadoop tool (MapReduce, Hive, Hbase, Pig, etc) can easily and transparently work with in-memory data without going through ETL. A special Pivotal HAWQ PXF driver allows federated queries executing within HAWQ to join data ingested by the real-time layer.

In the future, we plan to expose support for Objects and JSON in addition to SQL. Irrespective of whether the data is flat and structured, self describing(JSON) or nested (objects/JSON) applications will be able to issue SQL queries and index arbitrary nested fields for efficiency.

We also expect to offer the Spring Data API into this repository and integrate with Spring XD.

Come and join us at SpringOne as we walk through the design patterns enabled through this IMDG+ Hadoop architecture and learn more of the possibilities this new design pattern.

