# Planner 分阶段工作流

以下主流程的 current/test-plan/evidence、四文件和不生成 tasks-v* 规则仅适用于 v2。对未完成授权迁移的 v1 Root，逻辑阶段与门禁相同，但继续使用原 Intake/Profile/Spec/Task/Report 版本路径和 state.transitions；已发布路径不覆盖，修订发布新路径。不得在 v1 中偷偷创建 v2 当前入口或精简旧 state。

## 1. 建立仓库基线

对于 Brownfield 项目，在 current 的 Repository Profile 章节记录画像与预算，在 evidence 保留基线命令证据。对于 Continuation，还必须填写 Takeover Assessment 章节，把现有工作分类为已接受、未验证、部分完成、Stub、冲突、废弃、未知或未开始，并给出差距分析。

识别目标仓库已有的 Issue、Spec、测试、CI、发布和状态协议。在仓库画像的现有章节中记录与 DevFlow Gate 的语义映射；等价证据只有在作者职责分离、版本或 SHA 绑定、适用范围和时效性均满足时才能直接引用。不得为同一事实维护两套互相竞争的产物或状态；冲突时采用更严格的规则并记录依据。

保护所有任务开始前就存在的未提交修改。使用 [最小改动](../../ric-devflow/references/shared/minimal-change.md) 区分必要重构、附带重构和机会性重构。

当项目属于续接、存在脏工作区、分支或风格不明确、风险较高、基线不可信或接管风险显著时，请求执行 `BASELINE_REVIEW`。处理 Finding 时不得改写 Reviewer 的历史产物。

## 2. 定义行为和任务

遵循 [需求分析](requirement-analysis.md)，先在草稿阶段完成事实预检和规模判断，再在 current 中维护 Root、Spec、适用的 UI、决策、风险、回滚和预算。默认一次聚焦侦察加一次缺口补查，连续两次无新增事实就停止同类搜索；继续调查必须关联当前 AC、已观察失败或必要约束。必须明确分开当前行为、目标行为和保持不变的行为。

只对确有独立发布/验收障碍的大型请求在首次 G2 前做一次分期；优先在同一 Root 保留完整能力清单、阶段依赖和延期项，仅展开当前阶段。多文件、步骤多、耗时长或交接材料多本身不是拆分理由；先用路径/对象定位收紧交接，不递归创建 Root 或 Task。

遵循 [Task 与冻结 DAG](../../ric-devflow/references/shared/task-decomposition.md)，在 current 中维护最少必要节点及独立修订，不生成 tasks-v*、Review Target 包装或全套版本矩阵。Task 是完整交付结果，内部实现/测试/文档步骤不另立节点；只有真实依赖/发布/权限/所有权边界才拆分。记录适用门禁；按门禁策略直接省略不适用或已有有效证据覆盖的额外审核，不要求每个节点重走 G0–G4。

只有可发现事实已解决、必要假设和阻塞项已显式记录、当前交付阶段可独立审核时，才将精确版本交给 Reviewer 执行 `SPEC_REVIEW`。审核结果为 `APPROVE` 后，取得用户对同一产品行为版本的确认。随后要求 Tester 创建测试计划，再由 Reviewer 执行 `TEST_REVIEW`。以上三项批准没有同时保持有效前，不得进入正式实现。

## 3. 协调实现与集成

按[Git 策略](../../ric-devflow/references/contracts/git-policy.md)确认唯一功能/集成分支和真实目标分支，无命名规范时用 `feature/<REQ-ID>`，绝不能默认目标为 `main`。只有当所有依赖满足[有效依赖](../../ric-devflow/references/contracts/workflow-state.md#有效依赖与恢复)、存在最新的已验证集成 SHA、该 Task 的必需环境与权限可用，且写入区域不存在所有权冲突时，才把 Task 标记为 `READY`。不同环境优先在测试计划内分验证切片，不自动新增 Task；只阻塞受影响部分。必需验收条件缺失时不得进入 `READY`；冻结后的结构调整必须先满足例外条件。

每次只向 Implementer 委派一个 Task 或一个已归因为实现问题的 Defect，并提供当前 Spec、测试计划、所分配 detached worktree、来源 base_ref、不可变的 base SHA、续作候选 head、路径预算、仓库画像和参考实现。工作区绝对位置仅在会话/工具参数解析，持久记录沿用相对路径与 SHA。需要 Tester 编写独立测试时，按[测试代码交付](../../ric-devflow/references/shared/test-code-delivery.md)在原 Task 内串行交接，汇集各作者真实报告，再请求 Reviewer 对完整候选精确 SHA 范围执行 `CODE_REVIEW`。测试类 Defect 交给 Tester，不伪装成实现类问题。

只有 Planner 可以按照仓库既有合并策略和 DAG 顺序执行合并。每次合并后，都必须要求 Tester 针对新的集成 SHA 做增量验证。

G6 通过后 Task 保持 `VERIFIED` 以解锁依赖。G5 后补改测试也须重新绑定审核、集成与验证；补证期间撤销受影响 Task 的有效已验证状态，无关 Task 不重置。

失败必须先归类为 `SPEC`、`TEST`、`IMPLEMENTATION`、`ENVIRONMENT`、`BASELINE`、`SCOPE_CHANGE` 或 `UNKNOWN`，再路由给相应角色。同一个问题连续三轮未解决时，停止叠加补丁并针对实际失败归因；不自动重新规划或拆 Task。同一产物和审核模式连续两次 `REQUEST_CHANGES` 仍未收敛时，先暂停送审、汇总 Finding 并针对性修正；两种计数独立且不因拆分/换号清零。

## 4. 完整验证、发布审核与关闭

所有 Task 均已验证后，请求 Tester 对精确集成 SHA 执行完整验证，再由 Reviewer 对同一 SHA 和当前全部证据执行 `RELEASE_REVIEW`。只有发布审核通过且操作处于现有授权范围内，才能合并到仓库事实确定的目标分支。合并后要求 Tester 对目标 SHA 执行 Smoke Test。

只有 G0 至 G10 中所有适用证据都保持有效、目标 SHA 冒烟已通过、没有阻断 Finding 或 Defect，并且回滚、迁移、配置和文档义务都已完成时，才能先将当前交付对应 Task、随后 Root 设置为 `DONE`；不能在 Task 集成后提前关闭。
