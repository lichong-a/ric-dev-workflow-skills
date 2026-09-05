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
- 已发布的审核、测试报告和实现报告不可变；修正必须发布新版本并声明取代关系。

## 编辑与验证

- 保持共享契约和模板中的字段名及状态迁移不变。契约发生变化时，必须同步更新所有使用方。
- YAML 和 TOML 必须可解析，Skill 描述必须具备区分度，所有相对链接必须可解析。
- 四个 Skill 都必须通过随 Codex 提供的 Skill 验证器。
- 场景验证应检查可观察决策与产物，而不是比对精确措辞。
- 若缺少对应门禁要求的版本、Commit SHA、命令结果或其他证据，不得声称门禁已通过。
