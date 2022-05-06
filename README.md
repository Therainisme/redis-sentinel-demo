# 尝试用 Docker 小做一个 Redis Sentinel Demo

一主两从三哨兵

![image](https://user-images.githubusercontent.com/41776735/167107694-40f5de9f-b90d-4a19-8593-8b7fabfd900f.png)

## 初始配置

下面是于 Sentinel 配置相关的一些参数

`sentinel monitor <master-name> <ip> <port> <quorum>` ：Sentinel节点会定期监控主节点，`<quorum>` 代表要判定主节点最终不可达所需要的票数

`sentinel down-after-milliseconds <master-name> <times> `：每个 Sentinel 节点都要通过定期发送 ping 命令来判断 Redis 数据节点和其余 Sentinel 节点是否可达，如果超过了 down-after-milliseconds 配置的时间且没有有效的回复，则判定节点不可达，`<times>` （单位为毫秒）就是超时时间。

`sentinel parallel-syncs <master-name> <nums> `：在一次故障转移之后，每次向新的主节点发起复制操作的从节点个数

`sentinel failover-timeout <master-name> <times>` ：failover-timeout 通常被解释成故障转移超时时间

`sentinel auth-pass <master-name> <password> `：如果 Sentinel 监控的主节点配置了密码，sentinel auth-pass 配置通过添加主节点的密码，防止 Sentinel节点对主节点无法监控。

### Sentinel 配置

哨兵 1 —— sentinel-26379.conf

```conf
port 26379
dir /data

sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000 
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

哨兵 2 —— sentinel-26380.conf

```conf
port 26380
dir /data

sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000 
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

哨兵 3 —— sentinel-26381.conf

```conf
port 26381
dir /data

sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000 
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

### 主从节点配置

> 注意：只展示关键配置，其余配置已省略

主节点 —— redis-6379.conf：

```conf
port 6379
pidfile /var/run/redis_6379.pid
```

从节点 1 —— redis-6380.conf：

```conf
port 6380
pidfile /var/run/redis_6380.pid
replicaof 127.0.0.1 6379
```

从节点 2 —— redis-6381.conf：

```conf
port 6381
pidfile /var/run/redis_6381.pid
replicaof 127.0.0.1 6379
```

## 通过 docker-compose 启动

在项目根目录下执行

```
docker-compose up
```

随便选择一个 redis 容器的 redis-cli，通过**端口号**连接 redis-server 或 redis-sentinel。

- 连接主节点

```
docker exec -it redis-master-6379 redis-cli -p 6379
```

- 连接 Sentinel（其一）

```
docker exec -it redis-master-6379 redis-cli -p 26379
```

## 如果不小心玩崩了，想 REMAKE

```
cp ./svconf/* ./conf/
```