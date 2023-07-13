ELK 学习

# Elastic stack
> [Elastic stack](https://www.elastic.co/guide/index.html)

> It’s a fast and highly scalable set of components — Elasticsearch, Kibana, Beats, Logstash, 
> and others — that together enable you to securely take data from any source, in any format, 
> and then search, analyze, and visualize it.


包含三大部分：
- Ingest
  收集数据，securely take data from any source, in any format
  - collect and ship data
  例如 Filebeats，收集日志
  - enrich or transform data
  例如用 logstash 过滤日志

- Store
> Elasticsearch，the distributed search and analytics engine at the heart of the Elastic Stack.
> Elasticsearch provides a REST API that enables you to store data in Elasticsearch and retrieve it. 
存储数据以及索引查询数据

- Consume
> Use Kibana to query and visualize the data that’s stored in Elasticsearch. 
> Or, use the Elasticsearch clients to access data in Elasticsearch directly from common programming languages.

例如用 kibana 来图形化展示存储在 elasticsearch 中的数据


# Elasticsearch
> [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html)

> Elasticsearch is the distributed search and analytics engine at the heart of the Elastic Stack.
> Elasticsearch is where the indexing, search, and analysis magic happens.
> Elasticsearch provides near real-time search and analytics for all types of data.

- distributed document store
> Elasticsearch stores complex data structures that has been serialized as JSON documents.

- near-real-time--within 1 second search
> Elasticsearch uses a data structure called an inverted index that supports very fast full-text searches.

## inverted index 
> CS124 information retrieval 视频介绍：[information retrieval](https://www.youtube.com/watch?v=kNkCfaH2rxc&list=PLaZQkZp6WhWwoDuD6pQCmgVyDbUWl_ZUi&ab_channel=FromLanguagestoInformation)
> [CS 124: From Languages to Information](https://web.stanford.edu/class/cs124/)
> 科普文章：[inverted index](https://www.geeksforgeeks.org/inverted-index/)
> 维基百科介绍：[Inverted index](https://en.wikipedia.org/wiki/Inverted_index)


- Why it is called an inverted index?
> It is called an inverted index because it is simply an inversion of the forward index.

Forward Index:
Document 1: "apple", "banana"
Document 2: "banana", "orange"
Document 3: "apple", "orange"

Inverted Index:
"apple": Document 1, Document 3
"banana": Document 1, Document 2
"orange": Document 2, Document 3


- Difference between forward index and inverted index
> The forward index is faster in indexing whereas in the inverted index, searching is faster

Indexing 和 searching（querying）的区别：
indexing is the process of adding documents to Elasticsearch, while querying involves searching and retrieving relevant documents from the indexed data based on defined search criteria.


- Where is the inverted index used?
> An inverted Index is a data structure that is generally used in search engines and databases for locating relevant information quickly.

维基百科介绍：[Inverted index](https://en.wikipedia.org/wiki/Inverted_index)


## Data in: documents and indices

- An index can be thought of as an optimized collection of documents and each document is a collection of fields, which are the key-value pairs that contains your data.

- By default, Elasticsearch indexes all data in every field and each indexed field has a dedicated, optimized data structure. 

For example, text fields are stored in inverted indices, and numeric and geo fields are stored in BKD trees. 

The ability to use the per-field data structures to assemble and return search results is what makes Elasticsearch so fast.

- Elasticsearch has the ability to be schema-less

一个 document 中可以有多个不同的 fields，且允许 dynamic mapping，即 elasticsearch 能自动检测并添加新的 field


## Scalability and resilience: cluster, nodes, and shards

- Elasticsearch is built to be always available and to scale with your needs. It does this by being distributed by nature.


注意平衡 shards 的尺寸和数量，lots of small shards or a smaller number of larger shards
> [Quantitative Cluster Sizing](https://www.elastic.co/cn/elasticon/conf/2016/sf/quantitative-cluster-sizing)


- CCR (Cross cluster replication) provides a way to automatically synchronize indices from your primary cluster to a secondary remote cluster that can serve as a hot backup.

CCR is active-passive.
The index on the primary cluster is the active leader index and handles all write requests.
Indeices replicated to secondary clusters are read-only followers.


## Secure, manage and monitor elasticsearch cluster
- Kibana can manage a cluster
- [index lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) can manage data over time


## 节点 Node
> [moduels-node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)


- A node is an instance of Elasticsearch

- Cluster is a collection of connected nodes

- Every node in the cluster can handle HTTP and transport traffic by default

- The transport layer is used exclusively for communication between nodes

在配置文件 `/etc/elasticsearch/elasticsearch.yml` 中可以设置集群中的节点
```bash
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: ["10.0.0.201", "10.0.0.202", "10.0.0.203"]
```

- The HTTP layer is used by REST clients

例如可以通过 curl 命令访问 elastersearch，如查看某个节点的健康状态：
```bash
[root@es-2 ~]$ curl http://10.0.0.202:9200/_cat/health
1689215416 02:30:16 elk-cluster green 3 3 65 32 0 0 0 0 - 100.0%
```


- All nodes know about all the other nodes in the cluster and can forward client requests to the appropriate node

例如访问一个节点，查看该节点所在集群的的信息：
```bash
[root@es-2 ~]$ curl http://10.0.0.202:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
10.0.0.203           23          70   0    0.39    0.36     0.16 cdfhilmrstw -      es-node3
10.0.0.202           16          73   3    0.18    0.26     0.12 cdfhilmrstw -      es-node2
10.0.0.201           54          69   2    0.29    0.28     0.14 cdfhilmrstw *      es-node1
```

## Node roles
> [Node roles](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-roles)


可以在配置文件 `/etc/elasticsearch/elasticsearch.yml` 中通过 `node.roles` 指明当前节点的角色，如果手动指定，则需要手动指定集群中全部的节点角色，也可以不手动指定

每个集群至少需要两个节点：
- master
- data_content and data_hot，或者 data

查看集群的角色：
```bash
[root@es-2 ~]$ curl http://10.0.0.202:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
10.0.0.203           66          71   0    0.20    0.19     0.18 cdfhilmrstw -      es-node3
10.0.0.202           39          76   0    0.00    0.00     0.00 cdfhilmrstw -      es-node2
10.0.0.201           64          73   0    0.27    0.18     0.18 cdfhilmrstw *      es-node1
```
上面 node.role 显示的为角色，如 `c` 表示 coordinate，`d` 表示 data
master 为 `*` 表示为 elected master 节点

查看 elected master node
```bash
[root@es-2 ~]$ curl http://10.0.0.202:9200/_cat/master
P36drYpER7K9buvYhCJsug 10.0.0.201 10.0.0.201 es-node1
```

### master
- 控制和管理整个集群
such as creating or deleting an index, tracking which nodes are part of the cluster, and deciding which shards to allocate to which nodes. 
It is important for cluster health to have a stable master node.

master 节点必须有 `path.data` 目录，该目录用于存放集群的 metadata
集群的 metadata 描述怎么读 data nodes 的数据
```bash
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/lib/elasticsearch
```

- 一个集群只能有一个 active master node，可以配置多个 master-eligible nodes

### master-eligible node
master-eligibel node 如果不是 voting-only node，则有机会在 master election process 中被选举为 master 节点

配置文件中可以指定集群初始化时可以被选举为 master 的节点：
```bash
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["10.0.0.201", "10.0.0.202", "10.0.0.203"]
```


### dedicated master-eligible node
和 master-eligible node 不同，dedicated master-eligible node 只能有 master 角色，是将来成为 master 节点的候选节点

master-eligible 节点除了能被选举为 master 节点外，还可以有其他角色，如作为 coordinating node

> The most reliable way to avoid overloading the master with other tasks is to configure all the master-eligible nodes to be dedicated master-eligible nodes which only have the master role, allowing them to focus on managing the cluster. 

To create a dedicated master-eligible node:
```bash
node.roles: [ master ]
```

### voting-only master-eligible node
- 参与 master elections 但不作为 master 的候选节点
配置：
```bash
node.roles: [ master, voting_only ]
```

> Only nodes with the master role can be marked as having the voting_only role.

- Voting-only master-eligible nodes 也能有其他角色，如作为 data node
```bash
node.roles: [ data, master, voting_only ]
```

### data node
> Data nodes hold the shards that contain the documents you have indexed. 

- 最好创建一个 dedicated data node，即分开 master 和 data 角色
```bash
node.roles: [ data ]
```

- data node 可以分为多种，细分多种角色来处理不同的需求
In a multi-tier deployment architecture, you use specialized data roles to assign data nodes to specific tiers:
`data_content,data_hot, data_warm, data_cold, or data_frozen.`

A node can belong to multiple tiers, but a node that has one of the specialized data roles cannot have the generic data role.




### ingest node
> [Ingest node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-ingest-node)



### coordinating only node
> [coordinating only node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#coordinating-only-node)


> only route requests, handle the search reduce phase, and distribute bulk indexing
> They join the cluster and receive the full cluster state, like every other node, and they use the cluster state to route requests directly to the appropriate place(s).
```bash
node.roles: [ ]
```

集群中尽量不要设置太多 coordinating only nodes，这样会增加集群的负担，因为 master node 需要等每个节点的状态更新

data 节点也有 coordinate 节点的功能


## Data streams
> [Data streams](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-streams.html)


In Elasticsearch, a data stream is a way to organize and manage time-series data that is continuously generated or updated. 

> A data stream consists of one or more hidden, auto-generated backing indices. such as `.ds-logs-2009.09.09-000001`

> Every document indexed to a data stream must contain a @timestamp field, mapped as a date or date_nanos field type. 
> If the index template doesn’t specify a mapping for the @timestamp field, Elasticsearch maps @timestamp as a date field with default options.



## Elasticsearch cluster

> High availability (HA) clusters require at least three master-eligible nodes, at least two of which are not voting-only nodes. 
> Such a cluster will be able to elect a master node even if one of the nodes fails.


## Discovery and cluster formation
> [Discovery and cluster formation](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html)


- The discovery and cluster formation processes are responsible for discovering nodes, electing a master, formating a cluster, and publishing the cluster state each time it changes.


### Discovery
> [Discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html)

- 集群初始化时各节点发现彼此并加入集群
集群刚建立时，节点的配置文件中有指定 seed hosts 节点，节点将连接指定的 seed hosts，然后验证该节点是否是 master-eligible 节点

如果是 master-eligible 节点，则分享本节点的 known master-eligible peer，对方节点也会回应其知道的 master-eligible nodes

如果不是 master-eligible 节点，该 seed host 节点也能允许新节点连接到已存在的节点，其中可能包括 master-eligible 节点，如果依旧未找到 master-eligible 节点，则在 `discovery.find_peers_interval` 时间后重新进行 discovery 过程，默认时间为 1s

一般 seed hosts 写上一些 master-eligible 节点

```bash
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: ["10.0.0.201", "10.0.0.202", "10.0.0.203"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["10.0.0.201", "10.0.0.202", "10.0.0.203"]
#
# For more information, consult the discovery and cluster formation module documentation.
```

**********************************

seed hosts 节点的作用（chatgpt 3.5）：
Seed hosts serve an essential role in the discovery process, even if they are not master-eligible nodes. Here are a few important considerations:

1. Initial Contact Points: Seed hosts provide the entry points for new nodes to connect with the cluster. When a new node joins the cluster or an existing node needs to rejoin after being offline, it probes the seed addresses. These seed hosts may not be master-eligible nodes, but they act as initial contact points for the discovery and communication process.

2. Bootstrapping: Seed hosts help bootstrap the discovery process by providing the new node with information about other nodes in the cluster. Even if a seed host itself is not master-eligible, it may still be connected to other master-eligible nodes. It can share information about these master-eligible nodes with the new node, allowing it to connect and integrate into the cluster.

3. Network Topology: Seed hosts can be strategically chosen to ensure proper network connectivity and resilience. By selecting seed hosts in different network segments or geographical regions, you can enhance the chances of new nodes discovering and forming connections with the existing cluster.

4. Dynamic Cluster Changes: Seed hosts aid in handling cluster changes dynamically. If the cluster undergoes changes, such as nodes leaving or new nodes joining, seed hosts can provide updated information to help the discovery process adapt to the evolving cluster topology.

In summary, while seed hosts that are not master-eligible nodes may not directly participate in cluster coordination and leadership, they play a vital role in facilitating the initial connection, discovery, and integration of new or rejoining nodes into the Elasticsearch cluster. They serve as entry points into the cluster, provide bootstrap information, and help handle dynamic changes within the cluster.

**********************************

## 集群选举
- 由 master-eligible 节点参与选举，且实际参与选举的节点数目要超过总共 master-eligible 节点数目的一半
- master-eligible 节点数目最好为奇数
例如 master-eligible 节点为 3 个，则损坏一个节点，剩下 2 个节点依旧能提供服务，再损坏一个节点，则剩下的节点数少于一半，无法工作
而如果 master-eligible 节点数为 4 个，损坏两个节点后，剩下的节点数仍然不超过一半，效果和 3 个 master-eligible 节点相同

### Quorum-based decision making
master-eligible 节点的工作为：
- electing a master node
- changing the cluster state


quorum is a subset of the master-eligible nodes in the cluster
初始的 quorum 可以从配置文件中获取，该配置参数仅仅再集群第一个启动时有用，后面新的节点加入到已经建立的集群时不需要



Elasticsearch allows you to add and remove master-eligible modes to a running cluster

A decision is made only after more than half of the nodes in the voting configuration have responded


## 集群分片 shards
> [Size your shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html)


- There is a limit to the amount of data you can store on a single node.
- A cluster with too many indices or shards is said to suffer from oversharding. 


## Cross-cluster replication
> [Cross-cluster replication](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-ccr.html)


### replication 的作用
- 高可用
- 负载均衡
- Reduce search latency by processing search requests in geo-proximity to the user

### active-passive model
- primary shard 可以读写
- replica shard 仅可读

## 集群故障转移
> [Failure handling](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html#_failure_handling)

当集群中有节点出现故障时，集群能进行自动修复，这个过程需要一定时间


## ES 文档路由
> [Routing](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-routing)

ES 文档是分布式存储，在 ES 集群访问或存储一个文档时，由特定算法决定文档放在哪个主分片中，再结合集群状态找到存放此主分片的节点主机


## 写文档
> [Basic write model](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html#basic-write-model)


- primary shard is not required to replicate to all replicas.
- Elasticsearch maintains a list of shard copies that should receive the operation. 
This list is called **in-sync** copies and is maintained by the master node.

Each primary stage will not complete until the in-sync replicas have finished indexing the docs locally and responded to the replica requests.


## 读文档
> [Basic read model](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html#_basic_read_model)



# 集群健康状态
## green

## yellow
有部分数据丢失，但完整数据还在

## red



# ES 文档路由
replica shards location


# 安装
- ubuntu22.04
- 3G 内存，2核 CPU


## 包安装
> [官方安装说明](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/deb.html)

### 安装 
- wget 从镜像网站下载 .deb 包
清华大学镜像网站：[elasticsearch](https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/e/elasticsearch/)

- dpkg -i 安装
```bash
[root@es-1 src]$ dpkg -i elasticsearch-8.8.2-amd64.deb
```

### 优化配置

- 实验环境需要修改 JVM heap size
编辑 `/etc/elasticsearch/jvm.options`
```bash
################################################################
## IMPORTANT: JVM heap size
################################################################
##
## The heap size is automatically configured by Elasticsearch
## based on the available memory in your system and the roles
## each node is configured to fulfill. If specifying heap is
## required, it should be done through a file in jvm.options.d,
## which should be named with .options suffix, and the min and
## max should be set to the same value. For example, to set the
## heap to 4 GB, create a new file in the jvm.options.d
## directory containing these lines:
##
-Xms512m
-Xmx512m
```

- 关闭 xpack 功能
编辑 `/etc/elasticsearch/elasticsearch.yml`

```bash
# Enable security features
xpack.security.enabled: false
```

### 开启服务
```bash
[root@es-1 src]$ systemctl daemon-reload
[root@es-1 src]$ systemctl enable --now elasticsearch.service
```

### 验证
- `ss -ntl` 看到 9200 的端口打开
- curl 访问 9200 端口
```bash
[root@es-1 src]$ curl http://127.0.0.1:9200/
{
  "name" : "es-1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "8L0Gmyj4RkGMS8taazKr7g",
  "version" : {
    "number" : "8.8.2",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "98e1271edf932a480e4262a471281f1ee295ce6b",
    "build_date" : "2023-06-26T05:16:16.196344851Z",
    "build_snapshot" : false,
    "lucene_version" : "9.6.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```


### 配置文件优化
> [module-network](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/modules-network.html)


# 访问 Elasticsearch


# Beats 收集数据
> [Beats](https://www.elastic.co/guide/en/beats/libbeat/current/beats-reference.html)

> Beats are open source data shippers that you install as agents on your servers to send operational data to Elasticsearch. 


Beats 是 data shipper，有很多种 Beats：
- filebeat 
收集日志

- Heartbeat
> [heartbeat](https://www.elastic.co/guide/en/beats/heartbeat/current/heartbeat-overview.html)

用于探测服务是否正常运行

- Metricbeat
> [metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-overview.html)

用于收集系统的一些性能指标

- Filebeat
> [filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html)

用于收集日志文件


# Filebeat 收集日志
> [filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html)

> Filebeat is a lightweight shipper for forwarding and centralizing log data. 
> Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, 
> collects log events, and forwards them either to Elasticsearch or Logstash for indexing.


## 安装
> [Filebeat quick start: installation and configuration](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html)


最好和 elasticsearch 相同版本

- 下载镜像安装包
> [filebeat](https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/8.x/pool/main/f/filebeat/)

- 安装
```bash
[root@docker src]$ ls
filebeat-8.8.2-amd64.deb  kibana-8.8.2-amd64.deb
[root@docker src]$ dpkg -i filebeat-8.8.2-amd64.deb
```


## Filebeat 工作原理
> [how filebeat works](https://www.elastic.co/guide/en/beats/filebeat/current/how-filebeat-works.html)

> Filebeat consists of two main components: `inputs` and `harvesters`. 
> These components work together to tail files and send event data to the output that you specify.


### harvester
- A harvester is responsible for reading the content of a single file
- It reads the content of a single file
- It reads each file, line by line, and sends the content to the output
- The harvester is responsible for opening and closing file
- If a file is removed or renamed while it's being harvested, Filebeat continues to read the file. So the space is reserved until the harvested closes
- By default, Filebeat keeps the file open until `close_inactive` is reached


filebeat 是单进程，多线程工作，一个 harvester 对应一个线程
```bash
[root@docker src]$ pstree -p | grep filebeat
           |-filebeat(907)-+-{filebeat}(964)
           |               |-{filebeat}(965)
           |               |-{filebeat}(966)
           |               |-{filebeat}(967)
           |               |-{filebeat}(1008)
           |               |-{filebeat}(1009)
           |               `-{filebeat}(1636)
```
```bash
[root@docker filebeat]$ cat /proc/907/status | grep -i threads
Threads:        8
```

注意 harvester 的限制：[Too many open file handlers](https://www.elastic.co/guide/en/beats/filebeat/current/open-file-handlers.html)


### input
> [input](https://www.elastic.co/guide/en/beats/filebeat/current/how-filebeat-works.html#input)

- An input is responsible for managing the harvesters and finding all sources to read from

- input type 有很多种，默认的为 filestream 代替旧的 log 类型，input 会匹配定义的全部文件，然后为每个文件开启一个 harvester，each input runs in its own Go rountine


### How does Filebeat keep the state of files
1. Filebeat 记录每个文件的状态信息
> Filebeat keeps the state of each file and frequently flushes the state to disk in the registry file
> The state is usesd to remember the last offset a harvester was reading from and to ensure all log lines are sent 
> While Filebeat is running, the state information is also kept in memory for each input

Filebeat 会跟踪文件的状态并且记住 harvester 最后读取的行的 offset，因此如果 Filebeat 和 Elasticsearch 或 Logstash 断开连接，下次恢复通信后能接着之前的读的位置继续日志文件

注意 registry file 不要太大：[Regisry file is too large](https://www.elastic.co/guide/en/beats/filebeat/current/reduce-registry-size.html)

registry file 的路径：
```bash
[root@docker filebeat]$ ls /var/lib/filebeat/registry/filebeat/log.json
/var/lib/filebeat/registry/filebeat/log.json
```

部分内容如下：
```bash
{
  "k": "filestream::nginx-error-log::native::3703423-64768",
  "v": {
    "cursor": {
      "offset": 2257
    },
    "meta": {
      "source": "/docker-02/web/nginx/logs/error.log",
      "identifier_name": "native"
    },
    "ttl": 1800000000000,
    "updated": [
      2062294143957,
      1688911593
    ]
  }
}
```
- Key (`k`): `"filestream::nginx-error-log::native::3703423-64768"`
  - This is the unique identifier for the file tracked by Filebeat. 
    It includes information such as the filestream type (`filestream`), 
    the identifier name (`native`), and other details specific to the log file.
- Value (`v`):
  - `cursor`: 
    - `offset`: 2257
      - This represents the offset or position in the file where Filebeat last read. 
  - `meta`:
    - `source`: "/docker-02/web/nginx/logs/error.log"
      - This specifies the path or source of the file being monitored. 
    - `identifier_name`: "native"
      - This identifies the source or type of the file. 
  - `ttl`: 1800000000000
    - This indicates the time-to-live value for the record. 
  - `updated`:
    - [2062294143957, 1688911593]
      - These are timestamp values representing the last time the record was updated. 
        The first value is the high-resolution timestamp, while the second value is the Unix timestamp.


2. For each file, Filebeat stores unique identifiers to detect whether a file was harvested previously

Filebeat 通过 inode 和 device 来跟踪一个文件，因此一旦 Filebeat 开始 harvesting 一个文件，就会记器其 inode number，即使后面文件名被修改甚至删除，Filebeat can still continue to harvest and read it as long as the file descriptor remains open

查看文件的 inode 和 device：
```bash
[root@docker filebeat]$ stat log.json
  File: log.json
  Size: 49408           Blocks: 112        IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 6204255     Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2023-07-11 10:25:14.693354468 +0800
Modify: 2023-07-11 10:18:13.652004008 +0800
Change: 2023-07-11 10:18:13.652004008 +0800
Birth: 2023-07-09 14:32:03.775167501 +0800
```



注意，如果一个文件被删了，然后新建一个文件的 inode 和旧文件相同，可能造成 Filebeat 认为新文件和旧文件相同，见说明：[Inode reuse causes Filebeat to skip lines](https://www.elastic.co/guide/en/beats/filebeat/current/inode-reuse-issue.html)

## 配置 

### inputs
> [Configure inputs](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html)


- 默认 `input_type` 为 `filestream`


# Logstash
> [Logstash](https://www.elastic.co/guide/en/logstash/current/introduction.html)


## 安装

- 从镜像网站下载安装包
> [logstash](https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/8.x/pool/main/l/logstash/)

```bash
[root@logstash src]$ wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/8.x/pool/main/l/logstash/logstash-8.8.2-amd64.deb
```

- 安装 
```bash
[root@logstash src]$ dpkg -i logstash-8.8.2-amd64.deb
```

- 为 logstash 创建软连接
```bash

```


## Logstash 命令

`logstash --help` 查看帮助，需要等一段时间
### logstatsh -f
指定 `.conf` 配置文件，要指明绝对路径，相对路径是相对于 `/usr/share/logstash` 的路径而非当前路径


## Input 插件
> [Input plugins](https://www.elastic.co/guide/en/logstash/8.8/input-plugins.html)


## Filter 插件
> [Filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)


### Grok 插件
> [Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#_description_136)
> [Do you grok Grok](https://www.elastic.co/cn/blog/do-you-grok-grok)


- Grok 可以将日志文件的内容解析为特定结构的内容，如 json 格式的数据
> Grok is a great way to parse unstructured log data into something structured and queryable.

- Logstash 提供了约 120 种 grok 模式，也可以自己定义模式