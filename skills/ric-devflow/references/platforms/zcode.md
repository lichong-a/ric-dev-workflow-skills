# ZCode 准备

只在已确认当前宿主为 ZCode 时读取。先完成[准备与恢复](../bootstrap.md)的范围、固定同源候选与全部目标预检；不写 Codex/Claude 配置。

用户 Skill 可普通复制至 `~/.zcode/skills`，使用宿主刷新确认。既有安装若由宿主实际发现于项目/用户 `.agents/skills` 或其他受支持逻辑位置，保持该位置与范围；旧官方随包文档记录过 `.agents/skills` 发现能力，不能仅因本机有目录就声称本版本已发现。不创建 `.zcode/skills` 与 `.agents/skills` 双份规则，也不猜同名优先级。

原生模板为入口 `assets/agents/zcode/ric-devflow-<role>.md`。官方支持的目标为用户 `~/.zcode/agents/<name>.md`；项目入口准备完整开发时，只有四份角色定义可按已授权例外写入该用户目录，五 Skill 继续留在项目范围。独立角色只补该角色定义。不把项目 `.zcode/agents` 源文件存在当作已加载，不绑定某项目绝对路径；角色入口与共享根由实际发现后在交接中传入。

四 ID 为 `ric-devflow-planner/reviewer/tester/implementer`；保留 `model: inherit` 和 `injectAgentsMd: true`。Reviewer 仅 `Read, Grep, Glob`，其他角色增加 `Bash, Edit, Write`；白名单不保证文件路径隔离，不默认放开 MCP 或嵌套代理。已有同名自定义/异版本定义、禁用设置、其他模型或权限保持，冲突写入前停止。

Agent 定义在下一次运行加载；Skill 刷新后还要确认真正发现位置。文件安装、宿主加载、实际调用分别记录。需要下一运行时保留原任务、范围、固定候选及下一动作，恢复后读回现状，只补实际缺项。

主会话只转发已确认原生同级角色；唯一 Planner 子代理作决定并写状态，四子角色都不派生。完整开发检查五 Skill 与四原生角色；显式独立角色仅所需闭包。依据[原生平级调用](../shared/orchestration.md#原生平级调用)处理调用与恢复，不用通用代理冒充角色，不以 AGENTS.md 注入替代读取实际局部规则。

本包未宣称标准安装器支持 ZCode；使用普通复制和宿主官方管理入口。本文不证明原生实机已运行。2026-09-12 核验来源：[ZCode Skill](https://zcode.z.ai/cn/docs/skill)、[ZCode 子代理](https://zcode.z.ai/cn/docs/subagents)。
