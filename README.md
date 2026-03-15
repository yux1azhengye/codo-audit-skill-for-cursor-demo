# Codo Audit Skill for Cursor

基于 Cursor 的**代码安全审计 Skill**：按「参数与范围 → Recon → 专项扫描 → 二轮验证 → 攻击链与报告」五阶段执行，支持多语言与误报控制。

## 快速开始

1. 克隆或下载本仓库到本地。
2. 在 Cursor 中打开**待审计的项目**（或打开本仓库后指定目标路径）。
3. 对 AI 说：**「安全审计」**、**「漏洞扫描」** 或 **「code audit」**，即会按本 Skill 执行。

报告默认输出到 `./audit-report/`，可在对话中指定其他路径。

## Skill 位置与结构

- **主入口**：`.cursor/skills/codo-audit/SKILL.md`
- **终报模板**：`.cursor/skills/codo-audit/OUTPUT-TEMPLATE.md`
- **通用 Prompt**：`prompts/recon.md`、`round1-*.md`、`attack-chain.md`、`false-positive-rules.md`
- **按需语言**：`prompts/java/`、`go/`、`python/`、`php/`、`asp/`、`nodejs/`，及 `prompts/logic/`（业务逻辑专项）

详细说明见 [.cursor/skills/codo-audit/README.md](.cursor/skills/codo-audit/README.md)。

## 支持语言

Java、Go、Python、PHP、ASP/ASP.NET、Node.js；Step 0 自动识别技术栈后按需加载对应策略。

## License

MIT
