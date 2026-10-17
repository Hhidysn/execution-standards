# 角色精简方案（role-simplification）

**本文是一份待批准的方案文档，不是已生效的配置；本文不修改任何 settings.json 或 agent 定义文件。**

## 目录

- 目标角色集
- 逐角色处置表
- 已纠正的两条事实前提
- 已证实的事实错误（质询 M2）
- Owner 已确认的决定
- 执行顺序与回滚
- 未验证项

## 目标角色集

精简后日常职责共七种（含主会话），另保留一个按需角色：

| 职责 | 承担者 |
| --- | --- |
| orchestrator | 主会话（编排、写入归属裁决、进度维护） |
| scout | 内置 scout，只读定位 |
| architect | 用户角色，design / review / diagnose 三模式 |
| worker | 内置 worker，有界实现与写入 |
| designer | 用户角色，只读设计 |
| experience-checker | 新建用户角色，独立体验检查 |
| verifier | 用户角色，命令执行、生成物与证据采集 |
| researcher（按需） | 外网调研与独立 audit 模式 |

## 逐角色处置表

| 当前角色 | 处置 | 机制 | 代价 |
| --- | --- | --- | --- |
| architect | 保留 | 吸收 oracle 的升级诊断与 reviewer 的只读审查，成 design/review/diagnose 三模式 | 提示词需按模式分节；单次调用须声明模式 |
| oracle | 禁用 | `agentOverrides.oracle.disabled=true` | 失去独立诊断身份与 fork 上下文；命令诊断转交 verifier |
| reviewer | 禁用 | `agentOverrides.reviewer.disabled=true`，审查并入 architect 的 review 模式 | watchdog_diff 保留在架构审查；缺工具时由主会话提供固定 diff |
| verifier | 保留 | 明确允许生成物、隔离数据与证据写入，禁止源码修复 | 无 |
| council-gpt / council-deepseek / council-gemini / council-glm | 默认禁用 | `agentOverrides.<name>.disabled=true`，原文件保留可恢复 | 高风险议题改用同一只读角色的 fresh 实例，逐次指定不同模型；失去固定命名入口 |
| evidence-auditor | 禁用 | `agentOverrides.evidence-auditor.disabled=true`，职责并入 researcher 的独立 audit 模式 | 审计调用须显式声明 audit 模式 |
| researcher | 保留按需入口 | eject 为用户角色时移除 write 与自动进度写入，报告由宿主保存 | 无 |
| scout | 保留 | eject 时移除 bash/write 与自动进度写入，仅只读定位 | 无 |
| designer | 保留，收窄 | 只读设计：仅可写设计产物目录，不写产品代码；产品 UI 实现统一交 worker | UI 实现链路多一步编排 |
| ops | 默认禁用 | `agentOverrides.ops.disabled=true`，环境诊断归 verifier | 无 |
| delegate | 默认禁用 | `agentOverrides.delegate.disabled=true`，有界实现用 worker | 无 |
| worker | 保留 | 内置角色与现有模型（magpie/devin/swe-2）不变 | 无 |
| experience-checker | 新建用户角色 | 定义见 references/experience-checker.md；tools: read，defaultContext: fresh | 黑盒隔离围栏与视觉多模态能力均为未验证项 |

## 已纠正的两条事实前提

1. **architect 与 oracle 并非只差提示词。** architect（magpie/codex/gpt-6.1-sol，thinking xhigh）tools 仅 read/grep/find/ls，**无 bash**；oracle 同为 magpie/codex/gpt-6.1-sol、thinking max，但 settings override 含 bash，且内置默认 `context: fork`。合并方向不变：升级诊断并入 architect，需要执行命令的诊断归 verifier。
2. **reviewer 与 verifier 工具集不同。** reviewer 有 watchdog_diff、无 bash；verifier 有 bash。架构审查可保留 watchdog_diff；审查角色缺执行工具时，由主会话提供固定 diff，不能因此给审查角色开放 shell。

## 已证实的事实错误（质询 M2）

`defaultProgress` **不在** pi-subagents 的 override 支持字段表（C:/Users/24590/.pi/agent/npm/node_modules/pi-subagents/docs/agents.md:231）。该表仅含：description、advertise、machine、output、outputMode、defaultReads、model、defaultProvider、thinking、systemPromptMode、inheritProjectContext、inheritGlobalContext、inheritSkills、defaultContext、acceptanceRole、disabled、skills、tools、systemPrompt。

因此「用 override 改掉 worker 的 progress.md」不可行。替代方案：逐次调用时传 `progress:false`，或把 worker eject 为用户角色后在定义中关闭自动进度写入。

## Owner 已确认的决定

### D1=(b)：本轮只写方案，配置改动与 aiGame 文件同步执行

本方案未生效。执行前需**同批**修改：

- `F:/documents/software/aiGame/.pi/APPEND_SYSTEM.md`：
  - 第 13 行：明文授权「把 Gemini 3.8 Flash、DeepSeek Flash、GLM Flash 当作默认的交叉验证手段……（可用 council-gemini／council-deepseek／council-glm）」，需改为按需 fresh 实例交叉验证；
  - 第 23–28 行角色表：列出 worker、reviewer、evidence-auditor、oracle、council-*，需与本方案处置表一致。
- `F:/documents/software/aiGame/AGENTS.md`：第 8 行「pi 路径见 .pi/APPEND_SYSTEM.md」与第 30 行的同一授权表述，需同步更新。

### D2=(a)：designer 收窄为只读设计，不改用户级 AGENTS.md

用户级 `C:/Users/24590/.pi/agent/AGENTS.md` 第 35 行写明「所有文件改动由主代理，或由获准写入的 worker / designer 完成」，与本方案「designer 只读」存在张力。不改全局文件的原因：该行是跨项目的长期约定，改动会波及所有项目；收窄由 designer 自身定义与其调用约束在项目内实现。

### D5：designer 不加 web_search

素材由 orchestrator 派 researcher 提供。可选项：单独给 designer 加 `fetch_content` 以读取已知 URL 的视觉参考——若采用，归入 D1 同批执行。

## 执行顺序与回滚

1. **禁用前盘点调用点**：对将被禁用的角色名（oracle、reviewer、evidence-auditor、ops、delegate、council-*）在文档、脚本、常驻文件中按名称 grep，确认无硬引用。
2. **逐项禁用**：一次只加一条 `disabled` override，观察调用链再禁下一项。
3. **保留快照**：执行前备份 settings.json 与各 agent 定义文件，形成可回滚快照。
4. **逐项可恢复**：删除对应 override 条目即恢复原角色；自定义角色文件本身不删除。

settings.json 片段（仅作文档示例，不实际写入）：

```json
{
  "agentOverrides": {
    "oracle":           { "disabled": true },
    "reviewer":         { "disabled": true },
    "evidence-auditor": { "disabled": true },
    "ops":              { "disabled": true },
    "delegate":         { "disabled": true },
    "council-gpt":      { "disabled": true },
    "council-deepseek": { "disabled": true },
    "council-gemini":   { "disabled": true },
    "council-glm":      { "disabled": true }
  }
}
```

注：`disabled` 对自定义（用户级）代理同样有效，故 council-* 与自建角色均可经 override 禁用而不删文件。

## 未验证项

- **跨模型家族的实际可用性**：不同路由名不证明不同模型家族；尤其不能凭 Devin/SWE-2 的名称推断 worker 底层模型。交叉验证「换模型家族」的真实收益需实测确认。
- experience-checker 的黑盒隔离围栏与视觉多模态能力未验证，详见 references/experience-checker.md「未成立项」一节。

*本方案未生效。执行前需 owner 批准，并与 aiGame 常驻文件同批修改。*
