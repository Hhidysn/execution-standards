# execution-standards

多代理执行规范技能：规定已获授权的软件任务怎样实施、审查、验证和留下可信证据。

与 i-wish 的分工：i-wish 管流程（需求澄清、调研、设计门禁、交付）；本技能管执行（谁做、怎么做、证据放哪、谁验）。

## 快速开始

将本目录符号链接到 pi 的技能目录：

```powershell
New-Item -ItemType SymbolicLink -Path "C:/Users/24590/.agents/skills/execution-standards" -Target "F:/documents/software/skills/execution-standards"
```

之后 i-wish 进入执行阶段或主会话派发任务时，按 `SKILL.md` 的路由表加载对应 reference。

## 目录导航

| 路径 | 内容 |
| --- | --- |
| `SKILL.md` | 触发描述、分工边界、条件路由、共同底线、触发规则与未成立项 |
| `references/` | 各角色与任务类型的执行规范（orchestrator、architect、worker、designer、experience-checker、verifier、project-adapter、task-types/） |
