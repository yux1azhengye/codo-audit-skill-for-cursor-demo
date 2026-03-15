# Recon 阶段：首轮信息收集

## 目标

在专项扫描前，输出可被后续步骤消费的结构化信息：技术栈、路由、认证、依赖、敏感 Sink 与入口。**按主语言按需读取**对应子文件夹中的策略：
- Java： [java/java-audit.md](java/java-audit.md)
- Go： [go/go-audit.md](go/go-audit.md)
- Python： [python/python-audit.md](python/python-audit.md)
- PHP： [php/php-audit.md](php/php-audit.md)
- ASP/ASP.NET： [asp/asp-audit.md](asp/asp-audit.md)
- Node.js： [nodejs/nodejs-audit.md](nodejs/nodejs-audit.md)

## 输出结构（recon.json 或等价）

```json
{
  "tech_stack": ["语言", "框架", "关键库"],
  "routes": [
    { "method": "GET|POST|...", "path": "/api/...", "handler": "控制器/函数名", "file": "路径" }
  ],
  "auth": { "type": "JWT/Session/OAuth/...", "entry_points": ["登录/鉴权相关文件或路由"] },
  "dependencies": ["敏感依赖名与版本，如 ORM、HTTP 客户端、模板引擎"],
  "sinks": {
    "sql": ["文件:行或标识"],
    "exec": ["文件:行或标识"],
    "ssrf": ["文件:行或标识"],
    "file": ["文件:行或标识"],
    "deserialize": ["文件:行或标识"]
  },
  "entry_sources": ["请求参数、Body、Header、Cookie 等入口汇总"]
}
```

## 执行要点

1. **语言/框架识别**：根据项目根目录标志文件（pom.xml、go.mod、requirements.txt、composer.json、*.csproj、package.json 等）与目录结构确定主语言与框架。
2. **路由提取**：从路由配置、注解、注册表或约定（如 Flask route、Spring Controller、Gin Handler）枚举 API；若存在拦截器/过滤器/中间件，在 recon 中注明，供逻辑漏洞专项使用。
3. **认证与鉴权**：识别登录、鉴权、权限校验的入口与方式，便于 Round 1 认证/越权专项。
4. **Sink 初筛**：不深入分析，仅标记可能涉及 SQL、执行、SSRF、文件、反序列化的位置，供 Round 1 按类型选文件与策略。
5. **依赖**：列出与安全相关的依赖（ORM、HTTP 客户端、模板、序列化库等），便于后续 CVE/GHSA 关联（可选）。

## 与专项的衔接

- Round 1 各专项（SQL/RCE/SSRF/auth/file/logic）应优先使用 recon 中的 `routes`、`sinks`、`auth` 缩小范围，避免全库盲目扫描。
