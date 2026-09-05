# `.devflow` 运行期证据

此目录是四个 DevFlow 角色共用的、位于仓库本地且便于人工阅读的证据工作区。它不是 CLI 状态数据库。各角色使用普通文件工具创建 Markdown/YAML，并使用目标仓库自身的构建、测试、CI 和 Git 命令。

## 单次变更目录布局

每个 Root Issue 创建一个不会冲突的目录：

```text
.devflow/changes/REQ-YYYYMMDD-NNN/
├── state.yaml
├── intake.md
├── repository-profile.md          # Brownfield
├── takeover-assessment.md         # Brownfield 续作
├── root-issue.md
├── decisions/
├── specs/
├── ui/
├── tasks/
├── tests/
├── reviews/
├── implementations/
├── defects/
└── reports/
```

只创建当前变更需要的目录。从 `.agents/skills/_devflow_shared/templates/` 复制源结构，替换占位内容，并保持 YAML 可解析。

## 所有权

- 只有规划者可以写入 `state.yaml`、需求接收记录、仓库画像/接管评估、Root Issue、Spec/UI、DAG/Task、变更预算、决策、归因和关闭记录。
- 审核者负责生成审核载荷。只读 Custom Agent 将完整 YAML 返回规划者，规划者可以原样落盘，但不得修改审核结论。
- 测试者负责测试计划/用例、测试自动化改动、报告和 Defect 证据；测试者不得修改生产代码。
- 实现者负责所分配的生产/测试变更，并为一个 Task 或实现类 Defect 创建新的实现报告；实现者不得合并。

规划类草稿在首次送审前可以原地完善。产物一旦送交独立审核、产生审核结论或被 `state.yaml` 引用，即视为已发布；无论结论是否通过，均不可覆盖。修正必须使用新的 ID/版本并填写 `supersedes`；规划者负责把状态指向最新有效证据。局部修订只创建实际变化的产物，未变化材料通过原路径、版本或 SHA 引用，不复制整套证据。

## 证据与门禁

审核和用户批准必须绑定产物版本。代码审核必须绑定精确的 `base_sha..head_sha`，测试报告绑定 `tested_sha`，发布/冒烟决策绑定精确的集成或目标 SHA。分支名会移动，因此不能作为证据。

FAST 工作可以使用紧凑产物，但仍须保持角色分离，并经过适用的 G0-G10 门禁。证据缺失或过期时应返回 `BLOCKED`，不得推定已批准。Brownfield 工作还要记录基线失败和变更预算；续作场景还须记录既有工作清单与差距分析。

首次正式 Spec 审核前，规划者应完成有界事实预检、交付规模判断，并把宿主仓库已有审批、CI 和发布证据映射到对应 Gate。语义和身份等价的证据直接引用，不维护第二份竞争事实。角色交接只发送当前动作和不可变身份，完整内容由接收角色从本目录读取，不依赖完整对话历史。

测试证据必须注明实际环境权威层级。合成/Mock、本地、集成、live 和生产结果只证明各自声明的范围；低权威结果不得冒充更高权威验收。可独立的环境缺失只阻塞对应验证切片，不可拆分的必需证据缺失则对应 Gate 保持未通过。

不得在此目录写入 secret、Token、敏感日志、生产数据或未脱敏个人信息。运行期产物是否提交由宿主仓库策略决定；DevFlow 本身不强制规定。
