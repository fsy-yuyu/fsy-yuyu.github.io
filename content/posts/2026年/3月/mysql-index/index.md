---
title: "MySQL 索引：B+ 树、最左前缀与慢查询排查"
date: 2026-03-15T20:40:00+08:00
draft: false
description: "为什么用 B+ 树、联合索引的顺序为什么重要、EXPLAIN 该看哪几列，以及一条慢 SQL 的完整排查路径。"
categories: ["数据库"]
tags: ["MySQL", "索引", "性能优化"]
cover:
  relative: true
  image: "cover.png"
  alt: "MySQL 索引笔记封面"
  caption: "B+ 树与最左前缀"
---

索引不是「加上就快」，加错方向反而拖慢写入。这篇按「结构 → 规则 → 排查」的顺序整理。

## 为什么是 B+ 树

{{< figure src="btree.png" caption="非叶子节点只存键，数据都在叶子层，叶子之间用链表相连" alt="B+ 树结构示意" >}}

三个关键点：

1. **非叶子节点只存键值**，一个 16KB 的页能放更多键，树更矮（三层通常能覆盖千万级数据），磁盘 IO 更少。
2. **数据全在叶子节点**，查询路径长度稳定，不会出现「有的快有的慢」。
3. **叶子节点之间有双向链表**，范围查询（`BETWEEN`、`ORDER BY`、分页）可以顺着链表扫，不用回到根。

对比 B 树：B 树的非叶子节点也存数据，扇出小、树更高；也没有叶子链表，范围查询要反复中序遍历。

## 联合索引与最左前缀

```sql
CREATE INDEX idx_a_b_c ON t (a, b, c);
```

这个索引相当于按 `(a, b, c)` 排序的一本字典。能用的查询：

| 查询条件 | 能否用索引 | 用到几列 |
| --- | --- | --- |
| `WHERE a = 1` | 能 | a |
| `WHERE a = 1 AND b = 2` | 能 | a, b |
| `WHERE a = 1 AND b = 2 AND c = 3` | 能 | a, b, c |
| `WHERE b = 2` | 不能 | — |
| `WHERE a = 1 AND c = 3` | 部分 | 只用到 a，c 需回表过滤 |
| `WHERE a = 1 AND b > 2 AND c = 3` | 部分 | a、b 用于定位，c 用不上（范围之后失效） |
| `WHERE a = 1 ORDER BY b` | 能 | a 定位 + b 天然有序，无需额外排序 |

记忆口诀：**从左往右、遇到范围就断、跳列就断**。

所以建索引前先想清楚「哪些列总是和哪些列一起查」，把等值条件放前面、范围条件放后面。

## 覆盖索引：不回表就够快

如果查询要的列都在索引里，就不用回表取整行：

```sql
-- 索引 (user_id, created_at)
SELECT user_id, created_at FROM orders WHERE user_id = 100 ORDER BY created_at DESC;
-- Extra 里会出现 Using index，表示覆盖索引
```

反例：`SELECT *` 几乎必然回表，能只取需要的列就别 `SELECT *`。

## EXPLAIN 看哪几列

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 100 ORDER BY created_at DESC LIMIT 20;
```

| 列 | 关注点 |
| --- | --- |
| `type` | 访问类型，好坏顺序：`system` > `const` > `eq_ref` > `ref` > `range` > `index` > `ALL` |
| `key` | 实际用了哪个索引，`NULL` 表示没走索引 |
| `key_len` | 用了索引的几个字节，能反推用到第几列 |
| `rows` | 预估扫描行数，越大越危险 |
| `filtered` | 扫描后还剩多少比例，越低说明索引筛选效果越差 |
| `Extra` | 关键线索，见下表 |

`Extra` 常见值：

| 值 | 含义 | 要不要处理 |
| --- | --- | --- |
| `Using index` | 覆盖索引，很好 | 不用 |
| `Using where` | 服务层还要过滤 | 结合 rows 看 |
| `Using filesort` | 额外排序，没走索引顺序 | 通常要优化 |
| `Using temporary` | 用了临时表，常见于 `GROUP BY` | 通常要优化 |
| `Using index condition` | 索引条件下推，较好 | 不用 |

出现 `type=ALL` + `rows` 很大 + `Using filesort`，基本就是这条 SQL 的问题所在。

## 一条慢 SQL 的排查路径

```sql
-- 1. 确认慢查询日志开着
SHOW VARIABLES LIKE 'slow_query_log';
SHOW VARIABLES LIKE 'long_query_time';

-- 2. 找出最耗时的 SQL
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;

-- 3. 看执行计划
EXPLAIN ANALYZE SELECT ...;   -- 8.0.18+ 能给出实际耗时
```

然后按顺序试：

1. 补索引 / 调整联合索引顺序；
2. 把 `SELECT *` 收窄成需要的列，争取覆盖索引；
3. 分页太深时改成 `WHERE id > 上次最大 id LIMIT 20`（游标分页），别用大 `OFFSET`；
4. 拆分大 SQL，把聚合放到预计算表；
5. 以上都不行，再看分库分表或换存储。

## 索引的代价

- 每个索引都要占磁盘、占内存（buffer pool 里缓存索引页）。
- 每次 `INSERT`/`UPDATE`/`DELETE` 都要维护所有相关索引，写入越频繁越明显。
- 区分度低的列（如 `status` 只有 0/1）单独建索引收益很低，放进联合索引里当过滤条件更合适。

## 小结

- B+ 树矮、叶子有链表，适合磁盘和范围查询。
- 联合索引记三条：最左前缀、遇范围就断、跳列就断。
- 排查顺序：慢日志 → `EXPLAIN` 看 `type/key/rows/Extra` → 补索引或改写法。
