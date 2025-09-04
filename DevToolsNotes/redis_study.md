# Redis 入门

Redis 是开发高性能后端服务器的必备中间件。

## 一、Redis 介绍

> - Redis是 Key-Value 型 NoSQL 数据库；
>
> - Redis将数据存储在内存中，同时也能持久化到磁盘；
>
> - Redis常用于缓存，利用内存的高效提高程序的处理速度。
>
> **Redis 特点**
>
> - 速度快、持久化、主从复制；
> - 广泛的语言支持（常见的后端语言）；
> - 支持多种数据结构（String, List, Hash, Set, ZSet）；
> - 支持分布式和高可用。

### 1.1 Linux安装Redis

本部分参考 AI 即可，非常方便，以 Ubuntu20.04 为例：

```bash
# 1. 先安装相关依赖
sudo apt update
sudo apt install -y build-essential tcl

# 2. 在指定目录（也就是安装目录）下载Redis指定版本源码并且解压缩
wget https://download.redis.io/releases/redis-6.0.16.tar.gz
tar xzf redis-6.0.16.tar.gz
cd redis-6.0.16

# 3. 进入到安装目录后，编译且安装
make
sudo make install
```

## 二、Redis常用基本配置

安装了 redis 之后，其配置文件在安装目录下，文件名`redis.conf`即为配置文件，启动 redis 时，使用命令 `[安装目录]/src/redis-server [安装目录]/redis.conf`即可让配置文件生效。

> 配置文件一些关键项解析：
>
> | 配置项      | 示例              | 说明                    |
> | ----------- | ----------------- | ----------------------- |
> | daemonize   | daemonize yes     | 是否启用后台运行,默认no |
> | port        | port 6379         | 设置端口号,默认6379     |
> | logfile     | logfile 日志文件  | 设置日志文件            |
> | databases   | databases 255     | 设置redis数据库总量     |
> | dir         | dir 数据文件目录  | 设置数据文件存储目录    |
> | requirepass | requirepass 12345 | 设置使用密码            |

## 三、Redis常用命令

Redis 常用命令总结如下：

| 命令     | 示例              | 说明                                 |
| -------- | ----------------- | ------------------------------------ |
| `select` | `select 0`        | 选择0号数据库                        |
| `set`    | `set name lily`   | 设置 key=name, value=lily            |
| `get`    | `get hello`       | 获得 key=hello 结果                  |
| `keys`   | `keys he*`        | 根据 Pattern 表达式查询符合条件的Key |
| `dbsize` | `dbsize`          | 返回 key 的总数                      |
| `exists` | `exists a`        | 检查 key=a 是否存在                  |
| `del`    | `del a`           | 删除 key=a 的数据                    |
| `expire` | `expire hello 20` | 设置 key=hello 20秒后过期            |
| `ttl`    | `ttl hello`       | 查看 key=a 的过期剩余时间            |

## 四、Redis数据类型

在 Redis 中，不同的数据类型对应的命令不同。

### 4.1 String 类型



### 4.2 Hash 类型



### 4.3 List 类型



### 4.4 Set 类型



### 4.5 Zset 类型



