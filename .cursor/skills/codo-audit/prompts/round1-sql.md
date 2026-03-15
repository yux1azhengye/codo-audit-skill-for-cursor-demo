# Round 1 专项：SQL 注入

## 目标

从 Recon 给出的路由与 Sink 中，追踪用户可控数据到 SQL 执行点的完整数据流，识别未正确参数化或存在拼接的 SQL。

## Sink（按语言按需参考：prompts/java/、prompts/go/、prompts/python/ 下对应 *-audit.md）

- Java：`Statement` 拼接、MyBatis `${}`、HQL 拼接、`JdbcTemplate` 拼接等。
- Go：`db.Exec`/`Query` 拼接、`fmt.Sprintf` 拼 SQL、GORM 原生 SQL。
- Python：`cursor.execute(sql % ...)`、ORM `raw()`/`extra()`、`text()` 拼接。

## 策略

1. 从 Recon 的 `sinks.sql` 与 `entry_sources` 出发，建立 Source → Sink 数据流。
2. 识别所有“字符串拼接/格式化后传入执行函数”的代码路径，标记为疑似。
3. 确认是否使用预编译/参数化（如 `PreparedStatement`、`?` 占位、命名参数）；若仅为拼接且 Source 可控，记为发现。
4. 注意二次注入、ORDER BY/表名等无法参数化的位置，若用户输入直接进入则同样记为发现。

## 输出（写入 round1-findings）

每条记录：**Source（文件:行/参数名）**、**Sink（文件:行/语句形态）**、**数据流简述**、**是否参数化**。暂不要求完整 PoC，二轮再补。

## 可选案例

若需提高命中率，可引入 GHSA/CVE 中与 SQL 相关的代码模式（如 dict key 未转义、ORM 聚合注入）作为参考，控 token 使用。
