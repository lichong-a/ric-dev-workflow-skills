# DevFlow Skills 验证报告

> 当前验证日期：2026-09-07（Asia/Shanghai）
> 对象：2.1 单层调度、Task 内分步实现、冻结 DAG 与审核适用性；本地未提交改动
> 结果：适用静态检查、独立决定情境和一次规则一致性审查通过；未测量研发耗时
> 源码基线：`main` / `a376bf1738fb774544152e9a58d3a65755d306b0`；开始时工作区干净

## 2026-09-07 范围与真实检查

本轮只改角色入口、直接相关契约/参考、Compact current 模板说明、场景评测和项目文档。没有新增 Schema 字段、状态、Verdict、Skill、脚本或工作流程序；原 v1 模板不变。未访问 WanGoPlatform，未修改项目/全局 Agent 配置，未提交或推送。

| 实际命令或检查 | 结果 |
|---|---|
| `python3 <CODEX_HOME>/skills/.system/skill-creator/scripts/quick_validate.py .agents/skills/<role>`（四角色分别执行） | 4/4 输出 `Skill is valid!` |
| `python3 /tmp/devflow-dag-eval-Y9nz69/validate.py` | YAML 12、项目 TOML 5、Markdown 69、frontmatter 7、相对链接 95；无解析失败/重复键/断链 |
| 同一临时验证程序的结构/兼容检查 | 恰好 4 个入口；shared 无入口；无 scripts/包清单；调用策略和 Sandbox 不变；原 v1 模板及 Compact 字段不变 |
| 同一程序只读核对 Reviewer 与全局链接 | 项目/全局 Reviewer 均为 high；5 个全局 Skill 链接仍解析到本源树 |
| `git diff --check`、`git diff --cached --name-only` | 无空白错误；暂存区为空 |
| `git diff --binary` 的 SHA-256（独立审查时） | `ac2f88aa72136cf81164a57cc79f5a76e6d5572bc9076692e51b6caa29c728f6`；绑定规则补丁，不包含随后追加的本节验证报告 |

`<CODEX_HOME>` 仅脱敏实际命令中的本机用户目录。一次性 validate.py 是临时静态验证工具，不属于交付包；没有通过它模拟或声称强制执行产品状态机。

## 2026-09-07 独立决定情境

使用 3 个独立且不继承完整历史的有界会话，各 1 次请求：Planner 判断 A–F、Implementer 判断 G/H、规则审查者只读检查源码 diff。没有复审重试或追加子任务。前两者获得原始合成情境和 Skill，不提供预期结论；它们仅输出下一步安排，不实施真实业务变更/委派/Gate 迁移。以下由实际输出逐项判定，不靠匹配固定措辞。

| 情境 / 覆盖 | 可观察决定 |
|---|---|
| A / WF-23 | CSV 导出涉及 6 个文件、多个步骤，Planner 安排 1 个完整 Task；校验/转义/测试/说明不独立建节点，不增加服务/UI |
| B / WF-25 | 原 3 个 Task 及依赖保持不变；8 文件工作留在 TASK-002；非 AC 审计看板延期；拼写修正不单独送审，保留原批准绑定 |
| C / WF-06、WF-25 | 同 Defect 三轮失败后停止盲目第四次补丁，有界归因；仍为原 Task/Defect，不重建图或清零计数 |
| D / WF-26 | 编译失败及宿主先集成提供方规则证明依赖缺失，只补 TASK-001 → TASK-002，局部 G2 复审；TASK-003 不变 |
| E / WF-26 | 权限/租户行为变化使受影响旧批准失效，暂停对应实现并重开 G2/G3/G4；是否改图仍取决于真实障碍，不自动重建 |
| F / WF-27 | 不适用的 Greenfield 额外基线专项、内部步骤、相同对象有效审核不再新增调用；新候选 SHA 仍必须 G5，不伪造 APPROVE |
| G / WF-24 | Implementer 选择原 Task 内实现、自测失败修正、检查点恢复和完整候选交接；不建议派生角色/子 Task 或逐步骤审核 |
| H / WF-24 输入边界 | 缺少完整 base SHA 和有效计划时 BLOCKED，交回原 Planner/调用者；不猜测，不新找 Planner 重建流程 |
| 一次独立源码审查 | 完整读取 20 个规则/文档文件的 diff 及直接相关调用方，未确认 P0/P1/P2 冲突或门禁绕过；不是业务 CODE_REVIEW |

WF-23–27 的决定级检查均满足预期，不能表述为五条完整研发流程已运行。特别是 G 的自测失败属于输入情境，未真正编写 CSV 代码、运行失败用例或验证运行时禁止递归；输出明确区分了安排和已执行事实。三个会话均只写指定临时结果文件。

## 2026-09-07 限制与清理

- 未运行真实业务实现/测试/发布、长期递归压力测试或耗时/Token 对照；不声称研发周期下降某个比例。
- 未重跑旧 v1 迁移、本地远端恢复、完整 G0–G10 或其他无关 WF 场景；下面的 2026-09-06 及更早结果只是历史证据。
- 未重新运行 Codex doctor；本轮配置未改，只做 TOML/角色策略解析与只读核对，不复用历史 doctor 结果声称本轮诊断通过。
- 临时输入、3 份角色/审查输出及静态验证程序仅放在 `/tmp/devflow-dag-eval-Y9nz69`，采集上述结果后按 5 个精确路径删除并移除空目录，不进入 Skill 包或业务目录。

---

# 2026-09-06 历史验证报告（以下不是本轮重跑结果）

> 当前验证日期：2026-09-06（Asia/Shanghai）
> 对象：Compact v2 收敛、Git 分类与无损迁移，本地未提交改动
> 结果：本轮适用静态检查、隔离迁移/恢复及有限独立角色评测通过
> 源码基线：`main`，`96632ae8acbae3bc93b2e825dd8660472f002ef2`；工作开始时干净。当前是 Git 仓库，不是 NO_GIT。

## 本轮范围与结果

只修改 Skill、共享契约/参考、Compact 模板、评测、AGENTS、README、设计和本报告。新建 v2 容器；原 v1 模板、报告字段、Gate、状态名、四角色调用策略、项目 .codex 配置和源码演示忽略规则保持原状。未读取或修改 WanGoPlatform；未操作真实 GitHub/业务远端，未调整用户全局 Agent 配置。

| 检查 | 实际结果 |
|---|---|
| Codex quick_validate.py | 4/4 Skill 输出 Skill is valid! |
| 可发现入口与纯指令结构 | 4 个 SKILL.md；共享目录无入口；无 scripts/CLI/包清单/生成状态机 |
| YAML / TOML | 12 个 YAML、5 个项目 TOML 可解析；另核对 7 个 Markdown frontmatter；YAML 重复键检查通过 |
| 本地 Markdown 链接 | 最终复核 69 个文档、88 个相对链接，0 断链 |
| v1 与配置兼容 | 原 templates 根目录与 .codex、.gitignore 相对 HEAD 零差异；Planner 可隐式，其余 explicit-only；Reviewer read-only，其余 workspace-write |
| Reviewer / 全局链接 | 项目与用户全局 reasoning effort 均 high；五个全局 Skill 链接解析到当前源树 |
| Git 差异自审 | git diff --check 通过，暂存区为空；新增文件仅为 2 个参考和 4 个 Compact 模板 |
| 固定文件 / Task 局部修订 | 3 个 Task，TASK-002 两次技术修订；其余两个正文/修订不变；4 个核心文件，无版本目录 |
| Git 分类 | 核心文件及必要失败附件不被忽略；.local 被忽略；已跟踪临时文件不会因新增 ignore 自动移除 |
| 无 Git / 禁止跟踪 | 正式文档快照在当前文件改成 r2 后仍逐字保持 r1；未把本地快照声称为 Git 共享或代码 SHA |
| 全源迁移 | 14/14 源文件进入基线和逐项索引，含已修改文件、隐藏未跟踪文件、未知字段/文件及冲突来源 |
| 内容保留 | 采用有效 Spec v2，而非已拒绝 v3；AC、测试 Oracle、权限、Task/依赖、预算相关约束、延期项保留；旧批准仍失效 |
| 原始角色载荷 | 4/4 历史报告逐字一致；独立 Tester/Reviewer 原始输出追加后逐字包含且各出现 1 次 |
| 失败与顺序 | 真实改变源文件被检测；注入候选部分写入失败，旧 state/全部源字节仍在；有序候选构建中 4 次核对旧入口，最后才切 state、精确移除 13 个旧副本 |
| 标签冲突 | 原 v1-baseline 标签不移动，采用 v1-baseline-2，索引记录实际标签 |
| 本地远端 | 原子推送分支和基线标签至隔离 bare remote；普通重新克隆恢复 14/14；浅克隆补取标签恢复 14/14 |
| 独立迁移门禁 | Tester 6 项文档保留性核验 PASS；Reviewer BASELINE_REVIEW APPROVE 精确迁移候选；两者均声明不授予产品门禁 |

固定布局 demo 的“四个”不计有实际用途的 attachments。迁移 fixture 从 14 个旧文件变成 4 核心文件 + 1 索引，旧原文通过基线保留；该数字不是任何真实业务目录的迁移结果。设计文档从 2601 行整合为 143 行，细则路由到共享契约，未用行数变化推断推理时间改善。

## 真实执行命令

一次性验证程序仅位于 `/tmp/devflow-v2-validation-5kH1go/validate.py`，通过 apply_patch 构造合成 fixture，使用系统 Git；不是包内产品脚本，交付时清理。以下时间为单次本地命令实测墙钟，不含独立 Agent 时间，不是 DevFlow 端到端性能基准。

提交版对命令中的本机用户目录做路径脱敏：`<CODEX_HOME>` 表示执行时的 Codex 用户目录，命令参数和结果未改变。

| 命令 | 结果 |
|---|---|
| `python3 <CODEX_HOME>/skills/.system/skill-creator/scripts/quick_validate.py .agents/skills/<role>`（四角色逐个） | 4/4 通过 |
| `python3 /tmp/devflow-v2-validation-5kH1go/validate.py static` | PASS；108 项结构/语法/链接/策略/链接位置断言，记录一次约 0.028s，补写报告后复核约 0.032s |
| `python3 /tmp/devflow-v2-validation-5kH1go/validate.py prepare` | PASS；14 源文件，4 原始载荷，隔离候选，约 0.279s |
| `python3 /tmp/devflow-v2-validation-5kH1go/validate.py compact` | PASS；14 项固定文件/修订/Git 分类/去重/快照断言，约 0.143s |
| `python3 /tmp/devflow-v2-validation-5kH1go/validate.py faults` | PASS；7 项部分写入、源变更、已跟踪忽略行为检查，约 0.104s |
| `python3 /tmp/devflow-v2-validation-5kH1go/validate.py reviewprep` | 建立实际独立 Reviewer 的精确 Git 文档对象，约 0.032s |
| `python3 /tmp/devflow-v2-validation-5kH1go/validate.py finish` | PASS；34 项审核绑定、原样转录、发布/克隆恢复检查，约 0.427s |
| `python3 /tmp/devflow-v2-validation-5kH1go/validate.py ordered` | PASS；22 项先目标内容、最后 state、精确旧路径删除及与已审候选树一致性检查，约 0.146s |
| `git push --atomic <本地 bare 路径> develop refs/tags/devflow-migration/REQ-DEMO/v1-baseline-2` | 成功；没有真实远端写入 |
| `git clone --depth 1 --no-tags file://<本地 bare 路径> <隔离目录>`，随后精确 fetch 基线标签 | 初始无标签；fetch 后逐项 git show 旧路径与源字节一致 |
| `git show <baseline>:<old-path>` + cmp / SHA256 | 主验证与独立 Reviewer 均核对了全部源字节；重新克隆和浅克隆也核对全部 14 项 |
| Python yaml/tomllib 与唯一键 Loader（命令内检查） | 语法、frontmatter、无重复 YAML 键检查通过 |
| `git diff --check`、`git diff --cached --stat`、`git status --short --branch` | 无空白错误；无暂存内容；本轮仍为本地修改 |

本轮验证工具调用曾有两次 JavaScript 字符串构造错误（反引号和模板插值），均在工具执行前失败，未改动 fixture 或源码；修正转录后执行成功，不计为产品故障。最初 prepare 中对权限类失败的“保持未操作”观察没有被当成自动防护通过证明；之后用实际部分写入/源变更测试以及独立 Planner 情境判断覆盖对应边界。

## 独立角色前向评测

共使用 4 个有界子 Agent、7 次角色/审查请求，默认不继承完整历史；其中一个审查者复用处理独立迁移对象。没有启用常驻任务、后台监控或全量业务执行。

| 对象 / 实际请求 | 可观察结果 |
|---|---|
| 当前 Skill 一致性首次审查 | 返回 CF-001 / P2：Planner 后续步骤的 v2 写入要求可能错误覆盖 v1 续作 |
| CF-001 同审查者增量复核 | 增加 Planner 主流程布局分支、AGENTS 与 orchestration 限定后，CF-001 resolved；只核对 Delta 和邻域 |
| Reviewer 首轮 SPEC_REVIEW | 同一轮返回 3 项真实 Finding：DAG 环、手改生成物违背宿主规则、必需测试交付路径未授权 |
| 同一 Reviewer 窄技术复审 | 只修改 Task-001/002 路径、生成策略、依赖；SPEC 行为与 Task-003 不变。REVIEW-SPEC-002 supersedes REVIEW-SPEC-001，3 项稳定 ID 均 resolved，APPROVE |
| Planner 恢复/收敛情境 | 暂停第三轮送审先归因；两次无新增事实后停止同类搜索；不研究无关分布式架构；已有完整 REVIEW-B 只补事件/索引，不重发审核；只读且有活跃写入的旧 Root 继续 v1，不迁移 |
| Tester 迁移核验 | 6 项文档保留检查通过，明确未执行产品用例/live/生产，不把继承 Defect 或过期批准清除 |
| Reviewer 迁移审核 | BASELINE_REVIEW APPROVE 候选 7f8145b62365c551058abd923eb9abd72579a879；全部 14 源、4 原载荷、未知内容、状态/引用与独立测试证据核验 |

Reviewer 首轮输出重复描述了部分已列 Finding，依据该实测现象补充“完整审核不要求重复维度矩阵/哈希清单”的输出规则；随后同会话复审保持逐 Finding 关闭证据而未重新生成逐维度矩阵。未声称所有报告都已达到某个字节/时间阈值。

## 隔离 Git 身份与证据边界

- 旧内容基线：`2aa9d026f92a44dfec4e0c0bdc8c0608a5000e9c`。
- 标签冲突后实际保留引用：`devflow-migration/REQ-DEMO/v1-baseline-2`；原标签仍指向 `072a772ab13dda6e50e7a2807f834ab1dce506d4`。
- 独立批准的迁移候选：`7f8145b62365c551058abd923eb9abd72579a879`。
- 追加原始独立证据后本地远端分支：`3e3654fe95bf1b98c0b0991b89567b5cb7e9ba0a`，不冒充前一候选的受测 SHA。
- 单独验证有序切换且 Root 树逐字等于已审候选的提交：`5b793f68112558e0dd5e3268b6c0db9cab1ca88b`。
- Reviewer 首审/复审文档：`b9e038933e3965fbeda1857eddb47291cd031a8a` → `6bf87a23dd6b4ea746d376e6dfda8ba25c89c09b`。

上述均为已清理的合成临时仓库身份，不属于本源码仓库或真实业务，不提供清理后的永久对象解析承诺。正式业务迁移的基线标签禁止删除/移动，此处临时 fixture 的销毁不作为真实迁移清理示例。

清理先执行 `gio trash -- /tmp/devflow-v2-validation-5kH1go`，因临时目录所在内部挂载不支持回收站而失败。随后在命令内验证精确临时路径、fixture 标记与无符号链接，再用 Python shutil.rmtree 清理，确认原路径不存在。仅移除约 2.1 MiB 的合成仓库、一次性检查程序与报告，不能从回收站恢复；未删除任何项目或业务数据。

## 配置诊断与未运行项

首次 `codex --strict-config doctor --summary --no-color --ascii`：配置 loaded，17 ok / 1 idle / 1 warn / 1 fail；fail 为 TERM=dumb 终端能力。使用仅本进程的 `TERM=xterm-256color` 再执行同一命令：18 ok / 1 idle / 1 warn / 0 fail，退出成功。warning 为历史 rollout/state DB、重复 thread inventory；unrestricted sandbox 是当前宿主提示，非本次修改角色配置。未升级 CLI，未更改系统或 Agent 策略。

本轮明确未运行：

- WanGoPlatform 文件迁移、真实业务代码测试、live/生产/凭据验证和真实 GitHub 推送：不在授权范围。
- WF-01–15 的完整业务交付重演及长时间无变化等待/真实挂起熔断：本轮保留既有规则、静态核对并做相关恢复情境评测，不冒充全部原流程重跑。
- LFS、子模块、外部 CI 附件的真实远端恢复：当前 fixture 全部为普通 blob/100644；契约明确遇到未保全指针必须阻塞，但未把这些未运行分支计为通过。
- 真实大型项目效率基准、Token 节省和端到端耗时对照：未测量，不作比例承诺。

WF-16–22 覆盖由静态、真实本地 Git 操作、人工构造故障与独立角色判断组合完成；这是纯指令系统，故障注入不是产品状态机测试，也不能证明任意模型每次都遵守指令。迁移前置冲突/授权/只读情境通过角色决定验证，文件恢复/顺序通过实际 Git 与文件操作验证，两者分开计证。

## Change Log

| 日期 | 作者 | 变更 | 证据 |
|---|---|---|---|
| 2026-09-07 | Codex | 单层调度与冻结 DAG 的静态、独立决定情境与规则审查；未做耗时基准 | 本文 2026-09-07 章节 |
| 2026-09-06 | Codex | 记录 Compact v2 本轮真实验证，保留历史结果并纠正当前仓库身份 | 上述命令、独立角色报告与隔离 Git 结果 |
| 2026-09-05 | 历史执行 | v1.1 兼容增强验证 | 以下历史记录 |

---

# 历史验证归档（以下不是本轮重跑结果）

以下原报告描述 2026-09-05 及 2026-09-04 的当时环境；其中 NO_GIT、数量与“本轮”均只指历史执行，不代表当前仓库状态。

# 2026-09-05 历史验证报告

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
