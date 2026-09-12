---
schema_version: 2
root_issue_id: "REQ-YYYYMMDD-001"
revision: 1
author: ""
---

# 当前交付正文

<!-- 按进度填写；不适用章节注明原因，不创建其他版本目录。 -->

<a id="INTAKE"></a>
## INTAKE — 需求与完整范围

原始目标、非目标、约束/授权；完整能力清单、当前阶段、阶段依赖、延期项和最终完成条件。

<a id="BASELINE"></a>
## BASELINE — 仓库画像与接管

项目上下文、真实目标分支依据、受保护工作区、入口/必要契约、相近实现、工具/命令、已知失败证据、生成与写入边界；既有 Issue/Spec/CI/发布协议到 Gate 的等价映射及冲突规则。
Continuation 的相关单元分类与差距；当前可用环境及权限，缺口/停止条件。

<a id="SPEC"></a>
## SPEC — 行为契约

独立 revision；当前、期望、必须保持的行为。稳定 AC ID 的前提、动作、可观察结果与失败恢复；适用契约/数据/权限/兼容、UI、配置/迁移/回滚要求。区分草稿与已批准对象身份。

<a id="DECISIONS"></a>
## DECISIONS — 决策、预算与问题

稳定 Decision/Risk Acceptance ID、依据/备选/后果/作者/授权与有效期。允许/保护路径、依赖/框架/数据库/契约/生成策略、测试/文档在内的预算和余量；未解决 Finding/Defect 引用与延期理由。

<a id="TASKS"></a>
## TASKS — 当前阶段 DAG

<!-- 默认完整纵向 Task；步骤不另立节点。初始节点/边经 G2 通过后冻结，例外写已有 DECISIONS/Change Log。门禁适用性及复用依据放 notes，不另建审核矩阵。每个 Task 用稳定锚点与独立 revision，状态只在 state 中维护。 -->
<a id="TASK-001"></a>
### TASK-001 — 当前任务

- revision: 1
- root_issue_id: REQ-YYYYMMDD-001
- spec_version: 1
- acceptance_criteria: []
- depends_on: []
- base_policy: latest_verified_integration_sha
- allowed_paths: []
- protected_paths: []
- reference_implementations: []
- expected_deliverables: []
- verification_expectations: []
- risk: medium
- owner: ""
- notes: []

任务级预算/非范围、验证环境权威/权限/退出条件；必要 base_ref/base_sha 从 state 的就绪索引读取。内部实现清单可放 notes 或 .local，不分配子 Task ID、不逐步骤送审。未来阶段只保留能力与依赖，不展开同样详细的任务。

## Change Log

| revision | 时间（含时区） | 作者 | 类型 | 受影响 ID | 原因 | 证据/前一对象身份 |
|---|---|---|---|---|---|---|
