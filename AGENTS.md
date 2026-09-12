# DevFlow Skill 包规则

本仓库实现 `docs/DEVFLOW_SKILLS_DESIGN.md` 所描述的纯指令式 DevFlow 角色系统。

## 范围与结构

- 根 `skills/` 是唯一发布源，必须且只能保留五个实际 Skill 目录：`ric-devflow` 入口与 `ric-devflow-planner`、`ric-devflow-reviewer`、`ric-devflow-tester`、`ric-devflow-implementer` 四业务角色。入口不成为第五业务角色。
- 共享契约、参考与评测仅在入口 `references/` 保存一份，模板在 `assets/templates/`；不保留旧名别名、额外 Skill 或符号链接依赖。
- 整个包必须保持为纯指令实现。不得增加工作流 CLI、编排脚本、自定义 Git 包装器、包清单或生成式状态机代码。
- 运行期证据应写入 `.devflow/changes/<REQ-ID>/`；不得把演示用变更记录提交到本 Skill 包。
- 各角色入口应保持聚焦；特定模式的流程、Schema 和检查清单应放入入口链接的参考文件。
- 三平台四角色原生模板随入口 `assets/agents/` 分发；Codex ID 为 `ric_devflow_<role>`，Claude/ZCode 为 `ric-devflow-<role>`，不依赖源码外围平台目录。
- 准备按真实宿主、逻辑发现位置和已确认安装范围进行；只补同源固定 SHA 的缺件，保留定制、禁用及未知冲突。ZCode 项目安装仅原生角色定义有用户级目录例外，不能把文件存在当作加载或调用证据。

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
- G6 后 Task 保持 VERIFIED；目标 SHA 冒烟与关闭证据齐备后才 DONE。依赖解锁核对有效证据而非只比较状态名，旧状态矛盾用新事件纠正，不改写历史报告。
- Tester 的代码交付复用原 Implementation Report Schema 并如实署名，在原 Task 内串行交接、独立审核和集成；G5 后测试修正仍重新绑定 SHA，不伪装成实现类 Defect。
- Codex 保持 Root Planner 原调度、模型继承、推理档位与权限意图；五份 agents/openai.yaml 仅入口允许隐式调用。Claude/ZCode 四角色为平级子代理：唯一 Planner 决策和写状态，主会话只转发；四子代理均不派生代理。适配仅在已确认宿主/会话身份下生效。
- 范围内开发入口与已批准角色交接应主动实际派发，不等待用户提醒 subagent；不扩大触发范围、权限或嵌套层级，不把文案宣布当调用证据。
- 新平台 Reviewer 仅 Read/Grep/Glob，历史阅读缓存由 Planner 从精确 Git 对象提取并核验；正式来源不依赖 .local。其他角色的 Bash/编辑白名单不是文件路径沙箱，不得夸大隔离保证。
- 门禁按 Root/完整 Task/最终交付分层适用，不递归套在步骤上；不适用或有效证据已覆盖的额外审核直接省略，不能伪造 APPROVE 或跳过仍适用的强制门禁。
- YAML 和 TOML 必须可解析，Skill 描述必须具备区分度，所有相对链接必须可解析。
- 五个 Skill 都必须通过随 Codex 提供的 Skill 验证器；单角色安装的自包含守卫必须先于外部引用，必要共享闭包不能要求无关角色安装。
- 包级复现命令维护于 docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md；允许文档中的隔离检查片段，不将其提取为产品 CLI/常驻脚本。历史报告原文保留，本轮结果追加并绑定受检对象。
- 场景验证应检查可观察决策与产物，而不是比对精确措辞。
- 若缺少对应门禁要求的版本、Commit SHA、命令结果或其他证据，不得声称门禁已通过。
