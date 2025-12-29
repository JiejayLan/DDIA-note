## Chapter 7 Transaction


read commit

| 能保证  | 不能保证     |
| ---- | -------- |
| ❌ 脏读 | ❌ 不可重复读  |
|      | ❌ 读偏差    |
|      | ❌ 幻读     |
|      | ❌ 写偏差    |
|      | ❌ 跨多行一致性 |

Repeatable Read / Snapshop 和 Read Committed 的根本区别

它是怎么做到的？（直觉版）

事务开始时：

数据库记录一个 时间戳 / 版本号

每一行都有多个版本（MVCC）

你只能看到：

≤ 你事务开始时间的最新版本

| 对比点       | Read Committed | Repeatable Read / Snapshot |
| --------- | -------------- | -------------------------- |
| 一致性单位     | 每条 SQL         | 整个事务                       |
| 时间视角      | 会跳时间           | 时间冻结                       |
| Read Skew | ❌ 有            | ✅ 无                        |
| 可推理性      | 低              | 高                          |


what is race condition
| 表现         | 本质             |
| ---------- | -------------- |
| 丢更新        | race condition |
| Read Skew  | race condition |
| Write Skew | race condition |
| 库存超卖       | race condition |

| 类型          | 核心问题                   | 举例（会议室 / 库存 / 转账）          | 典型 SQL 问题                                                 | 典型 DDB 问题                                    |
| ----------- | ---------------------- | -------------------------- | --------------------------------------------------------- | -------------------------------------------- |
| Lost Update | 两个事务同时修改同一行，导致某个修改丢失   | 两个用户同时预定同一个会议室，A 的更新被 B 覆盖 | `UPDATE account SET balance = balance - 10 WHERE id=1` 并发 | DynamoDB / Cassandra 原子条件写，Redis SETNX 等原子操作 |
| Write Skew  | 两个事务读取多个行，独立判断，但组合违反约束 | 医生值班：两个医生同时下班导致没人值班        | SI 下事务 A、B 读取两个医生状态，各自更新自己的行                              | DDB 跨 shard，事务 A、B 同时看到资源可用，各自扣减 → 违反约束      |
| Read Skew   | 同一事务不同语句看到不同快照，违反业务逻辑  | 报表统计期间库存变化导致错误             | RC 下连续 `SELECT` 查询同一表，不可重复读                               | DDB 节点异步复制，读到不同副本版本                          |
| Phantom     | 新插入/删除的数据被事务“看见”，导致不一致 | 统计部门员工数，另一个事务插入新员工         | 统计 `WHERE department='Sales'`，可串行化隔离下避免                   | DDB 跨 shard，节点延迟，新插入行破坏约束                    |

SQL（关系型数据库）处理方式

| Race Condition | RC（Read Committed） | RR / SI（Snapshot）      | Serializable / 2PL / SSI             |
| -------------- | ------------------ | ---------------------- | ------------------------------------ |
| Lost Update    | ❌ 容易丢失             | ✅（单行 snapshot 可检测）     | ✅（锁或 SSI 防止覆盖）                       |
| Write Skew     | ❌                  | ❌（单行 snapshot，不检测组合约束） | ✅（2PL 或 SSI 检测冲突）                    |
| Read Skew      | ❌                  | ✅（同一事务 snapshot）       | ✅                                    |
| Phantom        | ❌                  | ❌                      | ✅（Predicate / Range Lock 或 SSI 冲突检测） |

DDB（分布式数据库）处理方式

| Race Condition | DynamoDB 方法                          | 工程建议             |
| -------------- | ------------------------------------ | ---------------- |
| Lost Update    | Condition Check / Version Attribute  | 原子操作 + 重试        |
| Write Skew     | TransactWriteItems / 条件表达式           | 多行组合约束事务，注意吞吐量限制 |
| Read Skew      | ConsistentRead                       | 保证读到最新数据         |
| Phantom        | Transaction + Condition Check / 应用层锁 | 跨行约束需原子检查或序列化提交  |

