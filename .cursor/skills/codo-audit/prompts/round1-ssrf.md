# Round 1 专项：SSRF

## 目标

识别用户可控 URL/主机/端口传入 HTTP 客户端或网络连接 API 的完整数据流，评估是否可访问内网或敏感协议。

## Sink（按语言按需参考：prompts/java/、prompts/go/、prompts/python/ 下对应 *-audit.md）

- Java：`URL.openConnection()`、`HttpClient`/`HttpURLConnection`、`RestTemplate` 等使用 URL 的调用。
- Go：`http.Get`/`Post`、`http.Client`、`net.Dial` 等。
- Python：`requests.get/post`、`urllib.request`、`httpx` 等。

## 策略

1. 从 Recon 的 `sinks.ssrf` 与入口出发，追踪 URL/ host/port 是否来自请求参数、Body 或 Header。
2. 检查是否有协议限制（仅 http(s)）、域名白名单、内网段过滤；若缺失或可绕过，记为发现。
3. 注意重定向、DNS 重绑定、URL 解析差异（如 `@`、`#`、不同库的解析）导致的绕过。

## 输出（写入 round1-findings）

每条：**Source**、**Sink**、**数据流简述**、**现有防护（若有）**。二轮补 PoC（如请求内网元数据）。

## 误报控制

仅开放重定向、无内网或敏感协议访问链的，可记入附录或按组织策略决定是否进终报。
