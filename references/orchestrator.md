# orchestrator：执行调度与证据裁决

主会话进入已授权任务的执行、派发、候选变化、恢复或收口时读这份文档。它规定怎么切任务、派给谁、冻结什么、何时让证据失效，以及裁决与账本怎么记。验收判据本身、各角色怎么做自己的活，分别由任务包和对应角色 reference 承载，本文件不重复。

## 目录

- 任务包最小字段
- 垂直切片
- 依赖与调度
- 单写者
- 角色加载清单
- 独立性与上下文
- 候选冻结
- 检查选择与证据失效
- 异常裁决
- 状态归属
- 账本与证据格式

## 任务包最小字段

每个派出的任务包至少写清：

1. 目标与非目标：本次切片要解决什么、明确不解决什么。
2. 允许影响的文件：收窄到具体路径或目录；未列出的路径默认不写。
3. 契约与版本：任务依据的需求/设计/接口记录的指针与版本号。
4. 依赖：本任务需要哪个先行任务的哪个输出（按产物写，不按任务写）。
5. 验收判据：可观察、可执行的判据；派发前先定，不是做完再补。
6. 停止条件：什么情况下停步上报（契约不清、越出允许文件、熔断触发）。

任务包写契约与已约定的接缝，不写具体实现代码与行号——静态预生成的代码在实现时极易失效，且「计划比代码长」本身就是信号（workflow-repos-comparison.md §D.4）。

## 垂直切片

按端到端的窄切片切任务：一个切片穿过所有受影响的层，证明一个可观察结果，体量以一个新上下文能闭环为限（workflow-repos-comparison.md §C.1 共识 2、§E）。

不要横向按层切（「先把所有数据层改完」）。横向切片在最后一个切片之前没有任何可验证的行为，失败会级联回溯；每个切片自带的测试与该切片同批落地，不集中到收尾阶段（workflow-repos-comparison.md Findings 5）。

## 依赖与调度

- 依赖写精确：只声明需要的输出产物（文件、接口、冻结的契约），不声明抽象的「等某任务做完」。被依赖任务只需交付该产物即解锁下游。
- 语义独立才并行：两个任务之间没有共享写路径、没有产物供需关系，才允许并行；只靠「看起来不冲突」不算独立。
- 有界实验并进任务：存在尚未排除的实现方案时，把「限定时间内试出答案」本身切成一个任务，给明确的停止条件与产出物（结论 + 关键差异），不占着实现任务的位置无限探索。

## 单写者

- 一个文件同一时刻只有一个写者；派发任务时按「允许影响的文件」划互不重叠的写域（roles-and-granularity.md §一.2「文件所有权」）。
- 一个 cwd / worktree 同一时刻只有一个写者；任务包写哪个目录树是独占的。
- 并发只用于只读工作（scout、审查、调研）。任何需要写的角色先排队拿独占写域。

## 角色加载清单

派发时在任务书里写明该角色要读哪份 reference 的绝对路径，不依赖相对路径或技能自动发现。对照表：

| 派发对象 | reference |
| --- | --- |
| architect | F:/documents/software/skills/execution-standards/references/architect.md |
| worker | F:/documents/software/skills/execution-standards/references/worker.md |
| designer | F:/documents/software/skills/execution-standards/references/designer.md |
| experience-checker | F:/documents/software/skills/execution-standards/references/experience-checker.md |
| verifier | F:/documents/software/skills/execution-standards/references/verifier.md |

项目适配字段从 F:/documents/software/skills/execution-standards/references/project-adapter.md 或仓库适配文件摘取，按角色只给必要部分；体验检查者只收其中的用户可见标准摘录。

## 独立性与上下文

派发记录四件事：身份（角色名与 run-id）、模型、上下文来源（fresh / fork）、实际输入清单（给了哪些文件、哪些结论）。

- 审查与验收一律用 fresh 上下文：不转发作者推理、自测结论或推荐答案，只给冻结候选、契约与检查对象。作者实例不复用为审查实例，角色名可以复用。
- 架构审查（review 模式）的实例模型家族须与被审设计/实现的作者不同；同家族不同 Persona 不能算独立审查——同模型家族共享盲区，一致性是虚假的（roles-and-granularity.md §三.2）。
- 派发时核对实际输入：子代理拿到的恰是清单上那些；多给了实现背景就重新打包。

## 候选冻结

进入审查或验证前先冻结候选，冻结记录写进账本：

- 干净提交：用 commit 标识。
- 脏树：除 HEAD 外，还须纳入相关未跟踪源文件、测试、工具与配置的指纹（路径 + 哈希），不能只写 HEAD。
- golden、验证脚本与工具版本属于候选的一部分：它们变了，候选指纹就变。

## 检查选择与证据失效

- 按改动范围与风险选最小充分检查：选能判定本次验收判据的命令集合，不默认全量。
- 修改源码、测试预期、验证工具或 golden 后，由该输入支撑的证据条目全部失效；只重跑受影响的检查，复用其余证据时在账本写明依据（该证据的 inputs 未受影响）。
- 失效判断靠 evidence.jsonl 的 inputs 与 assertionFreeze 字段对指纹，不靠记忆。
- 原始日志先写忽略目录；关闭任务前保留支撑最终结论的必要原件。已删原件的证据标记不可复查，不引用成独立验证事实。

## 异常裁决

- 歧义停步：产品目标、契约或授权范围不明确时，停的是受影响的那部分；其他独立工作继续。停步同时写清卡点与需要的输入。
- 连续失败熔断：同一问题连续三次修复失败后停止打补丁，保留差异与证据，升级 architect 诊断。
- 有界复审：审查或验证不通过时，复审轮次设上限（建议两轮）；到上限要么升级裁决、要么换作者实例重做，不把阻塞项搁着不动。
- 每次裁决记进账本：做了什么、为什么、代价、授权依据——记裁决，不记过程流水（workflow-repos-comparison.md §C.2 分歧 3，裁决优先于停顿）。

## 状态归属

只有主会话更新任务状态与综合结论。子代理返回事实、观察与建议，不返回「完成」判定；子代理也不能自行降低门槛（改判据、跳过检查、重定义验收）。

## 账本与证据格式

ledger.md 由主会话维护，一个任务一份：

```markdown
# <task-id> <任务名>
授权/需求/契约：<现有记录指针及版本>
目标与非目标：<本次垂直切片>
候选：<基线 revision + 被测树指纹/产物哈希；脏树附未跟踪文件指纹>
上下文清单：<角色、模型、fresh/fork、实际输入、reference 版本>
所有权：<实例 → 产品/测试/文档路径；允许的生成目录>
验收：<AC-ID → 检查/观察 → 判据 → 证据 ID>
用户可见变更：<是/否及依据；designer / experience-checker 身份>
状态：ready | running | checking | blocked | done
裁决：<做了什么；为什么；代价；授权依据>
未覆盖/例外：<影响、责任人、下一步>
```

evidence.jsonl 由 verifier 或运行者产生，每行一条命令记录：

```json
{"id":"E1","task":"T1","ts":"2026-10-17T15:04:22+08:00","candidate":"<被测树/产物指纹>","executor":"<verifier-run-id>","command":"<实际命令>","cwd":"<绝对目录>","exitCode":0,"rawOutput":"<stdout 原件位置>","stderr":"<stderr 原件位置>","assertionFreeze":{"ref":"<检查契约/验收清单位置>","sha256":"<哈希>"},"assertionSource":"<需求 | 契约 | 独立样本 | 字面量>","inputs":[{"path":"<本命令读取的源文件/测试/golden/工具>","sha256":"<哈希>"}],"expected":"<判据>","observed":"<实际观察>","status":"passed"}
```

规则：

- 四元组是 command、cwd、exitCode、rawOutput/stderr 位置——缺一项证据不成立。
- 命令未运行时 exitCode 写 null，不能填 0；exitCode 0 只说明进程退出，仍须满足 expected 的行为断言。
- assertionSource 区分断言出自需求、公开契约、独立样本还是明确字面量；验证工具与 golden 的哈希放进 inputs，使证据失效可计算（critique-design.md R5）。
- 质性检查（截图、帧号、语义节点）引用呈现物定位与规则，不伪造自己运行过命令。
