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

## elasticsearch 介绍
### inverted index 
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


### Data in: documents and indices

- An index can be thought of as an optimized collection of documents and each document is a collection of fields, which are the key-value pairs that contains your data.

- By default, Elasticsearch indexes all data in every field and each indexed field has a dedicated, optimized data structure. 

For example, text fields are stored in inverted indices, and numeric and geo fields are stored in BKD trees. 

The ability to use the per-field data structures to assemble and return search results is what makes Elasticsearch so fast.

- Elasticsearch has the ability to be schema-less

一个 document 中可以有多个不同的 fields，且允许 dynamic mapping，即 elasticsearch 能自动检测并添加新的 field


### Scalability and resilience: cluster, nodes, and shards

- Elasticsearch is built to be always available and to scale with your needs. It does this by being distributed by nature.


注意平衡 shards 的尺寸和数量，lots of small shards or a smaller number of larger shards
> [Quantitative Cluster Sizing](https://www.elastic.co/cn/elasticon/conf/2016/sf/quantitative-cluster-sizing)


- CCR (Cross cluster replication) provides a way to automatically synchronize indices from your primary cluster to a secondary remote cluster that can serve as a hot backup.

CCR is active-passive.
The index on the primary cluster is the active leader index and handles all write requests.
Indeices replicated to secondary clusters are read-only followers.


### Secure, manage and monitor elasticsearch cluster
- Kibana can manage a cluster
- [index lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) can manage data over time


## elasticsearch 安装


## 包安装
> [官方安装说明](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/deb.html)


环境：ubuntu22.04 虚拟机，3G 内存，2核

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

## docker 部署 ES 集群
> [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/docker.html)


根据官方提示，要用 docker 安装多节点集群，至少需要 4G 内存

实验环境：ubunt22.04 虚拟机，安装 eleasticsearch 集群，8G 内存，2核


### 安装前准备
- 禁用 swap 
编辑 /etc/fstab 将 swap 的行注释
```bash
# /etc/fstab: static file system information.
#
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-aL2z4qRF6RP6ygmuXVQ89hX7MrChz0jPApnaIZ1WSFWWkEAhejDEX9zi7SrwDOep / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/f81cc5f3-9189-4a0c-b53a-f67110325cc7 /boot ext4 defaults 0 1
# /swap.img	none	swap	sw	0	0
```

使配置生效：
```bash
[root@es docker]$ swapoff -a
```

查看内存：
```bash
[root@es docker]$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       3.1Gi       3.6Gi       1.0Mi       1.1Gi       4.2Gi
Swap:             0B          0B          0B
```


- 安装前先修改宿主机中的内核参数
```bash
[root@es-cluster ~]$ cat /proc/sys/vm/max_map_count
65530
[root@es-cluster ~]$ echo "vm.max_map_count=262144" >> /etc/sysctl.conf
[root@es-cluster ~]$ sed -n '/vm.max_map_count/p' /etc/sysctl.conf
vm.max_map_count=262144
[root@es-cluster ~]$ sysctl -p
```

- 安装 docker
> [Install Docker Engine](https://docs.docker.com/engine/install/)


- 安装 docker compose
> [Install docker compose](https://docs.docker.com/compose/install/)


从镜像网站下载想要安装的包
> [清华大学镜像源](https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/dists/jammy/pool/stable/amd64/)

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/dists/jammy/pool/stable/amd64/docker-compose-plugin_2.19.1-1~ubuntu.22.04~jammy_amd64.deb
```

dpkg  安装
```bash
 dpkg -i docker-compose-plugin_2.19.1-1~ubuntu.22.04~jammy_amd64.deb
```


### 制作 elasticsearch 镜像


#### 自己制作 elasticsearch 镜像
- 用 ubuntu22.04 做基础镜像
- 下载 .deb 安装包到镜像中，用 dpkg -i 安装
- 未做


#### 用官方提供的镜像
> 官方镜像：[elasticsearch](https://www.docker.elastic.co/r/elasticsearch)
> 官方镜像的 Dockerfile：[elasticsearch](https://github.com/elastic/elasticsearch/tree/main/distribution/docker/src/docker)


- 官方镜像默认进入容器后 id 为 elasticsearch(1000) 而非 root，且没有 sudo 执行权限

官方的镜像可以运行一个 elasticsearch 容器，需要指明环境变量为单节点，否则默认集群模式不能正常运行

```bash
docker run --name es-01 -d -p 9200:9200 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.8.2
```


### docker compose 部署 elasticsearch 集群
> 官方介绍：[Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/docker.html)
> 多节点集群：[Start a multi-node cluster with Docker Compose](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/docker.html#docker-compose-file)
> 官方提供的 docker-compose.yml 文件：[docker-compose.yml](https://github.com/elastic/elasticsearch/tree/7.5/distribution/docker)
> 测试用：[docker-compose.yml](https://github.com/elastic/elasticsearch/tree/main/distribution/docker)
> [docker-compose.yml](https://github.com/elastic/elasticsearch/tree/main/docs/reference/setup/install/docker)
> docker compose 官方文档：[compose-file](https://docs.docker.com/compose/compose-file/05-services/)


- 注意旧的 docker-compose 命令改为 docker compose，见官方说明：[Migrate to Compose V2](https://docs.docker.com/compose/migrate/)


- 将官方的 [docker-compose.yml](https://github.com/elastic/elasticsearch/tree/main/docs/reference/setup/install/docker) 文件和对应的环境配置文件 .env 下载下来，然后用 docker compose 命令执行

官方的 docker-compose.yml 开启了 xpack 功能，实验删除该部分
官方还提供 kibana，但 kibana 运行一段时间就退出了，暂时去掉 kibana
进入 docker-compose.yml 所在的目录用 `docker compose logs kibana` 查看日志发现，kibana 需要 es 的密码，没有密码因此退出
```bash
docker-kibana-1  |  FATAL  Error: [config validation of [elasticsearch].password]: expected value of type [string] but got [number]
```


简化如下：
```bash
version: "2.2"

services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - ${ES_PORT}:9200
    environment:
      - node.name=es01
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es02,es03
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1

  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - esdata02:/usr/share/elasticsearch/data
    environment:
      - node.name=es02
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es01,es03
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - esdata03:/usr/share/elasticsearch/data
    environment:
      - node.name=es03
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es01,es02
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1

volumes:
  esdata01:
    driver: local
  esdata02:
    driver: local
  esdata03:
    driver: local
```

里面用到的环境变量单独写在一个 env.sh 的文件中:
```bash
# Version of Elastic products
STACK_VERSION=8.8.2

# Set the cluster name
CLUSTER_NAME=es-docker-cluster

# Port to expose Elasticsearch HTTP API to the host
ES_PORT=9200
#ES_PORT=127.0.0.1:9200

# Increase or decrease based on the available host memory (in bytes)
MEM_LIMIT=1073741824
```

写一个运行脚本 run.sh：
```bash
#!/bin/bash 

. env.sh
docker compose up -d
```

### 验证 ES 集群
- 查看镜像是否正常运行
```bash
[root@es docker]$ docker ps -a
```

- 查看集群的健康状态
```bash
[root@es docker]$ curl -sXGET http://10.0.0.201:9200/_cluster/health?pretty=true
{
  "cluster_name" : "es-docker-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 1,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

- 查看集群中的各节点
```bash
[root@es docker]$ curl -sXGET http://10.0.0.201:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.18.0.2           30          89   5    0.13    0.17     0.25 cdfhilmrstw -      es01
172.18.0.4           32         100   5    0.13    0.17     0.25 cdfhilmrstw *      es02
172.18.0.5           50         100   5    0.13    0.17     0.25 cdfhilmrstw -      es03
```


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


### 读请求处理过程
- When a read request is received by a node, that node is responsible for forwarding it to the nodes that hold the relevant shards, collating the responses, and responding to the client.

当一个有 cooridinating 角色的节点收到读请求，而读操作涉及到从不同的节点获取 shards 的数据时，the node is responsible for forwarding the request to the relavant shard-holding nodes，从其他有分片的节点处收集到数据进行整合，最后响应客户端最终的结果


### 分片选择
- By default, Elasticsearch uses adaptive replica selection to select the shard copies.
一般一个分片会在其他节点上有 replica，默认 elasticsearch 使用 [Adaptive replica selection](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-shard-routing.html#search-adaptive-replica) 来选择从哪个分片进行读取，因为 primary 和 replica 分片都能提供读服务


- 如果一个分片读取失败，coordinating node 将向有其他分片的节点发送请求
为了保证快速响应，有些 API 会提供当一个或多个分片无法响应时仍返回部分结果：Search，Multi Search，Multi Get


### 读取失败情况
> [Failures](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html#_failures)


- primary 分片会等全部 in -sync 副本分片复制成功，因此如果有一个分片速度很慢，可能会影响 indexing 速度

- dirty reads
The primary shard becomes isolated but is not immediately aware of this isolation，因此 primary shard 和 replica shards 无法通信但仍然执行写操作

when a concurrent read request is made after the write operation has been indexed on the isolated primary shard but before the primary shard realizes it is isolated and syncs with the replicas or master. In this scenario, the read request may retrieve the data from the isolated primary shard, leading to inconsistencies or data loss.

Elasticsearch mitigates this risk by pinging the master every second (by default) and rejecting indexing operations if no master is known.


## ES 集群健康状态
> [Cluster health](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html#cluster-health-api-response-body)

- 查看集群的健康状态
```bash
[root@es docker]$ curl -sXGET http://10.0.0.201:9200/_cluster/health?pretty=true
{
  "cluster_name" : "es-docker-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 1,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

上面的 `status` 显示集群的健康状态

### green
- All shards are assinged

### yellow
- All primary shards are assigned, but one or more replica shards are unassinged

If a node in the cluster fails, some data could be unavailable until that node repaired

### red
- one or more primary shards are unassigned, so some data is unavailable



## 访问 Elasticsearch
> [Beginner's Guide to Elasticsearch API: Indexing and Searching Data](https://www.atatus.com/blog/rest-apis-of-elasticsearch-an-overview/)
> 官方介绍：[REST APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)


### CAT APIs
> [Compact and aligned text (CAT) APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html)


官方文档提供例子，可以设置服务器端地址，然后 Copy as curl 到终端执行：
```bash
[root@es docker]$ curl -X GET "10.0.0.201:9200/_cat/master?v=true&pretty"
id                     host       ip         node
qV3pPgSPQjOnmy8nG8KT2A 172.18.0.4 172.18.0.4 es02
```

### Cluster APIs
> [Cluster APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html#cluster-health-api-response-body)


## Elasticsearch 插件
使用一些插件更好的查看 Elasticsearch 的状态

### Head 插件
> [elasticsearch-head](https://github.com/mobz/elasticsearch-head)


可以直接在浏览器安装插件，如在 Google chrome 浏览器中安装 Multi Elasticsearch Head 插件


### Cerebro 插件
> [cerebro](https://github.com/lmenezes/cerebro)


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

安装在需要收集日志的系统上
最好和 elasticsearch 相同版本

### 包安装
- 下载镜像安装包
> [filebeat](https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/8.x/pool/main/f/filebeat/)

- 安装
```bash
[root@docker src]$ ls
filebeat-8.8.2-amd64.deb  
[root@docker src]$ dpkg -i filebeat-8.8.2-amd64.deb
```


### docker 安装
> [Run Filebeat on Docker](https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html)

- filebeat 和 elasticsearch 都用相同的版本 8.8.2


## Filebeat 工作原理
> [how filebeat works](https://www.elastic.co/guide/en/beats/filebeat/current/how-filebeat-works.html)

> Filebeat consists of two main components: `inputs` and `harvesters`. 
> These components work together to tail files and send event data to the output that you specify.


### harvester
- A harvester is responsible for reading the content of a single file
- It reads the content of a single file
- It reads each file, line by line, and sends the content to the output
- The harvester is responsible for opening and closing file
- If a file is removed or renamed while it's being harvested, Filebeat continues to read the file. 
  So the space is reserved until the harvested closes
- By default, Filebeat keeps the file open until `close_inactive` is reached


filebeat 是单进程，多线程工作，但一个 harvester 并非对应一个线程
```bash
[root@docker src]$ pstree -p | grep filebeat
           |-filebeat(907)-+-{filebeat}(964)
           |               |-{filebeat}(965)
           |               |-{filebeat}(966)
           |               |-{filebeat}(967)
           |               |-{filebeat}(1008)
           |               |-{filebeat}(1009)
           |                -{filebeat}(1636)
```
```bash
[root@docker filebeat]$ cat /proc/907/status | grep -i threads
Threads:        8
```

- 什么是 harvester？
The number of threads does not necessarily equal the number of harvesters in Filebeat. 
Each harvester in Filebeat is associated with its own goroutine (a lightweight thread) to handle the harvesting process for a specific file. 
However, Filebeat may use additional goroutines for other purposes such as network communication, internal processing, or handling events.

In Filebeat, a goroutine is a lightweight thread of execution that is used to perform concurrent tasks. 
Goroutines are part of the Go programming language, which is the language in which Filebeat is implemented.


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


## filebeat 配置
filebeat 包安装可以用 systemd 管理，默认的配置文件为 `/etc/filebeat/filebeat.yml`
filebeat 配置文件的格式介绍见：[Config file format](https://www.elastic.co/guide/en/beats/libbeat/8.8/config-file-format.html)


### 配置文件中使用环境变量
> [Environment variables](https://www.elastic.co/guide/en/beats/libbeat/8.8/config-file-format-env-vars.html)

- 可以再配置文件中使用环境变量，如指定 hosts 时用环境变量


### inputs
> [Configure inputs](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html)

- 默认 `input_type` 为 `filestream`
- 可以配置不同的输入类型


#### stdin 标准输入读取数据
> [Stdin input](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-stdin.html)

- 从标准输入读取数据处理


#### Multiline messages 管理多行内容
> [Manage multiline messages](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html)

- 有时一个事件可能生成多行日志，如 java 应用，因此需要将多行合并为一个日志给 filebeat 收集，
  防止一个错误最后生成多行 ES 文档记录

如：
```bash
parsers:
- multiline:
    type: pattern
    pattern: '^\['
    negate: true
    match: after
```

筛选日志文件的模式为以 `[` 开头的行，negate 为 true 表示不匹配该模式的行，
match 为 after，表示不匹配该模式的行追加到匹配该模式的行后面合并为一行

见官方文档中图示


#### container input
> [Container input](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-container.html)


#### filestream input
> [filestream input](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-filestream.html)


- 从 active log file 中读取数据，代替旧的 log 类型，默认类型


```bash
filebeat.inputs:
# filestream is an input for collecting log messages from files.
- type: filestream
  # Unique ID among all inputs, an ID is required.
  id: nginx-access-log
  # Change to true to enable this input configuration.
  enabled: true
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /docker-02/web/nginx/logs/access_json.log
  tags: ["nginx-access"]
  parsers:
    - ndjson:
      target: ""
      add_error_key: true
      message_key: log
  #fields:
  #  level: debug
  #  review: 1

- type: filestream
  id: nginx-error-log
  enabled: true
  paths:
    - /docker-02/web/nginx/logs/error.log
  tags: ["nginx-error"]
  parsers:
    - ndjson:
      target: ""
      add_error_key: true
      message_key: log
```


### output 
> [Configure the output](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-output.html)


- 从 filebeat 收集的日志可以输出到 redis，kafka，logstash，elasticsearch 或 console

#### redis
> [Configure the Redis output](https://www.elastic.co/guide/en/beats/filebeat/current/redis-output.html)


- 可以配置多个 redis host 地址，或者用环境变量代替
- 可以利用 inputs 模块定义的 tags 将访问日志和错误日志分别存放在 nginx 不同的 key 中 

```bash
# ------------------------------ Redis Output -------------------------------
output.redis:
  hosts: 
    - "10.0.0.208:6370"
    - "10.0.0.208:6371"
    - "10.0.0.208:6372"
  password: "123456"
  db: 0
  timeout: 5
  key: "nginx"
  keys:
    - key: "nginx-access"   
      when.contains:
        tags: "nginx-access"
    - key: "nginx-error"  
      when.contains:
        tags: "nginx-error"
```



# Logstash
> [Logstash](https://www.elastic.co/guide/en/logstash/current/introduction.html)


- logstash 可以动态的将多种不同来源的数据进行处理，过滤，统一格式发送给目的地
- logstash 可以水平伸缩，插件多

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
从 `/usr/lib/systemd/system/logstash.service` 文件中查看二进制文件的路径
```bash
ln -sv /usr/share/logstash/bin/logstash /usr/local/sbin/logstash
```

## Logstash 工作原理
> [How logstash works](https://www.elastic.co/guide/en/logstash/current/pipeline.html)


> The Logstash event processing pipeline has three stages: inputs → filters → outputs. Inputs generate events, filters modify them, and outputs ship them elsewhere.


## Logstash 配置文件
> [Logstash Configuration File](https://www.elastic.co/guide/en/logstash/current/config-setting-files.html)


> Logstash has two types of configuration files: pipeline configuration files, which define the Logstash processing pipeline, and settings files, which specify options that control Logstash startup and execution.


修改配置文件后通过 `systemctl restart logstash` 重启


### Pipeline Configuration Files
> [Creating a Logstash pipeline](https://www.elastic.co/guide/en/logstash/current/configuration.html)

- pipeline 配置文件用于配置 logstash 怎么收集数据以及过滤等处理流程
- 管道配置文件放在 `/etc/logstash/conf.d` 目录下，创建的的以 `.conf` 结尾的文件都会被加载


#### pipeline 配置文件的格式
> [Structure of a pipeline](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html)


配置文件中值的类型有多种

```bash
path => [ "/var/log/messages", "/var/log/*.log" ]
host => "10.0.0.208"
```


### Settings Files

#### logstash.yml
> [logstash.yml](https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html)

logstash 配置文件

如：
```bash
# ------------  Node identity ------------
#
# Use a descriptive name for the node:
#
node.name: logstash-node01
#
# ------------ Data path ------------------
#
# Which directory should be used by logstash and its plugins
# for any persistent needs. Defaults to LOGSTASH_HOME/data
#
path.data: /var/lib/logstash
```


#### pipelines.yml
Contains the framework and instructions for running multiple pipelines in a single Logstash instance.

```bash
[root@logstash logstash]$ cat pipelines.yml
# This file is where you define your pipelines. You can define multiple.
# For more information on multiple pipelines, see the documentation:
#   https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
  path.config: "/etc/logstash/conf.d/*.conf"
```

如上面定义管道配置文件会加载的文件


#### jvm.options
> [JVM settings](https://www.elastic.co/guide/en/logstash/current/jvm-settings.html)


JVM 配置，因为 logstash 是基于 Java 和 Ruby 语言开发

根据实际情况设置 heap space，设置依据参考官方介绍：[Setting the JVM heap size](https://www.elastic.co/guide/en/logstash/current/jvm-settings.html#heap-size)
```bash
## JVM configuration

# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms1g
-Xmx1g
```

#### log4j2.properties
> [Log4j2 configuration](https://www.elastic.co/guide/en/logstash/current/logging.html#log4j2)


## 修改 Logstash 用户
`systemctl status logstash.service` 查看 service 文件的路径
将 User 和 Group 修改为 root，防止 logstash 收集本机一些日志时权限不够

修改 service 文件后通过 `systemctl daemon-reload` 和 `systemctl restart logstash` 使其生效

## Logstash 命令
> [Running Logstash from the Command Line](https://www.elastic.co/guide/en/logstash/current/running-logstash-command-line.html)

`logstash --help` 查看帮助，需要等一段时间


### logstatsh -f 指定配置文件
指定 `.conf` 配置文件，要指明绝对路径，相对路径是相对于 `/usr/share/logstash` 的路径而非当前路径


### logstash -e 指定配置内容
直接将配置文件的内容写在命令行


### logstash -t 配置文件语法检查
- Check configuration for valid syntax and then exit
- Note that grok patterns are not checked for correctness with this flag

### logstash -r 重新加载配置文件
Monitor configuration changes and reload whenever the configuration is changed. 
NOTE: Use SIGHUP to manually reload the config.


## 创建 logstash pipeline
> [Creating a Logstash pipeline](https://www.elastic.co/guide/en/logstash/current/configuration.html)
> 配置例子：[Logstash configuration examples](https://www.elastic.co/guide/en/logstash/current/config-examples.html)


### Structure of a pipeline
> [Structure of a pipeline](https://www.elastic.co/guide/en/logstash/8.8/configuration-file-structure.html#array) 



### Accessing event data and fields
> [Accessing event data and fields](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#conditionals)


- Inputs generate events, filters modify them, and outputs ship them

- All events have properties
例如 nginx 访问日志有 status code，request path，HTTPverb (GET, POST) 等
Logstash 称这些 properties 为 fields

inputs 阶段没有 fields，因为 inputs 产生 events

inputs blocks 中也没有条件判断等，因为条件判断依赖 events 和 fields

#### Field references
> [Field references](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#logstash-config-field-references)


例如 event：
```bash
{
  "agent": "Mozilla/5.0 (compatible; MSIE 9.0)",
  "ip": "192.168.24.44",
  "request": "/index.html"
  "response": {
    "status": 200,
    "bytes": 52353
  },
  "ua": {
    "os": "Windows 7"
  }
}
```
- To reference top-level field as `agent`, just specify the fieldname
- To reference the `os` field, specify `[ua][os]`


#### sprintf format
> [sprintf format](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#sprintf)

- sprintf format enables you to embed field values in other strings

```bash
output {
  statsd {
    increment => "apache.%{[response][status]}"
  }
}

output {
  statsd {
    increment => "apache.%{[response][status]}"
  }
}
```

#### 条件判断 Conditionals
> [Conditionals](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#conditionals)


```bash
if EXPRESSION {
  ...
} else if EXPRESSION {
  ...
} else {
  ...
}
```

```bash
filter {
  if [action] == "login" {
    mutate { remove_field => "secret" }
  }
}
```
上面的 `action` 为 field

例子见：[Using conditionals](https://www.elastic.co/guide/en/logstash/current/config-examples.html#using-conditionals)


#### The @metadata field
> [The @metadata field](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#metadata)


- `@metadata` field 不会在 output 阶段作为 events 的一部分，可以用于条件判断，但不会输出该字段的内容

### 使用环境变量
> [Using environment variables Overview](https://www.elastic.co/guide/en/logstash/current/environment-variables.html) 


```bash
export TCP_PORT=12345
```
```bash
input {
  tcp {
    port => "${TCP_PORT}"
  }
}
```

## Input 插件
> [Input plugins](https://www.elastic.co/guide/en/logstash/8.8/input-plugins.html)


### Redis input plugin
> [Redis input plugin](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-redis.html)


- redis 输入插件中 host 配置多个主机出错
```bash
input {
    redis {
        host => [ "10.0.0.208:3670", "10.0.0.208:3671", "10.0.0.208:3672" ]
        password => "123456"
        db => "0"
        data_type => 'list'
        key => "nginx"
    }
}

output {
    stdout {
        codec => json
    }
    file {
        path => "/tmp/test.log"
    }
}
```

- 写 redis 从节点的 host 和 port 也会出错，因为 slave 节点是 readonly

- 写 redis master 节点，在 host 中指定 iP 和 端口，不单独写 port，也会出错，必须写 host 和 port
```bash
input {
	redis {
        host => "10.0.0.208:6372"
        #port => "6372"
		password => "123456"
		db => "0"
		data_type => 'list'
		key => "nginx"
	}
}
```
从官方文档可知，host 的 value type 为 string，port 的 value type 为 number

- 将 redis master 和 slave 节点分开都写到 input 插件中
三个 redis server 用 docker 运行，宿主机为 10.0.0.208，对外曝露端口分别为 6370 6371 6372

```bash
input {
	redis {
        host => "10.0.0.208"
        port => "6370"
		password => "123456"
		db => "0"
		data_type => 'list'
		key => "nginx"
	}
	redis {
        host => "10.0.0.208"
        port => "6371"
		password => "123456"
		db => "0"
		data_type => 'list'
		key => "nginx"
	}
	redis {
        host => "10.0.0.208"
        port => "6372"
		password => "123456"
		db => "0"
		data_type => 'list'
		key => "nginx"
	}
}
```

其中最后一个端口 6372 对应的是 master 节点，其余两个为 slave 节点
配置后 logstash 可以从 redis 获取数据，但会有错误提示，因为写的两个 slave 节点不能写数据


- 写 master 节点可以成功获取数据，logstash 消费了 redis 中的数据，redis 中就无数据了
```bash
input {
	redis {
        host => "10.0.0.208"
        port => "6372"
		password => "123456"
		db => "0"
		data_type => 'list'
		key => "nginx"
	}
}

output {
    stdout {
        codec => json
    }
    file {
        path => "/tmp/test.log"
    }
}
```

- host 和 port 等可以写成环境变量


## Filter 插件
> [Filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)






### Grok 插件
> [Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#_description_136)
> [Do you grok Grok](https://www.elastic.co/cn/blog/do-you-grok-grok)


- Grok 可以将日志文件的内容解析为特定结构的内容，如 json 格式的数据
> Grok is a great way to parse unstructured log data into something structured and queryable.

- Logstash 提供了约 120 种 grok 模式，也可以自己定义模式


#### Grok 基本语法
> [Grok Basics](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#_grok_basics)


##### 基本语法：`%{SYNTAX:SEMANTIC}`
`SYNTAX` 为要匹配的内容，即写模式
`SEMANTIC` 为自定义的名区分几种匹配模式：WROD, DATA and GREEDYDATA
如：`%{NUMBER:duration} %{IP:client}` 中匹配 `NUMBER` 模式的文本，将该字段命令为 `duration`
如模式 `WORD` 表示单词，可以匹配多个字段，但根据实际内容的含义，第一个匹配的文本命名为 `a`，即 `SEMANTIC` 为 `a`，而第二个匹配的文本命名为 `b`


注意区分几种匹配模式：WROD, DATA and GREEDYDATA，各个模式的定义见 [grok-patterns](https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/grok-patterns)

```bash
WORD \b\w+\b
DATA .*?
GREEDYDATA .*
```

`GREEDYDATA` 为贪婪匹配


1. 示例一
例如匹配下面内容：
```bash
[2023-07-26 10:30:45] [ERROR] This is an error message: Invalid input detected.
```

使用如下 grok pattern
```bash
\[%{DATA:timestamp}\] \[%{WORD:severity}\] %{GREEDYDATA:message}
```

最后匹配到的内容如下：
- `timestamp`: "2023-07-26 10:30:45"
- `severity`: "ERROR"
- `message`: "This is an error message: Invalid input detected."


2. 示例二
```bash
*1 open() "/data/www/index.htmls" failed (2: No such file or directory), client: 10.0.0.1, server: localhost, request: "GET /index.htmls HTTP/1.1", host: "10.0.0.208:8080"
```

如果匹配模式为：
```bash
grok {
    match => { 
        "message" => "%{DATA:error_message}," 
    }
}
```
则匹配到的内容为 `*1 open() "/data/www/index.htmls" failed (2: No such file or directory),`

如果用贪婪的匹配：
```bash
grok {
    match => { 
        "message" => "%{GREEDYDATA:error_message}," 
    }
}
```
则匹配到第一个逗号后会继续向后匹配，即匹配到最后一个逗号处


##### 模式 pattern
`NUMBER` 模式可以从官方提供的模式中查看：[grok-patterns](https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/grok-patterns#LL13)

查看文件夹 ecs-v1 中的内容

##### 正则表达式
pattern 的定义为正则表达式，grok 使用的 regular expression library is Oniguruma，见 [oniguruma](https://github.com/kkos/oniguruma/blob/master/doc/RE)


##### 自定义模式 Custom Patterns
如果 grok 官方中没有符合的模式，可以自己定义模式，模式使用 oniguruma 语法

```bash
(?<field_name>the pattern here)
```
如：
```bash
 (?<queue_id>[0-9A-F]{10,11})
```

自定义的模式写在一个自定义模式文件中，模式文件在目录 `patterns` 目录中，文件名随意，如 `postfix`：
```bash
# contents of ./patterns/postfix:
POSTFIX_QUEUEID [0-9A-F]{10,11}
```
自定义一种 pattern `POSTFIX_QUEUEID`，后面的正则表示式表示匹配模式

使用自定义模式时先指明模式的路径，用 `patterns_dir` 指明路径
```bash
filter {
    grok {
      patterns_dir => ["./patterns"]
      match => { "message" => "%{SYSLOGBASE} %{POSTFIX_QUEUEID:queue_id}: %{GREEDYDATA:syslog_message}" }
    }
  }
```
上面是官方例子，实际测试时（用 logstash -f 指定文件方式）自定义模式文件所在的目录最好写上绝对路径


#### match 选项
> [match](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#plugins-filters-grok-match)


```bash
filter {
      grok {
        match => {
          "speed" => "Speed: %{NUMBER:speed}"
          "duration" => "Duration: %{NUMBER:duration}"
        }
      }
    }
```
- `match` 选项匹配两个 field：speed 和 duration
其中 speed 字段中又进行匹配，格式为 `Speed: 数字`，数字命名为 speed


- 匹配多个模式
```bash
filter {
    grok {
      match => {
        "message" => [
          "Duration: %{NUMBER:duration}",
          "Speed: %{NUMBER:speed}"
        ]
      }
    }
  }
```
按照官方写法，对于 nginx 错误日志，用下面的筛选方法：
```bash
2023/07/26 11:39:36 [error] 13#0: *1 open() "/data/www/abc" failed (2: No such file or directory), 
client: 10.0.0.1, server: localhost, request: "GET /abc HTTP/1.1", host: "10.0.0.208:8080"
```

用下面的筛选方法：
```bash
filter {
    if [tags][0] == "nginx-error" {
        grok {
            match => { 
                "message" => [
                    "client: %{IP:client_ip}, server: %{HOSTNAME:server_name}, ",
                    "request: \"%{WORD:request_method} %{URIPATH:requested_page} %{DATA:request_protocol}\", "
                ]
            }
        }
    }
    else if [tags][0] == "nginx-access" {
        drop { }
    }
}
```
不能筛选出 `"request: \"%{WORD:request_method} %{URIPATH:requested_page} %{DATA:request_protocol}\", "` 部分，
单独只写其中一行时都可以筛选，合并为一行也可以筛选，但写在一行太长




#### overwrite 选项
> [overwrite](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#plugins-filters-grok-overwritev)


```bash
filter {
      grok {
        match => { "message" => "%{SYSLOGBASE} %{DATA:message}" }
        overwrite => [ "message" ]
      }
    }
```


#### Drop 插件
> [Drop filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-drop.html)


可以筛选一些数据丢弃不输出到 elasticsearch 或其他地方，但不影响输入，如输入来源为 redis，logstash 还是会消费 redis 的数据，然后在过滤阶段丢弃，如：
```bash
input {
	redis {
        host => "${REDIS_HOST}"
        port => "${REDIS_PORT}"
        password => "123456"
        db => "0"
        data_type => 'list'
        key => "nginx-access"
        tags => "nginx-access"
	}
	redis {
        host => "${REDIS_HOST}"
        port => "${REDIS_PORT}"
        password => "123456"
        db => "0"
        data_type => 'list'
        key => "nginx-error"
        tags => "nginx-error"
	}
}

filter {
    if [tags][0] == "nginx-error" {
        grok {
            patterns_dir => ["/etc/logstash/conf.d/patterns"]
            match => { 
                "message" => "%{NGINX_ERROR_LOG}" 
            }
        }
    }
    else if [tags][0] == "nginx-access" {
        drop { }
    }
}
```

`tags` 选项为 array 类型，因此用 tags[0]，将 nginx 访问日志丢弃，只处理 nginx 错误日志




### 例子
> [Logstash configuration examples](https://www.elastic.co/guide/en/logstash/current/config-examples.html)


#### 处理 nginx 错误日志
用 nginx 的默认错误日志，其内容如下：
```bash
2023/07/26 11:39:36 [error] 13#0: *1 open() "/data/www/abc" failed (2: No such file or directory), client: 10.0.0.1, 
server: localhost, request: "GET /abc HTTP/1.1", host: "10.0.0.208:8080"
```

在 logstash 的过滤配置文件目录下创建一个目录 patterns，在该自定义目录中创建一个文件 cus_patterns，
该文件中定义 nginx 错误日志的格式：
```bash
NGINX_ERROR_LOG %{DATESTAMP:timestamp} \[%{LOGLEVEL:loglevel}\] %{NUMBER:process_id}#%{NUMBER:connection_id}: %{DATA:error_message}, client: %{IP:client_ip}, server: %{HOSTNAME:server_name}, request: \"%{WORD:request_method} %{URIPATH:requested_page} %{DATA:request_protocol}\", host: \"%{HOSTPORT:host_port}\"
```

定义环境变量：
```bash
#!/bin/bash 

export REDIS_HOST="10.0.0.208"
export REDIS_PORT="6372"
```

然后过滤时用自定义的模式：
```bash
input {
	redis {
        host => "${REDIS_HOST}"
        port => "${REDIS_PORT}"
		password => "123456"
		db => "0"
		data_type => 'list'
		key => "nginx-access"
	}
	redis {
        host => "${REDIS_HOST}"
        port => "${REDIS_PORT}"
		password => "123456"
		db => "0"
		data_type => 'list'
		key => "nginx-error"
	}
}

filter {
    if [tags][0] == "nginx-error" {
        grok {
            patterns_dir => ["/etc/logstash/conf.d/patterns"]
            match => { 
                "message" => "%{NGINX_ERROR_LOG}" 
            }
        }
    }
    else if [tags][0] == "nginx-access" {
        drop { }
    }
}

output {
    stdout {
        codec => json
    }
    file {
        path => "/tmp/test.log"
    }
}
```
在 logstash 的 redis input plugin 中不用定义 tags，因为 redis 中已经定义了 tags



