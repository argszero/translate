http://markorodriguez.com/2013/01/09/on-graph-computing/

# On Graph Computing
# 图计算

The concept of a graph has been around since the dawn of mechanical computing and for many decades prior in the domain of pure mathematics. Due in large part to this golden age of databases, graphs are becoming increasingly popular in software engineering. Graph databases provide a way to persist and process graph data. However, the graph database is not the only way in which graphs can be stored and analyzed. Graph computing has a history prior to the use of graph databases and has a future that is not necessarily entangled with typical database concerns. There are numerous graph technologies that each have their respective benefits and drawbacks. Leveraging the right technology at the right time is required for effective graph computing.

图论随着机械计算的曙光而出现，在数学领域已经存在好几十年了。得益于这个数据库的黄金时代，图论证在软件引擎领域变的越来越流行。图数据库提供了持久化和处理图数据的一种方式。但是，图数据库并不是我们可以存储和分析图数据的唯一方式。图计算的历史中曾经使用图数据库，但未来并不是必须引入典型的数据库概念。有很多种图技术，每个都有各自的优点和缺点。为了进行高效的图计算，需要在合适的时间利用合适的技术

## Structure: Modeling Real-World Scenarios with Graphs
## 结构：使用图为现实世界场景建模

![](http://markorodriguez.files.wordpress.com/2013/01/vertex-edge.png?w=165)

A graph (or network) is a data structure. It is composed of vertices (dots) and edges (lines). Many real-world scenarios can be modeled as a graph. This is not necessarily inherent to some objective nature of reality, but primarily predicated on the the fact that humans subjectively interpret the world in terms of objects (vertices) and their respective relationships to one another (edges) (an argument against this idea). The popular data model used in graph computing is the property graph. The following examples demonstrate graph modeling via three different scenarios.

图（或网络)是一种数据结构。它有顶点和边组成。很多现实场景可以使用图来建模。它不一定是真正的内在固有特征，而是主要是人为的将这个世界解释为一组对象（顶点）和他们之间的关系（边）（an argument against this idea）。生成的用于图计算的数据模型就是图的价值。下面用三个不同的场景解释一下图模型

## A Software Graph
## 软件图

Stephen is a member of a graph-oriented engineering group called TinkerPop. Stephen contributes to Rexster. Rexster is related to other projects via software dependencies. When a user finds a bug in Rexster, they issue a ticket. This description of a collaborative coding environment can be conveniently captured by a graph. The vertices (or things) are people, organizations, projects, and tickets. The edges (or relationships) are, for example, memberships, dependencies, and issues. A graph can be visualized using dots and lines and the scenario described above is diagrammed below.

Stephen是一个名为TinkerPop的面向图的引擎小组的成员。Stephen是Rexster的一个贡献者。Rexster通过软件依赖而和其他项目产生关联。当一个用户发现Rexster的一个bug，他提交了一个Issue。通过图可以很容易捕获刚才描述的协调编码环境。顶点为人，组织，项目，tickets。边为成员关系，依赖，issue等。通过点和线将上面描述转为一张图如下：

![](http://markorodriguez.files.wordpress.com/2013/01/software-graph.png?w=350)

## A Discussion Graph
## 讨论图

Matthias is interested in graphs. He is the CTO of Aurelius and the project lead for the graph database Titan. Aurelius has a mailing list. On this mailing list, people discuss graph theory and technology. Matthias contributes to a discussion. His contributions beget more contributions. In a recursive manner, the mailing list manifests itself as a tree. Moreover, the unstructured text of the messages make reference to shared concepts.

Matthias对图感兴趣，他是Aurelius的CTO，同时还是图数据库项目Titan的lead。Aurelius有一个邮件组。人们在这个邮件组里讨论图理论和技术。Matthias参与了一个讨论，他的参与引来更多人参与。以递归的方式，邮件组将其显示为一个树。另外，非结构化的消息文本里有共享的主题。

![](http://markorodriguez.files.wordpress.com/2013/01/discussion-graph.png?w=320)

## A Concept Graph
## 概念树

A graph can be used to denote the relationships between arbitrary concepts, even the concepts related to graph. For example, note how concepts (in italics) are related in the sentences to follow. A graph can be represented as an adjacency list. The general way in which graphs are processed are via graph traversals. There are two general types of graph traversals: depth-first and breadth-first. Graphs can be persisted in a software system known as a graph database. Graph databases organize information in a manner different from the relational databases of common software knowledge. In the diagram below, the concepts related to graph are linked to one another demonstrating that concept relationships form a graph.

图可以用来表述任意概念之间的关系，包括图相关的概念。比如，注意下面的句子里概念（斜体）是如何关联的 。_图_ 可以表达为 _邻接表_ 。处理 _图_ 的一般方式是 _图遍历_ 。 有两种常见的 _图遍历_ ： _深度优先_ 和 _广度优先_ 。_图_ 可以通过称为 _图数据库_ 的软件系统持久化保存。图数据库和关系型数据库采用不同的方式
![](http://markorodriguez.files.wordpress.com/2013/01/concept-graph.png?w=270)

