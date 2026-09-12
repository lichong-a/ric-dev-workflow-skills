# Claude Code 准备

只在已确认当前宿主为 Claude Code 时读取。先完成[准备与恢复](../bootstrap.md)的范围、固定候选与全部目标预检；不写 Codex/ZCode 配置。

项目与用户 Skill 根分别为 `.claude/skills`、`~/.claude/skills`；按实际逻辑发现来源确定范围，普通复制即可，不依赖共享 symlink。原生模板位于入口 `assets/agents/claude-code/ric-devflow-<role>.md`，复制到同范围 `.claude/agents` 或 `~/.claude/agents`。只填缺失项，不替换同名定制定义，不覆盖 `CLAUDE.md`、`AGENTS.md` 或 settings。

四 ID 为 `ric-devflow-planner/reviewer/tester/implementer`；保持 `model: inherit`。Reviewer 只提供 `Read, Grep, Glob`，其他角色增加 `Bash, Edit, Write`。不加 Agent/MCP/其他客户端派生能力；工具白名单不是文件路径沙箱，Bash 仍须遵守角色和路径边界。原生定义从交接中的绝对 Skill 路径读取规则，不写死某项目位置。

已有项目或用户 agents 目录受宿主监视，修改通常数秒后可发现；首次创建目录或通过 add-dir 添加来源时需要新会话。先观察宿主实际加载再决定重载，不固定 sleep 后假称成功。Skill/同名 Agent 的真实来源、启用状态以当前宿主反馈为准；Codex `openai.yaml` 不替代 Claude 的触发语义。

完整开发检查五 Skill 和四原生角色；独立调用只准备入口、当前角色及其原生定义。主会话显式加载角色 Skill 时，准备就绪后实际调用同名子角色，不能亲自执行角色正文。唯一 Planner 子代理决定下一动作；主会话按其精确交接继续调用平级角色并原样回传，四子角色均不再派生。缺工具、未加载、权限拒绝或旧 Planner 生命周期未知时按[编排](../shared/orchestration.md#原生平级调用)停止受影响动作，不启动第二写入者。

文件安装、宿主加载、实际调用分列；需新会话时保留原任务与恢复信息。本文不证明真实 Claude 运行已通过。2026-09-12 核验来源：[Skills](https://code.claude.com/docs/en/skills)、[Sub-agents](https://code.claude.com/docs/en/sub-agents)。
