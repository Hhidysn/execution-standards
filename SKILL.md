---
name: execution-standards
description: 用于已授权的软件实现、代码修改、bug 修复、重构、独立审查和验证。规定多代理执行中的最小改动、单写者、独立验证、完成证据，以及 UI、文案和交互变更的设计与体验检查。进入编码或执行检查时使用；需求尚未明确或方案尚未批准时由流程技能先处理。
---

# execution-standards

## 1. 边界（一句话分工）

> i-wish 决定做什么、研究和批准什么、何时进入下一阶段及如何交付；execution-standards 规定已获授权的任务怎样实施、审查、验证和留下可信证据。

本技能不做：不重新采访、不做联网调研流程、不设批准门禁、不授权发布、不替 i-wish 判完成。

## 2. 路由表

| 当前在做什么 | 读哪份 reference |
| --- | --- |
| 当前承担执行调度、合并报告或恢复任务 | `references/orchestrator.md` |
| 当前处理架构边界、升级诊断或独立语义审查 | `references/architect.md` |
| 当前获派产品实现或独立测试实现 | `references/worker.md` |
| 当前制定用户可见变更的设计基准 | `references/designer.md` |
| 当前检查已经采集的外部呈现物 | `references/experience-checker.md` |
| 当前运行验收命令或采集运行证据 | `references/verifier.md` |
| 首次适配仓库、验证入口失效或项目事实发生变化 | `references/project-adapter.md` |
| 本次变更属缺陷修复 / 新功能 / 等价重构 | `references/task-types/bugfix.md` · `references/task-types/feature.md` · `references/task-types/refactor.md` |
| 已有项目适配文件 | 各角色读取必要字段；体验检查者只接收其中的用户可见标准摘录 |

references 一律用绝对路径读取（本环境 read 支持绝对路径）；派发时在任务书里写绝对路径，不要依赖相对路径或技能自动发现。

## 3. 共同底线

常驻四条，因为它们是每次执行都成立、且违反时无明显信号的前提：

- 最小改动：只改完成任务必须改的；不顺手重构、不预埋未来逻辑。
- 禁止无证据自证：作者的「完成」报告不能作为自己的验收结论。
- 目标驱动验证：实施前先知道成功判据，改完验证受影响的行为。
- 达成即停：通过已要求的检查后停止；不追加无关优化、不重复全量测试。

异常分支两条，只在对应条件触发时启用：

- 歧义停步：产品、契约或授权不明确时，暂停受影响部分，继续独立工作；普通实现选择自行处理。
- 连续失败熔断：同一问题连续三次修复失败，停止打补丁，保存差异与证据并升级诊断。禁止自动 reset、stash 或清理用户工作区。

多代理两条：

- 一个文件同一时刻只有一个写者。
- 产出者不能承担该产出的独立验收。

## 4. i-wish 角色 ↔ 本技能职责映射

| i-wish 角色 | 本技能的 reference 或说明 |
| --- | --- |
| 创意提案者 | 用同一只读角色的 fresh 实例，逐次指定不同模型（不新建岗位） |
| 研究者 | researcher（不在本技能内，沿用其发现契约） |
| 设计者 | `references/designer.md` |
| 设计质询者 | `references/architect.md` 的 review 模式 |
| 产出者 | `references/worker.md` |
| 独立验收作者 | `references/verifier.md` |
| 需求审查者 | `references/architect.md` 的 review 模式（需求轴） |
| 规范审查者 | `references/architect.md` 的 review 模式（规范轴） |
| 用户检查者 | `references/experience-checker.md` |

两轴审查必须分别给结论，一边通过不能抵消另一边失败。

## 5. 用户可见变更的触发规则

> 当变更可能改变用户看到、听到、读到或操作到的内容与状态时，必须安排 designer 在实现前确定基准，并安排 experience-checker 在实现后独立检查。

触发面：布局、主题、字体、文案与错误提示、导航、点击/键盘/手柄行为、焦点与语义、加载和恢复反馈，以及影响这些呈现的后端状态变化。

例外：纯内部等价重构且能说明呈现契约未变时不触发；机械文案修改可用已有基准做简短设计检查，不要求重新画图。

## 6. 未成立项（不得冒充通过）

在建立之前，对应结论保持未验证：

- 体验检查的黑盒隔离围栏尚未实现。`tools: read` 只限工具种类，不限读取路径；现有 pi-subagents 原生 permissions 只按工具名 allow/ask/deny，不能限路径。未建立围栏前，体验检查不得计为黑盒门禁通过。
- 体验检查角色的视觉多模态能力未验证。未验证时相关结论强制 NOT_VERIFIED。
