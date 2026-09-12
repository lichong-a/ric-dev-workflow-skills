# 工作流状态

只有当前负责的规划者可以更新 `.devflow/changes/<REQ-ID>/state.yaml`。

## Root Issue 状态

主路径：

```text
DRAFT -> REPOSITORY_BASELINE -> SPEC_REVIEW -> USER_APPROVAL -> TEST_REVIEW
      -> READY -> IMPLEMENTING -> INTEGRATING -> FULL_VALIDATION
      -> RELEASE_REVIEW -> READY_TO_MERGE -> POST_MERGE_VERIFY -> DONE
```

`REPOSITORY_BASELINE` 对 Greenfield 是条件步骤，对 Brownfield 则是必需步骤。辅助状态为 `BLOCKED`、`CHANGE_REQUESTED`、`CANCELLED` 和 `ROLLED_BACK`。

## Task 状态

```text
PLANNED -> READY -> IN_PROGRESS -> CODE_REVIEW -> APPROVED -> MERGED
        -> INTEGRATION_TEST -> VERIFIED -> DONE
```

辅助状态为 `BLOCKED`、`REWORK`、`CANCELLED` 和 `SUPERSEDED`。

## 状态迁移不变量

- 每次迁移都要记录操作者、带时区时间戳、前一状态、后一状态、原因及证据 ID/路径。v2 先追加完整事件到 evidence，再更新 state 当前索引；v1 继续使用原 transitions 字段。v2 不复制 Task 正文、Review 全文或完整迁移历史。
- `READY` 要求 Spec 已批准、当前行为版本已获用户批准，并且测试计划已批准。
- 只有在全部依赖满足下述“有效依赖”、Task 的 base SHA 是最新已验证集成 SHA，且其写入区域只有一个所有者时，Task 才能进入 `READY`。
- `APPROVED` 要求当前 `base_sha..head_sha` 已通过代码审核。
- `VERIFIED` 要求合并后集成 SHA 已有覆盖该 Task 的有效增量测试 `PASS`；集成完成后保持此状态以解锁依赖，不立即转为 `DONE`。
- `READY_TO_MERGE` 要求同一集成 SHA 已通过完整验证和发布审核。
- `DONE` 对 Task 与 Root 均要求当前交付的目标分支 SHA 已有有效冒烟 `PASS`，且没有未解决的阻塞发现、Defect 或过期证据；由 Planner 在关闭时将对应 Task、随后 Root 置为 `DONE`。
- 行为性 Spec 变更会使 Root Issue 返回 `SPEC_REVIEW`，并按 `change-control.md` 使下游状态失效。
- 不得因为产物很小就跳过状态。FAST 交付可以压缩产物和合并审核轮次，但必须保留相同证据。

以上 Task 状态只用于正式交付节点，不适用于内部步骤、用例、检查点或 Reviewer 动作。G0–G4 的有效 Root 级证据由 Task 引用，不为每个 Task 建一条 Root 状态链。不适用或有效证据已覆盖的额外检查按 gate-policy 省略；这不授权把未完成的必需 Task 状态伪装成已通过。

## 有效依赖与恢复

依赖 Task 处于 `VERIFIED`，或具有完整关闭证据的 `DONE`，且对应 Spec、代码审核、增量测试及集成血缘仍有效时，才可解锁后续 Task。`DONE` 还须核对原交付目标 SHA 的冒烟与关闭证据；状态名不能代替证据，新 Task 仍从当前最新已验证集成 SHA 开始。

恢复发现旧状态与证据矛盾时，先记录缺口并停止依赖该无效结论的派发。Planner 依据仍有效证据追加纠正事件：例如 G6 有效但尚无目标冒烟的旧 `DONE` 纠正为 `VERIFIED`；需修正代码则进入 `REWORK`，关键证据无法建立则 `BLOCKED`。保留全部历史报告，不批量改写历史，也不触发布局迁移。

G5 后发生生产或测试代码变化时，受影响 Task 经 `REWORK -> CODE_REVIEW` 重新绑定候选并完成集成验证；期间不能继续作为有效 `VERIFIED`/`DONE` 解锁依赖。已启动的下游按实际影响暂停并补证，无关 Task 不重置。详见[测试代码交付](../shared/test-code-delivery.md)。

若缺少必需版本、SHA、授权、环境或安全 worktree 边界，应使用 `BLOCKED`，并记录精确解阻条件。
