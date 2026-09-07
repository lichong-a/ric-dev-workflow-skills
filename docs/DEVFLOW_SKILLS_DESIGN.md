# DevFlow 四角色 Skill 总体设计

> 版本：2.1 · Compact 布局、单层调度与冻结 DAG
>
> 本文是系统导航和完成定义；具体规则只在共享契约维护，不复制全套模板或模式清单。v1.1 的文件版本布局由旧模板兼容，不要求已有 Root 强制迁移。

## 1. 目标与边界

DevFlow 是 instruction-only 的四角色软件交付系统。目标是可接管、可验证的最小完整交付，同时约束调查扩展、文档复制和重复审核。

必须保留独立 Reviewer、Tester、用户批准、G0–G10、不可变证据与真实 SHA 绑定；效率不能通过削弱门禁获得。不新增第五个 Skill、工作流 CLI、Git 包装器、编排/常驻迁移脚本、包清单或生成状态机。目标仓库的原生 Git、测试、CI、Issue/发布协议是执行工具，语义等价的已有证据直接引用，不维护第二套事实。

本版只改变新 Root 默认容器和未来执行策略；v1 记录、报告载荷、状态和 Verdict 继续有效。迁移只转换存储，不能批准产品变化。WanGoPlatform 不属于本包可写范围；全局 Skill 符号链接指向本源树，Custom Agent 配置独立且本轮不改变。

## 2. 分类与主流程

项目上下文：`GREENFIELD`、`BROWNFIELD`、`BROWNFIELD_CONTINUATION`。已有代码、契约、用户、测试、历史或脏工作区即按 Brownfield；续作须分类已接受/未验证/部分/Stub/冲突/废弃/未知/未开始。

主要工作类型为 `feature/continuation/bugfix/migration/refactor/maintenance/infrastructure`；重构不是功能的默认附赠。交付深度为 `FAST/STANDARD/HIGH_RISK`，与项目新旧正交。FAST 缩小文档和审核对象，不省略职责分离；权限、租户、数据、兼容、并发、安全或生产风险要求相应深度。

```text
G0 需求 → G1 条件基线 → G2 Spec 审核 → G3 用户批准 → G4 测试计划审核
→ 单 Task 实现 → G5 代码审核 → 集成 → G6 增量测试
→ 全部 Task 验证后 G7 完整测试 → G8 发布审核 → 目标分支合并
→ G9 目标 SHA 冒烟 → G10 关闭
```

门禁的精确输入和决策所有者见[门禁策略](../.agents/skills/_devflow_shared/contracts/gate-policy.md)。Root 主路径和全部辅助状态、Task 主路径与辅助状态仍由[工作流状态](../.agents/skills/_devflow_shared/contracts/workflow-state.md)定义，不增加迁移或 Compact 专用状态。

首次 G2 前执行一次聚焦事实侦察、一次必要缺口补查和规模判断。当前决策条件满足就进入下一步；连续两次同类检查无新事实停止搜索，继续调查须关联当前 AC、已观察失败或必要约束。默认一个完整纵向 Task；仅真实独立发布/验收边界在初始规划分期，保留完整目标，不因为文件多、上下文长或执行慢递归拆分。

DAG 首次完成即作为送审基线，G2 通过后冻结。内部实现、测试/文档步骤、检查点和审核动作不单独建节点、不重跑 Root 流程。已证实的结构性障碍才通过已有 Decision/Change Log 做最小改图与受影响门禁复审，不自动重建全图。详细判断只在[Task 与冻结 DAG](../.agents/skills/devflow-planner/references/task-decomposition.md)维护。

## 3. 四角色及调用边界

| 角色 | 调用 | 所有权 | 不得 |
|---|---|---|---|
| [Planner](../.agents/skills/devflow-planner/SKILL.md) | 开发请求可隐式；也可显式 | 需求、画像/接管、Spec/UI、Task DAG、预算、状态、归因、调度与授权内合并 | 写生产代码、批准自己、重复充当 Tester |
| [Reviewer](../.agents/skills/devflow-reviewer/SKILL.md) | 仅显式或 Planner 委派 | 只读独立结论与稳定 Finding | 修改被审对象、自创行为、更新状态/合并 |
| [Tester](../.agents/skills/devflow-tester/SKILL.md) | 仅显式或委派 | 测试计划、特征/契约/集成/E2E/回归/Smoke、测试和 Defect 证据 | 改生产/Spec、削弱正确预期 |
| [Implementer](../.agents/skills/devflow-implementer/SKILL.md) | 仅显式或委派 | 单获批 Task/实现类 Defect、单元/组件及模块内测试、实现报告 | 改需求、扩大预算、自审批准、合并 |

完整约束见[角色边界](../.agents/skills/_devflow_shared/contracts/role-boundaries.md)。Reviewer 模式仍为 BASELINE_REVIEW、SPEC_REVIEW、TEST_REVIEW、CODE_REVIEW、RELEASE_REVIEW。迁移完整性用限定范围的 BASELINE_REVIEW，不新增模式。

Root Planner 单层直接调度；其他三个角色不再派生 DevFlow 角色或子 Task。接收角色仅完成指定动作，步骤在原 Task/计划内进行，缺口交回原 Planner。已有 v1 也适用该调度约束，不要求为此迁移布局。

Custom Agents 使用仓库现有 `.codex/agents/*.toml`：Reviewer 只读，其他角色 workspace-write；模型继承宿主。推理强度和 Sandbox 不通过 Skill 静默改动，调用 policy 保持 Planner 可隐式、其余 explicit-only。全局配置是独立文件，安装前审阅，不覆盖同名配置。

## 4. Compact 默认布局与 Git 分类

```text
.devflow/changes/<REQ-ID>/
├── state.yaml
├── current.md
├── test-plan.md
├── evidence.md
├── attachments/   # 必要持久证据、正式快照、一次性迁移索引
└── .local/        # 可丢弃过程文件
```

按进度创建，详细结构见[Compact v2](../.agents/skills/_devflow_shared/references/compact-layout.md)，不在本文复制第二套 Schema。

| 位置 | 内容与所有者 | 业务 Git |
|---|---|---|
| state | Planner 当前状态、Task/有效证据/开放问题索引 | 提交 |
| current | Planner 当前需求、画像、Spec/UI、决策、预算、当前阶段 Task | 提交 |
| test-plan | Tester 的 AC/Oracle/环境/职责/退出条件 | 提交 |
| evidence | 原作者独立载荷，Planner 原样串行追加 | 提交 |
| attachments | 必要脱敏证据、快照和迁移索引 | 提交或引用稳定仓库/CI 产物 |
| .local | Prompt、搜索/Diff 中间件、临时日志/报告、会话 ID/cursor/PID/缓存 | 忽略 |

失败、BLOCKED、未运行项也是正式证据，不等于临时垃圾。证据至少记录可复现条件、cwd、命令、环境、退出码和关键输出；不足时保存脱敏附件，不能仅依赖 .local。凭据/敏感原始数据不写受跟踪产物。

业务仓库仅忽略临时目录，例如 `.devflow/changes/*/.local/`；本 Skill **源码**忽略演示 `.devflow/changes/`，不能照搬。ignore 不会取消已跟踪文件。精确暂存并遵守提交/推送的分别授权，详见[Git 策略](../.agents/skills/_devflow_shared/contracts/git-policy.md)。

## 5. 修订、身份与不可变性

current/test-plan 原地维护最新版，底部 Change Log 记录修订、时间、作者、类型、受影响 ID、原因与证据。Task 保持稳定 ID 和独立修订，不随父文件一起换号；未变对象只引用，不生成 tasks-v*、目标包装或冗余哈希矩阵。

草稿可连续编辑；正式送审前固定文档 Commit，代码审核前固定完整代码候选，验收/切换时合并提交证据与状态。文档身份是完整 Commit SHA + 路径 + 对象 ID；代码仍是 base/head/tested SHA。追加报告的提交不等于先前受测代码，不要求记录保存自身 SHA，避免自引用提交循环。

已送审对象即使被拒绝也可从 Git 历史逐字恢复。报告按记录只追加、保留原作者与全部载荷；更正使用新记录和 supersedes。v2 容器 schema_version 为 2，嵌入原报告保持 schema_version 1 与既有字段。旧 v1 模板原样保留，已发布旧路径不可覆盖。

无 Git、不允许跟踪证据或无提交授权：正式文档边界保存不可变持久快照与哈希，标注仅本地可恢复；不能用临时缓存承担唯一历史。缺少真实代码 SHA 的门禁仍阻塞。详细冻结/取代/失效规则见[生命周期](../.agents/skills/_devflow_shared/contracts/artifact-lifecycle.md)。

先证据后状态；同 ID 同载荷复用，异载荷阻塞；写入中断不得引用半条记录。state 不再累计完整 transitions 或迁移历史，事件进入 evidence。账本由 Planner 汇聚不改变各角色作者身份，原始载荷不可被“摘要优化”。

## 6. 收敛、复审与上下文

[变更控制](../.agents/skills/_devflow_shared/contracts/change-control.md)定义 EDITORIAL/TECHNICAL/BEHAVIORAL：

- 原 Task 内执行顺序/检查点不单独审核；纯 EDITORIAL 记录差异并保留原批准绑定，不声称新修订已审。预算/约束仅局部技术复审；改依赖先满足冻结例外；职责映射仅复核有关映射。
- 行为、权限、契约、兼容或测试预期变化重开受影响 G2/G3/G4 与下游证据。
- 未变对象直接沿用原绑定；批准适用到新的技术对象须有独立适用性记录。新 SHA/环境/输入按实际影响补证，不伪造旧测试在新输入执行。
- 首轮返回当前对象全部可发现阻断 Finding；窄 Delta 由可用且独立的原 Reviewer 复查旧 Finding 与邻域。行为/架构/显著风险扩张或更换 Reviewer 才完整复审对应范围。
- 两次同对象/模式 REQUEST_CHANGES 暂停送审归因；同 Defect 三轮失败停止补丁。分别计数，不因改名重置，不默认重建全套 Spec/DAG/测试模型。
- 推测性优化、无关债务进入延期，不成为返修理由；新发现真实严重问题仍报告。
- 实现者自测，Tester 独立验证，Reviewer 按风险复核，Planner 检查身份与覆盖，不重复完整套件。相同输入有效结果可引用，必须披露复用/新运行范围。

交接只携带 Root、单动作/模式、对象身份与适用 SHA、Finding/Defect、允许/保护路径、输出、首个检查点、完成/停止条件与风险相称预算。完整材料只引用，默认不继承全历史（支持时 fork_turns: none）。

恢复先 state，再按 ID 定位当前章节与证据；共享契约在同会话未变化时不重复加载。事件等待使用 cursor/revision，没有新状态不重发 Prompt、不反复读全任务、不因暂时无 commentary 杀掉正常推理。宿主硬性无进展/等待规则优先；真正错误范围、预算耗尽无进展、工具挂起或权限缺口从现有产物恢复。详见[编排交接](../.agents/skills/devflow-planner/references/orchestration.md)。

## 7. Brownfield、环境与完整交付

[Brownfield 策略](../.agents/skills/_devflow_shared/contracts/brownfield-policy.md)和[仓库侦察](../.agents/skills/_devflow_shared/references/repository-discovery.md)保留：真实分支来源、工作区保护、沿现有纵向路径续作、生成/保护区域、基线分类与特征测试。

默认一个足够相关的稳定实现作参考；有具体缺口再补。仓库规则、真实契约和工具配置优先；不照搬已知不安全模式。重构分 REQUIRED/INCIDENTAL/OPPORTUNISTIC，仅必要改动默认在范围。预算包含测试、生成物、必要文档及余量，不添加统一文件/行/字节硬阈值；宿主已有硬限制及其计数口径继续生效。

合成/Mock、本地真实边界、集成、live、生产验证的权威性分开。缺环境/权限只阻塞能独立拆分的切片，不擅自扩大权限、不以合成代替 live。不可拆分必需 AC 缺前提时 Task 不 READY。失败归因仍为 SPEC/TEST/IMPLEMENTATION/ENVIRONMENT/BASELINE/SCOPE_CHANGE/UNKNOWN；不得把新增需求伪装成 Defect。

## 8. v1 无损迁移

完整执行顺序与恢复命令仅在[迁移参考](../.agents/skills/_devflow_shared/references/legacy-migration.md)维护：

1. 一次聚焦建议；只读、有活跃写入者或授权不足时继续 v1。
2. 盘点全部受跟踪/未跟踪/dirty/未知内容及引用，解决安全保全与归属。
3. 用完整基线提交和持久标签 devflow-migration/<REQ-ID>/v1-baseline 保存原文；标签冲突保留原标签并编号。
4. 隔离生成候选，依据有效证据而非最大版本合并；只导入必要原始证据。
5. 提交逐文件 attachments/migration-v1.md，保留未知字段、所有源路径与恢复关系。
6. 验证字节与语义/引用完整性，独立 Reviewer 审核精确候选；过期 Gate 不复活。
7. 再验源未变，最后切 state；仅移除明确已核验的旧工作树副本，不递归清理 .devflow。

写入失败/源变化/未解决冲突均不完成切换。迁移索引可恢复检查点，不反复全量盘点。发布时按授权同时发布分支和基线标签，并从远端重新取回旧原文；未推送注明仅本地。禁止正常维护删除/移动标签；浅克隆补取标签，最新 ZIP 不包含历史承诺。

## 9. 参考路由与包结构

四个入口保持短而聚焦；_devflow_shared 无 SKILL.md。原 templates 根目录保留 v1，templates/compact 是 v2 四文件。角色 references 按当前模式加载；共享 references 只有在真实路径涉及对应语言/领域时读取，不一次加载所有检查表。

现有语言参考覆盖 Python、JS/TS、Go、Java/Kotlin、Rust、C#、PHP/Ruby、Shell、SQL；领域涵盖架构、API、前后端、数据库、安全、DevOps、分布式系统、AI Agent。UI 复用现有组件/Token/交互，覆盖适用加载/错误/空/权限/离线、响应/焦点/i18n；不创建平行视觉体系。实现准则仍由这些参考及宿主规则维护，不因本文精简而取消。

## 10. 评测与完成定义

[工作流评测](../.agents/skills/_devflow_shared/evals/workflow-cases.md)保留 WF-01–15，并新增 WF-16–22：固定文件/独立修订、Git 分类、全源迁移、失败恢复、本地远端历史恢复、独立角色行为、无 Git/不跟踪/只读/v1 兼容。另保留[触发评测](../.agents/skills/_devflow_shared/evals/trigger-cases.md)和[Brownfield 评测](../.agents/skills/_devflow_shared/evals/brownfield-cases.md)。

WF-23–27 覆盖单交付多步骤、非递归执行、冻结图的普通变化与必要例外、按适用性省略额外审核。WF-06 三轮失败改为针对根因而非默认重规划；评测不要求对无关旧迁移场景整套重跑，但必须说明未运行边界。

包完成条件：

- 四入口通过 Codex validator；YAML/TOML、链接、调用策略、Sandbox、diff 正确。
- 仅指令与必要模板/参考，无 CLI/脚本/清单/第五入口，v1 模板兼容。
- 隔离 Git demo 验证四核心文件、局部修订、正式证据去重、旧字节与当前语义保全。
- 验证中断、源变化、标签冲突、未提交归属、只读/无 Git/不跟踪及旧模式继续运行。
- 用隔离本地远端验证发布基线标签、重新 clone 与浅克隆恢复，不操作真实业务远端。
- 有限独立角色前向评测基于原始输入，检查可观察决定与原样载荷，而非单纯比对措辞。
- 验证报告区分静态检查、合成故障演示、独立评测、历史结果与未运行项；文件数、轮次和耗时不得推测。
- 演示仅安全临时目录，采集脱敏结果后清理；不写入 Skill 包或 WanGoPlatform。

实际运行命令、日期、局限见[验证报告](DEVFLOW_SKILLS_VALIDATION.md)。业务 Root 只有全部适用 G0–G10 有效、开放阻断清零、回滚/迁移/配置/文档齐备且目标 SHA 冒烟完成，才能 DONE；包静态通过不代表业务门禁通过。
