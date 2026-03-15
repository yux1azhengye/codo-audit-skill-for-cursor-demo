# codo-audit 代码审计 Skill

本 Skill 按「参数与范围 → Recon → 专项扫描 → 二轮验证 → 攻击链与报告」五步执行代码安全审计，产出物与误报规则均写死在 SKILL.md 及 `prompts/` 下。

## 如何触发

在 Cursor 中对该项目对话时，使用以下任一表述即可触发本 Skill：

- 安全审计
- 漏洞扫描
- code audit / security review
- 查找 SQL/RCE/SSRF/认证/文件/逻辑漏洞

## 产出物路径（默认）

- `./audit-report/recon.json` — Recon 阶段输出
- `./audit-report/round1-findings.md` — 首轮专项发现
- `./audit-report/confirmed-findings.md` — 二轮确认后的漏洞
- `./audit-report/audit-report.md` — 终报（唯一正式报告）

## 目录结构

- `SKILL.md` — 主流程与规则
- `OUTPUT-TEMPLATE.md` — 终报格式
- `prompts/` — 通用 Prompt（主文件夹）
  - `recon.md` — Recon 阶段
  - `round1-sql.md`、`round1-rce.md`、`round1-ssrf.md`、`round1-auth.md`、`round1-file.md` — 通用专项
  - `attack-chain.md`、`false-positive-rules.md` — 攻击链与误报规则
- `prompts/java/` — Java 按需：`java-audit.md`
- `prompts/go/` — Go 按需：`go-audit.md`
- `prompts/python/` — Python 按需：`python-audit.md`
- `prompts/php/` — PHP 按需：`php-audit.md`
- `prompts/asp/` — ASP/ASP.NET 按需：`asp-audit.md`
- `prompts/nodejs/` — Node.js 按需：`nodejs-audit.md`
- `prompts/logic/` — 业务逻辑专项按需：`round1-logic.md`

语言与业务逻辑相关 Prompt 放在子文件夹，执行时仅按主语言/需要加载对应文件，控制 token。

可修改报告目录（如指定其他路径），流程与格式不变。
