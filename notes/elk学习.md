ELK 学习

# Elastic stack
> [Elastic stack](https://www.elastic.co/guide/index.html)

> It’s a fast and highly scalable set of components — Elasticsearch, Kibana, Beats, Logstash, 
> and others — that together enable you to securely take data from any source, in any format, 
> and then search, analyze, and visualize it.


包含三大部分：
- Ingest
  收集日志，securely take data from any source, in any format
  - collect and ship data
  例如 Beats，初步收集日志
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
# 节点角色
> [moduels-node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)

## Master node

## data node


最终一致性


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
- Filebeat keeps the state of each file and frequently flushes the state to disk in the registry file

- The state is usesd to remember the last offset a harvester was reading from and to ensure all log lines are sent 

- While Filebeat is running, the state information is also kept in memory for each input

Filebeat 会跟踪文件的状态并且记住 harvester 最后读取的行的 offset，因此如果 Filebeat 和 Elasticsearch 或 Logstash 断开连接，下次恢复通信后能接着之前的读的位置继续日志文件
```bash
[root@docker filebeat]$ ls /var/lib/filebeat/registry/filebeat/log.json
/var/lib/filebeat/registry/filebeat/log.json
```

- For each file, Filebeat stores unique identifiers to detect whether a file was harvested previously


## 配置 

### inputs
> [Configure inputs](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html)


- 默认 `input_type` 为 `filestream`