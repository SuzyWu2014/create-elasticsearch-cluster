# Configuration

下面来介绍 ES 配置文件中的参数设置，如果你要将 ES 用于 production，那么以下的参数就显得尤为重要。

## Cluster 和 Node 的名字

```yaml
# ======================== Elasticsearch Configuration =========================
cluster.name: my-application
node.name: node-1
```

## 路径

一般来说不需要改变默认设置，只有当你的

```yaml
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
# path.data: /var/data/elasticsearch
#
# Path to log files:
#
# path.logs: /var/log/elasticsearch
```

## Gateway

当 cluster 中的 node 离线时，ES 会在正常的 nodes 里产生新的 shard/replica, 但当离线的 node 恢复正常，ES 就会启动 rebalencing, 先将之前离线的 node 里的 shards/replicas 都删除，因为他们极有可能已经不同步， 然后将新的 Shards/replica 再移动到恢复正常的 nodes 里。这个过程非常消耗性能，所以我们要设置他在满足一定的条件之后才能启动这个恢复过程。

```yaml
gateway.recover_after_nodes: 2  # 只有当 2 个 node 同时在线时，才会启动 recovery
gateway.expected_nodes: 3       # Cluster 中一共有几个 nodes
gateway.recover_after_time: 3m  # 如果一个 node 离线超过 3 分钟就还是 recovery
```

## Network

```yaml
# --------------------------------- Discovery ----------------------------------

# 防止将不属于这个 cluster 的 node 也添加进来，造成数据混乱
# hosts 设置成 cluster 中各个 node 的 IP，所有的 node 的设置应该是一样的
discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]
discovery.zen.ping.multicast.enable: false


# Prevent the "split brain" by configuring the majority of nodes (total number of nodes / 2 + 1):
#
discovery.zen.minimum_master_nodes: 2 # 至少有两个 master node 同时在线才能允许 cluster 被使用

```

## Roles

```yaml
# 最好的做法是将各个 node 的职能分开
node.master: True
node.data: True
node.client: False # 如果设成 True, 则会自动将 node.master 设置成 false

http.enabled: False # 在 data node 添加此设置，确保 data node 只能通过内部的网络来访问，不能直接响应请求

```

## `JVM heap/ Memory setting`

```yaml
# Make sure that the `ES_HEAP_SIZE` environment variable is set to about half the memory
# available on the system and that the owner of the process is allowed to use this limit.
ES_HEAP_SIZE = RAM／2

# 关闭系统的 memory swaping
# bootstrap.memory_lock: true Elasticsearch performs poorly when the system is swapping the memory.
bootstrap.mlockall: true
```

## `File Descripters／MMap`

```yaml
File Descriptor: 64,000
MMap: unlimited # 如果不允许设置成 unlimited, 就设置的越高愈好
```

