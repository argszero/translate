http://blog.cloudera.com/blog/2013/09/email-indexing-using-cloudera-search/

# Email Indexing Using Cloudera Search
# 使用Cloudera Search做邮件索引

    by Jeff Shmain September 25, 2013 no comments Tweet

Why would any company be interested in searching through its vast trove of email? A better question is: Why wouldn’t everybody be interested? 

为什么有公司会对大量邮件的搜索感兴趣？更好的问法是：为什么不是每个人都感兴趣？

Email has become the most widespread method of communication we have, so there is much value to be extracted by making all emails searchable and readily available for further analysis. Some common use cases that involve email analysis are fraud detection, customer sentiment and churn, lawsuit prevention, and that’s just the tip of the iceberg. Each and every company can extract tremendous value based on its own business needs. 

Email成了我们最普遍的较量方式，所以，使得邮件可以搜索并为将来的分析做好准备，可以提取出巨大的价值。邮件分析最常见的用例是欺诈检测，客户情绪及流失，预防投诉，这只是冰山一角。每个公司都可以给予他们自己的业务需求提取出巨大的价值。

A little over a year ago we described how to archive and index emails using HDFS and Apache Solr. However, at that time, searching and analyzing emails were still relatively cumbersome and technically challenging tasks. We have come a long way in document indexing automation since then — especially with the recent introduction of Cloudera Search, it is now easier than ever to extract value from the corpus of available information.

一年多前，我们描述了如何通过HDFS和Apache Solr做邮件归档和邮件索引。然而，在那时候，对邮件的搜索和分析仍然是相对繁琐并且有一定技术挑战的任务。从哪时起，我们已经在文档自动索引的路上走了很远--特别是近期发布的Cloudera Search，现在比以往任何时候都更容易从语料库中提取价值的可用信息

In this post, you’ll learn how to set up Apache Flume for near-real-time indexing and MapReduce for batch indexing of email documents. Note that although this post focuses on email data, there is no reason why the same concepts could not be applied to instant messages, voice transcripts, or any other data (both structured and unstructured). 

在这盘文章中，你将要学习如何部署Apache Flume来进行接近实时的构建索引，如何部署MapReduce啦进行批量邮件文档的索引构建。注意，虽然这里我们聚焦在邮件数据上，同样的概念同样可以应用与即时消息，语音文件或者其他数据（结构化或非结构化）

## Where to Start
## 起点


Every document that you would like to make searchable must live in an index. In addition, the documents themselves must be broken up to provide the capability of searching on specific fields.

每个你想搜索的文档都应该有索引。并且，文档应该被解析成多个字段以方便对这些字段进行搜索。

The diagram below provides a high-level view of how indexes, documents, and fields are related:

下面这张图是索引，文档以及相关字段的高层次视图

![](http://blog.cloudera.com/wp-content/uploads/2013/09/shmain11.png)

The first thing to do is to break down the emails into fields that are of particular interest to the use case.  The fields will be used for building the indexes and facets. To understand the fields of the email, we will use the Mime 1.0 standard, which is defined here. Below is the text from a sample email, with relevant fields highlighted (This email came from a public archive.)

第一件要做的事儿就是将邮件解析为用例感兴趣的多个字段。这些字段将用来构建索引和facets。为了能理解邮件的字段，我们采用描述的标准的Mime 1.0（[定义在这里](http://www.ietf.org/rfc/rfc1521.txt)。下面是示例邮件里的文本，相关的字段做了高亮（这个邮件是从公共存档拿来的）

![](http://blog.cloudera.com/wp-content/uploads/2013/09/shmain2.png)

Cloudera Search is based on Solr, which includes multiple other components such as Apache Lucene, SolrCloud, Apache Tika, and Solr Cell. Let’s begin by editing Solr’s schema.xml file to incorporate the fields we have identified in the previous exercise. Throughout this document, you will extensively use the solrctl command to manage SolrCloud deployments. (See the full command reference here.)

Cloudera Search基于Solr，Solr包含诸如Apache Lucene，SolrCloud，Apache Tika，Solr Cell等一堆组件。我们把上面联系中识别的字段编辑到Solr的schema.xml文件中。在本文中，我们会大量使用solrctl命令来管理SolrCloud部署。（[完整的命令参考这里](http://www.cloudera.com/content/cloudera-content/cloudera-docs/Search/latest/Cloudera-Search-User-Guide/csug_solrctl_ref.html)）

The command below will generate a template schema.xml file that you will update to fit our email use case.

下面这个命令会生成一个schema.xml的模板，我们稍后会为我们的邮件用例更新。  

    $ solrctl --zk localhost:2181/solr instancedir --generate $HOME/emailSearchConfig


(Please note that –zk localhost:2181 should be replaced with the address and port of your own Apache ZooKeeper quorum.)

(注意 - zk localhost:2181 应该替换为你自己的Apache Zookeeper quorum的地址）
 
Next, edit the schema.xml file (which can be found here). What follows is a brief overview of what was changed from the template generated above. 

然后，修改schema.xml文件（[可以从这里找到](https://github.com/jshmain/cloudera-search/blob/master/email-search/schema.xml))。下面我们简要概述一下我们基于模板做了哪些修改。

First, completely replace the <fields> and <fieldTypes> section with the one below and add the additional “to” line. The email addresses need to be tokenized differently from the rest of the text — therefore, we created email\_general field type below, using the solr.UAX29URLEmailTokenizerFactory tokenizer. Similarly, use a new field type, names\_general, for fields that contain names. Date fields use the tdate field type, which allows for easy date range faceting. All other fields are just set up as text\_general, which is supplied in the default Solr schema.xml file.
 
首先，使用下面的内容完全替换了 <fields> 和<fieldTypes> add the additional “to” line.。 邮件地址需要和其他文本做不同的标记-- 因此，我们使用solr.UAX29URLEmailTokenizerFactory创建了email\_general字段类型。类似的，为包含姓名的字段创建了names\_general类型。日期字段使用tdate类型，这个类型可以简单的进行范围查询。其他字段设置为Solr默认的schema.xml文件中就有的text\_general。

    <fields>
        <field name="message_id" type="string" indexed="true" stored="true" required="true" multiValued="false" />
        <field name="date" type="tdate" indexed="true" stored="true"/>
        <field name="from" type="email_general" indexed="true" stored="true"/>
        <field name="to" type="email_general" indexed="true" stored="true"/>
        <field name="subject" type="text_general" indexed="true" stored="true"/>
        <field name="from_names" type="names_general" indexed="true" stored="true"/>
        <field name="to_names" type="names_general" indexed="true" stored="true"/>
        <field name="cc" type="email_general" indexed="true" stored="true"/>
        <field name="bcc" type="text_general" indexed="true" stored="true"/>
        <field name="cc_names" type="names_general" indexed="true" stored="true"/>
        <field name="bcc_names" type="names_general" indexed="true" stored="true" />
        <field name="x_folder" type="text_general" indexed="true" stored="false" />
        <field name="x_origin" type="text_general" indexed="true" stored="false" />
        <field name="x_filename" type="text_general" indexed="true" stored="false"/>
        <field name="message" type="text_general" indexed="false" stored="false"/>
        <field name="body" type="text_general" indexed="true" stored="true"/>
        <field name="text" type="text_general" indexed="false" stored="false" />
        <field name="_version_" type="long" indexed="true" stored="true"/>
      </fields>
      <uniqueKey>message_id</uniqueKey>
    
    <fieldType name="names_general" class="solr.TextField" >
        <analyzer type="index">
          <charFilter class="solr.HTMLStripCharFilterFactory" />
          <tokenizer class="solr.StandardTokenizerFactory"/>
          <filter class="solr.LowerCaseFilterFactory"/>
        </analyzer>
        <analyzer type="query">
          <charFilter class="solr.HTMLStripCharFilterFactory" />
          <tokenizer class="solr.StandardTokenizerFactory"/>
          <filter class="solr.LowerCaseFilterFactory"/>
        </analyzer>
      </fieldType>
      <fieldType name="email_general" class="solr.TextField" >
        <analyzer type="index">
          <tokenizer class="solr.UAX29URLEmailTokenizerFactory" />
          <filter class="solr.LowerCaseFilterFactory"/>
        </analyzer>
        <analyzer type="query">
          <tokenizer class="solr.UAX29URLEmailTokenizerFactory" />
          <filter class="solr.LowerCaseFilterFactory"/>
        </analyzer>
      </fieldType>

 

One very important concept in SolrCloud deployments is the notion of collections. A “collection” is a single index that spans multiple Solr Instances. For example, if your email index is distributed across two Solr Instances, they all add up to form one collection. Let’s call our collection email\_collection and set it up using the commands below:

SolrCloud里一个非常重要的概念就是集合，一个`集合`是一个跨多个Solr实例的单一索引。比如，如果你的email索引分发到两个Solr实例上，则它们加在一起形成一个集合。我们把我们的集合称为email\_collection,这个集合使用下面的命令创建：

    $ solrctl --zk localhost:2181/solr instancedir --create email_collection $HOME/emailSearchConfig
    $ solrctl --zk localhost:2181/solr collection --create email_collection -s 2

Again, replace –zk localhost:2181 with your own ZooKeeper quorum configuration in both statements.

统一，将两处的zk localhost:2181替换成你自己的ZooKeeper quorum配置的地址。
 
Note that the “-s 2″ argument defines the number of shards. A “shard” is a very important concept in Solr and refers to a slice of an index. For example, if you have a corpus of 1 million emails, you may want to split it into two shards for scalability and to improve query performance. The first shard will handle all the documents that have a message-id between 0 – 500,000, and the second shard will handle documents with message id between 500,000 – 1,000,000. This logic is all handled by Solr internally; you need only specify the number of shards you would like to create with the -s option.

注意"-s 2"定义了碎片(shards)的个数。"碎片"在Solr里是一个非常重要的概念，它指向索引的一个片段。例如，你有一个1百万邮件的语料库，你可能因为扩展性或提高查询性能而希望将其分割为两个碎片，第一个碎片包含message-id在0到5十万之间的所有文档，第二个碎片包含message id在50万到1百万之间的所有文档。Solr内部处理了所有逻辑；你只需要通过在创建时通过-s选项指定碎片的个数。

The number of shards will depend on many factors and should be determined carefully. The following table describes some of the considerations that should go into choosing the optimal number:

碎片的个数受多种因素影响，应当仔细慎重决定。下表列出了再选择最优的个数时需要考虑的一些事情：

![](http://blog.cloudera.com/wp-content/uploads/2013/09/table1.png)

## How Email Parsing Happens
## Email是如何解析的

Email parsing is achieved with the help of Cloudera Morphlines.  The Morphlines library is an open source framework, available through the Cloudera Development Kit (CDK), that defines a transformation chain without a single line of code needed.

邮件的解析是通过Cloudera Morphlines来帮助实现的。Morphlines库是一个开源的框架，包含在Cloudera Development Kit(CDK)里，使用它不需要写一行代码就可以定义一个转换链。

The Morphlines configuration file defines the transformation chain. We will define a Morphlines configuration file and use it later for both batch and near real-time indexing.

Morphlines配置文件定义了转换链。我们将定义一个Morphlines配置文件，这个配置文件同时适用于批量和近实时的索引构建

The Email morphlines library will perform the following actions:
Email morphlines 库会进行下述操作：
 
1. Read the email events with the readMultiLine command
2. Break up the unstructured text into fields with the grok command
3. If Message-ID is missing from the email, generate it with the generateUUID command
4. Convert the date/timestamp into a field that Solr will understand, with the convertTimestamp command
5. Drop all of the extra fields that we did not specify in schema.xml, with the sanitizeUknownSolrFields command
6. Load the record into Solr for HDFS write, with the loadSolr command

1. 通过readMultiLine命令读取email event
2. 通过grok命令将非结构化的文本解析为多个字段
3. 如果email里缺失Message-ID，使用generateUUID命令生成一个
4. 使用convertTimestamp命令将date/timestamp的字段解析为Solr的类型
5. 通过sanitizeUnknownSolrFields命令，将我们没有在schema.xml里定义的字段丢弃
6. 使用loadSolr命令将记录加载到Solr

To view the full Morphline configuration file for this example, please visit this [link](https://github.com/jshmain/cloudera-search/blob/master/email-search/morphlines.conf).

可以从 [这里](https://github.com/jshmain/cloudera-search/blob/master/email-search/morphlines.conf) 查看本例中Morphline的完整配置。

## Which Tool to Use for Indexing
## 使用哪个工具在做索引

Cloudera Search can index emails in various ways. In a near-real-time scenario, Cloudera Search will utilize Flume to index messages on their way into HDFS.  As the data passes through Flume, relevant fields will be extracted using MorphlineSolrSink. Solr will then index the event and write the indexes to HDFS. For the batch-oriented scenarios, Cloudera Search provides MapReduceIndexerTool, which will read the data out of HDFS, build the indexes, and write them out back to HDFS. There are multiple options in the tool, including one that will merge the generated indexes into live Solr Servers.

Cloudera Search可以有多种方式来给email做索引。在近实时的场景下，Cloudera Search使用Flume来给正在写入HDFS途中的消息做索引。当消息在Flume中传递时，使用MorphlineSolrSink提取出相关字段。然后Solr给这个时间做索引并将索引写入到HDFS。在面向批处理的场景下，Cloudera Search提供了一个MapReduceIndexTool，这个工具读取HDFS中的数据，构建索引，然后将索引回写到HDFS。这个工具有很多选项，包括一个将索引merge到一个在线的Solr Server上的选项。

The appropriate indexing method will be dictated entirely by your use case and data ingestion strategy. For example, if you are doing customer churn analysis, you may initiate a periodic search query for words that signify a negative sentiment. As emails from customers come into the system, you may need to persist and index them right away. In that case, the indexing must be done in near-real-time and the data would be ingested through a Flume Agent. But suppose you already have five years of emails already stored on HDFS?  If you need to index them, the only scalable option is to run MapReduceIndexerTool.

选择哪种做索引的方法取决于你的用例和集成策略。例如，如果你在做客户流失分析，你可能需要启动一个定期的查询带有负面信息的词汇。在客户邮件进入系统时，你需要将其持久化并正确的生成索引。在这种用例下，索引必须采用近实时的方式生成，数据应该通过Flume Agent开始分析。但是，假设你已经有5年的email保存在HDFS上，如果你需要对他们做索引，你的选项就应该是跑一下MapReduceIndexerTool

In some use cases, both methods may be needed. For example, if most of the emails are indexed in real time but later on you identify a new field that was not indexed before, you may need to go back and run MapReduceIndexerTool to reindex the whole corpus. (There is an HBase near-real time indexer as well, but we’ll address that in a future post.)

在某些场景下，两种方式都是需要的。例如，如果大多数email都是实时索引的，但是，有一天你发现有个字段在之前没有索引，你需要回过头来运行MapReduceIndexerTool来对整个全集重建索引（还有个HBase的近实时的索引，我们会在后面的文章里描述它）

The table below describes some question that may help you identify optimal indexing tools: 

How to Run Batch Indexing

The flow of the MapReduceIndexerTool is straightforward:


 

    Read the HDFS directory with the files that need to be indexed.
    Pass them through the morphline, which will perform the following transformation chain:
        Break the text up into fields with grok command
        Change the date field into Date Time that Solr will understand
        Drop all of the extra fields that you did not specify in the schema.xml file
        Generate the indexes and store them to HDFS
         Merge the indexes into live SOLR servers

To run the MapReduceIndexerTool, execute the following command:

$ hadoop jar /contrib/mr/search-mr-0.9.2-cdh4.3.0-SNAPSHOT-job.jar \
org.apache.solr.hadoop.MapReduceIndexerTool \
--morphline-file  \
--output-dir  --go-live --zk-host clust2:2181/solr --collection email_collection

A few notes:

    <cloudera search directory> is the directory where search tools are installed. If you installed via packages, this would be /usr/lib/solr. If you installed via parcels, it would be the directory where the Search parcel was deployed. 
    <morphlines file> is the location of the morphlines config file that was created above.
    The –go-live option will actually merge the indexes into Solr Server.

For a full description of the above command, read this doc.
How to Index in Near-Real Time Using Flume

The flow for near-real-time indexing looks more complex than batch indexing, but it is just as simple:

    As the email files are dropped into the spooling directory, Flume will pick them up.
    Flume will replicate them into two different memory channels.
    The first channel is read by HDFS Sink and rolled directly into HDFS.
    The second channel is read by MorphlineSolrSink, which will perform the same transformation chain as described previously:
        Break the text up into fields with the grok command
        Change the date field into Date Time that Solr will understand
        Drop all of the extra fields that you did not specify in the schema.xml file
        Generate the Indexes and send them to Solr

One powerful feature of Cloudera Search is that the morphline file, which encodes the transformation logic, is identical in both examples.

For completeness, here is the section in Flume configuration that defines Solr Sink:

# SOLR Sink
tier1.sinks.solrSink.type=org.apache.flume.sink.solr.morphline.MorphlineSolrSink
tier1.sinks.solrSink.channel=solrChannel
tier1.sinks.solrSink.morphlineFile=/tmp/morphline.conf

 
How to Search and View Indexed Data

Once the data is indexable and accessible by Cloudera Search, there are many ways for users to interact with it, including the Solr GUI or the via Hue’s Search application, which provides a very rich and configurable interface with faceting, date ranges and much more. 


Solr GUI

 

Hue’s Search app

I could spend a significant amount of time describing Search visualization tools, but that is a matter for another post. 
Conclusion

To summarize, here are the important steps an enterprise can take to get more value from its email data:

    Decide which email fields are to be extracted and indexed.
    Use Morphlines to help with transformations. (No coding required!)
    Use MapReduceIndexerTool to index the emails in HDFS.
    Use Flume with MorphlineSolrSink to index the emails in near real time.

The big news is that with Cloudera Search, indexing and searching various types of data over small and large data sets has become much easier and much more flexible. We have all learned to search the internet for information by using popular tools like Google and Yahoo!. Now, this tool is available for data in your Hadoop and HBase environments – and conveniently provides a more integrated and unified process. You will be able to find your data and process it on the same platform, without having to switch context or move your data. 

Jeff Shmain is a solution architect at Cloudera.

