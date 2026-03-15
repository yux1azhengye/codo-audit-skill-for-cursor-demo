# Round 1 专项：文件操作（读/写/路径穿越）

## 目标

识别用户可控路径传入文件读写 API 的数据流，包括任意文件读、写、路径穿越（../）、 overwrite 敏感文件等。

## Sink（按语言按需参考：prompts/java/、prompts/go/、prompts/python/ 下对应 *-audit.md）

- Java：`FileInputStream`/`FileOutputStream`、`Paths.get()`、`new File()`、NIO 等。
- Go：`os.Open`/`os.Create`、`ioutil.ReadFile`/`WriteFile`。
- Python：`open()`、`pathlib.Path`、`send_file` 等。

## 策略

1. 从 Recon 的 `sinks.file` 与入口出发，追踪路径参数是否经过规范化与白名单/目录限制。
2. 检查是否存在 `../`、绝对路径、符号链接导致的穿越；是否可写系统或应用敏感文件。
3. 上传类：后缀/类型校验是否可绕过、是否可覆盖已有文件、是否可执行（如 web 目录下执行脚本）。

## 输出（写入 round1-findings）

每条：**Source**、**Sink**、**数据流简述**、**风险（读/写/穿越/上传）**。二轮补 PoC。
