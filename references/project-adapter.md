# project-adapter：项目适配层

## 目录

- [谁读、什么时候读](#谁读什么时候读)
- [怎么填](#怎么填)
- [空模板](#空模板)
- [MReader 填写示例](#mreader-填写示例)
- [aiGame 填写示例](#aigame-填写示例)

## 谁读、什么时候读

- 适配维护者：首次为仓库建立执行映射时读；验证入口、证据位置或项目底线发生变化时回来改。
- 各执行角色：只取自己需要的字段——命令、证据目录、写入范围、项目特有底线。通用执行规则由 SKILL.md 承担，本文件不重复。

适配文件是一份事实映射：把「这个仓库里什么算有效证据」落成可执行的表格。它不写流程、不设门禁，只保证角色拿到的命令和路径是真实存在的。

## 怎么填

- 每个仓库放一份 EXECUTION_ADAPTER.md 在根目录，控制在约 60–100 行；只映射项目事实与检查入口。
- 「事实来源」必须指向当前有效的项目规则文件，不复制其内容；适配文件失效的最早信号就是来源文件改了而适配没跟上。
- 「验证入口」每条注明适用条件、cwd、命令和判据；只填核实存在、实际运行过的命令，不从文档目录里抄未验证的入口。
- 「项目底线与触发」只写本仓库特有的约束，比如不可替换的检查、数据/权限边界、特殊失败规则；人人遵守的通用纪律留给技能。
- 轮次内产物：本轮两份填写示例仅作为本技能的材料，不写入项目仓库。

## 空模板

```text
# 执行适配 v1
项目/技术栈：
事实来源：<当前有效的项目规则与开发入口>
技能入口：execution-standards
验证环境：<shell、工具链、设备或引擎要求>

## 验证入口
| ID | 适用变更 | cwd | 命令/入口 | 通过判据与输出 |

## 证据
任务账本：<现有历史目录>/<task-id>/ledger.md
命令记录：<同任务目录>/evidence.jsonl
临时原件：<已忽略目录>/<task-id>/<run-id>/
长期原件：<必要证据的保留位置>
保留/脱敏：<规则、到期后的可复查状态>

## 所有权
产品与测试：worker，每任务缩小至明确路径。
设计与审查：只读；主会话/宿主保存报告。
验证：仅允许指定命令、生成目录与隔离测试数据。
账本及当前文档：由主会话指定唯一写者。

## 项目底线与触发
<不可替换的检查、数据/权限边界、UI 矩阵、特殊失败规则>
```

## MReader 填写示例

技术栈：Android、Kotlin、Compose、Gradle。

- 事实来源：PROJECT_DECISIONS.md；docs/README.md；受影响专题与测试导航。
- cwd：仓库根；Windows 用 .\gradlew.bat，其他现有环境用 ./gradlew。

验证入口：

| ID | 适用范围 | 命令/入口 | 判据 |
| --- | --- | --- | --- |
| COPY | 文案、资源及相关实现 | .\gradlew.bat verifyUiCopy | 硬校验通过；报告项单列，不擅自扩充基线或豁免 |
| ERROR | 引擎错误码及界面映射 | .\gradlew.bat verifyErrorCodes | 新错误码具备本地化映射，或有获批的逐条诊断豁免 |
| GEOMETRY | 间距、chip、布局 token | python tools/verification/measure_ui.py <截图> --rows --density <实测值> --json <输出> | 将数值与已批准间距/几何规则比较；返回 0 只说明测量完成 |
| TEST | 行为变化 | 受影响模块的既有测试入口；集成风险需要时运行 .\gradlew.bat test | 覆盖本次行为及回归，记录实际命令 |
| UI | 用户可见变更 | 真机安装、操作、截图；宽屏/大字用现有渲染断言入口 | 截图、语义/几何和状态一致；不用黑屏模拟器截图作视觉证据 |

证据位置：

- docs/history/testing/<date>-<task>/ 放 ledger.md、evidence.jsonl、必要的脱敏原件和报告。
- 临时原件放 tmp/execution/<task>/<run>/，沿用已有忽略规则。
- docs/testing/ 保留导航；当前功能、架构和状态文档只更新事实及证据链接。

项目底线：

- 原始本地文件只读；迁移和备份遵守当前项目决策。
- 引擎返回稳定错误码，app 层负责本地化；不降低 COPY/ERROR 判据。
- 48dp 是触控目标，不是 chip 视觉高度；几何以实际 density、bounds 和项目 token 核对。
- 真机禁止 connectedDebugAndroidTest；必要 instrumentation 按既有手动安装及 am instrument 方式执行，保护用户数据。
- 产品文件仅由获派 worker 修改；设备操作使用获准范围，不清空真实书架。
- 不新增根目录日志。

来源：PROJECT_DECISIONS.md 的验证纪律、文档治理、chip 行与间距节；measure_ui.py 的参数和返回行为。

## aiGame 填写示例

技术栈：Godot 4.7 mono、GDScript、C#；正式工程 .NET SDK 8.0.400。

- 事实来源：AGENTS.md；.pi/APPEND_SYSTEM.md（pi 侧委派授权与角色表所在）；NEXT_CHAT.md 最新替代节；research/development-guide.md；game/runtime/motion/README.md。
- cwd：仓库根；每次检查绑定新的输出目录。

验证入口：

| ID | 适用范围 | 命令/入口 | 判据 |
| --- | --- | --- | --- |
| MANAGED | C#、工程或构建来源变化 | python tools/managed_build.py --outdir <新目录> | 完整重建及来源、编译项、程序集身份和适用加载副本校验通过 |
| STAGED-CS | staged 工程 C# 接线 | python tools/worldgen_validation.py --case csharp --outdir <新目录> | 经 stage() 重建，实际引擎执行断言通过，无引擎错误 |
| WORLD | 世界生成相关变更 | python tools/worldgen_validation.py --case p0 --outdir <新目录> | 适用合同断言通过；不外推窗口、性能或完整玩法 |
| WINDOW | 实验入口用户可见变更 | python tools/worldgen_window_validation.py --outdir <新目录> | 自有窗口操作及外部呈现证据；其他入口改用其对应既有验证器 |
| FULL | 确有集成风险 | python prototype/run_validation.py | 只声明该入口实际覆盖范围 |

证据位置：

- 任务书、账本、报告和必要原件：research/history/<task>-<date>/。
- 大型临时原件与 staged 工程：third-party/_analysis/execution/<task>/<run>/，沿用忽略规则。
- NEXT_CHAT.md 新节只写当前状态、下一步和证据指针，明确替代范围，不复制全量日志。

项目底线：

- 保留 stage() 完整重建及 trust anchor；不用普通增量构建成功替代新鲜度。
- unsafe_skipped 不可作为证据；verify-only 也必须通过现有来源与加载身份校验。
- AI 只产生受限候选；可信系统决定状态、资格和费用，View 不获得世界写权限。
- 验证目录和测试存档隔离；破坏性恢复测试不得指向真实游玩存档。
- headless、窗口操作、性能、真实 API 和跨电脑验证分别报告。
- 每轮保留变更验证工具的源码快照，或至少哈希与行数；工具变化会影响证据时效。
- 当前交接已记录部分运动验证的时序抖动，不把 13/13 单独当成稳定门禁；真实 API 不作为自动回归重跑。

来源：development-guide.md 启动与验证节；motion/README.md 构建来源自证、起引擎前的构建/校验接线节；NEXT_CHAT.md 最新流程改进及既有测试脆弱性节。

---

*依据：standards-layering.md §2、§5；workflow-repos-comparison.md §D。*
