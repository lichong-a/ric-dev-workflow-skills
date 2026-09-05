# DevFlow Skills 验证报告

> 日期：2026-09-05
> 验证对象：`docs/DEVFLOW_SKILLS_DESIGN.md` v1.1 的兼容增强实现
> 结果：通过（PASS）

## 范围

验证覆盖包结构、Skill 元数据、UI 调用策略、Custom Agent 配置、YAML/TOML 语法、参考路由、角色边界、证据/版本/SHA 契约、效率与审核收敛不变量，以及一个新的隔离兼容增强演示。2026-09-04 的 Brownfield 续作演示作为历史回归证据单列保留。

未创建 DevFlow CLI、工作流脚本、自定义状态机程序、包清单或生产部署。演示只使用临时本地 Git 仓库，并在采集证据后删除。

## 静态验证

| 检查项 | 结果 |
|---|---|
| 可发现的 `SKILL.md` 数量 | 通过：恰好四个 |
| `_devflow_shared` 不是 Skill | 通过 |
| Skill 包内禁止的 `scripts/` | 通过：不存在 |
| 随附的 `quick_validate.py` | 通过：4/4 Skill |
| YAML 解析 | 通过：11/11 个 YAML 文件 |
| TOML 解析 | 通过：项目配置及 4/4 Custom Agent，共 5/5 个文件 |
| Markdown 相对链接 | 通过：62 个 Markdown 文件中的 44 个本地链接 |
| 中文叙述覆盖 | 通过：说明以中文为主；Skill、Root Issue、Reviewer、Gate 等协议标识保留英文 |
| 调用策略 | 通过：规划者可隐式调用；其他三个只能显式调用 |
| Custom Agent Sandbox 策略 | 通过：审核者只读；其他角色 workspace-write |
| 模型可移植性 | 通过：没有硬编码模型名称 |
| Reviewer 推理强度 | 通过：项目级和用户全局均为 `high` |
| 模板与 Schema 兼容 | 通过：模板目录零变更；状态、Gate、顶层字段和 `schema_version: 1` 不变 |
| Agent TOML 兼容 | 通过：项目 `.codex/` 相对变更前基线零差异 |
| Codex 项目配置加载 | 通过：Doctor 报告 18 项正常、1 项空闲、1 个历史 thread warning、0 失败 |

当前 Codex CLI 版本为 `0.151.0`。`codex --strict-config doctor --summary --no-color --ascii` 显示项目配置已加载、状态数据库健康且 `0 fail`。唯一 warning 是历史 rollout 文件未进入 state DB 及重复 thread inventory；它不属于 DevFlow Skill 或当前 Agent TOML 配置错误。Doctor 同时提示有 `0.153.4` 可用，本次未升级软件或改动系统配置。

当前目录不是 Git 仓库，因此本次包变更记录为 `NO_GIT`；没有虚构当前目录的 branch、HEAD 或 Commit SHA。验证期间把变更前内容保存在一次性临时基线中，通过目录差异完成最终自审后删除。

## 本轮执行命令与结果

| 命令/检查 | 真实结果 |
|---|---|
| `quick_validate.py <四个 Skill 路径>` | 4/4 输出 `Skill is valid!` |
| Python `yaml.safe_load` / `tomllib.load` | YAML 11/11、TOML 5/5 可解析 |
| Markdown 本地链接解析 | 62 个文档、44 个本地链接，0 断链 |
| 结构与兼容检查 | 4 个 Skill、0 个禁用脚本；模板和 `.codex/` 相对基线零差异 |
| Reviewer 配置核对 | 项目 `.codex/agents/devflow-reviewer.toml` 与用户全局对应文件均为 `high` |
| `codex --strict-config doctor --summary --no-color --ascii` | 18 ok、1 idle、1 warn、0 fail；warning 属于历史 thread inventory |
| `python3 -m unittest discover -s tests -v`（隔离 demo） | 1/1 通过 |
| 兼容增强场景断言（隔离 demo） | 全部通过，`DEMO_OK=1` |
| 非等价仓库证据映射断言（聚焦 demo） | `MAPPING_DEMO_OK=1`；未绑定版本的批准未复用为 G3 |

## 兼容增强隔离演示（2026-09-05）

本轮使用两个一次性本地 Git 仓库：主 demo 的基线分支为 `develop`，基线 SHA 为 `30717b88bad43390d1092bc85be670b4d9400f62`；另一个聚焦 demo 专门验证仓库既有证据的等价性判断。Fixture 包含既有产品范围、绑定 SHA 的 CI 要求，以及必须另行授权的 Vendor Sync live 门禁。

| 不变量 | 观察结果 |
|---|---|
| G2 前规模判断 | 完整目标保留为 3 个阶段；本地导出、live 推送和 UI 审计分别有独立 Root Issue |
| 局部阻塞 | 本地阶段 `READY`；live 阶段因未授权凭据为 `BLOCKED`；延期 UI 阶段保持 `DRAFT` |
| 仓库协议映射 | Repository Profile 显式覆盖 G0–G10；未绑定 Spec 版本的既有范围批准只作为输入，不能满足 G3；另行取得绑定 Spec v2 与内容哈希的用户批准；CI 文档只复用为命令策略，运行报告仍须绑定 tested SHA |
| Reviewer 完整披露 | 首次 `SPEC_REVIEW` 在同一报告返回 3 个不同章节的 P1/P2 Finding |
| 增量复审 | 同一 Reviewer 复核精确 Delta 和稳定 Finding ID，新 Review `supersedes: REVIEW-001` 并批准 |
| 送审冻结 | 被拒绝的 `spec-v1.md` SHA-256 始终为 `57d907a8b12eedb45148f8cb20eded7564534d757cc18f42b141a376d69309c4` |
| 低扇出 | 只有 2 个 Spec 版本；未变化的仓库画像、Root Issue 和测试计划只引用，不复制 |
| 紧凑交接 | Reviewer 增量复审交接为 10 行、571 字节，不包含 Spec 正文或完整对话 |
| Schema 兼容 | Task、Review、State 实例与原模板顶层键一致，均保持 `schema_version: 1` |
| 环境权威 | 测试计划明确区分本地与 live；没有读取凭据，也没有把 Mock/本地结果写成 live 通过 |

主 demo 的初始 Fixture 曾把未绑定 Spec 版本的既有范围批准直接列为 G3 复用。最终自审判定这不满足版本绑定要求，因此该项未计为通过；聚焦 demo 随后验证了正确规则：旧批准只作为需求输入，G3 必须另有绑定 Root Issue、Spec v2 和内容哈希的用户批准。上表及最终结论采用修正后的结果。

演示产物在采集上述脱敏结果后删除，不进入本 Skill 包，也未访问 WanGoPlatform。

## 历史 Brownfield 隔离演示（2026-09-04）

演示仓库用一个小型 Python 功能刻意组合了设计中的六类 Brownfield 风险：

- 当前分支 `feature/order-refund` 包含一段可工作的计算逻辑和两个 Stub；
- 仓库贡献指南和 CI 证据指向 `develop`，而非 `main`；
- 存在用户拥有且未跟踪的 `web/refund.css`，并将其设为受保护；
- 完整基线存在一个无关历史测试失败，而退款测试通过；
- 邻近遗留代码使用不安全 SQL 字符串插值；
- 一个假想的大范围错误/目录重构超出局部预算。

模拟的 DevFlow 产物明确标记为 `evaluation_fixture`；它们没有冒充真实用户或生产批准。Task 分支从完整 SHA `3648d00c29af7e32e1de7eec9b49ca91730b6c55` 开始，并产生 head SHA `5cc46cfa512af84bd9c98351e3e89a30a7885b2c`。

## 历史演示结果

| 不变量 | 观察结果 |
|---|---|
| 上下文分类 | `BROWNFIELD_CONTINUATION` |
| 目标分支 | `develop`，由两项仓库事实推导 |
| 续作行为 | 保留已测试计算逻辑；只补全两个 Stub |
| 变更半径 | Commit 只改动 `src/refund.py` 和 `tests/test_refund.py` |
| 脏 worktree 保护 | CSS SHA-256 保持不变；从未 staged 或 committed |
| 历史失败处理 | 变更前：1；变更后：仍为 1；新增失败：0 |
| 目标验证 | 退款测试从 2/2 增加到 4/4 通过 |
| 不安全遗留风格 | 新 SQL 使用两个占位符和独立参数 |
| 重构防火墙 | 模拟审核返回 `REQUEST_CHANGES`，并给出 P2 范围发现 |
| 证据身份 | 实现、审核、集成、发布和冒烟均匹配精确 SHA |
| 状态模型 | 从 `DRAFT` 到 `DONE` 的全部 12 次必需迁移连续 |
| 合并后冒烟 | `develop` 在发布 SHA 上的 4/4 退款测试通过 |

完整测试命令正确保留了非零退出码，因为有意设置的遗留 Fixture 仍然失败。集成报告没有把该命令声明为成功；它记录了已知失败，显示新增失败差异为空，并将 `PASS` 限定在已批准验收范围内。

## 场景与越权审核

`.agents/skills/_devflow_shared/evals/` 下的永久评测目录覆盖设计中的全部触发与 Brownfield 场景，以及过期证据、行为变更、三轮 Defect、角色越权和 `WF-08` 至 `WF-15` 的效率/收敛场景。对已实现 Skill 入口、共享契约和隔离演示的检查确认：

- 只有开发工作可隐式路由到规划者；
- 显式调用审核者但缺少精确模式/版本/SHA 时返回 `BLOCKED`；
- 实现者缺少当前 Spec、测试计划或 base SHA 时，在修改生产代码前返回 `BLOCKED`；
- 规划者把“直接修复”请求路由成 Task/实现者交接，而不是亲自编码；
- 审核者返回发现，而不是“边审边改”；
- 测试者报告 Defect，而不是修改生产代码；
- 实现者将合并所有权交还规划者；
- Spec 过期或 head/tested SHA 变化会使先前证据失效；
- Program/Epic 在 G2 前按独立价值和边界分期，同时保留完整目标；
- 交接只传递精确身份与增量，支持时默认不继承完整历史；
- Reviewer 一轮披露阻断 Finding，同 Reviewer 增量复审，两轮不收敛时先根因整理；
- 送审前草稿可原地完善，送审版本冻结，未变化产物只引用；
- 仓库已有证据按语义映射，环境缺失局部阻塞，低权威结果不冒充 live 验收。

## 剩余限制

这是纯指令系统，因此静态验证证明的是可发现性、语法、路由和契约一致性；隔离 demo 证明一条具有代表性的决策与产物路径，但不等同于长期真实项目中的延迟基准。本轮未启动额外独立多 Agent 前向评测，以遵守当前任务的代理授权边界。后续应在下一次真实 DevFlow 使用中观察审核轮次、版本数量、交接大小、无变化轮询和总墙上时间。

后续 Codex 版本可能调整配置字段；本包刻意不硬编码模型名称，`fork_turns: "none"` 仅作为平台支持上下文继承控制时的当前示例。升级 Codex 时应重新对照可用工具 Schema 和官方文档验证。
