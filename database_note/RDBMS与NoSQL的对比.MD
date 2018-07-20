# RDBMS与NoSQL的对比
## 关系数据库管理系统 
RDBMS =Relationship database manage system</br>
高度组织结构化数据，结构化查询语言SQL；</br>
数据和关系都存储在单独的表中，数据操纵语言；</br>
严格一致性，遵循ACID原则；</br>
适合于客户服务器编程，网络和商务应用的主导技术，适用于需要确保数据安全性、原子性、一致性的场景。</br>

### 关系型数据库遵循ACID规则
transaction 交易，国内称“事务”。</br>
* Atomicity 原子</br>
一笔交易开始时，完全做或者完全不做。事务要么成功，要么失败。</br>
* Consistency 一致</br>
前提：a+b=10，过程：a改变时，b也必须改变，结果：保证a+b=10。</br>
* Isolation 独立</br>
如果一个事务要访问的数据正在被另外一个事务修改，只要另外一个事务未提交，它所访问的数据就不受未提交事务的影响。用户a给用户b转钱，a转账的同时b查询他自己的数据，在a提交之前，b是看不到a新转来的钱的。
* Durability 持久</br>
一旦事务提交后，它所做的修改将会永久的保存在数据库上，即使出现宕机也不会丢失。</br>

## 非关系型数据库 
NoSQL = Not Only SQL</br>
非结构化和不可预知数据，无声明性查询语言；</br>
无预定义模式，item存储、列存储、文档存储、图形数据库；</br>
最终一致性，非ACID原则，遵循CAP原则；</br>
适用于超大规模数据的存储，常用于存储用户的个人信息，社交网络，地理位置，用户生成的数据和用户操作日志。</br>

### NoSQL遵循CAP Theorem
#### CAP Theorem组成
* Consistency 一致</br>
Every read receives the most recent write or an error.</br>
所有节点相同时间具有统一数据（相当于访问同一份最新数据副本）。</br>
* Availability 可用</br>
Every request receives a (non-error) response – without guarantee that it contains the most recent write.</br>
保证每个请求不仅成功或失败都响应。</br>
* Partition tolerance 分隔容忍</br>
The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes.</br>
系统中任意信息的丢失或失败不会影响系统的继续运作。</br>
（大概和其中一个路由挂了不会影响整个网络的使用相似。）</br>

#### CA/ CP/ PA 
CAP理论的核心：</br>
一个分布式系统不可能同时满足C、A、P三个需求，最多只能同时满足其中两个。</br>

* CA</br>
单点集群，满足一致性和可用性，扩展性差。（靠谱，较快，但事情一定要全干完，难以增加新设备）</br>
* CP</br>
满足一致性，分区容忍性的系统，通常性能不是特别高。</br>
（靠谱，有可能很慢，易加新设备，为保持一致性降低性能但够用）</br>
For example, we have 10 computers in the network and if two lose connection with others, every computer should wait.</br>
CP Works like: I have Partitions and they are consistent to each other all the time but it can happen that the whole system does not answer in time for write operations because one server died.</br>
* AP</br>
满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。</br>
（不那么靠谱，较快，易加新设备，事情可以只干一半，用户a看到的数据有可能和用户b不一样）</br>


#### 与BASE的关系
BASE：Basically Available, Soft-state, Eventually Consistent。</br>
BASE就是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案。</br>
* Basically Available 基本可用</br>
* Soft-state 软状态</br>
* Eventually Consistent 最终一致</br>

## 现时大多数据库的分类
* 常见CA系统</br>
1. 传统关系型数据库，比如Postgres和MySQL等(Relational) ;</br>
2. Vertica (Column-oriented) ;</br>
3. Aster Data (Relational) ;</br>
4. Greenplum (Relational) ;</br>

* 常见CP系统</br>
1. BigTable (Column-oriented) ;</br>
2. Hypertable (Column-oriented);</br>
3. HBase (Column-oriented) ;</br>
4. MongoDB (Document) ;</br>
5. Terrastore (Document) ;</br>
6. Redis (Key-value) ;</br>
7. Scalaris (Key-value) ;</br>
8. MemcacheDB (Key-value) ;</br>
9. Berkeley DB (Key-value) ;</br>

* 常见AP系统</br>
1. Dynamo (Key-value);</br>
2. Voldemort (Key-value) ;</br>
3. Tokyo Cabinet (Key-value) ;</br>
4. KAI (Key-value) ;</br>
5. Cassandra (Column-oriented) ;</br>
6. CouchDB (Document-oriented) ;</br>
7. SimpleDB (Document-oriented) ;</br>
8. Riak (Document-oriented) ;</br>
