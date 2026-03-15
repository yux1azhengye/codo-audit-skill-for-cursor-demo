# Round 1 专项：RCE / 命令执行

## 目标

从用户可控入口到系统命令执行、脚本执行或危险反序列化的数据流，识别可导致 RCE 的调用链。

## Sink（按语言按需参考：prompts/java/、prompts/go/、prompts/python/ 下对应 *-audit.md）

- Java：`Runtime.getRuntime().exec()`、`ProcessBuilder`、`ScriptEngine`、OGNL（Struts）等。
- Go：`exec.Command`、`os/exec`。
- Python：`os.system`、`subprocess`、`eval`、`exec`、`pickle.loads`、模板注入（Jinja2）。

## 策略

1. 从 Recon 的 `sinks.exec` 与入口出发，追踪参数是否未经校验/过滤传入命令或脚本。
2. 注意命令拼接、数组参数 vs 字符串参数（如 `exec(cmd)` 与 `exec(cmd, [arg1, arg2])`）的差异。
3. 模板注入：用户输入进入模板渲染引擎（如 Jinja2、Freemarker）且可闭合语法时，记为发现。
4. 反序列化导致 RCE：`ObjectInputStream.readObject`、`pickle.loads`、`yaml.load` 等，若反序列化数据来自请求，记为发现。

## 输出（写入 round1-findings）

每条：**Source**、**Sink**、**数据流简述**、**触发方式（命令/脚本/反序列化）**。二轮补 PoC。

## 误报控制

纯理论链、无代码证据或无法从当前代码触发的链，二轮排除，不进入终报。
