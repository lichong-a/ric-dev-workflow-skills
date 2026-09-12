# Codex 准备

只在已确认当前宿主为 Codex 时读取。先完成[准备与恢复](../bootstrap.md)的范围、固定同源候选与全目标预检；不写 Claude/ZCode 配置。

## 文件与必需配置

- 项目 Skill 根为项目 `.agents/skills`，用户根为 `~/.agents/skills`；跟随已发现的逻辑入口，普通目录复制可用。
- 原生定义源为入口 `assets/agents/codex/ric-devflow-<role>.toml`，目标为同范围 `.codex/agents/ric-devflow-<role>.toml`。
- 角色 ID 来自 TOML `name`，为 `ric_devflow_planner/reviewer/tester/implementer`，不是文件名推测。保留 `name`、`description`、`developer_instructions` 必需字段。
- 模型继承宿主；Planner `max`、Reviewer `high`、Tester/Implementer `xhigh`；Reviewer `sandbox_mode = "read-only"`，其他角色 `workspace-write`。这表达原权限意图，不能覆盖更严格的宿主限制。
- `[agents].enabled` 默认 true，无显式设置时不必为安装增加配置。明确 false 不反转。已有并发数、模型、权限、Agent 配置保留；[配置示例](../../assets/agents/codex/config.toml)是原基线示例，不能整段写入或把示例并发数强加给用户。
- 五份 Skill `agents/openai.yaml` 中只有 `ric-devflow` 允许隐式调用，四业务角色均为 false；不把该 Codex 策略声称为其他宿主硬开关。

仅补本次必要的原生角色定义；同名定制或不同版本先报告，不覆盖。独立主会话直接执行角色时不要求其他原生角色；完整开发必须核验四目标真正可用。交接携带已确认角色入口绝对路径、入口共享根与适用仓库规则，不将源码 checkout 路径永久写进 Agent 模板。

## 角色名称与界面边界

本节名称字段核验日期：2026-09-13；其余平台说明沿用下文标注的原核验日期。

原生 TOML 的 `name` 是稳定角色 ID；`description` 使用“规划者｜…”等中文职责说明，`developer_instructions` 要求首条进度和返回结果自述职责。当前官方文档确认的角色入口字段为 `name`、`description`、`developer_instructions`；本包不增加未经证实的 `nickname_candidates` 或 TOML `display_name`，也不改变原模型、推理、权限或并发配置。

实际调用工具支持 `task_name` 时按[职责命名](../shared/orchestration.md#职责命名与可读交接)使用 `ric_<role>_<work>`；只在工具确实提供人类标题字段时传“实现者｜Skill布局”等标题。平台昵称、调用任务名和 Skill `agents/openai.yaml` 的 `display_name` 是不同对象。当前工具没有旧子代理重命名接口时，不承诺修改 Skill UI 或 TOML 描述会改名、不重建会话来模拟改名、不追溯修改旧会话；通过后续进度和交接自述说明职责。

完整性要求核验四个原生角色，不要求同时创建四个代理；按[会话复用](../shared/orchestration.md#按需派发与会话复用)由同一 Root 按需使用现有角色。官方字段与宿主能力以[Codex subagents 文档](https://learn.chatgpt.com/docs/agent-configuration/subagents)及当前实际工具 Schema 为准，配置可解析不证明界面昵称已经变化。

## 加载与路由

Skill 变更通常自动检测；若未出现则按宿主提示重载。原生角色文件读回后核验当前会话实际可调用的 ID、来源、启用状态；需要新会话时保留 bootstrap 恢复信息，不把文件安装视为加载完成。

准备完成后，完整开发进入 Root Planner 并保持它直接调度角色；主会话显式角色按原角色流程执行。已启动子角色不安装、不派生。目标缺失时不能以 `default`/通用代理冒充 `ric_devflow_<role>`。

文档核验日期为 2026-09-12；配置解析/说明核验不代表原生实机已运行：[Codex subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)、[Codex Skills](https://learn.chatgpt.com/docs/build-skills)。
