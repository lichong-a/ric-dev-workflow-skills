---
name: ric-devflow
description: 当用户请求软件开发、缺陷修复、项目续作、重构、迁移或基础设施变更时主动使用；先确认宿主、安装范围与同源完整性，按需补齐 DevFlow Skill 和当前平台原生角色，再持续路由四角色交付。解释、一般建议、独立审核或测试、单纯文档与极小无风险编辑不启动完整流程。
---

# RIC DevFlow 入口

本入口负责准备和路由，不是第五个业务角色，不代替 Planner、Reviewer、Tester 或 Implementer，也不授予业务批准。保留用户原任务、约束、已有授权及进度，按以下顺序执行。

1. **任务分类**：范围内完整开发进入完整流程；显式 `ric-devflow-<role>` 或精确角色委派只准备该角色的必要闭包。解释、一般建议、独立只读审核/测试、单纯文档编辑、极小无风险编辑及无关 Skill 请求不隐式启动完整 DevFlow；遵从用户明确选择。
2. **宿主与身份**：依据实际宿主声明、可用工具及原生启动上下文确认 Codex、Claude Code 或 ZCode 和主/子会话；配置目录存在不作身份依据。不明确、不支持或身份矛盾时保留原任务并报告精确缺口。已启动子角色缺件时只回交宿主，不安装、不再派生代理。
3. **安装范围**：记录逻辑 `discovered_path`、可信候选 `source_root` 和 `install_scope`；写入跟随逻辑发现位置或已确认范围，不能由 symlink 的 realpath 反推。仅源码路径且范围未知时，先明确范围再写入。
4. **完整性与缺件守卫**：先检查本入口内 `references/bootstrap.md` 及当前平台材料是否可读，未齐备前不要打开下文缺失链接。主会话缺件时，先确认发布源 `https://github.com/lichong-a/ric-dev-workflow-skills`，优先完整可信本地 checkout 或安装记录完整 SHA；无记录只向已确认源取一次候选并固定 SHA，不猜历史、不执行下载代码。按该 SHA 完整清单核对本次闭包全部现存包文件的类型和字节，入口本身也必须匹配；源不明、范围不明、候选不完整、定制/异版本、未知文件或断链均保持安装目标零写入并报告缺口。匹配后只从可信候选源读取 bootstrap 和当前平台材料，列出所需 Skill 闭包、必需原生定义/配置、父目录及权限，**全部目标的冲突、明确禁用和权限预检通过前不复制任何缺件**；通过后才排他创建并普通复制缺项，写前复核仍缺失、写后逐字读回，中断后重新比对再补缺。已启动子角色只回交宿主，不自行恢复。文件齐备后读取[准备与恢复](references/bootstrap.md)继续；完整开发需要五 Skill 与当前宿主四原生角色，独立角色仅检查入口、该角色及必要传递引用/配置，不要求无关角色。
5. **补装与配置**：仅在存在缺件或需核验配置时，按 bootstrap 读取当前平台一份参考：[Codex](references/platforms/codex.md)、[Claude Code](references/platforms/claude-code.md) 或 [ZCode](references/platforms/zcode.md)。固定可信候选 SHA、核对整个待修复闭包的现存包文件、预检全部目标后只补缺；冲突、禁用和权限拒绝按原边界停止，不执行下载安装代码。
6. **加载核验**：分别报告文件安装、宿主加载和实际调用。文件齐备不等于角色已加载；能继续就沿原任务前进。需要重载时给出最小恢复信息，不重复安装，不伪造调用成功或业务 Gate。
7. **角色路由**：准备就绪后才读取目标角色入口。Codex 的 Root Planner 保持直接调度；Claude/ZCode 主会话按[原生平级调用](references/shared/orchestration.md#原生平级调用)转发，唯一 Planner 作决定并写状态，四子角色均不派生代理。无角色工具或目标未加载时报告缺口，不用通用代理冒充。

| 任务 | 准备完成后读取 | Codex 原生 ID | Claude/ZCode 原生 ID |
|---|---|---|---|
| 完整开发或显式规划 | [Planner](../ric-devflow-planner/SKILL.md) | `ric_devflow_planner` | `ric-devflow-planner` |
| 显式或委派审核 | [Reviewer](../ric-devflow-reviewer/SKILL.md) | `ric_devflow_reviewer` | `ric-devflow-reviewer` |
| 显式或委派测试 | [Tester](../ric-devflow-tester/SKILL.md) | `ric_devflow_tester` | `ric-devflow-tester` |
| 单个已批准 Task/实现类 Defect | [Implementer](../ric-devflow-implementer/SKILL.md) | `ric_devflow_implementer` | `ric-devflow-implementer` |

表中未选择角色的链接仅是导航，不是独立角色的安装依赖。只按当前动作读取共享契约和领域材料；同一会话中已读且未变的内容无需重复读取。包准备完成不改变 G0–G10、职责分离、Spec/测试批准、真实 SHA 绑定、冻结 DAG 或 VERIFIED 到 DONE 条件。
