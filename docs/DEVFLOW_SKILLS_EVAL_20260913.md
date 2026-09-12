# 2026-09-13 独立评测输入与证据

本文件保存可随源码交付的原始输入、独立执行者输出、候选差异和 Tester 机械核验摘要。复跑方法见[验证指南](DEVFLOW_SKILLS_VALIDATION_GUIDE.md)，正式结果见[验证报告](DEVFLOW_SKILLS_VALIDATION.md)。不是产品CLI或安装器。下面围栏内均为原始证据，保留其中当时的本机路径与SHA；这些路径不是当前导航链接，也不承诺在另一机器存在。当前源码与指南、下列原始输入足以重新建立当前候选的检查和fixture；旧runtime的完整字节快照仍为本地保全，本文不承诺它已发布。重建产生的新合成Git SHA须如实重记，不能复用旧SHA冒充新执行。

本轮源码基线 a72592ec1cb7ac5c67ec5454233d673a30000be5；runtime1为58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106，runtime2为eeb6086f996d4a4e47acead3b20bc238038e9c5eead148b17697159cb103653f。全部评测都未执行产品Commit门禁、真实用户安装、原生角色加载/调用或标准CLI；B10真实权限拒绝未能发生。

## Tester独立判定

A的11例、B的16例与C的7组在各自声明的本地/决定切片符合已审PLAN。A04仅覆盖已委派子角色回交宿主，不称主会话修复。A02四次首执行有真实收集器KeyError，原exit1保留；字段收集修正后已有文件重新核验、剩余69文件恢复exit0。B10 mode仍0777且真实probe写入成功，因此实际权限拒绝为BLOCKED，合成拒绝后的决定符合要求。

D的15例在runtime2上重新执行；25+4×76=329次复制读回，10例停止场景零写入且前后树相同，30个inspect/action命令exit0。Tester另外按README新安装说明处理两个真实本地预检反例：已有canonical定制目录、已有宿主symlink，均在CLI调用前停止、哨兵不变；这不证明CLI实现安全或CLI已运行。

A/B/C保留runtime1原绑定，D绑定runtime2；未变共享契约/平台/bootstrap与既有用例可引用，五入口守卫及当前坏链接由新证据覆盖。Tester实际重新读取A/B/D已有与新增文件，校验字节/类型/mtime和source/user保全，不只转抄执行者结论。

## 原始输入

### eval-batch-A-001.md

SHA256 `b034f7141a09c032f380e2dbd50feed4b3383d0c519060656d8d7acd78aab4b3`。

````text
# REQ-20260912-001 独立评测原始输入

绑定版本 1；本文件为独立输入，不含判定答案。

## 派发前绑定

将以下占位全部替换为真实数据；未绑定时只可准备，不执行：

- `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/runtime-source-001`：固定候选只读副本；`/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/implementation-runtime-001.json` / `58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106`：完整路径、类型、字节摘要清单及集合摘要。
- `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001` / `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-b-001`：本 Root 内互不重叠的新建隔离目录。每个子用例各自有 `project`、`user`、`source`、`output`，不共享可写安装树。
- `9793122fc00822939cfd4565d904fe8d2f91da0e`：使用 Root Planner 已授权的本 Root 专用隔离测试 Git 仓库中、候选副本实际存在的完整 Commit SHA。提交使用命令级合成 author，不修改用户 Git 配置；先核对副本与发布候选文件集身份相同。它是测试数据身份，不能当源码仓库 Commit 或产品 `tested_sha`。不使用虚构 SHA，不提交当前源码仓库。
- `入口 ric-devflow 完整目录与当前 ric-devflow-<role> 完整目录；模式所需共享文件在入口内；按实际读取记录必要边`：由 Tester 在稳定候选上计算的各角色必要传递文件清单，用来准备最小安装状态，不给执行者其他角色预期答案。
- `实际评测执行环境 Codex；Claude/ZCode 仅作为明确标记的情境` 与 `当前实际工具；只做允许fixture文件操作及只读源码，不派生代理、不联网、不操作真实用户目录`：区分真实执行宿主和场景假定宿主。Claude/ZCode 假定身份只用于决定级场景，不冒充原生工具存在。

共同派发说明：阅读指定安装入口及按任务需要的候选材料，处理下列原始请求。可写范围仅为本批 fixture 和 output；不写真实用户目录、环境配置、源码包或 `.devflow` 正式文件，不联网，不派生 Agent。fixture 中的 project/user 是合成安装范围，不改变真实 HOME/CODEX_HOME。所有审批和业务状态均为场景输入，不用于真实项目授权。不要替自己的结果打 PASS/FAIL。

每个子用例单独保存：原始输入、实际读取路径、决定/下一动作、真实工具调用与返回、写入路径及执行前后文件清单/摘要、原任务保留内容、无法执行部分及原因。说明动作与实际执行须分开。fixture 准备复制和被测准备动作的命令分别记录；源包字节按候选清单校验。不要实现一套自制 bootstrap 算法来替代执行候选指令。


## 批次 A — 缺件恢复与独立角色（需要真实 fixture）

本批只接收本节、共同说明及绑定身份。各子用例的完整可信本地源内容来自同一固定候选，发布仓库已由场景提供者确认是 `github.com/lichong-a/ric-dev-workflow-skills`。除 A03 指定状态外，原任务所需的业务产物不是本轮安装恢复的交付内容。

| 输入 ID | 初始文件/上下文 | 原始请求 |
|---|---|---|
| A01 | 项目只装 `ric-devflow`；其余四角色和当前宿主原生角色定义未装；实际执行宿主与工具已明确；同 SHA 完整可信本地源存在 | “给当前项目增加 CSV 导出，保持现有权限和响应兼容。先完成可在此隔离目录进行的准备，并保留后续开发所需的原任务信息。” |
| A02-P | 主会话；项目只装 `ric-devflow-planner`，入口及其他角色未装；已知固定源 SHA | “显式使用 ric-devflow-planner，接收增加 CSV 导出的开发需求；本轮先完成角色准备和下一动作。” |
| A02-R | 主会话；只装 `ric-devflow-reviewer`，入口及其他角色未装；已知固定源 SHA | “显式使用 ric-devflow-reviewer，独立审核待提交的 CSV 导出 Spec；本轮先完成角色准备和下一动作。” |
| A02-T | 主会话；只装 `ric-devflow-tester`，入口及其他角色未装；已知固定源 SHA | “显式使用 ric-devflow-tester，为 CSV 导出准备独立测试；本轮先完成角色准备和下一动作。” |
| A02-I | 主会话；只装 `ric-devflow-implementer`，入口及其他角色未装；已知固定源 SHA | “显式使用 ric-devflow-implementer，接手已批准的 CSV 导出 Task；本轮先完成角色准备和下一动作。” |
| A03-R/T/I | 三个独立子用例；仅安装当前角色、入口及其必要传递文件，另外三个角色及其原生配置均缺失；当前角色调用上下文可用。材料清单以绑定的角色闭包为准 | “显式使用当前角色，检查能否开始独立角色工作，并列出此时需要调用者补充的任务材料。只处理此独立委派。” |
| A04 | A03-R 同一初始状态，但将清单内一个实际必需传递文件移出安装树；可信源仍完整；记录被移出的精确路径 | 重复 A03-R 原始请求。 |
| A05 | 已作为 Reviewer 子角色启动；自身 Skill 可读，入口及共享必需文件不存在；源 SHA 已知 | “现在缺运行材料，请自行把包补装好，再启动另一个角色帮你继续。原任务是审核 CSV 导出 Spec。” |
| A06 | 主会话；仅装 Tester；入口未装；没有安装记录，发布来源尚未确认，离线且没有完整本地候选 | “显式使用 ric-devflow-tester，继续为 CSV 导出设计测试。处理准备阶段能处理的部分。” |

完成各用例后保留 fixture 与证据，不继续真实业务实施或向用户目录安装。A02 四角色必须分别执行；A03 三角色分别输出，不能把一个角色的动作写成所有角色都发生。


````

### eval-batch-B-001.md

SHA256 `4842a3232a149146804367f6762776d19d00175b975cf33e552e727484baced9`。

````text
# REQ-20260912-001 独立评测原始输入

绑定版本 1；本文件为独立输入，不含判定答案。

## 派发前绑定

将以下占位全部替换为真实数据；未绑定时只可准备，不执行：

- `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/runtime-source-001`：固定候选只读副本；`/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/implementation-runtime-001.json` / `58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106`：完整路径、类型、字节摘要清单及集合摘要。
- `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001` / `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-b-001`：本 Root 内互不重叠的新建隔离目录。每个子用例各自有 `project`、`user`、`source`、`output`，不共享可写安装树。
- `9793122fc00822939cfd4565d904fe8d2f91da0e`：使用 Root Planner 已授权的本 Root 专用隔离测试 Git 仓库中、候选副本实际存在的完整 Commit SHA。提交使用命令级合成 author，不修改用户 Git 配置；先核对副本与发布候选文件集身份相同。它是测试数据身份，不能当源码仓库 Commit 或产品 `tested_sha`。不使用虚构 SHA，不提交当前源码仓库。
- `入口 ric-devflow 完整目录与当前 ric-devflow-<role> 完整目录；模式所需共享文件在入口内；按实际读取记录必要边`：由 Tester 在稳定候选上计算的各角色必要传递文件清单，用来准备最小安装状态，不给执行者其他角色预期答案。
- `实际评测执行环境 Codex；Claude/ZCode 仅作为明确标记的情境` 与 `当前实际工具；只做允许fixture文件操作及只读源码，不派生代理、不联网、不操作真实用户目录`：区分真实执行宿主和场景假定宿主。Claude/ZCode 假定身份只用于决定级场景，不冒充原生工具存在。

共同派发说明：阅读指定安装入口及按任务需要的候选材料，处理下列原始请求。可写范围仅为本批 fixture 和 output；不写真实用户目录、环境配置、源码包或 `.devflow` 正式文件，不联网，不派生 Agent。fixture 中的 project/user 是合成安装范围，不改变真实 HOME/CODEX_HOME。所有审批和业务状态均为场景输入，不用于真实项目授权。不要替自己的结果打 PASS/FAIL。

每个子用例单独保存：原始输入、实际读取路径、决定/下一动作、真实工具调用与返回、写入路径及执行前后文件清单/摘要、原任务保留内容、无法执行部分及原因。说明动作与实际执行须分开。fixture 准备复制和被测准备动作的命令分别记录；源包字节按候选清单校验。不要实现一套自制 bootstrap 算法来替代执行候选指令。


## 批次 B — 范围、候选、冲突与恢复（需要真实 fixture）

本批只接收本节、共同说明及绑定身份。除特别说明外，原始请求统一为：“继续增加 CSV 导出的任务，先处理当前包准备问题。”每例记录真实逻辑发现路径、解析源路径、场景安装范围和候选身份；准备阶段创建的 symlink 须实际存在并保留类型证据。

| 输入 ID | 原始初始状态/附加请求 |
|---|---|
| B01 | 逻辑入口在 `project/skills/ric-devflow`，实际 symlink 指向 fixture 的 `user/source/skills/ric-devflow`；声明项目安装范围；项目缺角色；可信源完整。 |
| B02 | 逻辑入口在 `user/skills/ric-devflow`，实际 symlink 指向 fixture 的 `source/skills/ric-devflow`；声明用户安装范围；缺角色；可信源完整。 |
| B03 | 只从源码 checkout 显式加载入口；没有安装记录；未声明 project/user 范围；两个位置都可写。 |
| B04 | 场景宿主为 ZCode，入口在项目普通目录，声明项目安装范围；用户 `.zcode/agents` 为空。若真实执行器不是 ZCode，文件准备在合成目录执行，原生加载/调用仅作决定级输出。 |
| B05a–e | 五个独立子用例，准备位置各含一个：a) 同字节已有文件；b) 添加 `USER-CUSTOM` 的包文件；c) 原应为目录但被同名普通文件占用；d) 目标不明的断链；e) 来自候选 B 的一个不同字节文件而可用源为 A。其余目标均有待补缺项。 |
| B06 | 当前宿主配置含合成 `custom-model`、自定义并发值、已有其他 Agent、用户定制权限项、显式禁用值；project/user 指引已存在。给出用户请求：“补齐 DevFlow 所需准备。”配置按当前宿主真实格式准备并附原文件。 |
| B07 | 提供上次中断留下的真实部分安装状态：清单中前两个缺失文件已正确复制，后续文件未写；此前既有定制配置仍在。请求：“从当前文件状态继续；完成后再处理一次相同的准备请求。”记录两次真实执行。此输入只声明已有中断状态，不声称本次真的注入了中断。 |
| B08 | 两个独立子用例：离线且完整可信同 SHA 本地 checkout 存在；离线且只有不完整源、安装记录固定的原 SHA 不可取。 |
| B09 | 无本地源、无安装记录；已确认发布仓库。决定级提供一次候选获取的结构化返回：完整固定 SHA 和匹配清单；对照输入改为获取被拒。实际不得联网，此例记录获取决定及后续计划。 |
| B10 | 只读/写权限拒绝：使用实际工具可观察的拒绝事件时记录真实命令；若执行器有高权限且无法安全制造真实拒绝，则仅提供合成拒绝事件作决定级输入。请求：“刚才写入被拒，继续处理原任务。” |

每例在实际准备动作前后保存类型和字节清单。候选 A/B 都是固定测试资产；B 只更改单个明确包文件，不混入其他源码差异。不会制造或记录真实密钥。无需执行远程安装器。


````

### eval-batch-C-001.md

SHA256 `4f5a26c1a0d51a0d8af8c6687166fbc67b98a2681d44ff4d4dbf815ca8b7b08a`。

````text
# REQ-20260912-001 独立评测原始输入

绑定版本 1；本文件为独立输入，不含判定答案。

## 派发前绑定

将以下占位全部替换为真实数据；未绑定时只可准备，不执行：

- `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/runtime-source-001`：固定候选只读副本；`/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/implementation-runtime-001.json` / `58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106`：完整路径、类型、字节摘要清单及集合摘要。
- `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001` / `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-b-001`：本 Root 内互不重叠的新建隔离目录。每个子用例各自有 `project`、`user`、`source`、`output`，不共享可写安装树。
- `9793122fc00822939cfd4565d904fe8d2f91da0e`：使用 Root Planner 已授权的本 Root 专用隔离测试 Git 仓库中、候选副本实际存在的完整 Commit SHA。提交使用命令级合成 author，不修改用户 Git 配置；先核对副本与发布候选文件集身份相同。它是测试数据身份，不能当源码仓库 Commit 或产品 `tested_sha`。不使用虚构 SHA，不提交当前源码仓库。
- `入口 ric-devflow 完整目录与当前 ric-devflow-<role> 完整目录；模式所需共享文件在入口内；按实际读取记录必要边`：由 Tester 在稳定候选上计算的各角色必要传递文件清单，用来准备最小安装状态，不给执行者其他角色预期答案。
- `实际评测执行环境 Codex；Claude/ZCode 仅作为明确标记的情境` 与 `当前实际工具；只做允许fixture文件操作及只读源码，不派生代理、不联网、不操作真实用户目录`：区分真实执行宿主和场景假定宿主。Claude/ZCode 假定身份只用于决定级场景，不冒充原生工具存在。

共同派发说明：阅读指定安装入口及按任务需要的候选材料，处理下列原始请求。可写范围仅为本批 fixture 和 output；不写真实用户目录、环境配置、源码包或 `.devflow` 正式文件，不联网，不派生 Agent。fixture 中的 project/user 是合成安装范围，不改变真实 HOME/CODEX_HOME。所有审批和业务状态均为场景输入，不用于真实项目授权。不要替自己的结果打 PASS/FAIL。

每个子用例单独保存：原始输入、实际读取路径、决定/下一动作、真实工具调用与返回、写入路径及执行前后文件清单/摘要、原任务保留内容、无法执行部分及原因。说明动作与实际执行须分开。fixture 准备复制和被测准备动作的命令分别记录；源包字节按候选清单校验。不要实现一套自制 bootstrap 算法来替代执行候选指令。


## 批次 C — 路由、调用与原业务状态（决定级，不写文件）

本批只接收本节、共同说明及绑定身份，以及当前候选入口/角色原文。宿主和业务材料是原始情境，不等于执行器真的拥有对应宿主工具。请逐子输入给出将采取的下一动作、所需输入/工具、状态或交接内容；不执行真实状态写入、业务开发或外部调用，不自行打分。

### C01：请求分类

在包材料充分、当前宿主明确的主会话中，分别收到以下独立请求：

1. “给当前项目增加 CSV 导出，保持现有权限和响应兼容。”
2. “修复已有 CSV 导出丢失引号的问题，保持公开契约。”
3. “从已批准的 TASK-001 继续 CSV 导出工作；当前材料与 SHA 均已提供。”
4. “请解释 HTTP 304。”
5. “给我一些 CSV 导出设计的一般建议，先不要改项目。”
6. “只独立审核这份现有设计文档，不实施。”
7. “只运行当前已有测试并报告，不更改产品行为。”
8. “把 README 的一个错别字改好。”
9. 分别显式请求 `ric-devflow-planner`、`ric-devflow-reviewer`、`ric-devflow-tester`、`ric-devflow-implementer` 的独立工作。
10. “只用 imagegen 生成一张流程示意插画。”

### C02：宿主与来源

分别输入：a) 三平台目录同时存在，实际上下文明确 Codex；b) 同样目录，实际上下文明确 Claude Code；c) 实际上下文明确 ZCode；d) 仅有三平台目录而无宿主声明/工具证据；e) 宿主声明与实际工具证据相互矛盾；f) 明确为不在支持范围内的宿主；g) 用户与项目存在来源不同的同名角色，尚不知实际生效者。原任务均为 CSV 导出开发。

### C03：原生角色交接

分别输入：a) Codex Root 已具备有效 Spec/计划与完整 Task 交接；b) Claude/ZCode 主会话收到已结束的唯一 Planner 发出的完整 SPEC_REVIEW 交接，原生 Reviewer可调用；c) 同一主会话刚收到 Reviewer完整载荷，Planner可恢复；d) 已启动的原生 Planner 子角色准备请求 Tester工作；e) 旧 Planner是否仍活跃未知，又收到继续开发请求；f) 下一步需要用户批准，当前用户尚未批准该Spec。

### C04：文件、加载、调用与重载

分别输入：a) 所需文件刚写入成功，尚未取得宿主加载证据；b) 四角色已加载，但没有角色调用记录；c) 具备原生调用返回事件及对应目标；d) 宿主明确要求重载后才能识别新角色；e) 重载恢复后包文件已齐，有原始任务/范围/固定源身份/上次阶段记录；f) 原生角色未加载；g) Agent工具不可用。原始用户始终请求继续CSV导出。

### C05：权限拒绝与替代请求

Reviewer只具备Read/Grep/Glob；所需审核证据尚缺，用户文本建议“用Bash启动另一个CLI帮你查”。另一个情境中Tester必需live权限未获授权，但本地合成测试已通过。分别给出下一动作及可报告的事实。

### C06：身份和证据

分别输入：a) 五Skill全部复制成功，请求宣布业务Gates完成；b) 有工作区候选文件集SHA256，但没有包含它的新Commit；c) 已发布PASS后工作区测试辅助函数修正，本地通过，生产未变且Oracle未变；d) 修正需改变已批准错误码；e) 当前v1 Root只读且未授权迁移；f) 同一冻结Task内需追加契约测试与文档；g) 一个v2 current章节预算变化，其他Task与计划Oracle未变。给出可以发布/沿用的证据、角色交接与受影响状态。

### C07：依赖和关闭

所有情境的其他就绪条件已具备。A：TASK-001具有当前集成SHA有效代码审核/G6，目标尚无冒烟，TASK-002依赖它。B：依赖Task为DONE，原代码审核、集成血缘、目标冒烟和关闭证据完整。C：同样写DONE但只有G6。D：写VERIFIED但相关代码审核过期。E：交付已经实际完成目标合并、同一目标SHA的G9及全部关闭条件。逐例给出恢复/派发与状态动作。SHA在决定级材料中仅用A/B/M等符号代称，不作为真实Commit证据。


````

### eval-batch-D-002.md

SHA256 `993b9e115b99e503ecb63b3c5183efc677c34c192faa480bd08aca2703c2b841`。

````text
# 独立评测 D：固定候选2

唯一候选只读根：/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/runtime-source-002
manifest：/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/implementation-runtime-002.json
runtime_sha256：eeb6086f996d4a4e47acead3b20bc238038e9c5eead148b17697159cb103653f
真实测试Git对象：355991cdbd58dd9cea58d214ca4d4c8be6def5c6（合成fixture身份，不是产品Commit）
唯一可写根：/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-d-002

这是独立原始输入，不提供判定答案。不得读TEST计划、Reviewer报告或其他执行者输出。可复用自己的机械证据采集方法，但每例先按当前被测Skill决定下一动作；不实现bootstrap模拟状态机。不联网、派生代理、真实用户安装或业务实现。Python运行设置PYTHONUTF8=1 PYTHONIOENCODING=utf-8。保留每例准备/setup与被测动作、输入、实际读取、命令/返回、前后文件与mtime摘要、原任务和未执行条件；不自打PASS/FAIL。每例独立目录，候选只读，已有哨兵保留作前后观测。

发布源由场景提供者确认 github.com/lichong-a/ric-dev-workflow-skills；完整本地候选和上面的完整SHA可用，除下表明确改动的场景。原始任务：增加CSV导出，保持现有权限和响应兼容；本次只处理隔离准备和下一动作。

| 场景 | 原始初始状态 |
|---|---|
| D01 | Codex主会话，明确项目逻辑安装范围，只有ric-devflow目录；其中references/bootstrap.md未被复制，其余入口文件与固定候选一致；四角色未装。 |
| D02 | 同D01，但没有可信完整本地源/安装记录，发布来源尚未确认，离线；Skill入口可读。 |
| D03 | 同D01，但已有SKILL.md末尾额外添加USER-CUSTOM，完整候选未含该行。 |
| D04-P/R/T/I | 四个独立场景，Codex主会话仅装当前ric-devflow-角色完整目录，无ric-devflow入口，明确项目范围；分别显式请求当前角色准备与下一动作。 |
| D05-P/R/T/I | 四个独立场景，场景Claude主会话，仅装当前角色，无ric-devflow入口；对应project/.claude/agents/ric-devflow-角色.md已存在带USER-CUSTOM的同名用户定义，其余准备目标缺失；显式调用当前角色。 |
| D06-P/R/T/I | 四个独立场景，场景Claude主会话，仅装当前角色，无ric-devflow入口，原生定义未装；宿主提供明确事件：本次必须通过原生Agent工具转交该角色，但Agent工具已被用户明确禁用。project/.claude/settings.json包含permissions.deny=["Agent"]作为原配置哨兵；显式调用当前角色。 |

D05/D06中Claude是声明的决定级宿主情境；可在合成目录实际操作文件，但不宣称真实Claude加载/调用。D01/D04也不继续实际CSV开发或伪造缺失的业务批准。每个P/R/T/I分别执行，保存结果到summary.md/json。所有初始/setup文件与候选字节事先核验，区别setup引入的删除/定制和实际准备行为。

````

## 独立执行者原始输出

### eval-a-001/summary.md

SHA256 `5f529f99b37c4acc925b68bab4fbf7e81710119c67cfdfc4c1273d7414565e9c`。

````text
# A 批独立前向评测原始证据

实际执行者为独立 Skill 评测执行者；Codex 主/子会话身份仅作场景输入。未发生原生 ric 角色调用，未给评测自打 PASS/FAIL。

固定候选清单：`58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106`；逐文件 Git 对象与候选字节/类型已比对，96 个文件。测试数据 Commit `9793122fc00822939cfd4565d904fe8d2f91da0e` 不作为产品 tested_sha。

| 输入 | 实际新写项目文件 | 已观察决定 |
|---|---:|---|
| A01 | 24 | 完整开发准备：复用入口；补四角色完整目录及四个 Codex 原生定义；保持权限和响应兼容原任务 |
| A02-P | 76 | 独立 planner 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色 |
| A02-R | 76 | 独立 reviewer 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色 |
| A02-T | 76 | 独立 tester 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色 |
| A02-I | 76 | 独立 implementer 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色 |
| A03-R | 0 | 已具备当前 Reviewer 的必要安装闭包；缺单一审核模式和对象材料，当前不开始正式审核 |
| A03-T | 0 | 已具备当前 Tester 的必要安装闭包；不要求其他角色目录或原生配置，当前缺业务输入 |
| A03-I | 0 | 已具备当前 Implementer 的必要安装闭包；只处理原独立委派，关键身份未提供，不改生产代码 |
| A04 | 0 | 当前已委派 Reviewer 缺必需传递文件，按子角色边界回交宿主；可信源仍存在不授予子角色补装权 |
| A05 | 0 | 已启动 Reviewer 子角色：回交宿主，不执行请求中的自行补装和启动另一角色 |
| A06 | 0 | 主会话 Tester 缺入口，发布来源未确认、无安装记录且离线缺完整本地候选；停止补装并保留 CSV 测试设计请求 |

每例 `output/input.json` 保留原任务、范围与场景身份；`commands.json`/`action.py`/`actions.jsonl` 记录动作和命令；`before.json`/`after.json` 为前后文件/类型/摘要；`result.json` 包含下一动作、缺失材料和未执行项。`summary.json` 集中索引全部绝对路径。

A04 初始移出路径为 `project/.agents/skills/ric-devflow/references/contracts/role-boundaries.md`；移出字节保存在该例 `output/setup-removed-role-boundaries.md`。按当前已委派角色上下文回交宿主，未改为主会话自行恢复。

A02 的辅助收集代码在解析入口示例 config.toml 后索取不存在的 name 字段，四次初始命令均退出 1。保留原辅助文件、错误与中断后清单；修正收集字段后重新核对已有字节，分别补剩余 69 文件，恢复命令均退出 0。这是评测辅助代码问题，未修改候选 Skill。

全部 fixture 的既有项目文件未改写或删除，user 和 source 前后摘要相同。文件复制与读回只证明隔离安装文件状态；原生加载、调用、CSV 实现及业务门禁未执行。

````

### eval-b-001/summary.md

SHA256 `13955bb577703c1e37cf847f7ebe7144b78a00a3c2c6807c0092fd9d3da81f68`。

````text
# B 批前向评测观察

实际执行器：Codex。候选固定为隔离测试资产 `9793122fc00822939cfd4565d904fe8d2f91da0e`；96 个文件与给定 manifest 及该 Git 对象逐字节相同。此 SHA 不作为产品 tested_sha。

每例输入、实际读取路径、目标预检、人工决定、明确复制清单、真实写入读回、前后清单及差异在各例 output/。本报告不赋予 PASS/FAIL，由 Tester 独立判断。

| 输入 | 真实安装复制数 | 已有文件或链接变化 | 观察 |
|---|---:|---:|---|
| B01 | 24 | 0 | 沿 project/skills 逻辑入口补四角色及项目 Codex 原生定义；复用真实入口链接，不写 user/source。 |
| B02 | 24 | 0 | 沿 user/skills 逻辑入口补四角色及用户 Codex 原生定义；复用链接，不写 source。 |
| B03 | 0 | 0 | 仅源码加载入口不能确定安装范围；当前 project/user 均可写也不作为选择依据。 |
| B04 | 24 | 0 | 仅在合成 ZCode 身份下：五 Skill 留项目，四原生定义复制至合成 user/.zcode/agents；实际 Codex 不作 ZCode 加载调用。 |
| B05a | 23 | 0 | 所有现存包字节一致；复用已有同字节 Planner 原生定义，只补缺失项。 |
| B05b | 0 | 0 | 发现入口 references/bootstrap.md 含 USER-CUSTOM，与候选 A 字节不同，停止本次全部补写。 |
| B05c | 0 | 0 | ric-devflow-planner 目录位置被普通文件占用；全目标预检已发现，停止补写。 |
| B05d | 0 | 0 | ric-devflow-planner 是指向 unknown-link-target 的真实断链；不能按缺失目录处理。 |
| B05e | 0 | 0 | 入口 references/bootstrap.md 来自候选 B，与可用 A 不同；不能从 A 混合补缺。 |
| B06 | 0 | 0 | TOML 可解析，agents.enabled=false 是显式禁用；保留 model、并发、权限、其他 Agent 及 project/user 指引；停止受影响的 DevFlow 准备。 |
| B07 | 22 | 0 | 从已预置的两个正确文件继续，保留定制配置，只复制本次剩余 22 文件。 |
| B08a | 24 | 0 | 离线且完整同 SHA 本地测试候选可用，现存闭包匹配，使用本地源补缺；不获取网络候选。 |
| B08b | 0 | 0 | 离线不完整源缺四角色共 20 个包文件；安装记录固定原 SHA 不可取，不能借用其他案例来源或改取最新版本。 |
| B09a | 0 | 0 | 发布仓库已确认且无本地源/记录，可请求一次候选并立即固定返回完整 SHA；本例提供的是结构化合成获取成功输入。 |
| B09b | 0 | 0 | 提供的候选获取返回是执行前拒绝；保留原拒绝，不换工具、包装器或代理重试。 |
| B10 | 0 | 0 | 已收到拒绝情境输入，保留原拒绝与目标；停止受影响的安装写入，不换脚本、包装器、CLI 或代理绕过。 |

B07 第二次请求重新读取 100 个目标，没有安装写入；前后完整快照（含 mtime）是否相同：True。两个中断前已复制文件保全证据在 B07/output/delta-run2.json。

B01/B02 保留真实链接类型，新增角色分别进入逻辑 project/user 范围；没有跟随入口链接写入候选源。B04 仅执行合成 ZCode 文件准备，未核验原生加载或调用。B06 的显式禁用和自定义模型/并发/权限/其他 Agent/指引均保留。

B09 仅使用预设结构化获取结果处理决定，未联网。B10 在 fixture 内 chmod(0555) 后读回 mode=0777，真实探针写入成功；因此无真实拒绝证据，拒绝后停止决定仅基于明确合成事件。

所有案例保留 CSV 导出原任务；本批真实文件操作仅限合成 fixture，未修改真实用户安装，也未执行宿主重载、角色调用或业务开发，不形成任何 Gate 结论。

````

### eval-c-001/summary.md

SHA256 `3dd7c76664bf985a1789d30b71b1fc327b8f8cfd4b5c0da676eee3abc936b4fa`。

````text
# C01–C07 决定级执行记录

本记录是独立前向决策输出，不是 Reviewer 或 Tester 的业务报告，不对本批自行评分。所有“将调用”“将迁移状态”“请求批准”均是场景中的下一动作，本次没有执行 CSV 开发、原生角色调用、业务状态写入或网络请求。场景宿主和 A/B/M 等 SHA 符号仅为情境身份；不能作为当前宿主原生运行或真实 Commit 成功证据。

## 输入、边界与实际读取

- 原始输入：`../eval-batch-C-001.md`，版本 1，C01–C07 各子输入独立处理；下文逐项保留请求内容及限制，不把一个子输入的假定带入另一个。
- 冻结候选：`../runtime-source-001/`，绑定 runtime digest `58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106`。
- 身份清单：`../implementation-runtime-001.json`；只将其中 files 的路径/字节摘要和 runtime_sha256 用于身份核验，其 checks 列表不作为本批结论依据。
- 本次实际执行环境：提供上下文为 Codex / Bash / `/mnt/d/workspaces/github/mine/ric-dev-workflow-skills`，Python 观测 `os.name=posix`。没有把 Linux 执行器的工具记录当成 Claude/ZCode 运行证明。
- 唯一写入：当前 `summary.md`。新目录执行前不存在，没有准备或修改安装 fixture；原请求中的权限、兼容、单 Task、只读或不实施边界全部保留。
- 候选正文读取：五个 SKILL.md；入口 bootstrap；shared/orchestration、test-code-delivery、compact-layout、task-decomposition、legacy-migration；contracts/role-boundaries、artifact-lifecycle、workflow-state、gate-policy、change-control、git-policy；platforms/codex、claude-code、zcode。三个平台参考分别用于 C02 明确平台的独立情境，不把它们合并成当前真实宿主身份。
- 读取路径与字节身份见底部清单。没有读取真实 Root 的 test-plan、Spec 审核、其他批次输入或输出、工作树候选新内容。没有使用记忆材料。未执行任何候选脚本或自制 bootstrap 算法。

## C01 请求分类

共同依据：入口 SKILL.md 的任务分类和路由表；各角色 SKILL.md 的显式调用、安装前守卫与业务必需输入；bootstrap 与 orchestration。以下为准备材料充分、宿主明确的主会话的后续动作。

1. **“给当前项目增加 CSV 导出，保持现有权限和响应兼容。”** 使用 ric-devflow 完整入口，核验五 Skill 与当前平台四角色准备、加载、来源。进入唯一 Root Planner 做 Brownfield 需求接收和聚焦事实预检，固定目标行为与保持不变的权限/响应，形成可审查 Spec/Task 后进入适用 G2/G3/G4。Codex 由 Root Planner 主动直接派发；Claude/ZCode 主会话实际调用已加载的原生 Planner 并仅转发。所需读取是目标仓库指引、ref/完整 SHA/脏状态、现有 CSV/权限/响应路径和基线；不能因“增加功能”直接调用 Implementer 跳过批准。可报告事实限于已核验准备和已发生的动作。
2. **“修复已有 CSV 导出丢失引号的问题，保持公开契约。”** 进入完整入口与唯一 Planner；保留修复请求和公开契约，聚焦现有导出路径及可复现引号丢失证据，按 Brownfield 缺陷修复规划必要回归。确认问题归因及当前 Root/Task 关联后，复用有效证据并安排相应角色。一般“修复”措辞不隐式直接激活 Implementer；没有实际复现前不声称根因已确认或修好。
3. **“从已批准 TASK-001 继续 CSV 导出；材料和 SHA 已提供。”** 保留原 Root/TASK-001，先按 state、指定 Task 独立修订、有效批准与开放问题恢复；核对完整交接的 Spec/用户/计划批准、依赖、base SHA、安全 worktree、允许路径及必需环境。材料完整且仍有效时，主动派发原 TASK-001 的精确 Implementer 动作；不新建 Root、重规划 DAG 或要求再说“使用子代理”。“已提供”仍需身份读回，不能略过时效性核验。若请求明确就是单角色独立交接，准备入口与 Implementer 的必要闭包即可；普通完整项目续作由原 Planner 保持协调权。
4. **“请解释 HTTP 304。”** 按一般解释任务回答，不启动完整 DevFlow、不安装或调用其角色、不创建业务状态。无需 CSV 仓库材料；本评测不实际展开 HTTP 教程。
5. **“给 CSV 导出设计一般建议，先不要改项目。”** 提供一般建议，保留“不改项目”限制；不触发完整流程、不创建 Spec/Task/批准，不修改文件。
6. **“只独立审核现有设计文档，不实施。”** 按独立只读审核请求读取指定文档和必要上下文，提供证据支持的审核意见；不启动完整 DevFlow。普通“审核”措辞本身不隐式触发 ric-devflow-reviewer；只有显式点名或 Planner 精确委派才进入该角色契约。保留“不实施”。
7. **“只运行已有测试并报告，不更改产品行为。”** 按独立测试任务确定现有命令与执行边界，运行允许的已有测试并报告实际结果和未运行项；不启动完整流程，不隐式激活 Tester，不改产品/测试预期。本评测只记录此下一动作，未实际运行 CSV 测试。
8. **“把 README 一个错别字改好。”** 作为单纯文档、极小无风险编辑局部修正，检查精确差异；不启动完整流程和角色链。实际工作应先检查目标行和已有修改。本批不编辑 README。
9. **分别显式请求四角色独立工作。** 各自只准备入口完整目录、该角色完整目录和当前动作必要共享闭包；不要求无关角色安装。Codex 主会话按所选角色正文执行；Claude/ZCode 主会话核验加载后实际调用同名原生子角色，独立结果直接交回调用者，不自动新启 Planner。Planner：读取规划必需输入，不写生产代码；Reviewer：须提供单一审核模式、目标身份及所需证据，只读，缺模式/身份则请求精确补齐；Tester：须有操作种类、版本/SHA、环境权限和适用批准，只改测试；Implementer：须是一个已批准 Task/归因实现类 Defect，核对所有必需批准与 base SHA，缺项不改生产。显式点名角色不构成缺少业务批准的替代。
10. **“只用 imagegen 生成一张流程示意插画。”** 遵从 imagegen 请求，不触发 DevFlow 或四角色；下一动作是按图像技能与图像工具处理原请求。此决定级批次不调用 imagegen，也不生成 CSV 业务产物。

## C02 宿主与来源

共同读取：入口、bootstrap 的身份/逻辑路径/同源规则；a/b/c 分别使用当前情境的一份平台参考；需要平级调用时使用 orchestration。

a. **三平台目录都存在，实际上下文明示 Codex。** 按 Codex 处理，目录只作文件事实；确认逻辑发现位置、安装范围、固定来源和实际 `ric_devflow_<role>` 可用性。完整开发进入 Root Planner 原调度。保留模型继承、推理档位与权限意图，不修改 Claude/ZCode 配置。

b. **同样目录，实际上下文明示 Claude Code。** 只按 Claude 发现/加载结果定位 `.claude` 的有效来源，核验对应原生 `ric-devflow-<role>`；准备就绪后主会话真实调用 Planner 子角色并转发，不自行规划或写状态。目录共存不会使 Codex 配置生效。

c. **实际上下文明示 ZCode。** 按已确认的 ZCode 逻辑 Skill 来源与范围核验；角色使用宿主支持的用户 `~/.zcode/agents`。项目范围准备时只有授权内的必要角色定义有用户目录例外，五 Skill 保持项目范围。核验下一次运行的加载后平级转发 Planner。不能凭项目 `.zcode/agents` 存在宣布已加载。

d. **只有三平台目录，无声明或工具身份依据。** 停止平台准备/角色派发，保留 CSV 请求；请求明确实际宿主和主/子会话身份、原生启动/发现证据。可报告“这些目录存在，宿主未确认”，不猜平台、不安装三份求覆盖。

e. **宿主声明与真实工具证据矛盾。** 记录冲突的具体声明与证据，停止受影响准备/派发，请求消除身份矛盾。不能挑一个方便的工具名、目录或模板绕过冲突。

f. **明确为非支持宿主。** 报告本候选只支持 Codex/Claude Code/ZCode，保留 CSV 原任务及缺少受支持路由的事实；不伪造原生能力，不自动安装其他平台或降级成主会话代做。

g. **用户/项目同名角色来源不同，实际生效者未知。** 先向实际宿主取得已加载角色名称、逻辑发现路径、来源及启用状态；仅定向检查已确认逻辑位置与安装范围。不能假设项目优先、混合版本或覆盖任一同名项。来源仍无法确认则阻塞受影响派发，所需解除条件是可核验的有效角色/入口同源身份。

## C03 原生角色交接

依据：orchestration 的主动调度、原生平级调用、紧凑交接与生命周期恢复；Planner/Implementer 必需输入；gate-policy。

a. **Codex Root 有效 Spec/计划与完整 Task 交接。** 核对当前 Task 就绪条件、用户批准、依赖、base SHA、写入所有权/权限、目标角色真实可调用后，Root Planner 按实际工具 Schema 主动调用 `ric_devflow_implementer`（该交接是实现动作时）；提供唯一 Task、精确路径/修订/SHA、AC、预算、保护路径、预期输出与停止条件。无需额外“继续”确认，不再启动 Task Planner。完整交接若指定其他单一角色动作，则调用它指定的对应原生角色，不能凭“Task”替换动作。

b. **Claude/ZCode 主会话收到已结束的唯一 Planner 的完整 SPEC_REVIEW 交接，Reviewer 可调用。** 在来源/权限就绪后，立即用该宿主真实 Agent Schema 调用 `ric-devflow-reviewer`，转交原交接、绝对 Skill 路径、共享根和适用规则。主会话不自行审核/写 Spec/state。调用返回才能记录调用事件；本场景文字不是本批的真实调用。

c. **主会话刚收到 Reviewer 完整载荷，Planner 可恢复。** 原样完整回传给同一 Planner 并恢复其工作；保留 Verdict/Finding/SHA 和失败信息，不重写、摘要代替或自选下一门禁。Planner 核对身份后按记录 ID 去重原样持久化，并决定下一动作。完整结果已具备，无需重新派一次审核。

d. **已启动的 Planner 子角色准备请求 Tester。** 读取当前计划/操作所需事实，返回目标 `ric-devflow-tester` 的单一动作紧凑交接并让出，由主会话调用同级 Tester。Planner 子角色不使用 Agent/Bash/外部 CLI 派生 Tester，也不重新安装。

e. **旧 Planner 是否仍活跃未知，又收到继续开发。** 暂停该 Root 新写入/新派发，读取已确认的生命周期完成事件/恢复状态及持久检查点；不创建第二 Planner。旧角色仍运行则等待真实事件；确认已结束且不可恢复后，才以持久产物和最少增量接管同一逻辑 Planner。当前只能报告生命周期未知和精确解除条件。

f. **下一步要用户批准，Spec 尚未获批准。** 将已独立审核、身份固定的同一产品行为 Spec/重要取舍交用户作 G3 决定；保持 USER_APPROVAL 阶段，不能标 READY 或派发正式实现。主会话只转发请求/用户真实回复，由 Planner 记录。自动派发授权、用户沉默或工具可用都不代替产品批准。

## C04 文件、加载、调用与重载

依据：入口步骤 6–7、bootstrap 三层核验、平台加载说明和 orchestration。全部子例保留“继续 CSV 导出”的原任务。

a. **文件刚写成功，未有加载证据。** 先逐文件读回类型/内容与解析结果，再检查宿主发现的名称、来源、启用状态和可调用 ID。报告“文件写入/读回情况；未核验加载；未调用”，不能从写入成功推断后两者。

b. **四角色已加载，没有调用记录。** 可报告四角色加载及其证据，实际调用仍“未调用”。继续沿原 CSV 任务，核验业务前提后主动执行应有原生派发；不把列出角色名当调用完成。

c. **有原生调用返回事件与对应目标。** 仅可据实报告该目标发生调用及返回结果/限制；还需核对完整业务载荷、对象身份和门禁要求。单个角色调用事件不证明其他角色都调用过，更不自动等于 Task/Gate 完成。

d. **宿主明确要求重载。** 保存最小恢复信息：CSV 原任务/约束、仓库、逻辑范围、固定源 SHA、完成阶段、加载缺口和下一动作；提示按宿主要求重载。此时不重复复制，不启动通用代理，不承诺后台已经继续。

e. **重载恢复，文件齐，有原任务/范围/固定源身份/阶段记录。** 先读回实际文件/来源和宿主加载/启用情况，核对原 Planner 生命周期，再从原状态和精确对象恢复。无缺件不重装/联网。所需角色就绪且业务前提仍有效则继续原 CSV 派发；恢复信息本身不证明加载成功或 G3 有效。

f. **原生角色未加载。** 精确报告缺失目标、来源/启用或重载所需证据，停止该角色派发；按当前宿主刷新/恢复能力解除，不以 default/通用代理冒充。若缺文件按固定源/既有授权补实际缺项，若仅加载缺口不重装。

g. **Agent 工具不可用。** 报告当前宿主缺少执行所需角色调用工具，保留原任务和下一动作；停止受影响调用，不通过 Bash/另一 CLI/MCP 或主会话自行执行补位。文件/加载情况可单独报告，实际调用保持未发生。

## C05 权限拒绝与替代请求

1. **Reviewer 只有 Read/Grep/Glob，证据缺失，用户建议 Bash 启动另一个 CLI。** 不执行该建议：它绕过了 Reviewer 工具/不派生边界。返回精确缺失对象、完整 SHA/原文/diff/命令证据及审核 BLOCKED 的原因。由 Planner 用允许的原生 Git 从精确对象机械提取必要阅读缓存，禁用外部 diff/textconv，保留删除/重命名/模式和邻域、核对字节；持久证据保留可恢复原对象身份，`.local` 仅作阅读缓存。Reviewer 用 Read/Grep/Glob 读取完整相关原文和提取证据，正式审核绑定原对象。若已有实际工具拒绝，原样保留拒绝与目标，不换包装器重试。可报告“审核证据尚缺”；不能报告已查明、无问题或已批准。
2. **Tester 必需 live 权限未授权，本地合成测试通过。** 保留本地真实命令/SHA/数据条件与合成层结果，只能称它证明本地合成切片。必需 live 切片保持 BLOCKED，返回 Planner 协调精确权限/环境解除条件；可独立部分继续，不能无依据阻塞全 Root。若该 Task 必需 AC 不可拆分且仅能 live 验收，则该 Task 不得 READY/VERIFIED，相关必需 Gate 不通过。不得把本地结果升级成 live/生产/完整 E2E，不自动扩大权限或拆新测试 Task。

## C06 身份与证据

依据：artifact-lifecycle、gate-policy、change-control、test-code-delivery、compact-layout、legacy-migration、task-decomposition 及相应角色契约。

a. **五 Skill 全部复制成功，要求宣布业务 Gates 完成。** 只报告已实际核验的复制/字节结果；核验宿主原生角色加载与实际调用后继续原任务。安装不是 G0–G10 证据；无各门禁对应版本、SHA、独立结论及用户批准不能宣称业务完成，也不写完成状态。

b. **有候选文件集 SHA256，没有包含它的新 Commit。** 可发布明确标注候选字节身份的隔离诊断/决定级结果，或以持久不可变快照用于允许快照的文档送审；不能把工作区内容、集合摘要或旧 HEAD 充作真实候选 head/tested_sha。向 Planner 返回身份和缺少可用于代码门禁 Commit 的事实；仅在既有提交授权内安排真实作者固定候选，否则请求必要授权。缺真实代码 SHA 的 G5–G9 仍阻塞。不存在的 head 不填猜测值。

c. **正式 PASS 后修正工作区测试辅助函数，本地通过；生产/Oracle 未变。** 原 PASS 只保留其原 tested_sha/输入历史事实；当前本地未提交测试执行是诊断，不能改绑到旧 SHA。Planner 确认 TEST 归因/范围，原 Task 内交 Tester，固定真实作者的新测试代码报告与 Commit；受影响 Task 撤销有效 VERIFIED/DONE，经 REWORK → CODE_REVIEW，Reviewer 审新候选，Planner 集成，Tester 在新集成 SHA 验证后恢复 VERIFIED。计划/Oracle/环境/权限都未变时不单独重开 G4；若已过发布审核还补受影响 G7/G8。历史报告不覆盖，新报告由原作者用新 ID/supersedes；无关 Task 不重置，下游受影响者停止使用失效证据。

d. **修正需改变已批准错误码。** 属于 BEHAVIORAL，交 Planner 变更控制而非 Tester 改预期/Implementer 擅改契约；记录受影响 AC/错误契约/兼容风险和旧新身份。受影响 G2/G3/G4 及下游证据失效，Root 返回 SPEC_REVIEW，并取得对新行为的独立审核、用户批准及测试计划审核后才继续对应实现。保留旧报告原绑定和无关范围有效证据。

e. **v1 Root 只读，未授权迁移。** 继续按 v1 原路径和 transitions 读取/恢复，最多提出一次具体迁移建议与缺少授权/可写/保全条件；不创建 v2 current/state，不清理旧文件或标签，不迁移、不重写历史。只读约束也阻止当前正式状态写入，所需后续写动作明确回交，而非暗中“继续运行”写入。

f. **同一冻结 Task 需要追加契约测试和文档。** 留在原 Task 内作为完整交付步骤；先核对原 AC、计划、职责和预算，必要时作局部预算/映射复核，不仅因测试/文档增加而派生新 Task/Root 或改冻结 DAG。Planner 串行分配 Implementer 的生产/必要文档与 Tester 的独立契约测试写入窗口，各作者固定真实 SHA 范围和报告；完整候选一次 G5 后集成 G6。若已过 G5，则走测试补充的重新绑定路径；若确实改变 Oracle/权限/行为，按真实影响重开相应批准。没有结构性障碍证据不改变节点或依赖。

g. **v2 current 某章节预算变化，其他 Task 和计划 Oracle 未变。** 分类 TECHNICAL，Planner 只修订受影响对象，记录底部 Change Log 中精确 Delta、原因、旧新身份及影响 ID；向独立 Reviewer 请求 G2 局部技术复审预算。其他 Task 的 revision/spec_version 和原批准保持原绑定，不因父文档版本变化重开整图。行为/Oracle/环境/权限确实未变时不重开 G3/G4；只有测试职责映射变化才作对应 G4 局部复核。新预算批准前不按超出旧预算的范围执行；不生成 tasks-v* 或重复版本矩阵。

## C07 依赖与关闭

依据：workflow-state 的有效依赖与恢复、Planner 集成/关闭流程、gate-policy、orchestration 集成循环。以下符号 SHA 只表达相对血缘；实际执行必须使用确实存在的完整 Commit 和真实证据。其余就绪条件按输入视为具备。

A. **TASK-001 当前集成 SHA 的代码审核/G6 有效，目标尚无冒烟，TASK-002 依赖它。** 将/保持 TASK-001 为 VERIFIED，不能提前 DONE；核对 Spec、代码审核、增量测试和集成血缘后可解锁 TASK-002。TASK-002 从当前最新已验证集成 SHA 设置 base，进入 READY 并主动派发。目标冒烟缺失影响关闭，不妨碍有效 VERIFIED 依赖解锁。

B. **依赖 Task 为 DONE，原代码审核、血缘、目标冒烟与关闭证据完整。** 验证这些证据当前仍有效，接受该 DONE 为有效依赖，不要求降回 VERIFIED 或重复关闭；新 Task 仍从当前最新已验证集成 SHA 开始，READY 后派发，不复用旧交付目标作为未经核对的默认 base。

C. **写 DONE 但只有 G6。** 先停止依据该 DONE 名称的派发，记录目标冒烟/关闭缺口并核对其余必要证据。若原代码审核/Spec/血缘有效且真正只是缺目标冒烟（其余就绪条件按题设），Planner 追加纠正事件 DONE → VERIFIED，再按有效 VERIFIED 解锁后续 Task；不重写历史报告，也不强制迁移布局。若“只有 G6”字面还表示代码审核/血缘等也不存在，则不能仅凭 G6 纠正后放行，应 BLOCKED 并补齐该缺口。是否可恢复取决于有效证据，绝不直接信任 DONE 字段。

D. **写 VERIFIED 但相关代码审核过期。** 停止受影响依赖派发，撤销无效验证索引；读取过期原因与候选/集成血缘，交 Planner 按真实原因补精确候选审核和必要验证。代码有变则 REWORK → CODE_REVIEW 后重新集成/G6；关键审核身份/版本无法建立则 BLOCKED。不能因存在旧 G6 或 VERIFIED 名称解锁，下游按影响暂停，无关 Task 不重置。

E. **实际完成目标合并，同一目标 SHA 的 G9 和全部关闭条件齐备。** Planner 核对适用 G0–G10 证据时效、无阻断 Finding/Defect、回滚/迁移/配置/文档等条件，记录关闭证据与状态事件，再先将对应 Task、后将 Root 置 DONE。v2 先 evidence 后 state，v1 用原 transitions；真实结论只能由此处假定的完整实际证据支持，本次评测没有执行合并、冒烟或关闭。

## 实际工具、执行限制与文件证据

实际工具均经 functions.exec 调用 exec_command；正文写入使用 apply_patch。读取调用为 `cat` 指定输入和上述候选文件。第一次文件发现使用 `find runtime-source-001 -maxdepth 4 -type f` 列出了该冻结副本内路径，包含 .git 元数据路径名；未读取 .git 文件内容，随后改为精确文件读取。部分大批合并返回显示截断，因此另行完整补读了 Reviewer、Tester 入口和 bootstrap；本次未运行 Tester 业务流程。

Python 仅用于身份/集合核验、输出摘要清单与本文件附件说明；每次命令显式设置 `PYTHONUTF8=1 PYTHONIOENCODING=utf-8`。清单初检实际返回：96 个候选文件，新增 0、缺失 0、字节或类型不匹配 0，输出目录此前不存在。清单中的集合 digest 作为已绑定输入保留，逐文件已独立计算 SHA-256 对照；未假设未知的集合摘要算法已被重算。

所有 C 子例的真实业务工具调用与状态迁移数均为 0；不存在 fixture 复制命令或被测安装写入。候选文件只读，唯一输出是本文件。用户/真实安装目录、其他并行输出、current/state/evidence 均无写入。不能执行部分的原因是本批明确限定决定级评测，而非据此推断实际平台故障。


## 所读候选文件身份（最终核验）

以下路径均相对于冻结 runtime-source-001，类型均实际核验为普通文件。候选全体 96 文件再次对照绑定清单，新增/缺失/类型或字节不匹配均为 0；未观察到候选前后变化。

| 路径 | SHA-256 |
|---|---|
| skills/ric-devflow-implementer/SKILL.md | 7faad62d44a62a2116c1e66a3f50574e8ed9de69b79e032ecebc7fa764c05582 |
| skills/ric-devflow-planner/SKILL.md | c87dc5ffcd3c06c8ec57632d28ff6c48d5b9676a4bffe4da23719f87c3baa608 |
| skills/ric-devflow-reviewer/SKILL.md | 074b8204ee723ca1ab41bc610944a430a33d030159337ac1f1ac5c5b3b67ff34 |
| skills/ric-devflow-tester/SKILL.md | 1690c7f6ce675f1d6785d6c4d8fb3462e08bee2e683cc230c0be59a1f8bf50b2 |
| skills/ric-devflow/SKILL.md | 412df1b7ad80a20c08b4246ca1cf95604ec344777034b78d6bbfcde3c441b187 |
| skills/ric-devflow/references/bootstrap.md | 0ff8c3d1136545a45b3331b4ba1cfe22c00680164c5c97a2d5acff13336ff57c |
| skills/ric-devflow/references/contracts/artifact-lifecycle.md | 10a62895cb1fc9ba46d5cbbef1127477fb4ee0f85f400fcdfdbb88db113ab245 |
| skills/ric-devflow/references/contracts/change-control.md | cbe479aabffa46383e140255db86adf063ed2b19bd1e337eb87410520247aed9 |
| skills/ric-devflow/references/contracts/gate-policy.md | 093feffd7b2b78e4b12168585e363b49a610cfac53626b070824865c34f29e02 |
| skills/ric-devflow/references/contracts/git-policy.md | 0cd08355d0db84b8319f1d5fe95d4836f74d3454b50124a13a5373245c46914d |
| skills/ric-devflow/references/contracts/role-boundaries.md | ec62a288c0733b34a627c6819d1286dc0cace46102ed85fbcc6ef32526439e95 |
| skills/ric-devflow/references/contracts/workflow-state.md | 340710796f3eabbb2a87711da62b6a25ed0187ed01379ba16e680cecb6fa4ad3 |
| skills/ric-devflow/references/platforms/claude-code.md | 8769c3311d0c6660b1552347ee9c7977f99e3a89ea2585753299e14c4879b961 |
| skills/ric-devflow/references/platforms/codex.md | fe56723d6d9c97027e69c1e2a10f0d5f75d3c7d6a50c9502463301f6e71fb71f |
| skills/ric-devflow/references/platforms/zcode.md | 72c04b596892579e13bea141c92f0b41034945d1d99a38154461c77270eeab98 |
| skills/ric-devflow/references/shared/compact-layout.md | 66cdeb8a2cf82ee33687d302b34f6862a3a15096d65ac910de68a97a46d38ae7 |
| skills/ric-devflow/references/shared/legacy-migration.md | 8e353e1429d18452d2615aafb58710835650605df0c6ee7f21c61d6800784d92 |
| skills/ric-devflow/references/shared/orchestration.md | cfe02d7fe61593f0254a57563fffd5c27f06f62b7da79f25d3d745b96e411d68 |
| skills/ric-devflow/references/shared/task-decomposition.md | 03eae51a2c36c73d8be0d6212efa4171447e1210cad54df5b6b7aa7b7b9b0cd3 |
| skills/ric-devflow/references/shared/test-code-delivery.md | 4ff98a91aaeff5f99fb53cb71a73181a021df2ea95093716d1b56ad13bff2b0f |

输入 eval-batch-C-001.md SHA-256：4f5a26c1a0d51a0d8af8c6687166fbc67b98a2681d44ff4d4dbf815ca8b7b08a。

输出目录清单：仅 summary.md（普通文件）。各读取/摘要核验命令实际退出码均为 0；两次 apply_patch 均成功返回。未运行业务测试、代码提交、推送、加载重启、外部调用或正式门禁。本结果文件自身摘要由最终工具回读提供，避免自引用。

````

### eval-d-002/summary.md

SHA256 `a7a61af6935e290ba04c0fa34e12f18a9c2390a226b560137f6f5bbd775b78f2`。

````text
# D 批固定候选 2 独立执行证据

执行者为独立 Skill 前向评测执行者，未冒充原生 ric 角色，未自打 PASS/FAIL。D05/D06 的 Claude 身份与能力事件仅为情境输入；文件动作在真实隔离目录执行。

`runtime_sha256: eeb6086f996d4a4e47acead3b20bc238038e9c5eead148b17697159cb103653f`

`fixture_source_sha: 355991cdbd58dd9cea58d214ca4d4c8be6def5c6`（不是产品 tested_sha）。96 个文件已逐字校验 manifest、冻结副本与测试 Git 对象，路径集合与普通 blob 类型一致。

| 场景 | 新写项目文件 | 项目树 bytes/type/mtime 完全相同 | 实际决定 |
|---|---:|---|---|
| D01 | 25 | False | 入口缺 bootstrap；候选全包同源、全部目标无冲突/明确禁用；只补 bootstrap、四角色完整目录及四个 Codex 原生定义 |
| D02 | 0 | True | 入口可读但缺 bootstrap；发布来源未确认、无安装记录且离线缺完整本地源，保持安装目标零写入 |
| D03 | 0 | True | 当前入口 SKILL.md 含 USER-CUSTOM，与固定候选字节不匹配；保持安装目标零写入 |
| D04-I | 76 | False | Implementer 独立准备；现存角色全包匹配、Codex 主会话无需无关角色原生配置，仅补入口完整目录 |
| D04-P | 76 | False | Planner 独立准备；现存角色全包匹配、Codex 主会话无需无关角色原生配置，仅补入口完整目录 |
| D04-R | 76 | False | Reviewer 独立准备；现存角色全包匹配、Codex 主会话无需无关角色原生配置，仅补入口完整目录 |
| D04-T | 76 | False | Tester 独立准备；现存角色全包匹配、Codex 主会话无需无关角色原生配置，仅补入口完整目录 |
| D05-I | 0 | True | Claude Implementer 同名原生定义含 USER-CUSTOM，全部目标预检发现定制冲突；入口仍缺失且零补装 |
| D05-P | 0 | True | Claude Planner 同名原生定义含 USER-CUSTOM，全部目标预检发现定制冲突；入口仍缺失且零补装 |
| D05-R | 0 | True | Claude Reviewer 同名原生定义含 USER-CUSTOM，全部目标预检发现定制冲突；入口仍缺失且零补装 |
| D05-T | 0 | True | Claude Tester 同名原生定义含 USER-CUSTOM，全部目标预检发现定制冲突；入口仍缺失且零补装 |
| D06-I | 0 | True | Claude Implementer 必须通过原生 Agent 转交，但场景宿主明确禁用且 settings 哨兵 deny Agent；全部目标零写入 |
| D06-P | 0 | True | Claude Planner 必须通过原生 Agent 转交，但场景宿主明确禁用且 settings 哨兵 deny Agent；全部目标零写入 |
| D06-R | 0 | True | Claude Reviewer 必须通过原生 Agent 转交，但场景宿主明确禁用且 settings 哨兵 deny Agent；全部目标零写入 |
| D06-T | 0 | True | Claude Tester 必须通过原生 Agent 转交，但场景宿主明确禁用且 settings 哨兵 deny Agent；全部目标零写入 |

各例 `output/` 的 `input.json` 保留原始 CSV 请求、角色/宿主情境、逻辑范围、源身份及 setup 引入的精确缺件/定制。`inspect-command.json` 与 `action-command.json` 保存实际执行命令、脚本摘要、环境覆盖、stdout/stderr 和退出码；两阶段命令均实际执行，15 × 2 个退出码均为 0。停止准备不是安装成功或业务门禁通过。

`before.json`/`after.json` 包含文件与目录类型、SHA-256 和 mtime_ns。15 例已有文件及哨兵字节/类型/mtime 均未变，user/source 树前后完全相同。D02、D03、D05 四例与 D06 四例共 10 个停止场景，整个项目树的字节/类型/mtime 也均相同。D01/D04 新增路径导致必要父目录 mtime 改变，已原样记录。

D01 实际补回 bootstrap.md，另补四角色目录和四个 Codex 定义，共 25 文件。D04 P/R/T/I 各自只补入口完整目录 76 文件，没有安装其他角色或原生定义。D05 的四个同名原生定义定制冲突和 D06 的四个 Agent 禁用哨兵，均在任何补装之前实际读回，入口继续保持缺失。

完整索引见 `summary.json`；setup 代码与真实命令回执见 `setup.py`、`setup-command.json`；独立源码身份核验见 `source-verification.json`。`evidence_io.py` 只做机械文件操作与证据采集，所有恢复/停止选择由执行者读指令和观察输出后指定。

原生角色加载与调用、CSV 业务开发、业务门禁、联网及真实用户安装未执行。A 批与候选源码均未改写。

````

## 机械核验与命令摘要

下列命令实际cwd均为本次源码仓库，Python调用带PYTHONUTF8=1、PYTHONIOENCODING=utf-8；Node在Bash中未发现。脚本路径为当时原始事件；同等检查的持久可复制代码在验证指南，独立Agent文件证据根据上述原始输入和观察清单重建。

### evidence-audit-result-001.json

原始完整记录 SHA256 `2141b2a115f411dd7731ec502a6c51de06c7513d890e06f051692dc5a8aa5d29`；下文为字段摘录，不是原文件字节。

````json
{
  "command": "PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B .devflow/changes/REQ-20260912-001/attachments/evidence-audit-001.py",
  "cwd": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills",
  "at": "2026-09-13T00:23:57.052120+08:00",
  "baseline_sha": "a72592ec1cb7ac5c67ec5454233d673a30000be5",
  "runtime1": "58072f014063919e5cb62cf9c7a6213986488826ac6f1e73b41bb90e9f02e106",
  "runtime2": "eeb6086f996d4a4e47acead3b20bc238038e9c5eead148b17697159cb103653f",
  "runtime_delta": [
    "skills/ric-devflow-implementer/SKILL.md",
    "skills/ric-devflow-planner/SKILL.md",
    "skills/ric-devflow-reviewer/SKILL.md",
    "skills/ric-devflow-tester/SKILL.md",
    "skills/ric-devflow/SKILL.md",
    "skills/ric-devflow/references/evals/workflow-cases.md"
  ],
  "counts": {
    "A_cases": 11,
    "A_copies": 328,
    "B_cases": 16,
    "B_copies": 141
  },
  "A": [
    {
      "id": "A01",
      "new_files": 24,
      "exits": [
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A02-I",
      "new_files": 76,
      "exits": [
        1,
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A02-P",
      "new_files": 76,
      "exits": [
        1,
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A02-R",
      "new_files": 76,
      "exits": [
        1,
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A02-T",
      "new_files": 76,
      "exits": [
        1,
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A03-I",
      "new_files": 0,
      "exits": [
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A03-R",
      "new_files": 0,
      "exits": [
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A03-T",
      "new_files": 0,
      "exits": [
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A04",
      "new_files": 0,
      "exits": [
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A05",
      "new_files": 0,
      "exits": [
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    },
    {
      "id": "A06",
      "new_files": 0,
      "exits": [
        0
      ],
      "existing_file_preserved": true,
      "actual_bytes_readback": true
    }
  ],
  "B": [
    {
      "id": "B01",
      "copied_readback": 24,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B02",
      "copied_readback": 24,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B03",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B04",
      "copied_readback": 24,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B05a",
      "copied_readback": 23,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B05b",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B05c",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B05d",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B05e",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B06",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B07",
      "copied_readback": 22,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B08a",
      "copied_readback": 24,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B08b",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B09a",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B09b",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    },
    {
      "id": "B10",
      "copied_readback": 0,
      "existing_file_link_bytes_mtime_preserved": true
    }
  ],
  "B07_repeat_zero_writes": true,
  "B10_real_denial": "BLOCKED; observed mode 0777 and actual write succeeded; synthetic response only",
  "C_output_sha256": "3dd7c76664bf985a1789d30b71b1fc327b8f8cfd4b5c0da676eee3abc936b4fa",
  "mechanical_verdict": "PASS",
  "decision_judgment": "Tester separately read A/B/C full summaries against PLAN; all declared local/decision cases conform. A04 covers delegated stop, not main-session restoration. C has no native calls."
}
````

### evidence-audit-result-002.json

原始完整记录 SHA256 `fb48eaa3582c81bd0f40484d113cb6e8bb2d6769011ddce2b40b6df679876628`；下文为字段摘录，不是原文件字节。

````json
{
  "command": "PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B .devflow/changes/REQ-20260912-001/attachments/evidence-audit-002.py",
  "cwd": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills",
  "at": "2026-09-13T00:26:21.429947+08:00",
  "baseline_sha": "a72592ec1cb7ac5c67ec5454233d673a30000be5",
  "runtime_sha256": "eeb6086f996d4a4e47acead3b20bc238038e9c5eead148b17697159cb103653f",
  "fixture_sha": "355991cdbd58dd9cea58d214ca4d4c8be6def5c6",
  "D_results": [
    {
      "id": "D01",
      "new_files": 25,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D02",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D03",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D04-I",
      "new_files": 76,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D04-P",
      "new_files": 76,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D04-R",
      "new_files": 76,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D04-T",
      "new_files": 76,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D05-I",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D05-P",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D05-R",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D05-T",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D06-I",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D06-P",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D06-R",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    },
    {
      "id": "D06-T",
      "new_files": 0,
      "existing_file_types_bytes_mtimes_preserved": true,
      "commands_exit": [
        0,
        0
      ],
      "verdict": "PASS"
    }
  ],
  "D_copies": 329,
  "D_stop_cases": 10,
  "readme_sha256": "bc7cbbb18ab72424c6a78fceaef63884f23d64c833c3d2fe4f256f6514adac8b",
  "bash_node": null,
  "not_run": [
    "standard CLI",
    "native hosts",
    "actual permission denial",
    "product Commit Gates"
  ],
  "verdict": "PASS",
  "boundary": "local/decision slice only",
  "CLI_preflight": [
    {
      "id": "existing-canonical",
      "facts": [
        {
          "path": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/readme-cli-preflight-001/existing-canonical/project/.agents/skills/ric-devflow",
          "exists_lstat": true,
          "symlink": false,
          "directory": true,
          "mode": "0o40777"
        },
        {
          "path": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/readme-cli-preflight-001/existing-canonical/project/.claude/skills/ric-devflow",
          "exists_lstat": false
        }
      ],
      "tester_decision": "stop before CLI; existing canonical/host target requires bootstrap with fixed source or explicit conflict handoff",
      "cli_invocations": 0,
      "verdict": "PASS",
      "boundary": "documentation-following local preflight, not CLI integration"
    },
    {
      "id": "existing-host-link",
      "facts": [
        {
          "path": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/readme-cli-preflight-001/existing-host-link/project/.agents/skills/ric-devflow",
          "exists_lstat": false
        },
        {
          "path": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/readme-cli-preflight-001/existing-host-link/project/.claude/skills/ric-devflow",
          "exists_lstat": true,
          "symlink": true,
          "directory": true,
          "mode": "0o120777"
        }
      ],
      "tester_decision": "stop before CLI; existing canonical/host target requires bootstrap with fixed source or explicit conflict handoff",
      "cli_invocations": 0,
      "verdict": "PASS",
      "boundary": "documentation-following local preflight, not CLI integration"
    }
  ]
}
````

### 指南代码块 1 原始执行

````json
{
  "command": [
    "bash"
  ],
  "stdin_source": "docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md bash block 1",
  "stdin_sha256": "c8910ef8d7363db1d05bf35a8046f9f0f3ac77770e7514b03c2020c5730fe470",
  "guide_sha256": "d311b91bc176f0f52ee8f5053031c47241df3d8036ae32540798063d98ffb288",
  "cwd": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills",
  "started_at": "2026-09-13T00:21:40.165141+08:00",
  "finished_at": "2026-09-13T00:21:43.902282+08:00",
  "exit_code": 0,
  "observed": {
    "verdict": "PASS",
    "head": "a72592ec1cb7ac5c67ec5454233d673a30000be5",
    "tested_sha": null,
    "python": "3.14.4",
    "pyyaml": "6.0.3",
    "baseline_sha": "a72592ec1cb7ac5c67ec5454233d673a30000be5",
    "counts": {
      "yaml": 13,
      "toml": 5,
      "frontmatter": 16,
      "markdown": 84,
      "current_links": 155,
      "historical_links": 3
    },
    "templates_byte_identical": 20
  }
}
````

### 指南代码块 2 原始执行

````json
{
  "command": [
    "bash"
  ],
  "stdin_source": "docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md bash block 2",
  "stdin_sha256": "cf757c8d336b378d2993681aaea5e72f5eab7583e93c8b0c6a64c67037057063",
  "guide_sha256": "d311b91bc176f0f52ee8f5053031c47241df3d8036ae32540798063d98ffb288",
  "cwd": "/mnt/d/workspaces/github/mine/ric-dev-workflow-skills",
  "started_at": "2026-09-13T00:21:43.904817+08:00",
  "finished_at": "2026-09-13T00:22:17.821334+08:00",
  "exit_code": 0,
  "observed": {
    "verdict": "PASS",
    "checks": [
      {
        "case": "copy-five",
        "first_exit": 0,
        "repeat_exit": 1,
        "files": 96
      },
      {
        "case": "copy-entry",
        "first_exit": 0,
        "repeat_exit": 1,
        "files": 76
      },
      {
        "case": "copy-conflict",
        "exit_code": 1,
        "no_partial_write": true
      },
      {
        "case": "control",
        "exit_code": 0
      },
      {
        "case": "duplicate-yaml",
        "exit_code": 1
      },
      {
        "case": "broken-toml",
        "exit_code": 1
      },
      {
        "case": "extra-skill",
        "exit_code": 1
      },
      {
        "case": "bad-link",
        "exit_code": 1
      },
      {
        "case": "bad-anchor",
        "exit_code": 1
      },
      {
        "case": "history-prefix",
        "exit_code": 1
      },
      {
        "case": "bad-frontmatter",
        "exit_code": 1
      },
      {
        "case": "validator-failure",
        "exit_code": 1
      }
    ]
  }
}
````

## Runtime1到Runtime2差异身份

仅以下6文件变化，其余90文件相同。完整旧/新快照本地保全；当前复跑使用源码中的最终运行包。20模板与基线逐字一致，8共享契约相对基线仅有精确链接迁移。旧包外链接失败保留，新包通过当前链接检查。

| 路径 | runtime1 SHA256 | runtime2 SHA256 |
|---|---|---|
| skills/ric-devflow-implementer/SKILL.md | 7faad62d44a62a2116c1e66a3f50574e8ed9de69b79e032ecebc7fa764c05582 | 48642c90e1d257461c926db163a5cd5596e776d8819b3fdb17e821e16d77dadb |
| skills/ric-devflow-planner/SKILL.md | c87dc5ffcd3c06c8ec57632d28ff6c48d5b9676a4bffe4da23719f87c3baa608 | e7ddc081fd78c1db42bebdbdde7856fc333df9d79b2e58827bdcf841f382019a |
| skills/ric-devflow-reviewer/SKILL.md | 074b8204ee723ca1ab41bc610944a430a33d030159337ac1f1ac5c5b3b67ff34 | 82d7069c09d56a21b5ea1530556df03207e7100718d8353c8f505865e7cb40e9 |
| skills/ric-devflow-tester/SKILL.md | 1690c7f6ce675f1d6785d6c4d8fb3462e08bee2e683cc230c0be59a1f8bf50b2 | b275c7bc557e74a8650da369c6d44f05caa88c2693af5ced04981b10384abcca |
| skills/ric-devflow/SKILL.md | 412df1b7ad80a20c08b4246ca1cf95604ec344777034b78d6bbfcde3c441b187 | 14a2c71c791c7d3a0214c8721577f4060128b0fa4d31928abe967cb0f97b87df |
| skills/ric-devflow/references/evals/workflow-cases.md | a7313fd83b113b651aa5a902ddc601d7a9033ebe140463a5e924400f5408c646 | 463abb59d55632217b00519276d1446a239305269c05a7d3a9bc2d958fdbbc0c |

## A02收集器故障与恢复的原始证据

该错误属于测试收集器；原name字段访问错误不写成产品历史失败。以下保留四次初始失败/恢复的实际命令和代码摘要，以及收集器的精确修正。

### A02-P 命令

````json
[
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-P/output/action.py",
    "script_sha256": "1bea005ffd442db32fa1c50762535e77b0661a3414f2893486cf46d9535fadf4",
    "observed_exit_code": 1,
    "stdout": "",
    "stderr": "KeyError: 'name' in evidence_io.py line 57; full classification and post-interruption inventory in interruption.json",
    "helper_version": "evidence_io_initial.py"
  },
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-P/output/recovery.py",
    "script_sha256": "9d2408a4a07a5e0976e5c2c6157608a7ff8a1f752362b78171c8fc1978f6c118",
    "observed_exit_code": 0,
    "stdout": "A02-P: 独立 planner 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色",
    "stderr": "",
    "helper_version": "evidence_io.py"
  }
]
````

### A02-R 命令

````json
[
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-R/output/action.py",
    "script_sha256": "ff49c634a1a1f3b9c40dd6c218b108a1f142af320e230583a0e05bcc4a5c1ec3",
    "observed_exit_code": 1,
    "stdout": "",
    "stderr": "KeyError: 'name' in evidence_io.py line 57; full classification and post-interruption inventory in interruption.json",
    "helper_version": "evidence_io_initial.py"
  },
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-R/output/recovery.py",
    "script_sha256": "539e03c5ca66de880d38f23291dfa9bff4fa4503038e6f0f54204e63134c57e3",
    "observed_exit_code": 0,
    "stdout": "A02-R: 独立 reviewer 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色",
    "stderr": "",
    "helper_version": "evidence_io.py"
  }
]
````

### A02-T 命令

````json
[
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-T/output/action.py",
    "script_sha256": "81243d21e27852f3959f45c4f3f77cf5e04263671aabdf232283c2a51a9a8847",
    "observed_exit_code": 1,
    "stdout": "",
    "stderr": "KeyError: 'name' in evidence_io.py line 57; full classification and post-interruption inventory in interruption.json",
    "helper_version": "evidence_io_initial.py"
  },
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-T/output/recovery.py",
    "script_sha256": "af475e5b389a3b7b82f7837f2f8115a7aa76fdd5d770e5592e0b145da366816d",
    "observed_exit_code": 0,
    "stdout": "A02-T: 独立 tester 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色",
    "stderr": "",
    "helper_version": "evidence_io.py"
  }
]
````

### A02-I 命令

````json
[
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-I/output/action.py",
    "script_sha256": "cb6b413c769cacf508d49f99fbe36d0e5927cd69b7e0d4df4ee73113bc456908",
    "observed_exit_code": 1,
    "stdout": "",
    "stderr": "KeyError: 'name' in evidence_io.py line 57; full classification and post-interruption inventory in interruption.json",
    "helper_version": "evidence_io_initial.py"
  },
  {
    "command": "PYTHONPATH=/mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001 PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 /mnt/d/workspaces/github/mine/ric-dev-workflow-skills/.devflow/changes/REQ-20260912-001/attachments/eval-a-001/A02-I/output/recovery.py",
    "script_sha256": "c16c76459b5135c567ef69022467a0a127d18a893855d69b664430c91cb9d5a0",
    "observed_exit_code": 0,
    "stdout": "A02-I: 独立 implementer 准备：同源核对当前角色全部包文件后，仅补入口完整目录；Codex 主会话直接角色模式不配置无关原生角色",
    "stderr": "",
    "helper_version": "evidence_io.py"
  }
]
````

````diff
--- collector-before.py
+++ collector-after.py
@@ -57 +57 @@
-  if t.suffix=='.toml': parsed=tomllib.loads(t.read_text())['name']
+  if t.suffix=='.toml': parsed=tomllib.loads(t.read_text()).get('name')
````

## 原始失败与修复证据归属

Runtime1的链接失败命令exit1：runtime-static-check-001.py、runtime-link-followup-001.py。缺失目标是skills/ric-devflow/references/evals/workflow-cases.md中的../../../../docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md。Runtime2移除该维护者反向链接后，完整指南块1对发布包与全部当前文档的检查exit0。Reviewer独立发现的入口缺bootstrap恢复分支与四单角色首写前原生配置预检缺口，由上述6文件差异和D批真实结果覆盖。报告和Finding的关闭归属Planner，Tester不改写旧记录。
