http://www.cubrid.org/blog/dev-platform/meet-impala-open-source-real-time-sql-querying-on-hadoop/

# Meet Impala: Open Source Real-Time SQL Queries on Hadoop
# 结识Impala: Hadoop之上的开源实时SQL查询

 __UPDATED on July 16 2013__ : the article has been updated inline considering the feedbacks on Twitter and comments below.

In this article I would like to introduce you to Cloudera Impala, an open source system which provides real-time SQL querying functionality on top of Hadoop. I will quickly go over when and how Impala was created, then will explain in more details about Impala's architecture, its advantages and drawbacks, compare it to Pig and Hive. You will also learn how to install, configure and run Impala. At the end of this article I will show the performance test results I have obtained when comparing Impala with Hive, Infobright, infiniDB, and Vertica.

## Analyzing results in real time through Hadoop

With the advent of Hadoop in 2006, big data analytics was no longer a task that could be performed by only a few companies or groups. This is because Hadoop was an open-source framework, and thus many companies and groups that needed big data analytics could use Hadoop easily at low cost. In other words, big data analytics became a universal technology.

The core of Hadoop is Hadoop Distributed File System (HDFS) and MapReduce. Hadoop stores data in HDFS, a file system that can expand the capacity in the form of a distributed file system, conducts MapReduce operations based on the stored data, and consequently gets the required data.

There is no limit to one's needs. The Hadoop user group tried to overcome Hadoop's limits in terms of functionality and performance, and develop it more. Complaints were focused on the use of MapReduce. MapReduce has two main disadvantages.

1. It is very inconvenient to use.
2. Its processing is slow.

To resolve the inconveniences of using MapReduce, platforms such as Pig and Hive appeared in 2008. Pig and Hive are sub-projects of Hadoop (Hadoop is also an ecosystem of multiple platforms; a variety of products based on Hadoop have been created). Both Pig and Hive have a form of high-level language, but Pig has a procedural form and Hive has a declarative language form similar to SQL. With the advent of Pig and Hive, Hadoop users could conduct big data analytics easier.

However, as Hive and Pig are related to the data retrieval interface, they cannot contribute to accelerating big data analytics work. Internally, both Hive and Pig use MapReduce as well.

This is why HBase, a column-based NoSQL appeared. HBase, which enables faster input/output of key/value data, finally provided Hadoop-based systems with an environment in which data could be processed in real time.

This progress of Hadoop (Hadoop eco-System) was greatly influenced by Google. HDFS itself was implemented on the basis of papers on GFS published by Google, and Hbase appears to have been based on Google's papers on BigTable. Table 1 below shows this influential relationship.



<table>
    <caption>__Table 1: Google Gives Us A Map__ (source: Strata + Hadoop World 2012 Keynote: Beyond Batch - Doug Cutting)</caption>
    <tr>
        <th>Google Publication</th>
        <th>Hadoop</th>
        <th>Characteristics</th>
    </tr>
    <tr>
        <td>GFS & MapReduce (2004)</td>
        <td>HDFS & MapReduce (2006)</td>
        <td>Batch Programs</td>
    </tr>
    <tr>
        <td>Sawzall (2005)</td>
        <td>Pig & Hive (2008)</td>
        <td>Batch Queries</td>
    </tr>
    <tr>
        <td>BigTable (2006)</td>
        <td>HBase (2008)</td>
        <td>Online key/value</td>
    </tr>
    <tr>
        <td>Dremel (2010)</td>
        <td>Impala (2012)</td>
        <td>Online Queries</td>
    </tr>
    <tr>
        <td>Spanner (2012)</td>
        <td>????</td>
        <td>Transactions, Etc.</td>
    </tr>
</table>

Cloudera's Impala, introduced in this article, was also established under the influence of Google. It was created based on Google's Dremel paper, which was published back in 2010. Impala is an open-source system under an Apache license, which is an interactive/real-time SQL query system that runs on HDFS.

SQL is very familiar to many developers and is able to express data manipulation/retrieval briefly.

As Impala supports SQL and provides real-time big data processing functionality, it has the potential to be utilized as a business intelligence (BI) system. For this reason, some BI vendors are said to have already launched BI system development projects using Impala. The ability to get real-time analytics results by using SQL makes the prospect of big data brighter and also extends the application scope of Hadoop.

## Cloudera Impala

Cloudera, which created Impala, said they had been technically inspired by Google's Dremel paper, which made them think that it would be possible to perform real-time, ad-hoc queries in Apache Hadoop.

In October 2012, when announcing Impala, Cloudera introduced it as follows:

>“Real-Time Queries in Apache Hadoop, For Real”

Impala adopted Hive-SQL as an interface. As mentioned above, Hive-SQL is similar in terms of syntax to SQL, a popularly used query language. For this reason, users can access data stored in HDFS through a very familiar method.

As Hive-SQL uses Hive, you can access the same data through the same method. However, not all Hive-SQLs are supported by Impala. For this reason, you had better understand that Hive-SQLs that are used in Impala can also be used in Hive.

The difference between Impala and Hive is whether it is real-time or not. While Hive uses MapReduce for data access, Impala uses its unique distributed query engine to minimize response time. This distributed query engine is installed on all data nodes in the cluster.

This is why Impala and Hive show distinctively different performance in the response time to the same data. Cloudera mentions the following three reasons for Impala's good performance:

1. Impala has reduced CPU load compared to Hive, and thus it can increase IO bandwidth to the extent that CPU load is reduced. This is why Impala shows 3-4 times better performance than Hive on purely IO bound queries.
2. If a query becomes complex, Hive should conduct multi-stage MapReduce work or reduce side joins. For queries that cannot be efficiently processed through a MapReduce framework (a query that contains at least one join operation), Impala shows 7 to 45 times better performance than Hive.
3. If the data block to analyze has been file-cached, Impala will show much faster performance, and in this case, it performs 20 to 90 times faster than Hive.


## Real-time in Data Analytics

The term "real time" is emphasized with the introduction of Impala. But one may naturally ask, "__How much time is real time?__" On this question, Doug Cutting (the person who created Hadoop) and Cloudera's senior architect gave their opinions on what real time is.

* "When you sit and wait for it to finish, that is real time. When you go for a cup of coffee or even let it run overnight, that's not real time."
* "'real-time' in data analytics is better framed as 'waiting less.'

Although it would be difficult to clearly define the criteria for real time numerically, if you can wait for a result, watching your monitor, that may be called real time.


## Impala Architecture

Impala is composed largely of __impalad__ and __impala state store__.

__Impalad__ is a process that functions as a distributed query engine. It designs a plan for queries and processes queries on data nodes in the Hadoop cluster. __The impala state store__ process maintains metadata for the impalads executed on each data node. When the impalad process is added or deleted in the cluster, metadata is updated through the impala state store process.

![Figure 1:  Impala High-level Architectural View.](http://www.cubrid.org/files/attach/images/220547/566/694/impala_high_level_architectural_view.png)
Figure 1:  Impala High-level Architectural View.

## Data Locality Tracking and Direct Reads

In Impala, the impalad process processes queries on all data nodes in the cluster instead of MapReduce, which is Hadoop's traditional analytic framework. Some advantages of Impala with regard to this configuration include data locality and direct read. In other words, impalad processes only the data block within the data node to which it belongs, and reads the data directly from the local directory. Through this, Impala minimizes network load. Moreover, it can also benefit from the effect of file cache.

## Scale Out

Impala provides a horizontal expansion like Hadoop cluster. In general, you can expand Impala when a cluster is horizontally expanded. All you have to do to expand the Impala that is running the impalad process on the server when a data node is added (metadata for the addition of impalad will be updated through impala state store).

This is very similar to databases based on massively parallel processing (MPP).

## Failover

Impala analyzes data stored in Hive and HBase. And HDFS used by Hive and HBase provides a certain level of failover through replication. For this reason, Impala can perform queries if the replica of a data block and at least one impalad process exist.

__Update__: in addition, if Imapala is used in CDH4 (Cloudera’s Distribution including Apache Hadoop), it is possible to configure HA for HDFS.

## Single Point of Failure (SPOF)

One of the huge concerns about all systems which use HDFS as a storage medium is that their name node is SPOF (we have discussed this in details when comparing HDFS with other distributed file systems in our previous article Overview and Recommendations for Distributed File Systems). Some solutions to prevent this have recently been released, but resolving the problem fundamentally is still a distant goal.

~~ In Impala, the namenode is SPOF as well. This is because you can't perform queries unless you know the location of a data block.~~
 According to Impala Performance and Availability, there is no SPOF in Impala. "All Impala daemons are fully able to handle incoming queries. If a machine fails however, all queries with fragments running on that machine will fail. Because queries are expected to return quickly, you can just rerun the query if there is a failure."

## Query Execution Procedure

The following is a brief account of the query execution procedure in Impala:

1. The user selects a certain impalad in the cluster, and registers a query by using impala shell and ODBC.
2. The impalad that received a query from the user carries out the following pre-task:
  1. It brings Table Schema from the Hive metastore and judges the appropriateness of the query statement.
  2. It collects data blocks and location information required to execute the query from the HDFS namenode.
  3. Based on the latest update of Impala metadata, it sends the information required to perform the query to all impalads in the cluster.
3. All the impalads that received the query and metadata read the data block they should process from the local directory and execute the query.
4. If all the impalads complete the task, the impalad that received the query from the user collects the result and delivers it to the user.

## Hive-SQL Supported by Impala

Not all Hive-SQLs are supported by Impala. As Impala supports only some Hive SQLs, you need to know which statements are supported.

## SELECT QUERY

Impala supports most of the SELECT-related statements of Hive SQL.

## Data Definition Language

In Impala, you cannot create or modify a table. As shown below, you can only retrieve databases and table schemas. You can only create and modify a table through Hive.

* SHOW TABLES
* SHOW DATABASES
* SHOW SCHEMAS
* DESCRIBE TABLE
* USE DATABASE

## Data Manipulation

Impala provides only the functionality of adding data to an already created table and partition.

* INSERT INTO
* INSERT OVERWRITE

## Unsupported Language Elements

There are still many Hive SQLs not supported by Impala. Therefore, you are advised to see the Impala Language Reference before using Impala.

* ~~ Data Definition Language (DDL) such as CREATE, ALTER, DROP. ~~ Impala stable version already supports DDL.
* Non-scalar data types: maps, arrays, structs
* LOAD DATA to load raw files
* Extensibility mechanisms such as TRANSFORM, custom User Defined Functions (UDFs), custom file formats, custom SerDes
* XML and JSON functions
* User Defined Aggregate Functions (UDAFs)
* User Defined Table Generating Functions (UDTFs)
* Lateral Views
* etc.

## Data Model

Since Impala 1.0 has dropped its beta label, Impala supports a variety of file formats: Hadoop native (Apache Avro, SequenceFile, RCFile with Snappy, GZIP, BZIP, or uncompressed); text (uncompressed or LZO-compressed); and Parquet (Snappy or uncompressed), the new state-of-the-art columnar storage format.

The highest interest, however, lies in whether Impala will support Trevni, a project currently led by Doug Cutting. Trevni is a file format that stores a table record comprising rows and columns in the column-major format instead of the existing row-major format. __Why is support for Trevni a matter of keen interest?__ It is because Impala could provide better performance with Trevni.

As Trevni is still being developed, you will only see a brief account of the column file format mentioned in the Dremel paper.

Dremel highlights the column file format as one of the factors affecting performance. What benefits most from the column file format is Disk IO. As the column file format uses a single record by dividing it into columns, it is very effective when you retrieve only some of the columns from the record. In the existing row-unit storage method, the same disk IO occurs whether you see a single column or all columns, but in the column file format, you can use Disk IO more efficiently, as Disk IO only occurs when you access the specific required column.

![Figure 2: Comparison of Row-unit Storage and Column-unit base (source: Dremel: Interactive Analysis of Web-Scale Datasets).](http://www.cubrid.org/files/attach/images/220547/566/694/comparison_of_row_unit_storage_and_column_unit_base.png)
Figure 2: Comparison of Row-unit Storage and Column-unit base (source: Dremel: Interactive Analysis of Web-Scale Datasets).

Looking at the result of the two tests conducted with regard to the column file format in Dremel, you can estimate the degree of contribution of the column file format to performance.

![Figure 3: Comparison of the Performance of the Column-unit Storage and the Row-unit Storage (source: Dremel: Interactive Analysis of Web-Scale Datasets).](http://www.cubrid.org/files/attach/images/220547/566/694/comparison_of_performance_of_column_unit_vs_row_unit_storage.png)
Figure 3: Comparison of the Performance of the Column-unit Storage and the Row-unit Storage (source: Dremel: Interactive Analysis of Web-Scale Datasets).

![Figure 4:  Comparison of the Performance of MapReduce and Dremel in the Column-unit Storage and the Row-unit Storage (300 nodes, 85 billion Records) (source: Dremel: Interactive Analysis of Web-Scale Datasets).](http://www.cubrid.org/files/attach/images/220547/566/694/comparison_of_performance_of_mapreduce_vs_dremel.png)
Figure 4:  Comparison of the Performance of MapReduce and Dremel in the Column-unit Storage and the Row-unit Storage (300 nodes, 85 billion Records) (source: Dremel: Interactive Analysis of Web-Scale Datasets).

In __Figure 3__, (a), (b) and (c) show the execution time according to the number of columns randomly selected in the column file format, while (d) and (e) show the existing record reading time. According to the result of the test, the execution of the column file format is faster. In particular, the gap becomes bigger when it accesses a smaller number of columns. Reading records using the previous method, it took a very long time even when accessing a single column, as if it was reading all columns. However, the smaller the number of selected columns, the better performance the column file format (a, b and c) shows.

__Figure 4__ compares the execution time when MapReduce work is processed in the column file format data and when it is not (while this is a comparison test of MapReduce and Dremel, we will just check the result of application/non-application of the column file format to MR). In this way, the column file format improves performance significantly by reducing Disk IO.

Of course, as the above test result is about the column file format implemented in Dremel, the result of Trevni, which will apply to Impala, may be different from the result above. Nevertheless, Trevni is also being developed with the same goal as that of the column file format of Dremel. For this reason, it is expected to have a similar result to the test result above.

__Update__: Intead of Trevni, Impala will support Parquet, an efficient general-purpose columnar file format for Apache Hadoop. In the current GA release of Impala only a preview form of Parquet is available.

## Installing, Configuring and Running Impala

## Installing

To install Impala, you must install the following software (for detailed installation information, visit the Impala website):

* Red Hat Enterprise Linux (RHEL) / CentOS 6.2 (64 bit) or above.
* Hadoop 2.0
* Hive
* MySQL
 * MySQL is indispensable for Hive metastore. Hive supports a variety of databases used as metastore. However, currently, you should use MySQL to make Impala and Hive interwork.

If interworking with HBase is required, you should install HBase as well. This article will not discuss the installation procedure for each software solution in detail.

If you have installed all required software, you should now install the Impala package to all the data nodes in the cluster. Then, you should install the impala shell in the client host.

## Configuring

For impalad to access HDFS file blocks directly from the local directory and for locality tracking, you should configure some settings for Hadoop core-site.xml and hdfs-site.xml. As this configuration is critical to the performance of Impala, you must add the settings. After changing the settings, HDFS should reboot.

Refer to Configuring Impala for Performance without Cloudera Manager for more information on configuration.

## Running

Before running Impala, you should carry out some pre-tasks, such as table creation and data loading through Hive Client, as Impala does not currently support these.

If you need to analyze an existing table in HBase or a new table, you can redefine it as an extern table through Hive. Refer to https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration.

To enter a query in Impala, you can enter it through Impala shell, Cloudera Beeswaw (an application that helps users to use Hive easily), or ODBC. Here you will see a query method by using the Impala shell.

```
$ sudo –u impala impala-shell
Welcome to the Impala shell. Press TAB twice to see a list of available commands.
Copyright (c) 2012 Cloudera, Inc. All rights reserved.
(Build version: Impala v0.1 (1fafe67) built on Mon Oct 22 13:06:45 PDT 2012)
 
[Not connected] > connect impalad-host:21000
[impalad-host:21000] > show tables
custom_url
[impalad-host:21000] > select sum(pv) from custom_url group by url limit 50
20390
34001
3049203
[impalad-host:21000] >
```

Before entering a query, you should select an impalad to be the main server from the impalads in the cluster. Once the connection is complete, enter a query and see the result.

## Impala Functionality/Performance Test

In this article I want to share with you the test results I have obtained when I first tried Impala beta version. Though many things have been changes since the beta release, I want you to see the trend in Impala performance. I conducted a simple test to see how well it performs within an available scope.

 __Table 2__ below shows the equipment and the version of the software used. The Hadoop cluster was composed of one Namenode/JobTracker, 3 to 5 Data/Task nodes, and one commander. And the HDFS replication was set to "2". The Map/Reduce capacities of node were also "2". Data for the test is approximately 1.3 billion records with 14 column data, totaling approximately 65 GB.
Table 2: Hardware and Software Versions.

<table>
    <caption>Table 2: Hardware and Software Versions.</caption>
    <tr>
        <th></th>
        <th>Classification</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>Equipment</td>
        <td>CPU</td>
        <td>Intel Xeon 2.00GHz</td>
    </tr>
    <tr>
        <td>Memory</td>
        <td>16GB</td>
        <td></td>
    </tr>
    <tr>
        <td>Software</td>
        <td>OS</td>
        <td>CentOS release 6.3(Final)</td>
    </tr>
    <tr>
        <td>Hadoop</td>
        <td>2.0.0-cdh4.1.2</td>
        <td></td>
    </tr>
    <tr>
        <td>HBase</td>
        <td>0.92.1-cdh4.1.2</td>
        <td></td>
    </tr>
    <tr>
        <td>Hive</td>
        <td>0.9.0-cdh4.1.2</td>
        <td></td>
    </tr>
    <tr>
        <td>Impala</td>
        <td>0.1.0-beta</td>
        <td></td>
    </tr>
</table>

The table schema and query created for the test are as follows:

Code 1: Hive Table Schema.

```
hive> describe custom_url;
OK
sd_uid bigint
type int
freq int
locale int
os int
sr int
browser int
gender int
age int
url int
visitor int
visit int
pv int
dt int
```

Code 2: Test Query.

```
select sr, url, sum(pv) as sumvalue from custom_url
WHERE sr != -1 and url != -1 and (sd_uid=690291 or sd_uid=692758)
group by sr, url order by sumvalue desc limit 50

```

The query was simply composed of frequently used simple statements, including select, from, group by and order by.

## Test Result

With cluster sizes of 3 and 5 nodes, the same query in the same table was performed through Impala and Hive, and the average response time was measured.

<table>
    <tr>
        <th></th>
        <th>Impala</th>
        <th>Hive</th>
        <th>Hive/Impala</th>
    </tr>
    <tr>
        <td>3 nodes</td>
        <td>265s</td>
        <td>3,688s</td>
        <td>13.96</td>
    </tr>
    <tr>
        <td>5 nodes</td>
        <td>187s</td>
        <td>2,377s</td>
        <td>13.71</td>
    </tr>
</table>


As mentioned above, Impala shows much better performance than the analytic work by using MapReduce. In addition, the bigger the cluster size is, the better response time Impala shows linearly. (After expanding the cluster, the test was conducted after evenly distributing data blocks through rebalancing.)

![Figure 5: Impala's Response Time According to Cluster Size.](http://www.cubrid.org/files/attach/images/220547/566/694/impala_response_time_according_to_cluster_size.png)
Figure 5: Impala's Response Time According to Cluster Size.


In addition, to compare the performance of Impala with that of other column-based commercial databases, such as Infobright, infiniDB and Vertica, which have MPP as a common denominator with Impala, a test was also conducted with the same data and query (the open source version of Infobright and infiniDB and the Community Edition version of Vertica were used for the test). The test of the three databases above was conducted on a single server.

<table>
    <caption>Table 4: Execution Time of Commercial Column-based Databases.</caption>
    <tr> <th>Database</th> <th>Execution time</th> </tr>
    <tr> <td>Infobright</td> <td>200s</td> </tr>
    <tr> <td>infiniDB</td> <td>32s</td> </tr>
    <tr> <td>Vertica</td> <td>15s</td> </tr>
</table>

As the test result in __Table 4__ shows, Infobright, infiniDB and Vertica, which are commercial enterprise products, show much better results than Impala. This is because Impala is still in the initial development stage, and thus may have structural and technical shortcomings. For Vertica, which showed the best performance of the four, in a three-server environment, it showed 30% higher performance than the result specified in Table 4.

However, as Vertica is a commercial product, if you want to configure a cluster of two or more servers, there is a higher cost.

## Conclusion

Less than a year has passed since Impala beta was released and two months since the production 1.0 version is released. Since Impala 1.0 a variety of file formats are now supported, a subset of ANSI-92 SQL is supported including `CREATE`, `ALTER`, `SELECT`, `INSERT`,`JOIN`, and subqueries. In addition, it supports partitioned joins, fully distributed aggregations, and fully distributed top-n queries.

Apart from this, MapR, which does business by using Hadoop like Cloudera, suggested Drill, an open source system based on the Dremel paper, like Impala, which is being prepared mainly by its HQ developers, to the Apache incubator. ~~ It is still just a proposal ~~ (not any more, it is currently undergoing incubation at The Apache Software Foundation), but Drill is expected to make fast progress once it is launched, as it is being promoted mainly by MapR developers.

By Kwon Donghun, Software Engineer at Data Infra Lab, NHN Corporation.


