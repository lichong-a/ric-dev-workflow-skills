# Compact v2：四文件与精确对象

新 Root Issue 的默认存储契约。不是独立 Skill，也不是另一个流程。G0–G10、Verdict、状态和原报告载荷继续使用既有契约。

## 固定布局与所有权

```text
.devflow/changes/<REQ-ID>/
  state.yaml
  current.md
  test-plan.md
  evidence.md
  attachments/   # 必要持久附件、正式边界快照、一次性迁移索引
  .local/        # 可丢弃过程材料、本机会话状态
```

按阶段创建，不为了凑文件数创建空计划/空报告。四个核心文件是整个交付的默认上限布局，不是把必要附件藏入临时目录的限制。

- Planner 写 `state.yaml`、`current.md`；后者在现有章节中容纳 Intake、Root、Repository Profile、Continuation Assessment、Spec/UI、Decision/Risk Acceptance、预算与当前阶段 Task。逻辑内容仍须完整，旧模板可作字段检查表，不再实例化为一堆文件。
- Tester 独立写 `test-plan.md`，包含 AC 映射、确定性步骤/预期、环境权限、测试职责、退出和未运行条件。
- 各角色是各自证据的唯一作者；Planner 是 `evidence.md` 的唯一串行持久化写入者，逐字转录原始载荷。Reviewer 保持只读；Tester/Implementer 返回报告，不直接并发编辑账本。
- `state.yaml` 只索引当前阶段、Task 状态、有效证据和开放问题。Task 正文不重复存入状态，完整 Review、迁移索引和历史 transitions 不塞进 state。

模板：[State](../../assets/templates/compact/state.yaml)、[Current](../../assets/templates/compact/current.md)、[Test Plan](../../assets/templates/compact/test-plan.md)、[Evidence](../../assets/templates/compact/evidence.md)。v2 容器以 `schema_version: 2` 标记；嵌入的原 Review/Test/Implementation/Defect 载荷仍用原模板与 `schema_version: 1`，保留所有字段和未知扩展，不把容器版本写入原报告。

## 正文、修订和寻址

`current.md` 与 `test-plan.md` 原地维护当前有效正文及明确标记的草稿增量。用稳定 ID/显式锚点定位 Spec、Task、Decision、Test Case。每个 Task 有自己的 `revision`；父文档的编辑不能令未变化 Task 全部重新发布。Task 的 AC、依赖、base 策略、允许/保护路径、交付物、验证与风险必须齐全；动态状态以 state 为准。

文件底部 Change Log 记录：修订、带时区时间、作者、`EDITORIAL/TECHNICAL/BEHAVIORAL`、受影响 ID、原因、证据。记录每次正式修订的变化，不复制旧全文、不把每次 commentary 当修订。全 Program 的能力、依赖、延期、最终完成条件保留，只展开当前阶段 Task 与测试细节。

一次正式交接使用：
```text
REQ-ID；角色动作/模式；
对象 {id, revision, path, commit_sha}；
代码 {base_sha, head_sha, tested_sha}（仅适用字段）；
原 Review/Finding/Defect ID 与精确 Delta；
允许/保护路径；输出；首个检查点；完成/停止条件与预算。
```
`path` 为仓库相对文件路径，`id` 定位该文件内对象；交接时另给仓库绝对路径。文档 `commit_sha` 必须是包含该路径/对象的完整已存在 Commit。使用 `git show <doc_sha>:<path>` 读取受审内容，不能以当前工作树或移动分支替代。局部 Task 审核只读所指章节和相关邻域，不必读取整个账本。

无可用文档提交身份时，用 `{id, revision, path, snapshot: {path, sha256}}`：快照保存送审文件的完整字节，位于持久 `attachments/` 或宿主批准的稳定位置，发送前校验哈希，永不覆盖。未跟踪证据注明“仅本地可恢复”，不声称 Git 共享；当前文件不是历史快照。原 v1 在无 Git 下可继续使用其不可变文件。代码 G5–G9 缺少真实 SHA 时仍阻塞，不能把快照伪装成 Commit。

## 账本记录

每条记录一个独立锚点，容器元数据包括 `record_id`、`author`、`created_at`、`kind`、`objects` 与 `supersedes`。记录 ID 可直接沿用载荷的 Review/Test/Implementation/Defect ID；同一账本唯一。新发布的报告将目标对象的完整文档身份放在 `objects`，原载荷中的代码 SHA、Verdict、Finding 和全部字段原样保留。导入的历史载荷不补写猜测身份，未知或旧引用由迁移索引解析且明确限制。

完整原始载荷使用足够长的 fenced block 隔离，保留空白、未知字段、旧 ID 和引用；仅在围栏之外补充定位说明。用户批准、状态事件、适用性与 Finding 处置由其真实作者记录，不捏造独立结论。账户/会话 ID、cursor、PID 不进入正式记录；角色作者用可审计且不含本机运行状态的身份。

先在 `.local/` 收齐完整输出，校验与作者输出一致后串行追加；确认整条记录完整且无 ID 冲突，再更新 state。可使用宿主原生安全写入工具，不要求新脚本。写入中断留下的未发布尾部只能在确认边界后恢复，不得截断完整历史；来源丢失则阻塞该记录而非猜补。证据追加成功、state 更新失败时按 ID 补索引，不重发同一角色任务。追加后冻结是记录级约束，并不禁止继续追加后续记录。

命令证据须包含完整 SHA（适用时）、cwd、命令、脱敏环境/依赖版本、复现数据条件、退出码、关键输出、范围、未运行原因。摘要不足以复现时，把必要原始脱敏材料提升到 `attachments/` 或稳定仓库/CI 产物，不能仅给临时路径。不得把失败或 BLOCKED 报告当垃圾删掉。

## 当前索引与恢复顺序

state 的 `artifacts` 定位当前文件；`approvals`、`tasks[].evidence` 与 `open_findings/open_defects/open_blockers` 使用已存在的记录/对象引用。草稿索引允许尚无发布身份，但有效批准必须指向绑定不可变身份的记录。状态事件先写 evidence，再更新当前状态，不在 state 维护完整历史。

恢复顺序：state → 当前目标章节/独立修订 → 有效批准及开放问题对应记录 → 必要历史。历史旧路径先查一次迁移索引。同一会话已读且未变化的共享契约无需重复加载；不以全量历史扫描代替按 ID 寻址。

Git 分类、提交授权、文档与代码 SHA 区分见[Git 策略](../contracts/git-policy.md)。迁移仅按[无损迁移](legacy-migration.md)执行。
