序
Hi，大家。你近期是否已经看到过很多AI代码审计的落地方案，授人以鱼不如授人以渔。市面上的方案虽多，但直接照搬往往水土不服。因此，在这篇文章中，我们将换一种思路：不盲从单一工具，而是从底层逻辑出发。接下来，我将带大家集中拆解目前业内极具代表性的几个优秀方案。我们的核心目标是“取其精华”——深度剖析这些方案背后的设计理念、Prompt工程技巧以及工程化落地的思路，最终将这些优点吸收转化，从0到1编写一个属于我们自己的代码审计 Skill。
本篇文章我们将以 Cursor 作为主要载体进行实操演示。但请记住，工具只是外壳，当我们真正掌握了 AI Skills 的编写方法和评价逻辑后，你可以游刃有余地将这套方法论迁移到你未来的任何安全任务中。
Cursor Skills 编写基础
在一切开始之前，我们需要简单知道Skills是什么，以及他是如何作用的。
Skills是什么
 Skills 本质上是一套“给 AI 的规则和工具说明“，你可以把它理解为：给 AI 提供一组“可调用的能力模块”。

基础语法：
载体	作用
.cursorrules	项目级规则（代码风格、安全规范），相当于项目系统 Prompt 
Cursor Skills（如 SKILL.md）	放于 .cursor/skills/，这里面写 name、description；description 决定何时触发
Skill 正文	输入/输出约定、分步流程、子 Agent 调用方式、产出物路径与格式 
编写要点：触发条件写清（如(安全审计)(漏洞扫描)）；输出格式硬性规定（问题单含 Source→Sink、PoC、修复建议）；分阶段可追溯（Step 0/1/2…、TodoWrite）。
规则体系：
类型	作用	示例
流程规则	阶段划分、谁来做、产出物	Recon → 专项扫描 → 二轮验证 → 攻击链 → 报告
输出规则	单条发现的呈现结构	API URL、Source→Sink、HTTP PoC、修复建议
误报控制	何类需二次确认或过滤	DoS、纯理论、无代码证据不进入终报
语言/框架规则	按目标选 Sink 与策略	Go 用 go-.md，Java 用 java-.md，首轮识别技术栈
好的方案都在规则体系上做显式设计，而不是靠一段笼统 Prompt。
Claud code -- 大模型原生代码安全方案
参考：claude-code-security-review核心：围绕(变更)扫描 + 双层误报过滤。
Diff-Aware 扫描机制
传统 SAST 全库扫描噪音大，且全库扫描对于AI来说上下文过长无法控制。该方案只分析 PR 的 diff：

触发于 PR 创建/更新，Action 拉取变更文件与行；Claude 只看到(本次引入的改动)，结果以 PR 行内评论 挂到对应代码行。

预设的误报过滤
两层设计：
1. 规则过滤：检查以下场景
- Markdown 文档文件
- DOS/资源耗尽
- 速率限制问题
- 内存/CPU 安全问题
- 开放重定向等
2. AI 过滤：对通过一层的发现，用 Claude + 可配置的 false-positive-filtering-instructions 再判(是否值得报出)。还可通过 custom-security-scan-instructions 注入项目/组织定制要求。

在二轮验证甚至多轮中要求对首轮发现逐条复核(是否可触发、是否有有效防护)，仅确认项进终报；维护(本组织视为误报)清单并写进 Prompt。

yhy0 -- 漏洞数据库驱动方案
参考：https://github.com/yhy0/ghsa-skill-builder
yhy0的思路是通用 Prompt 缺(真实漏洞模式)，AI 只会泛化知识。该方案用 GHSA + HackerOne 公开报告 作数据源，让 Claude 提炼成可被 Agent 调用的结构化 Skill。
框架级定制化 Prompt
Skill 分两层：
Real-World Cases：在第二层Skill中他做了一个来自按需加载的步骤，这些 CVE/GHSA/H1，含根因、Source→Sink、漏洞代码片段、攻击路径、(为何传统扫描器易漏)。先策略、后案例，控 token 又提命中率。
按漏洞类型与语言拆多 Skill（例如这里 Python 注入、Go 认证绕过），每 Skill 内(策略 + 案例)两层。

高质量数据库的吸收
● GHSA：有 patch、CWE、版本，适合提炼代码级模式（如 dict key 未转义、ORM 聚合注入）；可按生态、时间、严重程度、CWE 拉取。
● HackerOne：侧重攻击链与利用过程，与 GHSA(代码模式 + 实战手法)互补。
陌陌安全团队  -- 逻辑漏洞挖掘
参考：https://mp.weixin.qq.com/s/mz4yhuzfylCk69nNRin7Vw
陌陌安全的这篇文章主要谈了常规的自动化工具极难发现深层业务逻辑漏洞。该文章提到逻辑漏洞没有定式的Source-Sink形态，所以逻辑漏洞的有效上下文应该为API的完整逻辑链路。与此同时，真实业务仓库中通常会利用注解符来对API注入拦截器、过滤器以及Controller增强类，在实际的AST解析时无法识别到这一引用关系，我们通过在项目扫描时，对这一部分单独提取扫描。这里的思路就无法直接照搬，我们重点看可提取的点。
针对权限/逻辑漏洞的逻辑强化：
攻击者视角的角色设定作为强化：在 Prompt 中深度赋予 AI “理解业务的高级渗透测试专家”的角色，强制其打破常规的正向开发思维，重点审视参数篡改、状态机绕过、越权访问（IDOR）以及并发条件竞争等复杂的逻辑缺陷。利用模型对业务变量名（user_id、role、is_admin、skip_auth）的语义理解，定位敏感接口，查其是否参与鉴权/过滤、是否可被篡改。要求枚举接收上述参数的路由，追踪在 Service 层的用法（业务逻辑）。以及还有个关键点就是这里的上下文提取步骤，这里是以路由有限的提取方法。

实践：落地你自己的skill
我认为可以可抽象为五步，再通过每个人的想法按需修改：
1. Step 0 参数与范围：目标路径、报告目录、主语言（或自动识别）、扫描范围；可选(cursor的话是可以针对本地变更来做分析)。
2. Recon：语言相关首轮 Prompt（如 java/python/go-audit.md）输出结构、技术栈、路由表、认证、依赖，供后续选 Sink 与策略。
3. Round 1 专项扫描：按语言与漏洞类型拆 Prompt（SQL、RCE、SSRF、认证、文件、反序列化等），每专项一类 Sink+数据流；采用(策略 + 可选案例)，业务逻辑单独专项并加业务对象/状态、标志位/权限分支。
4. 二轮验证与误报过滤：按报告分组，子 Agent 逐组复核数据流、可触发性、有效防护；先规则排除再 AI 复核，合并结果为已确认漏洞唯一来源。
5. 攻击链与报告：对确认漏洞做产出物×攻击面关联（见 attack-chain-prompt.md），按模板出终报；硬性要求每漏洞带 API URL、Source→Sink、PoC、修复建议，有 HTTP的MCP 可加动态验证的标记。
规则体系写死在 Skill 里：流程、产出物路径、格式、误报界定都白纸黑字，便于在任何载体上复现同一套审计流程。

---

## 本仓库已落地的 Skill

已按上述五步与规则体系实现 Cursor 可用的代码审计 Skill，路径：

**`.cursor/skills/codo-audit/`**

- 主入口：`SKILL.md`（五阶段流程、输入输出约定、误报控制）
- 终报模板：`OUTPUT-TEMPLATE.md`
- 通用 Prompt（主文件夹）：`prompts/recon.md`、`prompts/round1-*.md`（SQL/RCE/SSRF/auth/file）、`prompts/attack-chain.md`、`prompts/false-positive-rules.md`
- 按需子文件夹：`prompts/java/`、`prompts/go/`、`prompts/python/`、`prompts/php/`、`prompts/asp/`、`prompts/nodejs/`、`prompts/logic/`（各含对应 *-audit.md 或 round1-logic.md）

在 Cursor 中对该项目说「安全审计」「漏洞扫描」或「code audit」即可触发；报告默认输出到 `./audit-report/`。