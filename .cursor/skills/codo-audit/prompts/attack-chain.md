# 攻击链与报告阶段

## 目标

对已确认漏洞（confirmed-findings.md）做「产出物 × 攻击面」关联，并按 OUTPUT-TEMPLATE 生成终报。

## 攻击链关联

1. **按攻击面分组**：同一 API/路由下的多条发现可归为一组，便于阅读与修复优先级。
2. **链式关系**：若某漏洞可与其他漏洞组合（如 SSRF + 内网未授权 → RCE），在报告中注明“可组合为攻击链”。
3. **入口与影响**：每条确认漏洞已包含 API URL、Source→Sink；本阶段仅做关联与汇总，不改变单条格式。

## 终报生成

1. 仅将 **confirmed-findings.md** 中的条目纳入终报，不得加入未通过二轮验证的发现。
2. 每漏洞必须满足：**API URL**、**Source→Sink**、**PoC（或复现步骤）**、**修复建议**。
3. 若存在 HTTP MCP 或动态验证能力，可对已验证项加「已动态验证」标记。
4. 输出路径：`<报告目录>/audit-report.md`，格式见 OUTPUT-TEMPLATE.md。

## 可选：严重程度与优先级

可按 CVSS 或内部标准对每条标注严重程度（Critical/High/Medium/Low/Info），并据此排序；若用户未要求可省略。
