---
name: codo-audit
description: >-
  Runs a phased code security audit: recon, vulnerability scanning by type,
  second-round verification, and final report. Use when the user asks for
  安全审计、漏洞扫描、code audit、security review, or wants to find
  SQL/RCE/SSRF/auth/file/logic vulnerabilities in the codebase.
---

# 代码审计 Skill（Code Audit）

按五阶段执行：参数与范围 → Recon → 专项扫描 → 二轮验证与误报过滤 → 攻击链与报告。规则与产出物格式写死在本文及引用文件中，便于复现。

## 输入约定

- **目标路径**：用户指定或默认当前仓库根目录。
- **报告目录**：默认 `./audit-report/`，可指定。
- **主语言**：用户指定或 Step 0 自动识别（Java/Python/Go/其他）。
- **扫描范围**（可选）：全库 或 仅本地变更（git diff）。Cursor 场景下可针对未提交变更做 diff-aware 分析。

## 输出约定

- 单条发现必须包含：**API/入口 URL**、**Source → Sink**、**PoC（或复现步骤）**、**修复建议**。
- 终报路径：`<报告目录>/audit-report.md`。
- 中间产物：Recon 输出 `<报告目录>/recon.json`（或等价结构）；Round 1 原始发现 `<报告目录>/round1-findings.md`；二轮确认后 `<报告目录>/confirmed-findings.md`。

## 误报控制（硬性规则）

以下类型**不进入终报**，二轮前即可规则过滤：

- Markdown/文档文件中的“漏洞”描述。
- 纯 DoS/资源耗尽、速率限制类（除非用户明确要求）。
- 内存/CPU 理论安全问题且无代码证据。
- 开放重定向等低价值且无利用链的单独项（可记入附录）。
- 无 Source→Sink 或无可复现 PoC 的条目 → 仅可进入“待确认”，不得进终报。

先规则排除，再对剩余项做 AI 复核（见 [prompts/false-positive-rules.md](prompts/false-positive-rules.md)）。

---

## Step 0：参数与范围

1. 确认或推断：目标路径、报告目录、主语言、扫描范围（全库 / 仅变更）。
2. 若未指定语言，扫描项目根目录识别技术栈（如 `pom.xml`/`build.gradle` → Java；`go.mod` → Go；`requirements.txt`/`setup.py` → Python；`composer.json` → PHP；`*.csproj`/`*.sln` → ASP/ASP.NET；`package.json` → Node.js）。
3. 创建报告目录；用 TodoWrite 记录本步完成。
4. 输出简短摘要：目标路径、报告目录、主语言、扫描范围。

---

## Step 1：Recon

1. 根据主语言选用首轮 Recon 策略，见 [prompts/recon.md](prompts/recon.md)。
2. 语言相关细节**按需读取**（仅加载与 Step 0 识别出的主语言一致的文件）：
   - Java → [prompts/java/java-audit.md](prompts/java/java-audit.md)
   - Go → [prompts/go/go-audit.md](prompts/go/go-audit.md)
   - Python → [prompts/python/python-audit.md](prompts/python/python-audit.md)
   - PHP → [prompts/php/php-audit.md](prompts/php/php-audit.md)
   - ASP/ASP.NET → [prompts/asp/asp-audit.md](prompts/asp/asp-audit.md)
   - Node.js → [prompts/nodejs/nodejs-audit.md](prompts/nodejs/nodejs-audit.md)
3. 产出物（写入 `<报告目录>/recon.json` 或等价）：
   - 技术栈、框架与关键依赖；
   - 路由/API 表（URL/方法/控制器）；
   - 认证/鉴权方式与入口；
   - 敏感 Sink 与数据流入口（供 Round 1 选型）。
4. 用 TodoWrite 标记 Recon 完成。

---

## Step 2：Round 1 专项扫描

1. 按语言与漏洞类型拆分任务，每类对应一组 Sink + 数据流策略：
   - **SQL 注入**：见 [prompts/round1-sql.md](prompts/round1-sql.md)
   - **RCE/命令执行**：见 [prompts/round1-rce.md](prompts/round1-rce.md)
   - **SSRF**：见 [prompts/round1-ssrf.md](prompts/round1-ssrf.md)
   - **认证/鉴权/越权**：见 [prompts/round1-auth.md](prompts/round1-auth.md)
   - **文件操作（读/写/路径穿越）**：见 [prompts/round1-file.md](prompts/round1-file.md)
   - **业务逻辑**（参数篡改、状态机、IDOR、并发）：见 [prompts/logic/round1-logic.md](prompts/logic/round1-logic.md)
2. 采用「策略 + 可选案例」：先按 Sink 与数据流扫描，必要时注入 CVE/GHSA 类案例（控 token）。
3. 业务逻辑专项：以攻击者视角（渗透测试专家），关注 `user_id`、`role`、`is_admin`、`skip_auth` 等语义，枚举相关路由并在 Service 层追踪用法。
4. 若为 diff-aware：仅对变更涉及的文件/模块执行对应专项，避免全库噪音。
5. 产出物：`<报告目录>/round1-findings.md`，每条暂不要求终报格式，但需包含 Source、Sink、简要描述。
6. 用 TodoWrite 标记 Round 1 完成。

---

## Step 3：二轮验证与误报过滤

1. **规则过滤**：对 `round1-findings.md` 应用误报规则，剔除不进入终报的类别（见上文误报控制）。
2. **分组**：按漏洞类型或模块对剩余条目分组。
3. **AI 复核**：对每组逐条复核：
   - 数据流是否完整（Source→Sink）；
   - 是否可触发（有 PoC 或明确复现步骤）；
   - 是否已有有效防护（若已防护则降级或剔除）。
4. 使用 [prompts/false-positive-rules.md](prompts/false-positive-rules.md) 中的复核指令；可维护「本组织视为误报」清单并写入 Prompt。
5. 合并结果为**已确认漏洞**唯一来源，写入 `<报告目录>/confirmed-findings.md`。
6. 用 TodoWrite 标记二轮完成。

---

## Step 4：攻击链与报告

1. 对 `confirmed-findings.md` 做产出物×攻击面关联，见 [prompts/attack-chain.md](prompts/attack-chain.md)。
2. 按 [OUTPUT-TEMPLATE.md](OUTPUT-TEMPLATE.md) 生成终报。
3. **硬性要求**：每漏洞必须包含 API URL、Source→Sink、PoC、修复建议；若有 HTTP MCP/动态验证，可加「已动态验证」标记。
4. 终报输出路径：`<报告目录>/audit-report.md`。
5. 用 TodoWrite 标记报告完成。

---

## 流程小结

| 阶段       | 产出物                  | 下一步           |
|------------|-------------------------|------------------|
| Step 0     | 参数摘要                | Recon            |
| Step 1     | recon.json              | Round 1 选 Sink  |
| Step 2     | round1-findings.md      | 二轮验证         |
| Step 3     | confirmed-findings.md   | 攻击链与报告     |
| Step 4     | audit-report.md         | 结束             |

所有规则、产出物路径与格式均固定，便于在其他载体上复现同一套审计流程。
