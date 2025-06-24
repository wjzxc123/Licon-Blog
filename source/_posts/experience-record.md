---
title: 难点记录
excerpt: "难点记录"
date: "2025/6/16 11:22:11"
tag: [面试]
categories: [面试]
---

#### mysql索引失效的场景
##### 1. 对索引列进行运算或函数处理
>场景: WHERE YEAR(create_time) = 2023, WHERE amount * 1.1 > 100
> 
>原因: 索引存储的是列的原始值。对列进行计算或应用函数后，MySQL 无法直接使用索引树查找计算结果。
>
>解决方案: 重写查询，避免对索引列操作。例如：

```sql
WHERE create_time BETWEEN '2023-01-01' AND '2023-12-31' -- 替代 YEAR(create_time)
WHERE amount > 100 / 1.1 -- 移除非索引列上的计算
```


##### 2. 使用 NOT LIKE、<>、NOT IN 操作符
>场景: WHERE name NOT LIKE 'A%', WHERE status <> 1, WHERE id NOT IN (1, 2, 3)
>
>原因: 这些操作符需要检查大部分数据行，优化器认为全表扫描可能比回表多次检查索引更快（尤其是数据量大时）。
>
>解决方案: 考虑改写逻辑，或确保结果集很小。


##### 3. 模糊查询以通配符开头 (% 前置)
>场景: WHERE name LIKE '%search', WHERE name LIKE '%search%'
>
>原因: B+树索引按值从左到右排序。前置 % 使索引无法定位起始点。
>
>解决方案:
>- 避免前置 %：WHERE name LIKE 'search%'（后缀匹配）通常可用索引。
>- 使用全文索引 (FULLTEXT) 进行复杂文本搜索。
>- 使用专门的搜索引擎 (如 Elasticsearch)。

##### 4. 隐式类型转换

>场景: 索引列是字符串类型 (VARCHAR)，查询用数字比较：WHERE phone = 13800138000 (phone 是 VARCHAR)
>
>原因: MySQL 会隐式转换索引列类型（如将字符串转数字），等同于在列上应用了转换函数，导致索引失效。
>
>解决方案: 确保查询条件与索引列类型严格一致：
```sql
WHERE phone = '13800138000' -- 使用字符串字面量
```

##### 5. OR 连接非索引列条件
>场景: WHERE indexed_col = 10 OR non_indexed_col = 20
>
>原因: 即使 indexed_col = 10 可用索引，但 OR 要求结果取并集。只要有一个条件涉及非索引列，优化器可能放弃索引选择全表扫描。
>
>解决方案:
>
>将 OR 拆分成两个查询用 UNION 合并：

```sql
SELECT ... WHERE indexed_col = 10
UNION
SELECT ... WHERE non_indexed_col = 20
```


##### 6. 违反联合索引的最左前缀原则
>场景: 有联合索引 (col1, col2, col3)。
>
>WHERE col2 = 'abc' AND col3 = 'xyz' (缺少 col1)
>
>WHERE col1 = 10 AND col3 = 20 (跳过 col2)
>
>原因: 联合索引按定义顺序排序存储。缺失最左列或跳过中间列时，索引无法高效定位数据（如同查电话簿跳过姓氏直接查名字）。
>
>解决方案:
>- 查询条件必须包含最左列（col1）。
>- 条件中列的顺序尽量与索引定义一致（优化器会调整，但最左列必须存在）。
>- 考虑建立覆盖所需查询的索引。

##### 7. 联合索引中使用范围查询后的列
>场景: 索引 (col1, col2, col3), 查询 WHERE col1 = 'a' AND col2 > 10 AND col3 = 'c'
>
>原因: col2 > 10 是范围查询，其后的索引列 (col3) 在索引树中不再有序，无法直接利用索引快速定位 col3 = 'c'。
>
>解决方案:
>- 尽量将等值查询列放在范围查询列之前。
>- 为 col3 单独建立索引（需评估查询频率和数据分布）。

##### 8. 使用 IS NULL / IS NOT NULL (取决于优化器选择)
>场景: WHERE indexed_col IS NULL
>
>原因: 如果表中允许 NULL 的行非常少，优化器可能使用索引。反之，如果 NULL 行很多，优化器认为全表扫描更快。
>
>解决方案: 考虑表数据分布。如果 NULL 值极少且需要快速查询，可尝试索引。

##### 9. 优化器判断全表扫描更快
>场景: 小表查询，或索引列区分度极低（如状态列只有几个值）。
>
>原因: 使用索引需要回表查数据，可能产生大量随机 I/O。优化器估算后认为直接读取连续的数据页（全表扫描）成本更低。
>
>解决方案:
>- 使用 FORCE INDEX 强制使用索引（需谨慎测试）。
>- 优化表结构和索引设计。


##### 10. 索引统计信息过时
>场景: 表数据发生大量变化（增删改）后，原本高效的查询变慢。
>
>原因: MySQL 优化器依赖存储的索引统计信息来选择执行计划。统计信息未及时更新导致优化器做出错误判断（如高估索引效果）。
> 
>解决方案: 定期执行 ANALYZE TABLE table_name; 更新统计信息。

##### 11. 索引本身损坏 (罕见)
>场景: 极端情况，如服务器崩溃、磁盘错误。
>
>原因: 索引文件物理损坏。
>
>解决方案: 使用 REPAIR TABLE table_name; 修复表（通常 MyISAM 需要，InnoDB 有自愈能力）。

##### 如何诊断索引是否失效？
1. EXPLAIN 是神器: 在查询前加 EXPLAIN 或 EXPLAIN FORMAT=JSON 分析执行计划。

关键字段：

- type: ALL 通常表示全表扫描（最差）。index 表示全索引扫描。range 表示使用了索引范围扫描（较好）。ref/eq_ref/const 表示高效索引使用（最佳）。

- key: 显示实际使用的索引。NULL 表示未使用索引。

- rows: 估算的需要扫描的行数（越小越好）。

- Extra: 包含重要信息，如 Using where (在存储引擎层后过滤), Using index (覆盖索引), Using filesort (需额外排序), Using temporary (需临时表)。

2. 开启慢查询日志: 记录执行时间超过阈值的查询，便于事后分析。

3. 性能监控工具: 使用 MySQL Enterprise Monitor、Percona Monitoring and Management (PMM)、Prometheus + Grafana 等监控数据库性能指标。

> 核心原则： 让索引列以"原始"、"未加工"的形式出现在查询条件中，并符合索引的数据结构和组织方式（特别是联合索引的顺序）。理解 B+树索引的工作原理是避免索引失效的关键💡。