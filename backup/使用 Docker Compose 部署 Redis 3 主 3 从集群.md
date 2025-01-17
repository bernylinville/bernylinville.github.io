本文将说明如何使用 docker compose 在 3 台服务器上部署一个 3 主 3 从的 Redis 集群。

## 部署架构

我们将在 3 台服务器上部署 6 个 Redis 节点，每台服务器运行 2 个节点。

## Docker Compose 配置

以下是用于部署 Redis 集群的 docker compose 配置文件示例：

```yaml
services:
  redis-7001:
    image: redis:6.2.16
    container_name: redis-7001
    restart: on-failure:3
    network_mode: host # 使用主机网络避免性能损耗
    volumes:
      - ./redis-7001.conf:/etc/redis/redis.conf
      - ./data/redis-7001:/data
      - ./logs/redis-7001:/var/log/redis
    environment:
      TZ: Asia/Shanghai
    command: redis-server /etc/redis/redis.conf
    healthcheck:
      test: ["CMD", "redis-cli", "-h", "127.0.0.1", "-p", "7001", "-a", "password", "--no-auth-warning", "ping"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 10s
    deploy:
      resources:
        limits:
          memory: 4gb
          cpus: '2.0'

  redis-7002:
    image: redis:6.2.16
    container_name: redis-7002
    restart: on-failure:3
    network_mode: host
    volumes:
      - ./redis-7002.conf:/etc/redis/redis.conf
      - ./data/redis-7002:/data
      - ./logs/redis-7002:/var/log/redis
    environment:
      TZ: Asia/Shanghai
    command: redis-server /etc/redis/redis.conf
    healthcheck:
      test: ["CMD", "redis-cli", "-h", "127.0.0.1", "-p", "7002", "-a", "password", "--no-auth-warning", "ping"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 10s
    deploy:
      resources:
        limits:
          memory: 4gb
          cpus: '2.0'
```

## Redis 配置文件

每个 Redis 节点都需要一个配置文件。以下是 `redis-7001.conf` 和 `redis-7002.conf` 的示例配置：

```conf
port 7001
requirepass password
masterauth password
protected-mode no
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
# 集群节点对外 IP 需要根据实际情况修改
cluster-announce-ip 192.168.10.8
cluster-slave-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage no

appendonly yes
appendfsync everysec
save 900 1
save 300 10
save 60 10000

# 最大内存限制，根据实际情况调整
maxmemory 4gb
maxmemory-policy volatile-lru
maxmemory-samples 5

timeout 300
tcp-keepalive 300
maxclients 50000
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
lazyfree-lazy-server-del yes

loglevel notice
logfile /var/log/redis/redis.log
syslog-enabled no

slowlog-log-slower-than 10000
slowlog-max-len 128

repl-backlog-size 64mb
repl-backlog-ttl 3600
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

## 部署步骤

1. **准备环境**：确保每台服务器上都安装了 docker 和 docker compose。
2. **配置文件**：根据上述示例创建 `redis-7001.conf` 和 `redis-7002.conf` 文件，并根据实际情况调整配置。
3. **启动集群**：在每台服务器上运行 `docker-compose up -d` 启动 Redis 节点。
4. **配置集群**：使用 `docker exec -it redis-7001 redis-cli --cluster create 192.168.10.8:7001 192.168.10.8:7002 ip:port --cluster-replicas 1` 配置集群。
