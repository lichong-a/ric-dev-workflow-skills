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
- 只有在全部依赖均为 `VERIFIED`、Task 的 base SHA 是最新已验证集成 SHA，且其写入区域只有一个所有者时，Task 才能进入 `READY`。
- `APPROVED` 要求当前 `base_sha..head_sha` 已通过代码审核。
- `VERIFIED` 要求合并后集成 SHA 已有集成测试报告。
- `READY_TO_MERGE` 要求同一集成 SHA 已通过完整验证和发布审核。
- `DONE` 要求存在目标分支冒烟证据，且没有未解决的阻塞发现、Defect 或过期证据。
- 行为性 Spec 变更会使 Root Issue 返回 `SPEC_REVIEW`，并按 `change-control.md` 使下游状态失效。
- 不得因为产物很小就跳过状态。FAST 交付可以压缩产物和合并审核轮次，但必须保留相同证据。

以上 Task 状态只用于正式交付节点，不适用于内部步骤、用例、检查点或 Reviewer 动作。G0–G4 的有效 Root 级证据由 Task 引用，不为每个 Task 建一条 Root 状态链。不适用或有效证据已覆盖的额外检查按 gate-policy 省略；这不授权把未完成的必需 Task 状态伪装成已通过。

若缺少必需版本、SHA、授权、环境或安全 worktree 边界，应使用 `BLOCKED`，并记录精确解阻条件。
