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
