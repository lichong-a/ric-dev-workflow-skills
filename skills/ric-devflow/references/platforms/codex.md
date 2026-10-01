# Codex 准备

只在已确认当前宿主为 Codex 时读取。先完成[准备与恢复](../bootstrap.md)的范围、固定同源候选与全目标预检；不写 Claude/ZCode 配置。

## 文件与必需配置

- 项目 Skill 根为项目 `.agents/skills`，用户根为 `~/.agents/skills`；跟随已发现的逻辑入口，普通目录复制可用。
- 原生定义源为入口 `assets/agents/codex/ric-devflow-<role>.toml`，目标为同范围 `.codex/agents/ric-devflow-<role>.toml`。
- 角色 ID 来自 TOML `name`，为 `ric_devflow_planner/reviewer/tester/implementer`，不是文件名推测。保留 `name`、`description`、`developer_instructions` 必需字段。
- 模型继承宿主；Planner `max`、Reviewer `high`、Tester/Implementer `xhigh`；Reviewer `sandbox_mode = "read-only"`，其他角色 `workspace-write`。这表达原权限意图，不能覆盖更严格的宿主限制。
- `[agents].enabled` 默认 true，无显式设置时不必为安装增加配置。明确 false 不反转。已有并发数、模型、权限、Agent 配置保留；[配置示例](../../assets/agents/codex/config.toml)是原基线示例，不能整段写入或把示例并发数强加给用户。
- 五份 Skill `agents/openai.yaml` 中只有 `ric-devflow` 允许隐式调用，四业务角色均为 false；不把该 Codex 策略声称为其他宿主硬开关。

仅补本次必要的原生角色定义；同名定制或不同版本先报告，不覆盖。独立主会话直接执行角色时不要求其他原生角色；完整开发核验四份定义及宿主提供的加载信息，实际调用随必要交接逐角色确认，不以一例成功证明四目标均可用。交接携带已确认角色入口绝对路径、入口共享根与适用仓库规则，不将源码 checkout 路径永久写进 Agent 模板。

## 角色名称与界面边界

本节名称字段核验日期：2026-09-13；其余平台说明沿用下文标注的原核验日期。

原生 TOML 的 `name` 是稳定角色 ID；`description` 使用“规划者｜…”等中文职责说明，`developer_instructions` 要求首条进度和返回结果自述职责。当前官方文档确认的角色入口字段为 `name`、`description`、`developer_instructions`；本包不增加未经证实的 `nickname_candidates` 或 TOML `display_name`，也不改变原模型、推理、权限或并发配置。

实际调用工具支持 `task_name` 时按[职责命名](../shared/orchestration.md#职责命名与可读交接)使用 `ric_<role>_<work>`；只在工具确实提供人类标题字段时传“实现者｜Skill布局”等标题。平台昵称、调用任务名和 Skill `agents/openai.yaml` 的 `display_name` 是不同对象。当前工具没有旧子代理重命名接口时，不承诺修改 Skill UI 或 TOML 描述会改名、不重建会话来模拟改名、不追溯修改旧会话；通过后续进度和交接自述说明职责。

完整性要求核验四个原生角色，不要求同时创建四个代理；按[会话复用](../shared/orchestration.md#按需派发与会话复用)由同一 Root 按需使用现有角色。官方字段与宿主能力以[Codex subagents 文档](https://learn.chatgpt.com/docs/agent-configuration/subagents)及当前实际工具 Schema 为准，配置可解析不证明界面昵称已经变化。

## 加载与路由

Skill 变更通常自动检测；若未出现则按宿主提示重载。Skill 已发现不代表原生角色已加载。补装或复用定义后，按[加载与调用核验](../bootstrap.md#文件加载调用分别核验)核对当前会话的 ID、来源和启用状态，以下一次必要交接确认调用，不虚构枚举工具或额外派发 Root Planner 自身。

文件与配置正确但当前会话目标未注册、未加载或未暴露目标调用能力时，明确提示“请重启 Codex，并在同一项目重新调用 Skill”，附已核验路径、宿主反馈和恢复信息。只有工具缺失且原因未知时，说明重启后仍需重新验证；配置/禁用/权限等故障单独报告。不自动重启客户端，不把其他会话或 CLI 的成功当当前会话证据；重启后重验，同一缺口仍在时诊断而不循环提示。

准备完成后，完整开发进入 Root Planner 并保持它直接调度角色；主会话显式角色按原角色流程执行。已启动子角色不安装、不派生。目标缺失时不能以 `default`/通用代理冒充 `ric_devflow_<role>`。

同会话能力提示遵循入口守卫和[提示核验](../bootstrap.md#同会话能力提示)，不为 warm 动作重读整个恢复流程，也不把旧调用当新调用。Root Planner 按共享契约执行有界 READY 波次与末 Task G6/G7 条件交接；既有宿主并发、模型和权限配置不改动。Codex Reviewer 继续直接只读 Git，不引入为缺 Git/Bash 宿主准备的导出缓存。

文档原核验日期为 2026-09-12；2026-10-01 复核 subagents 的定义目录与角色 ID 规则，加载结论仍以当前会话反馈为准，不据此宣称自动热加载或重启必然有效。配置解析/说明核验不代表原生实机已运行：[Codex subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)、[Codex Skills](https://learn.chatgpt.com/docs/build-skills)。
