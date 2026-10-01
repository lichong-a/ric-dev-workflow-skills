# ZCode 准备

只在已确认当前宿主为 ZCode 时读取。先完成[准备与恢复](../bootstrap.md)的范围、固定同源候选与全部目标预检；不写 Codex/Claude 配置。

用户 Skill 可普通复制至 `~/.zcode/skills`，使用宿主刷新确认。既有安装若由宿主实际发现于项目/用户 `.agents/skills` 或其他受支持逻辑位置，保持该位置与范围；旧官方随包文档记录过 `.agents/skills` 发现能力，不能仅因本机有目录就声称本版本已发现。不创建 `.zcode/skills` 与 `.agents/skills` 双份规则，也不猜同名优先级。

原生模板为入口 `assets/agents/zcode/ric-devflow-<role>.md`。官方支持的目标为用户 `~/.zcode/agents/<name>.md`；项目入口准备完整开发时，只有四份角色定义可按已授权例外写入该用户目录，五 Skill 继续留在项目范围。独立角色只补该角色定义。不把项目 `.zcode/agents` 源文件存在当作已加载，不绑定某项目绝对路径；角色入口与共享根由实际发现后在交接中传入。

四 ID 为 `ric-devflow-planner/reviewer/tester/implementer`；保留 `model: inherit` 和 `injectAgentsMd: true`。Reviewer 仅 `Read, Grep, Glob`，其他角色增加 `Bash, Edit, Write`；白名单不保证文件路径隔离，不默认放开 MCP 或嵌套代理。已有同名自定义/异版本定义、禁用设置、其他模型或权限保持，冲突写入前停止。

Agent 新定义在下一次运行加载；定义变更需要新建会话，已启动会话不热更新。Skill 刷新后还要确认真正发现位置，不能把 Skill 刷新当作子代理加载。补装或复用定义后，按[加载与调用核验](../bootstrap.md#文件加载调用分别核验)核对当前会话反馈，以下一次必要交接确认调用。

文件与配置正确但当前会话目标未加载或无调用能力时，明确提示“请重新启动 ZCode Agent 会话，并在同一项目重新调用 Skill”，附路径、原反馈、原任务、范围、固定来源及下一动作；不代用户关闭客户端。仅缺工具且原因未知时说明重启后仍需验证，其他故障单独报告。恢复后重新核验当前会话，复用正确文件，只补实际缺项；同一缺口仍在则诊断，不循环要求重启。文件安装、宿主加载、实际调用仍分别记录。

主会话只转发已确认原生同级角色；唯一 Planner 子代理作决定并写状态，四子角色都不派生。完整开发检查五 Skill 与四原生角色；显式独立角色仅所需闭包。依据[原生平级调用](../shared/orchestration.md#原生平级调用)处理调用与恢复，不用通用代理冒充角色，不以 AGENTS.md 注入替代读取实际局部规则。

warm 准备只按入口守卫及当前动作核对能力提示；加载/来源变更使相关提示失效，旧运行不证明本次已加载或调用。Planner 决定有界 READY 波次和末 Task G6/G7 条件批次，主会话不自行合并动作或重跑工作。完整载荷首传、后续固定指针和失败恢复按[紧凑交接](../shared/orchestration.md#紧凑且精确的交接)；只读 Reviewer 不写 Git 缓存，由 Planner 保持原对象和必要持久证据可恢复。

本包未宣称标准安装器支持 ZCode；使用普通复制和宿主官方管理入口。本文不证明原生实机已运行。2026-09-12 核验来源：[ZCode Skill](https://zcode.z.ai/cn/docs/skill)；2026-10-01 复核定义位置及新会话生效条件：[ZCode 子代理](https://zcode.z.ai/cn/docs/subagents)。
