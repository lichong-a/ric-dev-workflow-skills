# 测试代码交付与 SHA

当 Tester 新增或修正需要保留的测试代码时读取本参考。测试计划、测试代码交付、测试执行是三种不同产物；均留在原 Task，不创建测试子 Task，不改变角色批准权或原报告 Schema。

## G5 前形成完整候选

1. Planner 确认原 Task 的 AC、测试计划、测试允许路径和预算；生产路径对 Tester 始终受保护。涉及构建配置或生产 Hook 的缺口先交回 Planner，不能以“让测试跑起来”扩大权限。
2. Planner 串行分配 Implementer 与 Tester 的写入窗口，切换前确认上一作者已停止写入、其改动归属清楚且身份固定。默认在同一 Task 的隔离工作区先完成生产候选，再由 Tester 基于该精确 head 添加独立测试；Tester 不修改或提交上一作者遗留的脏文件。无法安全交接时先阻塞相关写入。
3. 各作者只在已有提交授权内固定自己的候选和报告。例：Implementer 报告 `A..B`，Tester 基于 B 提交测试并报告 `B..C`。两个报告保持原绑定；Planner 将它们与完整 `A..C` 候选送 G5，不把 Implementer 的 B 改写成 C，也不代其补报告。Reviewer 检查真实净差异、血缘及各报告实际覆盖，不能仅拼接报告摘要判定完整。
4. Reviewer 对包含必要生产代码和测试的完整候选独立给出 `CODE_REVIEW`；Planner 按授权与仓库策略集成，Tester 在新集成 SHA 上执行 G6。生产候选到测试候选的串行提交是原 Task 内步骤，不是逐步骤 Gate 或私自合并。

没有提交授权时，交回当前不可变 diff/快照和明确缺口；缺少真实候选 SHA 时不得发布可用于 G5 的正式代码报告。不同工作区间需要合并或冲突处理时由 Planner 协调，冲突代码仍由对应作者修正，Tester/Implementer 不自行合并。

## 代码交付报告与作者

复用 [Implementation Report](../../assets/templates/implementation-report.yaml) 的全部原字段、`schema_version: 1` 和唯一 `IMPL-NNN`；`task_id` 为原 Task，已归因测试问题时 `defect_id` 为原 Defect。

- Tester 在 `summary` 明确“Tester 的测试代码交付”、可审计作者、测试范围和本次动作；`base_sha/head_sha`、`changed_files`、预算合规、命令/结果、风险与未运行项来自实际差异。测试代码报告不等同于测试执行 PASS。
- v2 容器的 `author` 为真实 Tester 作者，`objects` 绑定原 Task/计划，Planner 原样追加；v1 继续原报告路径，作者说明放在既有 `summary`，不添加新必填字段或偷偷迁移布局。
- Implementer 与 Tester 分别发布自己的报告；新报告仅取代本作者需要纠正的旧报告。先前作者未变的交付直接引用，不因后续作者追加提交要求其重写历史。

## G5 后的测试补充或修正

Planner 先确认这是原 Task 内必要测试工作；超预算走局部预算复核，Oracle/环境/权限变化走变更控制。仅测试实现修正不重新规划或重开 G4。

受影响 Task 经 `REWORK -> CODE_REVIEW` 返回候选审核，撤销当前无效的已验证索引，保留历史报告。尚未集成时在原候选上串行固定新 head；已经集成且相关验证有效时从最新已验证集成 SHA 准备隔离修正候选。若首次 G6 失败或原验证已失效，Planner 核对失败报告与血缘，以包含待修测试的当前失败集成 SHA 为隔离修复基线，明确其未验证、不得解锁依赖；不能退到不含该 Task 的旧 PASS 提交，也不要求失败候选先通过才准修复。由 Tester 仅修改测试并报告真实差异，Reviewer 按现有 Delta/完整复审条件审核，Planner 集成，Tester 对新集成 SHA 验证后恢复 `VERIFIED`。这是原 Task 的修复路径，不是新 Task 的 READY 基线例外。

测试类 Defect 由 Planner 归因为 `TEST` 并交给 Tester，不能为了满足旧代码审核入口而伪装成 `IMPLEMENTATION`。审阅接受同一 Task 的测试代码报告。主会话、Planner 和 Tester 都不能替代 Reviewer 批准测试代码；无关 Task 不重置，受影响下游不能使用失效证据继续派发。发布审核后再改测试还须更新受影响的 G7/G8 证据。

## 正式执行与诊断证据

正式 Test Report 的 `tested_sha` 默认是同时包含受测生产代码和所用测试代码的完整提交。执行前核对隔离工作区与该提交一致；未提交测试、临时断言或工作区叠加修改只能产生明确标注的诊断结果，不能记作该旧 SHA 的正式 G6–G9 PASS。

确需复用仓库既有的外部测试资产时，保持生产 `tested_sha` 不变，在现有 `environment`、`commands` 和证据引用中记录测试资产的不可变提交/制品摘要、依赖/配置版本及实际运行组合；缺少可恢复身份则相关验证 BLOCKED。不增加平行测试框架或新报告 Schema。

G2 前窄范围 Brownfield 特征测试仍可按[特征测试](characterization-testing.md)建立诊断/基线：明确基线代码 SHA 与新增测试快照，不冒充包含该测试的已批准候选或正式验收。需要长期保留时，在获批 Task 内纳入预算、固定代码身份并完成上述审核/集成；不为提前取证伪造 Task 或批准。
