# DevFlow Skill 包规则

本仓库实现 `docs/DEVFLOW_SKILLS_DESIGN.md` 所描述的纯指令式 DevFlow 角色系统。

## 范围与结构

- `.agents/skills/` 下必须且只能保留四个可发现 Skill：`devflow-planner`、`devflow-reviewer`、`devflow-tester` 和 `devflow-implementer`。
- `_devflow_shared` 是资源库，绝不能包含 `SKILL.md`。
- 整个包必须保持为纯指令实现。不得增加工作流 CLI、编排脚本、自定义 Git 包装器、包清单或生成式状态机代码。
- 运行期证据应写入 `.devflow/changes/<REQ-ID>/`；不得把演示用变更记录提交到本 Skill 包。
- 各角色入口应保持聚焦；特定模式的流程、Schema 和检查清单应放入入口链接的参考文件。

## 不可妥协的角色边界

- 规划者负责需求接收、仓库接管、Spec、Task DAG、全局状态、协调、合并顺序和关闭；不得编写生产代码，也不得批准自己的工作。
- 审核者对被审核产物保持只读，只能给出绑定证据的 `APPROVE`、`REQUEST_CHANGES` 或 `BLOCKED` 结论。
- 测试者可以编辑测试和测试证据，但不得编辑生产代码或已批准 Spec。
- 实现者每次只处理一个已批准 Task 或实现类 Defect；不得修改需求、自审批准或合并。
- 已发布的审核、测试报告和实现报告按记录不可变；v2 汇聚账本时 Planner 仅原样追加，修正由原作者新记录声明取代关系。

## 编辑与验证

- 新 Root 默认 Compact 四文件容器 schema_version: 2；旧 v1 模板与读取能力保留，原报告载荷字段、Verdict、Gate、状态迁移不变。契约变化必须同步所有使用方。
- v2 current/test-plan 原地维护独立对象修订和底部 Change Log；正式历史用 Git 或持久快照保全；不生成 tasks-v* 或重复版本矩阵。未授权迁移的 v1 继续原版本路径与 transitions，不混用写入布局。
- 业务仓库默认跟踪核心四文件与必要脱敏附件，仅忽略 .local；本源码仓库继续忽略演示 changes，不得把该排除策略推广到业务仓库。
- 旧布局迁移须有单 Root 授权、全部源字节基线/持久标签、逐文件索引与独立审核，最后切换并仅移除明确已核验旧路径。只读、活跃写入或保全不完整时不切换；不操作 WanGoPlatform。
- 调查默认聚焦一轮加一轮缺口补查；两次无新增事实停止同类搜索，继续调查须关联当前 AC/失败/约束。
- 采用 Root Planner 单层调度；默认完整纵向 Task，内部实现/测试/文档步骤不派生子 Task 或角色链。DAG 首次 G2 通过后冻结，只有已证实无法在原 Task 内安全完成 AC 的结构性障碍才局部改图。
- 门禁按 Root/完整 Task/最终交付分层适用，不递归套在步骤上；不适用或有效证据已覆盖的额外审核直接省略，不能伪造 APPROVE 或跳过仍适用的强制门禁。
- YAML 和 TOML 必须可解析，Skill 描述必须具备区分度，所有相对链接必须可解析。
- 四个 Skill 都必须通过随 Codex 提供的 Skill 验证器。
- 场景验证应检查可观察决策与产物，而不是比对精确措辞。
- 若缺少对应门禁要求的版本、Commit SHA、命令结果或其他证据，不得声称门禁已通过。
