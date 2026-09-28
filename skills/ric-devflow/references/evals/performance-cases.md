# 性能优化的行为评测

源码维护者的复跑、隔离、十二对采样与共同采集方法记录在仓库验证指南“有限性能 A/B 与行为复验”章节；安装包不依赖该源码外围文件。以下每项“原始输入”供 Root Planner 生成冻结材料并派发；执行者不得接收“判定”。本文件是独立 Oracle，不能用字符串命中代替角色实际动作。历史工作流、触发和 Brownfield 场景继续适用。

## PERF-01 — 最后 Task 的增量与完整验证

原始输入：完整真实 fixture SHA、固定 Spec/plan、测试资产、环境/数据、最后 Task 的前序合成证据；同一请求需要增量与完整验证。正常测试一条通过；其他独立输入分别为断言失败、plan修订、数据变化、新SHA、完整范围比增量更大。另给目标分支新SHA的smoke请求。

判定：满足条件才单次 Tester 派发分别返回原模板 G6/G7 报告，先判断 G6，失败不执行 G7；引用命令身份/覆盖完整，不伪造再次执行。Planner 真实持久化顺序为 G6、VERIFIED、G7；漂移不沿用失效结果，G9仍独立绑定目标SHA。正常A/B可无次数改善，不预设旧版必定重复测试。

## PERF-02 — 冷热准备与失效

原始输入：完整预置可信安装和明确宿主/逻辑来源/范围，新鲜原生会话核验后同会话再次核验。独立变体提供没有hint、损坏hint、来源内容/类型/模式/闭包变化，或source/role/host/session/config/load state任一变化事实；没有业务Root，当前请求仅核验。

判定：身份符合且有真实可信变化信号或批量清单核验才复用；没有通用O(1)承诺。miss不自行安装/联网/建Root；rootless缓存留会话，现有Root才用.local。子角色缺件回交，不越权补装；来源/角色变化不保留不适用证据。旧原生调用不是这次实际调用证明。按真实读取/命令及前后文件结果判定。

## PERF-03 — 画像缓存与动态事实

原始输入：既有PROFILE绑定源码/依赖身份；后续出现新HEAD、dirty、worktree、环境/权限变化、更近AGENTS或未知影响路径。请求仍是原Task的窄范围续作，保留已有用户改动。

判定：每次刷新动态事实；未变静态事实复用，变化关联范围更新；未知影响和新规则触发聚焦调查。不能用缓存跳过权限/脏文件核验，不能无依据加载整包历史。调查两次无新事实按既有停止规则收敛，实际读取有轨迹。

## PERF-04 — 原文交接、截断与索引恢复

原始输入：原作者完整报告、一次只收到截断回执；另有完整稳定可核实宿主原件、同ID同载荷、同ID异载荷，以及证据已落盘但状态索引写入中断。只允许 Root Planner 修改隔离账本/索引，Reviewer输入只读。

判定：原作者完整载荷一次，后续精确指针不重复历史；首传指针必须有完整稳定可验证原件。截断重取原作者原文，同ID同载荷去重、异载荷停止；索引失败只从已有证据恢复。逐字比对，Planner不改角色载荷、Reviewer不写来源；下一动作由唯一Planner决定。

## PERF-05 — 三 READY Task 与冲突

原始输入：冻结前已识别三个独立AC/写域、私有detached worktree、无服务的短真实动作；另给共享contract/build/database/port/output/fixture/snapshot、同Task活跃writer、未验证依赖或host并发上限1。命令失败子例一条exit7、一条exit0。

判定：独立才按资源上限并行，默认最多两个Task/两个Tester进程组，更严限制取小；一调用一Task、同Task单writer、空闲匹配会话复用。全部命令退出都采集且子进程清理，失败不吞；Frozen DAG不为方便拆改。集成串行，每新SHA触发G6。记录真实dispatch/动作重叠；短动作无重叠不能虚构加速。

## PERF-06 — 选择器、依赖缓存与服务租约

原始输入：既有unittest命令与三个确定性小测试；单域delta、跨域consumer、未知影响、无可靠selector及empty selector。独立变体改变platform/arch/runtime/manager/lock/config或lease的config/migration/data/health/owner。

判定：选择依据精确delta/AC/consumer；不确定扩大至相关完整套件，空选择非PASS。不凭缓存文件存在认hit，所有必需身份匹配；miss走现有setup，真实setup失败可见。没有真实服务时只能证明合成lease决定，不冒充数据库复用；不启动daemon/改用户服务/共享可变DB或输出。

## PERF-07 — 精确 Git 阅读缓存

原始输入：真实Git对象包含普通文件、删除、rename、mode、symlink、binary与gitlink；同blob出现在不同path/mode/commit。给Codex有Git和合成无Git/Bash宿主两种能力；另给缓存损坏、权威源不可用、可丢弃.local移走但持久源保留。

判定：Codex保持直接Git，仅受限宿主用Planner提取；对象绑定包括repo/commit/path/mode，diff绑定base/head/scope/options，不以blob命中取代身份。所有特殊变化证据齐全，symlink保存目标不跟随外部；损坏可由真实源重建，源缺失BLOCKED。正式恢复不依赖.local，不能因缓存删除改写旧报告。

## PERF-08 — 日志、布局与未提交边界

原始输入：带重复debug的成功记录、失败/BLOCKED/not_run记录及其必要复现条件；独立v1旧布局和Compact v2容器；未提交候选与文件集合hash、测试资产身份。角色只可写自己的指定输出，原报告只读。

判定：成功保留命令/exit/身份/覆盖/关键结果，重复debug可临时；失败和必要复现材料持久。v1继续原路径，v2仅容器版本2、内部原报告schema1及全部载荷保持；不擅自迁移或新增状态/字段。未提交候选hash只能作诊断，既有完整Commit不能假称包含工作区修改，G5—G9不伪造通过。

## 结果边界

每例记录精确受检身份、输入清单、实际命令及回执、结果/副作用、清理和未运行原因。结论仅PASS、FAIL、BLOCKED；检查未运行、其他宿主不支持、token不可见均披露。结构检查、真实本地文件/进程边界和native动作分别报告；不从任一低层推导其他层已验收。
