---
title: clickhouse要点
excerpt: "clickhouse要点"
date: "2025/4/24 11:22:11"
tag: [clickhouse,列式数据库]
categories: [clickhouse]
---

## docker compose部署

可以先注释掉config.xml和users.xml，等容器启动后将容器路径下/etc/clickhouse-server/config.xml和/etc/clickhouse-server/users.xml文件复制到宿主机路径下，再启动容器
```yaml
version: '3'

services:
  clickhouse:
    image: clickhouse:25.3
    container_name: clickhouse
    ports:
      - "8123:8123"
      - "9000:9000"
      - "9009:9009"
    volumes:
      # - ./data:/var/lib/clickhouse:rw    
      # /var/lib/clickhouse下有windows 中有个文件无法被挂在，所以我们只选择数据文件和导入文件的挂在路径即可
      - ./data/data:/var/lib/clickhouse/data
      - ./data/user_files:/var/lib/clickhouse/user_files
      - ./config.xml:/etc/clickhouse-server/config.xml
      - ./users.xml:/etc/clickhouse-server/users.xml
```
配置的用户名和密码在users.xml中配置,在users标签下添加如下：root/root
```xml
<root>
    <password>root</password>
    <networks incl="networks" replace="replace">
        <!--<ip>::/0</ip>-->
        <ip>0.0.0.0/0</ip><!--允许所有人访问-->
    </networks>
    <profile>default</profile>
    <quota>default</quota>
</root>
```

## clickhouse常用数据类型

### 基础数据类型
| 类型名称               | 描述                     | 示例值                  |
|----------------------|------------------------|---------------------|
| UInt8, UInt16, UInt32, UInt64 | 无符号整数（范围递增）       | 255, 1000000        |
| Int8, Int16, Int32, Int64     | 有符号整数                 | -128, 32767         |
| Float32, Float64             | 单/双精度浮点数            | 3.14, -0.001        |
| String                      | 任意长度字符串（UTF-8编码）   | 'Hello', '中文'      |
| FixedString(N)              | 定长字符串（不足补空字节）    | FixedString(5) 'abc'|
| Date                        | 日期（精度到天）            | '2023-10-01'        |
| DateTime                    | 时间戳（精度到秒，默认时区为服务器时区） | '2023-10-01 12:34:56' |
| DateTime64(precision, [timezone]) | 高精度时间戳（微秒/毫秒级） | DateTime64(3) '2023-10-01 12:34:56.789' |
| Boolean                     | 布尔值（底层用 UInt8 存储，0=False，1=True） | true, false         |

### 复合数据类型
| 类型名称               | 描述                     | 示例值                  |
|----------------------|------------------------|---------------------|
| Array(T)             | 由同类型元素组成的数组       | [1, 2, 3], ['a', 'b'] |
| Tuple(T1, T2, ...)  | 异构元组                 | (1, 'a'), (3.14, now()) |
| Map(KeyType, ValueType) | 键值对集合（ClickHouse 23.3+ 支持） | {'key1': 1, 'key2': 2} |
| Nested(Name1 Type1, Name2 Type2, ...) | 嵌套表结构（实际存储为多个数组列） | tags Nested (id UInt32, name String) |


### 特殊用途类型
| 类型名称               | 描述                     | 典型场景          |
|----------------------|------------------------|---------------|
| LowCardinality(String) | 低基数字符串优化存储（适用于重复值多的列） | 性别、国家代码等枚举字段 |
| Nullable(T)           | 允许列值为 NULL（会增加存储和查询开销）   | 可选字段       |
| Enum('val1', 'val2') | 枚举类型（存储为整数，提升查询性能）     | 状态码、类型分类 |
| Decimal(P, S)         | 高精度定点数（P 总位数，S 小数位）       | 金融金额、科学计算 |
| IPv4/IPv6            | 存储 IP 地址（优化存储和查询）           | 日志分析、网络监控 |
| UUID                 | 存储 UUID 字符串（16字节二进制）         | 分布式唯一标识   |

```sql
CREATE TABLE user_actions (
    user_id UInt64,
    action_type Enum('click', 'view', 'purchase'),
    event_time DateTime,
    price Decimal(10, 2),
    tags Array(String),
    properties Map(String, String)
) ENGINE = MergeTree()
ORDER BY (user_id, event_time);
```

```sql
CREATE TABLE products (
    id UInt32,
    name String,
    reviews Nested (
        user_id UInt32,
        rating UInt8,
        comment String
    )
) ENGINE = MergeTree()
ORDER BY id;
```

## 一、数据库引擎（Database Engines）

定义数据库的存储和管理方式：

| 引擎名称             | 用途                                                         |
|----------------------|------------------------------------------------------------|
| Atomic               | 默认引擎，支持原子性DDL操作（推荐生产环境使用）。                           |
| MySQL                | 将ClickHouse数据库映射到MySQL表，实现跨引擎查询。                         |
| Lazy                 | 延迟加载数据，适用于日志类低频访问场景。                                   |
| MaterializedPostgreSQL | 与PostgreSQL实时同步数据（需启用实验性功能）。                          |

## 二、表引擎（Table Engines）

### 1. MergeTree 系列（核心引擎，支持高性能分析）

| 引擎名称               | 特点                                                         |
|----------------------|------------------------------------------------------------|
| MergeTree            | 基础引擎，支持分区、索引、TTL（数据生命周期管理）。                           |
| ReplacingMergeTree   | 自动去重（相同排序键保留最新数据）。                                   |
| SummingMergeTree     | 合并时对数值列自动求和，适用于预聚合场景。                               |
| AggregatingMergeTree | 合并时预聚合，配合 AggregateFunction 类型使用。                         |
| CollapsingMergeTree  | 通过标记列（sign）处理行级数据变更（如删除/更新）。                       |
| VersionedCollapsingMergeTree | 支持多版本折叠，处理复杂更新场景。                                 |
| GraphiteMergeTree    | 专为Graphite监控数据优化，支持自动降采样。                               |

### 2. 日志引擎（简单场景，数据量小）

| 引擎名称   | 特点                                                         |
|-----------|------------------------------------------------------------|
| TinyLog    | 数据不压缩，单文件存储，适用于临时测试（不支持索引）。                           |
| Log        | 按列存储，支持并行查询（比TinyLog高效）。                                   |
| StripeLog  | 按块存储，支持多线程写入（查询性能较低）。                                   |


### 3. 集成引擎（与外部系统交互）

| 引擎名称   | 用途                                                         |
|-----------|------------------------------------------------------------|
| Kafka     | 从Kafka主题消费数据，支持实时流处理。                                   |
| MySQL     | 映射MySQL表，允许在ClickHouse中查询MySQL数据。                         |
| PostgreSQL | 类似MySQL引擎，集成PostgreSQL数据源。                                 |
| ODBC/JDBC | 通过标准协议连接外部数据库（如Oracle、SQL Server）。                   |
| HDFS      | 直接读写HDFS文件（Parquet、ORC格式）。                                 |
| S3        | 读写Amazon S3存储的数据。                                             |


### 4. 特殊用途引擎

| 引擎名称           | 用途                                                         |
|------------------|------------------------------------------------------------|
| Memory           | 数据仅存储在内存中，重启后丢失（用于临时计算）。                           |
| File             | 将数据导出为指定格式（CSV、JSON等），直接操作文件。                       |
| URL              | 通过HTTP/HTTPS读写远程服务器数据。                                   |
| EmbeddedRocksDB  | 基于RocksDB的键值存储，支持高吞吐点查（实验性功能）。                   |
| Set              | 存储唯一键值集合，用于 IN 查询优化。                                 |
| Join             | 预存右表数据，加速JOIN操作（类似内存Hash表）。                         |


### 5. 虚拟引擎（元数据操作）

| 引擎名称   | 用途                                                         |
|-----------|------------------------------------------------------------|
| Generate   | 动态生成测试数据（如 `SELECT * FROM generate(...)`）。                   |
| Null       | 写入数据被丢弃，用于测试或ETL管道占位。                                   |
| Dictionary | 将外部字典映射为表，支持字典查询。                                       |


## 一些测试数据

建表
```sql
CREATE TABLE trips (
    trip_id             UInt32,
    pickup_datetime     DateTime,
    dropoff_datetime    DateTime,
    pickup_longitude    Nullable(Float64),
    pickup_latitude     Nullable(Float64),
    dropoff_longitude   Nullable(Float64),
    dropoff_latitude    Nullable(Float64),
    passenger_count     UInt8,
    trip_distance       Float32,
    fare_amount         Float32,
    extra               Float32,
    tip_amount          Float32,
    tolls_amount        Float32,
    total_amount        Float32,
    payment_type        Enum('CSH' = 1, 'CRE' = 2, 'NOC' = 3, 'DIS' = 4, 'UNK' = 5),
    pickup_ntaname      LowCardinality(String),
    dropoff_ntaname     LowCardinality(String)
)
ENGINE = MergeTree
PRIMARY KEY (pickup_datetime, dropoff_datetime);
```

导入数据

下载数据文件 https://storage.googleapis.com/clickhouse-public-datasets/nyc-taxi/trips_{0..2}.gz

放入容器的  /var/lib/clickhouse/user_files/{demo} 下
```sql
INSERT INTO trips
SELECT
    trip_id,
    pickup_datetime,
    dropoff_datetime,
    pickup_longitude,
    pickup_latitude,
    dropoff_longitude,
    dropoff_latitude,
    passenger_count,
    trip_distance,
    fare_amount,
    extra,
    tip_amount,
    tolls_amount,
    total_amount,
    payment_type,
    pickup_ntaname,
    dropoff_ntaname
FROM file(
    'demo/trips_{0-1}.gz',
    'TabSeparatedWithNames'
);
```

```sql
让我们运行几个查询。这个查询显示接送次数最多的前10个社区：

SELECT
   pickup_ntaname,
   count(*) AS count
FROM trips
GROUP BY pickup_ntaname
ORDER BY count DESC
LIMIT 10;

这个查询显示根据乘客人数的平均收费：

SELECT
   passenger_count,
   avg(total_amount)
FROM trips
GROUP BY passenger_count;
```
