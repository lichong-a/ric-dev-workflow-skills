# DevFlow Skills 验证报告

> 当前验证日期：2026-09-07（Asia/Shanghai）
> 对象：2.2 三平台原生配置适配；本地未提交改动
> 源码基线：main / e6f559b70c0917d0286a3acf0128c5b87f177567；开始时工作区干净
> 结果：适用静态检查、隔离安装/Git demo、有限角色情境与独立规则自审通过；Claude/ZCode 原生运行未验证

## 2.2 本轮改动与边界

新增 .claude/agents 和 .zcode/agents 各四份实际 Markdown 定义，.claude/skills 为 ../.agents/skills 相对链接。共享入口与现有编排参考补充宿主/主子身份、平级转发和精确历史阅读缓存；同步角色边界、项目规则、README、设计及 WF-28–33 / TRIGGER-11–14。

没有新增 Skill 正文副本、adapters、调度 Schema、工作流脚本或第五角色。Codex 配置、openai.yaml、原模板字段/状态/报告载荷未改；未操作 WanGoPlatform、未修改用户全局配置、未提交或推送。实现采用 skill-creator 的按需路由方式，平台差异留在薄配置和已有参考，不复制完整流程。

## 2.2 实际命令与静态结果

一次性测试工具位于 /tmp/devflow-native-validation-OQem8i/validate.py，属于本轮隔离验证辅助程序，不属于交付包。下表不是客户端实际启动记录。

| 实际命令或检查 | 结果 |
|---|---|
| python3 <CODEX_HOME>/skills/.system/skill-creator/scripts/quick_validate.py .agents/skills/devflow-<role>，四角色逐一执行 | 4/4 Skill is valid! |
| python3 /tmp/devflow-native-validation-OQem8i/validate.py static | 238 项检查；12 YAML、5 TOML、77 Markdown、15 frontmatter、120 相对链接；无重复 YAML 键或断链/失效锚点 |
| 同一 static 的原生配置与结构检查 | 8 份实际定义、正确工具白名单；ZCode 独有 injectAgentsMd=true；四份真实 Skill，Claude 链接不制造副本；无 scripts/adapters/包清单 |
| 同一 static 与基线逐字比较 | .codex、openai.yaml、全部既有模板共 29 个文件不变；Codex 调用策略与 Reviewer read-only 保持 |
| python3 /tmp/devflow-native-validation-OQem8i/validate.py install | 31 项检查；实际运行 README 新平台 Bash 片段主体，Claude 9 个链接、ZCode 4 个链接；同源重复安装可复用；已有文件/失效链接阻止安装且不覆盖、不部分创建。另验证隔离 Codex 五目录链接、四份原 TOML 复制与相对引用 |
| git diff --check | 通过 |
| command -v claude | PATH 中未发现 Claude Code；未安装客户端 |
| dpkg-query -W -f='${Package} ${Version}\n' zcode | zcode 3.11.2-6792；这是包版本，不是原生运行结果 |

<CODEX_HOME> 仅代替实际命令中的本机用户目录。测试程序读取受检文档并生成隔离缓存/链接，不执行产品状态机；静态断言不能证明模型必定遵从指令。

额外只读检查：使用 Node fs 读取本机 /opt/ZCode/resources/app.asar 中 out/host/index.js 的解析函数片段，确认原生 tools 支持逗号分隔字符串及列表，inherit 归一为继承，injectAgentsMd 被解析；未加载/执行整个应用模块。读取随安装包附带的 zcode-configuration-guide，确认 .agents/skills 发现路径。以上是安装包源码/文档静态证据，不是客户端会话验证。

## 2.2 隔离 Git demo

临时仓库只含合成 normalization 行为、测试和四核心示例文件；示例明确没有产品批准，不宣称完整 G0–G10 运行。

- python3 /tmp/devflow-native-validation-OQem8i/validate.py seed：创建隔离 Git 基线 f3246e92382577e091c40f03696bc80601c1425c。
- 通过 apply_patch 改候选、重命名及删除指定 fixture 文件，再运行 validate.py candidate：候选 4c94bd08f4953e010847f4ecef16d0ce19db8dfe。以上两个 SHA 只属于临时 demo，不是本源码仓库提交。
- 候选固定后，通过 apply_patch 在工作树加入 WORKTREE ONLY 内容，保持它不属于候选。
- validate.py demo：24 项检查通过。实际 git cat-file / merge-base / diff --no-ext-diff --no-textconv / show / ls-tree 验证对象、祖先、R100 重命名、删除及 120000 链接模式；6 份读取缓存与对应 Git 原文逐字相等，未混入脏工作树或跟随链接读取仓库外文件。
- 缓存中的精确候选实际执行 python3 -B -m unittest -v：2/2 通过。只测试合成 normalization 行为，不是三平台运行测试。
- git check-ignore 与 git ls-files：四核心文件已跟踪、不被忽略；.local 缓存被忽略；缺少 Git 路径返回失败，可与空内容区分。
- validate.py recover：清除明确的生成缓存后，git show 仍逐字恢复候选源码，恢复检查通过。

首次 demo 因一次性测试程序把 .local 目录计入“核心文件”而断言失败；修正为只计直接子文件并重新运行，24 项通过。失败不属于 Skill 运行问题，首次失败没有计为通过。

## 2.2 有界独立评测

共 3 个独立、不继承完整历史的评测会话，每个一次请求，没有追加评测轮次、实现子任务或真实平台调用。两个角色情境会话读取当前规则后给决定；另一个只读审阅全部规则/配置差异。未向评测者提供预期答案。

| 实际观察 | 覆盖与结果 |
|---|---|
| 路由 A–D | Codex 不因 .claude/.zcode 存在改路径；Claude 主会话转交 Planner；ZCode 独立 Reviewer 直接返回调用者；已启动 Planner 返回 SPEC_REVIEW 交接而不再次派发。WF-28 / TRIGGER-11–13 决定符合预期 |
| 路由 E–G | 身份不明、同名异版/缺 shared、原 Planner 生命周期未知时均停止相应派发/写入，不猜测、不混包、不启动第二个 Planner。WF-29/30 / TRIGGER-14 决定符合预期 |
| 路由 H 原始载荷 | 主会话只按原调用关系转发；实际用 apply_patch 输出唯一获准的临时 relay-return.yaml。cmp 与原载荷一致，370 字节；两个 Finding、REQUEST_CHANGES、中文/Ω/引号/缩进均保留 |
| 角色 1 | 完整候选继续在原 Task/DAG 内处理，不因内部自测修正派生测试/文档子 Task；补齐必要义务后才交完整候选审核 |
| 角色 2–4 | Reviewer 缺提取/核验证据及实现报告则 BLOCKED；Tester 缺必需 live/MCP 不把本地通过写完整 PASS；Implementer 缺 base SHA/批准不写代码，也不借 Bash 派生其他客户端。WF-31/32 决定符合预期 |
| 角色 5–6 | 可恢复原 Reviewer 做窄复审；不可恢复且已结束时新 Reviewer 按原谱系完整复审该对象；两个可观察阻断项同轮披露，偏好不进入返修链。WF-33 决定符合预期 |
| 一次独立规则自审 | 阅读 11 个受跟踪文件 diff、8 份新增定义、链接及相关完整规则；核对 Codex 基线零差异，未确认 P0/P1/P2 问题。验证报告未纳入该次审核，后续由主代理自审 |

原始转发载荷的 cmp 与 sha256sum 实际执行；两份 SHA-256 均为 af3c0af6c0dfbb009dfe2d184cba8c07f1638c16220df960ad5465d85b9305f0。它是合成转发样本，不是本项目的真实 Review 或 Gate 结果。

WF-28–33 已按上表完成适用静态、fixture 或决定级覆盖，不代表六条完整研发流程实际运行。截断重传、状态写入中断恢复及宿主无进展等待仅经过规则自审，未注入真实客户端故障；不得把这些子情境算为实机通过。

## 2.2 未运行项与清理

- 未启动 Claude/ZCode 原生代理、权限拦截、会话恢复或端到端开发。Claude 未发现；ZCode 本轮仅只读检查安装包，不修改用户配置、不读取登录凭据。包配置/行为评测通过不等于宿主实机通过。
- 未运行 Codex doctor、真实业务 CI/发布、性能/Token/研发耗时对照、完整 G0–G10 或旧 v1 全源迁移。配置未改和历史结果不能代替本轮实际运行。
- 原规则继续保留 v1、过期证据、门禁及三轮 Defect 约束；本轮只重测与宿主适配有关的范围。
- 临时安装链接、fixture Git 仓库、缓存、转发样本与测试程序仅在上述精确临时目录，完成脱敏采集后清理；不提交到本包或任何业务仓库。下面为历史验证，不计作本轮重跑。

---

# 2.1 历史验证报告（以下不是本轮重跑结果）

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


## 状态、测试交付与主动派发验证（2026-09-08）

本节是本轮新增记录，以上历史报告保持原始字节；历史统计和未运行项不作为本轮结果。复跑方法集中于[验证指南](DEVFLOW_SKILLS_VALIDATION_GUIDE.md)，行为原始输入与独立判定分别保存在[工作流场景](../.agents/skills/_devflow_shared/evals/workflow-cases.md)的 WF-34–39 和[触发场景](../.agents/skills/_devflow_shared/evals/trigger-cases.md)的 TRIGGER-15–18。

### 受检身份与执行环境

- 源码基线：`main`，`84954fbda3c1d8c47ef2a5ee9fb43e18ab4a3c4a`。本轮交付尚未提交，不能用这个 HEAD 代表修改后的文件。
- 受检候选：下方 97 个文件及符号链接的路径、模式和内容 SHA256；集合 SHA256 为 `622edd0004a90870640626018c355691d30b538079b1d48822a07dc4ed8ce4f8`。摘要算法采用指南第一段输出；报告自身参与链接检查，但排除摘要，避免自引用。
- 环境：Python `3.14.4`、PyYAML `6.0.3`、Git `2.53.0`、Bash。四个 Skill 使用宿主随附 `quick_validate.py` 验证器。
- 在源工作区直接执行指南片段，并在仅含受检源码、无用户计划文件的干净隔离 Git 副本复跑两段完整 Bash 块。副本中的提交仅用于隔离检查，不是本源码仓库提交，也不是角色门禁产物。

### 静态与隔离操作结果

| 实际执行 | 本轮结果与证据边界 |
|---|---|
| 验证指南第一段完整 Bash 块 | 退出 0；四 Skill 验证器 4/4 通过 |
| YAML/TOML/frontmatter 解析与重复键检查 | YAML 12、TOML 5、frontmatter 15，全部通过 |
| Markdown 相对链接与锚点 | 报告追加前的干净副本：79 文档、148 本地链接、23 锚点引用，0 失败；报告追加后的复检见本节末尾 |
| 四角色、共享目录、符号链接及平台工具配置 | 通过；四个实际 Skill，无第五入口；Claude 相对链接与各平台定义符合约束 |
| 验证指南第二段完整 Bash 块 | 退出 0；42 项隔离检查通过，临时 fixture 清理另行确认通过 |
| Claude/ZCode 隔离安装 | 两平台首次安装、同源重跑、普通文件冲突、失效链接冲突均符合预期；冲突前预检，无部分安装；仅使用临时目标 |
| 合成 Git 与测试代码交付 | 生产实现候选运行 1 条单元测试通过；新增辅助函数在实际候选上产生预期失败；从该失败 SHA 修正测试后，精确候选运行 2 条用例通过 |
| Git 对象恢复与失败语义 | 测试修改限于测试路径；祖先关系、精确对象字节/模式、脏工作区排除、删除/重命名/符号链接、缺失对象失败及退出码 7 保留均通过 |
| 兼容性逐字检查 | `.codex`、四份 `openai.yaml` 和原模板共 29 个文件内容保持不变；用户原有 `.zcode/plans/` 文件内容保持不变 |
| `git diff --check` 与暂存区检查 | 通过；本轮未暂存、提交或推送源码 |

本次隔离 Git fixture 的合成身份如下，便于区分执行时使用的组合；fixture 已按指南清理，不承诺这些演示对象今后仍可解析，可由完整片段重新创建。

| 对象 | 执行时的合成 SHA |
|---|---|
| 基础输入 | `e8a12d666b0c17609dd8d162a946a82bca58c540` |
| 生产实现及一条已通过测试 | `9da19fcfd8f769b58e299a7a58c153092a1ac7c4` |
| 含失败测试辅助函数的候选 | `f2defa35149d6e5bb284e1c55c17efed19524342` |
| 修正后实际运行两条测试的候选 | `644ee7dfcbec05f60f74593b61776aeec6d75c32` |
| 仅用于删除、重命名和符号链接演示的后续提交 | `e7f4e3ada075bd578f392a32d383370781fe8d17` |

`transport_sha` 没有被声称为测试通过的 SHA；以上 Git/测试事实也没有冒充 G5、G6、G9 或真实项目验收。

### 失败注入与修正

以下注入均在可丢弃副本或隔离执行环境中完成；不是对真实源树或用户配置的破坏性测试。正常副本在修正后重新执行两段完整指南，均退出 0。

| 输入或环境变体 | 观察结果 |
|---|---|
| 重复 YAML 键 | 第一段退出 1 |
| 不存在的相对链接 | 第一段退出 1 |
| 不存在的标题锚点 | 第一段退出 1 |
| 第一个 Skill 的名称无效 | 第一段退出 1，后续 Skill 验证未继续，不会被后续成功掩盖 |
| `PYTHONOPTIMIZE=1` 分别执行两段 | 两段均在执行检查/安装前退出 1，明确要求禁用优化模式 |
| README 安装目录赋值改写 | 第二段退出 1，未创建意外目标 |
| README 安装目录赋值重复 | 第二段退出 1，未创建意外目标 |

独立只读审查指出并复核关闭了以下实际问题：失败集成上的测试修正不能错误回到不含该测试的旧 PASS 基线；早期基线和 Root 测试计划不能被强制要求尚不存在的 Task；Python 优化模式不能移除检查断言；安装目录参数必须在替换前确认唯一且精确匹配。最终规则和失败注入覆盖这些修正。审查者执行了只读文本检查和有限最小复现，没有将主代理的指南执行结果声明为独立重跑。

### 独立子代理前向评测

实际启动两个独立 Codex 子代理会话，分别评估状态/测试交付和入口/连续派发；另有一个只读审查子代理。两组评测输入只含原始请求、必要事实与允许读取的规则路径，未提前提供预期答案，也未让评测者读取用例判定、README 或本报告。评测完成后由主代理按场景判定。这是规则输入下的决定级评测；实际工具操作限于读取和独立评测会话调用，没有启动 ZCode/Claude 原生会话或伪造业务门禁记录。

| 原始场景组 | 实际观察到的决定 |
|---|---|
| 两个有依赖的 Task | 前项 G6 通过后保持 `VERIFIED`；证据有效时解锁后项，待 G9 及关闭条件满足再关闭 Task 与 Root |
| 有效与失效的 `DONE` | 完整有效证据可以复用；只有 G6 的旧 `DONE` 通过新事件纠正为 `VERIFIED`；失效审核证据不能凭状态放行 |
| Tester 在原 Task 添加测试 | 保留 Implementer 的 A..B 报告与 Tester 的 B..C 报告；独立审核完整 A..C，正式测试绑定含测试代码的集成 SHA |
| 首次 G6 失败与 G5 后测试修正 | 从含失败测试的当前 M 修正；记录 TEST 问题、重新绑定审核与验证，不新增测试子 Task，不借用旧 PASS |
| 未提交测试、测试预期、外部资产 | 未提交测试仅属诊断；普通实现修正不重开 G4；预期变化复核受影响批准；外部资产需要不可变版本及组合 |
| ZCode 普通 CSV 开发请求，未提角色名 | 选择实际调用同名 Planner；不要求用户补充 subagent 提醒 |
| Planner 完整交接与角色回传 | 主会话继续调用指定 Reviewer；完整结果原样回传已有 Planner，不等待新用户消息 |
| G3 尚缺用户产品批准 | 保留绑定 Spec 的必要用户决定，未推断已批准 |
| 缺失角色、工具、来源或旧 Planner 身份 | 报告具体缺口；不回退主会话实现，不绕行客户端，不创建第二个状态写入者 |
| Planner 子代理身份与非开发输入 | 子代理返回精确交接，不嵌套派发；HTTP 304 解释和文案润色不触发 DevFlow |
| Codex 宿主 Planner | 按既有 underscore 角色映射直接委派，不套用 ZCode 平级转发模式 |

两组观察结果均符合独立判定标准。这里验证的是决策及证据选择，不是“ZCode 已自动派发成功”；角色调用的实际原生目标、回传事件和连续执行轨迹仍须原生会话证明。

### 本轮未运行与适用限制

- 未运行 ZCode/Claude 原生自动调用、角色连续交接及真实客户端加载验证；未执行全局安装或改写用户/业务项目配置。README 只交付可合并的最小入口提示。
- 未运行 `codex doctor`、真实业务仓库 G0–G10、v1 布局迁移、生产服务或性能基准；历史章节中的对应结果不计入本轮。
- 本轮不改变原报告 Schema、Compact 四文件布局、角色名称和原生 Codex 调用策略；静态与隔离检查不等于宿主强制权限沙箱或真实发布验收。

### 受检文件清单

以下 JSON 是指南第一段的候选清单输出，按路径排序。符号链接摘要取链接文本；普通文件记录权限模式与字节摘要。报告自身不在此清单中。

<details>
<summary>97 个候选文件的路径、模式与 SHA256</summary>

```json
[
  {
    "path": ".agents/skills/_devflow_shared/contracts/artifact-lifecycle.md",
    "mode": "0o664",
    "sha256": "822fde59c33193f3bfec218c05611974dd090558f018c8308ae89ba2ece23ac4"
  },
  {
    "path": ".agents/skills/_devflow_shared/contracts/brownfield-policy.md",
    "mode": "0o664",
    "sha256": "92675ac94db6c1093cb561f2adf7e4b807c19b1444122ded8d3e3c69b670de06"
  },
  {
    "path": ".agents/skills/_devflow_shared/contracts/change-control.md",
    "mode": "0o664",
    "sha256": "4d3d2d55d84288810b6eeed6d4cc19d02a14fb701f3ed4a6a1b4a32da2a11e51"
  },
  {
    "path": ".agents/skills/_devflow_shared/contracts/gate-policy.md",
    "mode": "0o664",
    "sha256": "2cc199d29f1ffb617990ab651af7003800c57c0fbdc8da2cc3be18929bdc4aad"
  },
  {
    "path": ".agents/skills/_devflow_shared/contracts/git-policy.md",
    "mode": "0o664",
    "sha256": "4a24cdd3f91fbc005359f5fc7801aca6d48b6132a550870fef967223956b4996"
  },
  {
    "path": ".agents/skills/_devflow_shared/contracts/review-severity.md",
    "mode": "0o664",
    "sha256": "078e41dddd6f444dab3f1df7226616629050ea4587b1e504d27530fcf582dc05"
  },
  {
    "path": ".agents/skills/_devflow_shared/contracts/role-boundaries.md",
    "mode": "0o664",
    "sha256": "8a8dc0f8bd8eec9fb9124c7c59dea9a55ed81b17c43e9d4e16b47b5b07492048"
  },
  {
    "path": ".agents/skills/_devflow_shared/contracts/workflow-state.md",
    "mode": "0o664",
    "sha256": "f3bf1af5e8883b8bdb7f8e6cf7f07d03945484d84ccca9fc0c5945bdd8400907"
  },
  {
    "path": ".agents/skills/_devflow_shared/evals/brownfield-cases.md",
    "mode": "0o664",
    "sha256": "236f4f3e8a119ef468465edd612b6f33435e92d19db4b73d4c2310ade546c1a5"
  },
  {
    "path": ".agents/skills/_devflow_shared/evals/trigger-cases.md",
    "mode": "0o664",
    "sha256": "896ba069c7775cf3119c64e32a3688917826f3e3731ef04dd798ac8e8013bcf4"
  },
  {
    "path": ".agents/skills/_devflow_shared/evals/workflow-cases.md",
    "mode": "0o664",
    "sha256": "47b99baf36f948a56b465f11940c2906daa5dfba9c18343a77a6e1df4735b53f"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/ai-agent-review.md",
    "mode": "0o664",
    "sha256": "6f59514e32b0ac8a6b6a6a9254459323a33a8d1abd9eebd30dd8a9fa2c96e2f7"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/api-review.md",
    "mode": "0o664",
    "sha256": "3f270a5fceb5b1341deb4e492a0d0bf76b26bc9ccd90d375e8adf713ad312668"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/architecture-review.md",
    "mode": "0o664",
    "sha256": "6361f3cfa93b62d85a414f7d75de3959b3a2ed97049d2a038584451fadaab776"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/backend-review.md",
    "mode": "0o664",
    "sha256": "fb92a6a734f2f4533d0e6b6da643dfe55e610e45f9831e8c530b246be7ac212b"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/compact-layout.md",
    "mode": "0o664",
    "sha256": "40918340cc9aca441aed7b1bf12f0e83773003ace17937eed8c9e779d788e006"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/database-review.md",
    "mode": "0o664",
    "sha256": "644b984f49e19708b393ab5fa844cf93bceb4e5fe3c9c5f2178c00e86beffec9"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/devops-review.md",
    "mode": "0o664",
    "sha256": "cc1b20e38d6ea54191fd9d130ad7f0792bba1821b2f8afe7c3f0ab4d0ba4b429"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/distributed-systems-review.md",
    "mode": "0o664",
    "sha256": "213a260d8ca572c7bc2c80a192ae7ceefb0f63b07cdda71a6e6f80d2c9584974"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/frontend-review.md",
    "mode": "0o664",
    "sha256": "9db848a487e56ec9e05e9dc97e7b8c0d9cdfc0bc3ac5ec3f68962477ff37fe5f"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/csharp.md",
    "mode": "0o664",
    "sha256": "e75eec074e06a53fa07bfe7bda62615f6aa8a671efa82adf6c1526d9472f965f"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/go.md",
    "mode": "0o664",
    "sha256": "719566797a9fa8d0d357ad0a2df52f92e56b70fb96d33f84b0c6d4e1fe1c7980"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/java-kotlin.md",
    "mode": "0o664",
    "sha256": "5dbe7c5d15786f643ffc91d7ba6e481f27d209bc267ec8b7f2616a3bc8406f29"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/javascript-typescript.md",
    "mode": "0o664",
    "sha256": "4ee8d8ed044873753321d289554c823248ed1f2cf76d7dc66cdbea6c7fac46e6"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/php-ruby.md",
    "mode": "0o664",
    "sha256": "13849b4f26db0dfccc8972dbf337d18b854029a7b9bc80145e4881de1e6c3208"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/python.md",
    "mode": "0o664",
    "sha256": "50c866d36f021cd76eb9b94cb2508d300531b93dfd047f836d43ab9cbf935e63"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/rust.md",
    "mode": "0o664",
    "sha256": "0d0282081dadbadc65193218417451fc5eb15d9387efe4749c836b072a733cb7"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/shell.md",
    "mode": "0o664",
    "sha256": "800109410876379d333f9e8b86fa7621ca7c22e0de855b7a3f1eb530885497e1"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/languages/sql.md",
    "mode": "0o664",
    "sha256": "f082058d6ffd3c5aa089e71b56e211d4f7269c3dd366cde0c7366a4901a5c870"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/legacy-migration.md",
    "mode": "0o664",
    "sha256": "8e353e1429d18452d2615aafb58710835650605df0c6ee7f21c61d6800784d92"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/minimal-change.md",
    "mode": "0o664",
    "sha256": "42bed23b00fd0848bd63b1baa5988877df671baca07f135980e239e832d91fe6"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/repository-discovery.md",
    "mode": "0o664",
    "sha256": "ca8c8fe2612ec5d8f10892d66b482696993d647beb243e622ef44e75a76bd67c"
  },
  {
    "path": ".agents/skills/_devflow_shared/references/security-review.md",
    "mode": "0o664",
    "sha256": "eb519d9670acd3326745cd96b3c3bda67d804fc8e5e116ded40be247902a5b76"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/compact/current.md",
    "mode": "0o664",
    "sha256": "fbba2e6c3a911aa494e134c42d223d29fdbaf15cbf1f890628800a62db326296"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/compact/evidence.md",
    "mode": "0o664",
    "sha256": "f5abef67535b05ebe47d0b4e1a54728742a76db783cb74b966941fdbd9d74487"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/compact/state.yaml",
    "mode": "0o664",
    "sha256": "92f98bcd4bf25da860b8e3080d8ecdaff6dbad6d090061ffe0a38fd0c816a59e"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/compact/test-plan.md",
    "mode": "0o664",
    "sha256": "aa32544496f5e533d31f61690abdf36533e7d67af9f589b7fd22573cfaeae54b"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/decision.md",
    "mode": "0o664",
    "sha256": "6e8f942ce220a2433ddd1ea0f18cdf3cf702e5d52541d3ccba1dd7230f6e80a6"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/defect.yaml",
    "mode": "0o664",
    "sha256": "6e6a6ceec80f781be9f2872b2a4d856da87030618bcef9121b518f2caec7b0e3"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/implementation-report.yaml",
    "mode": "0o664",
    "sha256": "3a987f1c28b768afa66728594579cc324f7ab47efaf61c4e49ccb063d4c28c19"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/intake.md",
    "mode": "0o664",
    "sha256": "fe866a9631e446b130ff86d99d1289dc53c23eff409f0a18e4c6219818d553f8"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/repository-profile.md",
    "mode": "0o664",
    "sha256": "576a7b7112143dc36f895a73ae4a6644f3627508a6c296a4d5e812d2d122bf61"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/review.yaml",
    "mode": "0o664",
    "sha256": "09f5b03879f5ca735b7282b61f4a1461489c1bcec5f8da74f7c660711388fa17"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/risk-acceptance.md",
    "mode": "0o664",
    "sha256": "501f3ef12b80b3ae16114c4976d5110707a2cfe3913bb145ba742589ae4c2c15"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/root-issue.md",
    "mode": "0o664",
    "sha256": "ab911e109c73325191a8df9ab30d20efca7a962e8f6712b60ccb6729a6349649"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/spec.md",
    "mode": "0o664",
    "sha256": "c9b9a317f11bbc3742870dbec7b8fe8609edf6e9f022e7a18d269356086b6b78"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/state.yaml",
    "mode": "0o664",
    "sha256": "ef11f88518c3cefb962d0194cd5741029751bc8f0f4a5bb479981d961c760b3c"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/takeover-assessment.md",
    "mode": "0o664",
    "sha256": "9d1867920df2f1911d6171abc3bbfe91c6417247470538d89b5c7317bd1ba80e"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/task.yaml",
    "mode": "0o664",
    "sha256": "c101d0fa1d9ebc7301272b65d80fe3a48edaa54270152251d60107221375c77a"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/test-case.yaml",
    "mode": "0o664",
    "sha256": "0aa66a76f92f9d9ddb642c9fd5f7a987c01086511ff6cb0f1d8dd3a8686c25b1"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/test-plan.md",
    "mode": "0o664",
    "sha256": "527257fd2b92b164d4ec16584108967eb47a71bfa0ff32d0f11bfc87ca7a5a96"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/test-report.yaml",
    "mode": "0o664",
    "sha256": "828138d296acc062936515d69e9f655bc13489943a8e45329de609133493a305"
  },
  {
    "path": ".agents/skills/_devflow_shared/templates/ui-spec.md",
    "mode": "0o664",
    "sha256": "79b42170ce2cc31fe8fdcb79f13e2dd1fa44831f4d124c274483542beb943ca7"
  },
  {
    "path": ".agents/skills/devflow-implementer/SKILL.md",
    "mode": "0o664",
    "sha256": "caa2002e58dd22855a9dfee64370f4deebd06b6b4b1a2d58a1b02ed50bc96289"
  },
  {
    "path": ".agents/skills/devflow-implementer/agents/openai.yaml",
    "mode": "0o664",
    "sha256": "19321a1355042d661f7c00cabce6d6e371f7986c4e56e80ee51a1d3ad738461d"
  },
  {
    "path": ".agents/skills/devflow-implementer/references/brownfield-implementation.md",
    "mode": "0o664",
    "sha256": "28317604240647d29edf14d2f690135de64ddb2b8b2cddec5d06c0ea7cc17715"
  },
  {
    "path": ".agents/skills/devflow-implementer/references/implementation-workflow.md",
    "mode": "0o664",
    "sha256": "c262a0e33d54d58cfda5fc36ccb8c7b82b81f6746a726884f707b5a8a7b782a1"
  },
  {
    "path": ".agents/skills/devflow-implementer/references/self-review.md",
    "mode": "0o664",
    "sha256": "25b8538b57ec1b6cbc6e7fd04d54034f558274bfa5b8e4e2ba9fb05772825782"
  },
  {
    "path": ".agents/skills/devflow-planner/SKILL.md",
    "mode": "0o664",
    "sha256": "ff91c9ed9dda25862e4f6fa336f51fc46ad94f84e8fd673e28348bef9436aa24"
  },
  {
    "path": ".agents/skills/devflow-planner/agents/openai.yaml",
    "mode": "0o664",
    "sha256": "f27ee780e5794031fc06e943d1ab512371eb09e67d38ffd66d40085004de6c39"
  },
  {
    "path": ".agents/skills/devflow-planner/references/orchestration.md",
    "mode": "0o664",
    "sha256": "18f4425adaab6c609d3725fd369f0c71d8f50edb2729ea92b9820ad46a39ce99"
  },
  {
    "path": ".agents/skills/devflow-planner/references/requirement-analysis.md",
    "mode": "0o664",
    "sha256": "5584e15b586f3a0878152ac4234c81b0499ed93d41e8e1833f58033ece0399bc"
  },
  {
    "path": ".agents/skills/devflow-planner/references/task-decomposition.md",
    "mode": "0o664",
    "sha256": "a379e4b2e13f9b046b768bec446d1a43f1d745f0e3fe2f83be1191f52cf7f1eb"
  },
  {
    "path": ".agents/skills/devflow-reviewer/SKILL.md",
    "mode": "0o664",
    "sha256": "3e3062ce6d876863b24049b7af5025726f530d25ea2579044cc3e30a78c4f268"
  },
  {
    "path": ".agents/skills/devflow-reviewer/agents/openai.yaml",
    "mode": "0o664",
    "sha256": "41f513c72e7c4f000162cc0f849d523890a29b7622eabfe9ffc99fdb14fad3a3"
  },
  {
    "path": ".agents/skills/devflow-reviewer/references/baseline-review.md",
    "mode": "0o664",
    "sha256": "3fe209def6407d77a6b9be63a862b691e72c4113ef72cbca84c53c3ed4524ed0"
  },
  {
    "path": ".agents/skills/devflow-reviewer/references/code-review.md",
    "mode": "0o664",
    "sha256": "7e735aa17eec1baf4b465a9a5047981c9d07229f5d3108aa09bf39ad36f6a090"
  },
  {
    "path": ".agents/skills/devflow-reviewer/references/release-review.md",
    "mode": "0o664",
    "sha256": "b1fde3f4bb41acc313f3e406497b46a3f09417404a21c347548befe23f6bd589"
  },
  {
    "path": ".agents/skills/devflow-reviewer/references/spec-review.md",
    "mode": "0o664",
    "sha256": "8e4c12a250e92fd85f64f2aae14103cc282ae2177325db3388537a84bbb366af"
  },
  {
    "path": ".agents/skills/devflow-reviewer/references/test-review.md",
    "mode": "0o664",
    "sha256": "def7c6ee28ba4fad5777c634e10f5bece8b640c9461c019a6a4d700d86d4cd8e"
  },
  {
    "path": ".agents/skills/devflow-tester/SKILL.md",
    "mode": "0o664",
    "sha256": "636b3d56018e0d6c2d25b0d7d4d1f031437d60e443992e338f8dfc73fad12f12"
  },
  {
    "path": ".agents/skills/devflow-tester/agents/openai.yaml",
    "mode": "0o664",
    "sha256": "c45e8ae2b8248ca10a3a221fa86abebfb9a6befe1234bbf926c930204034a9a9"
  },
  {
    "path": ".agents/skills/devflow-tester/references/characterization-testing.md",
    "mode": "0o664",
    "sha256": "b9015482fb7642db35a81d951090e34e71b7921f6e0f0c58d06fa386020bdb06"
  },
  {
    "path": ".agents/skills/devflow-tester/references/defect-reporting.md",
    "mode": "0o664",
    "sha256": "06453801ec36fcb8e42a41be9360c3a74fd5d5bfef8ee51182ecb1545b75f941"
  },
  {
    "path": ".agents/skills/devflow-tester/references/integration-testing.md",
    "mode": "0o664",
    "sha256": "4e94b590fc116082e00dc7279f9ae2661a4903da3da0189e00fd4d443c07bdeb"
  },
  {
    "path": ".agents/skills/devflow-tester/references/test-code-delivery.md",
    "mode": "0o664",
    "sha256": "c5a0755d4e23411bcec49a2078e13fa02d2d9afc836c3ba226ae7e7fbe98581e"
  },
  {
    "path": ".agents/skills/devflow-tester/references/test-strategy.md",
    "mode": "0o664",
    "sha256": "4ac59477bc456eac69c536348e5714deee3915d08263f1ec29672c09229c9027"
  },
  {
    "path": ".claude/agents/devflow-implementer.md",
    "mode": "0o664",
    "sha256": "717f494c88e768afecd3b8847187da3b228c5808f45589cbd2927364f5729830"
  },
  {
    "path": ".claude/agents/devflow-planner.md",
    "mode": "0o664",
    "sha256": "644b8342d1e8b5503f8c9c104257fcca22fd39f2f623e837f1d2d4b868592226"
  },
  {
    "path": ".claude/agents/devflow-reviewer.md",
    "mode": "0o664",
    "sha256": "8d2be354ec3ff8bbd2e9fd518ac26dfa3edd45591b3ee5d01637fa8b580e9926"
  },
  {
    "path": ".claude/agents/devflow-tester.md",
    "mode": "0o664",
    "sha256": "c9f926acd33a700a8a229a1acdbdc3e605e3091ecefe6b1fefbf959d1805f447"
  },
  {
    "path": ".claude/skills",
    "mode": "symlink",
    "sha256": "3c74c93317f015d16370349a76529b6f8332c808bdd589e7640eeba6a1b1b874"
  },
  {
    "path": ".codex/agents/devflow-implementer.toml",
    "mode": "0o664",
    "sha256": "ad35441288e8f9f1165fb180008254d27533d9962b76eabcab5c233b685404f0"
  },
  {
    "path": ".codex/agents/devflow-planner.toml",
    "mode": "0o664",
    "sha256": "043817a3d28760c6c020380ab491b066547a0a0f4f269d9a1fa33561ebcb4c41"
  },
  {
    "path": ".codex/agents/devflow-reviewer.toml",
    "mode": "0o664",
    "sha256": "5b47fee175e1a6e86180f770c0879f821e1d870c4e4551eef0bc2d5adf71e9bf"
  },
  {
    "path": ".codex/agents/devflow-tester.toml",
    "mode": "0o664",
    "sha256": "82a463ac4f075ffc823675260108e78a9003cbf965880b6bf20ee7ead7388b5f"
  },
  {
    "path": ".codex/config.toml",
    "mode": "0o664",
    "sha256": "fb26706a468cd952f3f64fc51f9fbadca8ea40b1a8c7072a2bdfb761edadb976"
  },
  {
    "path": ".devflow/README.md",
    "mode": "0o664",
    "sha256": "fecaec502d60c172075b8d20bb7eedf3dc9f15166d6d3ebfebe838057b03c909"
  },
  {
    "path": ".gitignore",
    "mode": "0o664",
    "sha256": "efd130085d0b6ab966e1fb9396c07b01ca0f41e9f15ebdf16271b571ce1f12dc"
  },
  {
    "path": ".zcode/agents/devflow-implementer.md",
    "mode": "0o664",
    "sha256": "b38d56ebb715f104ad3c61d1d6d972a9ed42cdc6050ba60058ec3014b165954b"
  },
  {
    "path": ".zcode/agents/devflow-planner.md",
    "mode": "0o664",
    "sha256": "e004d5f7bbe2bb2df4740201fc263b68c0b3293a20c11db64798c30f0a582a26"
  },
  {
    "path": ".zcode/agents/devflow-reviewer.md",
    "mode": "0o664",
    "sha256": "fafac0c63641d36d4f3d92a385a5f1b57021c50e15bff5959f19dd050a91f061"
  },
  {
    "path": ".zcode/agents/devflow-tester.md",
    "mode": "0o664",
    "sha256": "1d01fac5222a044b0ece5c30716c3bd8d8bdff522e4537d76dd4474b6454af2d"
  },
  {
    "path": "AGENTS.md",
    "mode": "0o664",
    "sha256": "ac3c2dbe975000f2a31d80db7d12b3e0df344a53f3da8d4e601b0d24c6458f55"
  },
  {
    "path": "README.md",
    "mode": "0o664",
    "sha256": "252baea44ae58b860018771c6ba5b1b132c7ed61f175d1f13ab4a309ebcb9f26"
  },
  {
    "path": "docs/DEVFLOW_SKILLS_DESIGN.md",
    "mode": "0o664",
    "sha256": "fac64bc0f747e2b5f99402e5b30b9071ca5366bc2ba83385b53212c0dd830f50"
  },
  {
    "path": "docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md",
    "mode": "0o664",
    "sha256": "003ee76c15a8d1a6985378f9cbc59c7fbc0f449d0627c7065738c9dc1dcd0d29"
  }
]
```

</details>

### 最终收尾复检

追加报告后的源树复检退出 0：4 个 Skill、12 YAML、5 TOML、15 frontmatter、79 个 Markdown 文档、151 个本地链接和 23 处锚点引用均通过，候选集合 SHA256 与干净副本一致。历史报告前 34935 字节保持不变；29 个受保护包文件和 1 个用户计划文件摘要一致。仅有本轮交付和原有用户计划处于未提交状态，暂存区为空。指南创建的 fixture 与本轮额外创建的隔离副本、失败注入文件和临时日志均已清理，源码分支和 HEAD 未变。

## Implementer 业务命名与既有代码保护补充（2026-09-08）

本节追加于前次验证记录之后，历史原文保持不变。新增规则由 Codex、Claude Code、ZCode 共同读取的 Implementer Skill 承载；Claude/ZCode 原生入口显式引用，自审增加命名冲突、共享调用方及回归检查。命名应表达当前业务含义，避免作用域遮蔽和同名异义；实施优先局部、预算内的最小完整变更，保持范围外行为与契约，不批量重命名旧代码。规则没有把命名清晰等同于消除 Git 合并冲突。

本次仅执行验证指南第一段完整静态检查，退出 0：四 Skill 验证器 4/4 通过，YAML 12、TOML 5、frontmatter 15、Markdown 79、本地链接 154、锚点引用 26，均无失败。源码 HEAD 仍为 `84954fbda3c1d8c47ef2a5ee9fb43e18ab4a3c4a`；相对前次候选，仅下列四份规则文件内容变化，原有其他候选文件、Codex 配置、模板和用户计划保持不变。另追加本报告，不暂存、提交或推送。

当前候选集合 SHA256 为 `2cae75ba4d0f1745250e63bbb61b038dda0601e765537cc42c0cac453a26db10`。受检集合仍为前节的 97 个文件；用下列四条替换前节清单中的同路径条目即可重建本次完整清单，摘要算法不变，报告自身仍排除。未重跑隔离安装/Git fixture 或独立代理行为评测：本次为局部指令补充，安装代码、状态/测试交付规则未变。未运行原生客户端，不宣称此静态检查证明实机执行效果。

```json
[
  {
    "path": ".agents/skills/devflow-implementer/SKILL.md",
    "mode": "0o664",
    "sha256": "ea97e45afe9e0bdbfbe68b31fbd45981274ac1438d38e7eca4faef985ebcf2a7"
  },
  {
    "path": ".agents/skills/devflow-implementer/references/self-review.md",
    "mode": "0o664",
    "sha256": "394d0dc5f8bf56b434f729c3b1f2feff517ac54317d7a2ae8c5dc3b5f06c52ac"
  },
  {
    "path": ".claude/agents/devflow-implementer.md",
    "mode": "0o664",
    "sha256": "f04fd7401e001f968389f4b676e186f6ec07b162882a841dc5b7c1cfd0b0899a"
  },
  {
    "path": ".zcode/agents/devflow-implementer.md",
    "mode": "0o664",
    "sha256": "a9be5b77dcabf44377dcaf3409b86ae94dc78e71b1bf018e8fb434c2b52478b4"
  }
]
```


## 2026-09-13 — 标准五 Skill 仓库：源码与隔离验收

本轮用户授权的**源码改造与隔离验收结果为 PASS**。正式 Commit 测试报告的 Verdict 为 **BLOCKED**：当前没有包含候选与测试材料的真实产品 Commit，不能填写产品 `tested_sha` 或宣布 G5–G9 通过。此限制不改变本轮已完成的源码交付，也不要求为范围外的提交、全局安装或原生环境另行扩大任务。

- Root：REQ-20260912-001；唯一 TASK-001；Spec revision 1、PLAN revision 1。Spec 快照 SHA256 为 `ec137e56df04ae11b70635e46d741e6744569ef3949e200085ed122756b81d5b`，独立审核通过的 PLAN 快照为 `2d74c82faf42a0290f6348cc30d1a806f027899074244bf2790653712978bdde`。
- 产品完整基线：`a72592ec1cb7ac5c67ec5454233d673a30000be5`；候选文件集 SHA256：`0e9c93a7a0e2ff5d1dc732e31120a00ed27ccc1b578ff73cca45dd5d9c704078`。集合包含仍存在的跟踪文件与新增 skills/docs，报告自身仅参与历史/链接检查、不进入集合摘要。未提交源码不能只用旧 HEAD 标识。
- 最终运行包：96 个普通文件、5 个实际 Skill 目录；runtime SHA256 `eeb6086f996d4a4e47acead3b20bc238038e9c5eead148b17697159cb103653f`。临时 Git fixture `355991cdbd58dd9cea58d214ca4d4c8be6def5c6` 仅为合成测试对象，不能当产品 Commit。
- 实际环境：Linux/Bash、Python 3.14.4、PyYAML 6.0.3、Git 2.53.0；Python 每次显式设置 UTF-8。Bash 未发现 Node，没有全局安装依赖。
- 本文件前 63575 字节与基线 Git 对象逐字相同；3 个历史链接按该对象核验。新增内容与全部当前包/文档继续执行当前链接检查，无整包或整报告豁免。

### 实际执行与独立证据

完整可复制命令在[验证指南](DEVFLOW_SKILLS_VALIDATION_GUIDE.md)。原始输入、独立执行者观察输出、命令/文件摘要和 Tester 判定保存在[本轮评测证据](DEVFLOW_SKILLS_EVAL_20260913.md)，可随源码保留；未把数千份 fixture、副本 Git 对象或临时安装脚本加入发布包。详细本地原始回执仍保留在本 Root attachments，公开复跑不以这些 ignored 路径为唯一方法来源。

| 检查或执行 | 实际结果与证明范围 |
|---|---|
| 五个 bundled quick_validate | 五次均 exit 0；不是仅保留循环最后一个退出码 |
| YAML/TOML/frontmatter | 13 YAML、5 TOML、16 frontmatter 可解析，重复 YAML key 会失败 |
| 发布闭包与兼容 | 当前相对链接/锚点均有效；四角色闭包不依赖无关角色；20模板原字节一致；8共享契约只发生精确链接迁移；三平台 ID、模型/工具/推理/沙箱意图保持 |
| README 普通复制 | 真实复制五目录96文件、仅入口76文件；重复请求按该“仅新空目标”示例退出1并保全；已有普通文件冲突在任何部分复制前停止。这不代表bootstrap幂等或CLI集成 |
| 检查器负控 | 8例：重复YAML key、损坏TOML、额外Skill、当前坏链接、坏锚点、历史前缀改变、错frontmatter名称、缺description导致bundled验证器失败；各自真实子进程exit1，干净对照exit0 |
| A批，runtime1 | 11例；328文件真实复制/读回；四个单角色分别恢复入口，最小闭包与子角色回交/来源缺口符合其声明范围 |
| B批，runtime1 | 16例；141文件真实复制/读回；逻辑project/user范围、真实symlink、合成ZCode目录例外、定制/版本/类型冲突、显式禁用与配置保全符合要求；B07第二次读回100目标、零写入 |
| C批，runtime1 | 7组原始决定输入逐项判定：触发排除、实际宿主证据、平级转发、安装/加载/调用分列、权限/批准、v1/v2与SHA、VERIFIED到DONE；没有真实业务调用或状态写入 |
| D批，runtime2 | 15例；329文件真实复制/读回；10个停止场景零写入，原bytes/type/mtime及source/user保全；30个inspect/action命令exit0。入口缺bootstrap恢复、四单角色正向及原生定义冲突/明确禁用先于首次补装均实测 |
| README 标准CLI说明负向跟随 | 2个真实隔离预检：已有canonical定制目录、已有宿主symlink；按修订说明首个CLI调用前停止，哨兵bytes/type/mtime未变。CLI调用数为0，只证明说明预检的本地决定 |

A/B/C结果始终绑定runtime1 `58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106` 及其合成fixture `9793122fc00822939cfd4565d904fe8d2f91da0e`；没有改绑旧报告。runtime2仅改变五入口守卫和一个eval链接，共6文件，其余90文件相同；受影响分支由D与新完整静态检查覆盖。引用未变输入下的既有结果，不重跑无关套件。

### 验收标准追踪

下列PASS均限定为当前源码、真实隔离文件与独立决定切片；原生/CLI等更高边界见后续未运行项。

| AC | 用例 | 结果及主要证据 |
|---|---|---|
| AC-001 | TC-001/003 | PASS：根skills五实际目录、名称/UI、五验证器与类型清单 |
| AC-002 | TC-002/014 | PASS：入口顺序与按需规则、C请求分类和路由决定 |
| AC-003 | TC-003/004 | PASS：发布包普通复制闭包，唯一共享源、模板/配置迁移，旧坏链接已独立复测 |
| AC-004 | TC-005–008 | PASS：A恢复/最小闭包与D最新守卫实际文件结果；子角色不安装 |
| AC-005 | TC-009/018 | PASS：C明确/未知/冲突宿主及工具缺口决定 |
| AC-006 | TC-010 | PASS：B真实逻辑路径/symlink范围，合成ZCode用户Agent例外 |
| AC-007 | TC-011/012 | PASS：A/B可信本地固定源、现存全包匹配/冲突、离线；获取服务仅决定级 |
| AC-008 | TC-013 | PASS：B/D全目标预检、配置与哨兵保全、重复/恢复；CLI说明两个负向预检 |
| AC-009 | TC-015/016 | PASS：三平台配置基线比较、C单Planner与平级转发决定 |
| AC-010 | TC-017/018；条件TC-024 | PASS于声明切片：C三阶段/重载/缺口；原生实机未运行 |
| AC-011 | TC-019–021 | PASS：现用文档/链接、安装说明、原历史前缀和8负控 |
| AC-012 | TC-004/022/023 | PASS：模板/契约保持、C原证据/状态/DAG/职责决定，安装不替代Gate |

### 失败、修复与未运行项

1. **runtime1真实产品失败**：eval仍链接包外维护指南；独立链接命令exit1（112相对链接中1失败）。Defect `DEFECT-REQ-20260912-001-RUNTIME-LINK-001` 交Planner归因实施，runtime2修正后完整当前包检查exit0。旧失败原件保留。独立Reviewer另发现缺bootstrap时入口无恢复分支、四单角色先写入口再查原生冲突/禁用；实现修正后D覆盖新分支，旧Review不改写。
2. **A02测试收集器失败**：四个首次执行真实exit1，错误为 `KeyError: 'name'`，配置示例TOML本来没有原生角色name字段。只修收集字段，候选不变；重新核验已写7文件后各补69文件，恢复exit0。事件 `DEFECT-REQ-20260912-001-EVAL-COLLECTOR-001` 的原错误、命令和恢复证据均保留，不冒充产品历史失败，也未弱化Oracle。
3. **真实权限拒绝未建立**：B10 chmod后仍观测mode0777，窄probe实际写入成功；因此真实权限失败切片BLOCKED，拒绝后的停止只是合成事件的决定级验证。不能称真实权限隔离通过。
4. **标准CLI、PowerShell与三平台原生加载/调用未运行**：没有执行npx安装器、真实用户级安装、宿主重载或原生角色调用；静态配置、普通复制、合成Claude/ZCode身份均不能替代它们。README已明确CLI仅适用于全部实际/canonical目标不存在的新安装，既有安装走bootstrap；去除-y不被当成充分保护。
5. **正式Commit门禁未执行**：没有新产品head/tested Commit，也没有合并、发布或目标冒烟；正式载荷Verdict BLOCKED。源码与隔离验收已完成，不把范围外条件写成产品实现失败，不请求新增提交或安装权限。

本轮未新增产品CLI、包清单、常驻脚本或全局依赖；Tester仅修改验证指南、新的证据文档与本报告追加段，未修改生产包、Spec或全局状态。原业务Gate、Schema、职责与历史记录保持。


## 2026-09-13 — 调度、职责名称与 .local 维护补充

本次是用户明确要求的调度说明、名称可读性与忽略配置维护。沿用前轮未提交源码，未重新启动完整业务流程。本段由主会话记录实际维护检查，不是新的正式 Commit 测试报告，也不改绑上轮测试或审核。

- `.gitignore` 保留整个 `.devflow/changes/` 的包级忽略，另加 `/.devflow/changes/*/.local/`。实际 `git check-ignore -v .devflow/changes/REQ-20260912-001/.local/probe.txt` exit 0；`git ls-files -- .devflow/changes/REQ-20260912-001/.local` 为空。父目录规则仍是实际命中来源。
- 在本 Root 的 `.local` 内创建临时 Git 仓库，只放新增规则：指定及另一 Root 的 `.local` 两个正例命中；current/evidence/state/test-plan、attachments 和无关 `.local` 六个反例不命中。该规则自身不扩大证据忽略；临时仓库已精确清理，既有文件未删除。
- 实际执行[验证指南](DEVFLOW_SKILLS_VALIDATION_GUIDE.md)第一个完整 Bash 块，exit 0：五次 bundled Skill validator、13 YAML、5 TOML、16 frontmatter、167 当前链接及锚点、3 历史链接检查通过；20 模板与基线原字节一致。四原生 ID、模型/推理/沙箱、Claude/ZCode 工具及 UI 调用策略保持。
- 当前检查文件集 SHA256 为 `266c939d0e7b66cb90e1890b3e63a69dd843223a35d86831c3786cf5851cc0d1`，算法与指南一致，排除报告自身以避免自引用；它不是产品 Commit。未改动的安装场景与检查器负控未重复执行，也未把旧行为结果重新绑定为本次原生实机结果。
- 本轮只复用既有实现者和审核者，没有新增子代理。新增规则要求按需派发、同职责优先复用、增加盲评执行者前说明必要性、在真实工具支持时使用 `ric_<role>_<work>` 任务名，以及中文职责自述。Skill UI 标题和原生描述已改善；未修改原生稳定 ID、权限、模型或并发配置。
- **Node/CLI 实际限制**：当前执行环境为 Linux x86_64，PATH 中仍没有 node/npm；只读发现 Windows FNM 的 `v24.16.0/installation/node.exe`，直接启动实测为 `errno 8: Exec format error`。目录版本标记不是成功运行的版本回执；标准安装器 CLI 仍为 NOT_RUN，没有下载新运行时或修改全局环境。
- **界面与性能限制**：未执行三平台原生加载、新任务标题展示或旧会话改名，也未测量修改后的长期代理数量/耗时；不承诺 Skill `display_name` 会改变子代理昵称。当前工具未提供旧子代理重命名接口。
