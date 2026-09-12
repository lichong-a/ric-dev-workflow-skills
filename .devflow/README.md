# 运行期证据目录

本 Skill **源码仓库**通过根 .gitignore 排除 `.devflow/changes/`，仅保留此说明；演示记录不要提交在此处。业务仓库不能照搬这条排除规则。

新 Root 默认按进度创建：
```text
changes/<REQ-ID>/
  state.yaml
  current.md
  test-plan.md
  evidence.md
  attachments/
  .local/
```

业务仓库默认跟踪四个核心文件和必要脱敏附件/迁移索引，仅忽略 `.devflow/changes/*/.local/`。正式失败、阻塞和未运行结果仍是证据，不能只有临时路径。ignore 不会自动取消已跟踪文件；提交和推送分别遵守授权。

Planner 维护 current/state，Tester 独立维护 test-plan，各角色报告由 Planner 原样串行追加 evidence。正文保持最新版和底部 Change Log，Task 独立修订；正式送审对象用 Git 完整 SHA/路径/ID 固定，报告记录不可覆盖。后续记账提交不是先前 tested_sha，不要求记录自身 Commit。无 Git 或不许跟踪时用持久不可变本地快照并明确仅本地可恢复。

旧 schema_version 1 继续按原布局读取。只有取得单 Root 迁移授权且全部原文已保全、索引完备、独立审核通过才切换；不自动清理。详见 [Compact 契约](../skills/ric-devflow/references/shared/compact-layout.md)、[迁移参考](../skills/ric-devflow/references/shared/legacy-migration.md)与 [Git 策略](../skills/ric-devflow/references/contracts/git-policy.md)。

G0–G10、独立审核/测试、用户批准、源/受测 SHA、工作区保护和环境权威要求不变；存储变小不代表验收范围缩小。
