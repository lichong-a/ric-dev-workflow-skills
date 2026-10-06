# Claude Code 准备

只在已确认当前宿主为 Claude Code 时读取。先完成[准备与恢复](../bootstrap.md)的范围、固定候选与全部目标预检；不写 Codex/ZCode 配置。

项目与用户 Skill 根分别为 `.claude/skills`、`~/.claude/skills`；按实际逻辑发现来源确定范围，普通复制即可，不依赖共享 symlink。原生模板位于入口 `assets/agents/claude-code/ric-devflow-<role>.md`，复制到同范围 `.claude/agents` 或 `~/.claude/agents`。只填缺失项，不替换同名定制定义，不覆盖 `CLAUDE.md`、`AGENTS.md` 或 settings。

三个子角色 ID 为 `ric-devflow-reviewer/tester/implementer`；保持 `model: inherit`。Reviewer 只提供 `Read, Grep, Glob`，Tester/Implementer 增加 `Bash, Edit, Write`。主会话直接加载 Planner Skill，不安装或调用 Planner 原生定义。不加 Agent/MCP/其他客户端派生能力；工具白名单不是文件路径沙箱，Bash 仍须遵守角色和路径边界。原生定义从交接中的绝对 Skill 路径读取规则，不写死某项目位置。

会话启动时已存在的项目或用户 agents 目录受宿主监视，修改通常数秒后可发现；会话中首次创建 agents 目录、修改 add-dir 来源下的定义，或以 `--disable-slash-commands` 启动时，需要重启以加载变更。先观察当前会话反馈，按[加载与调用核验](../bootstrap.md#文件加载调用分别核验)使用下一次必要角色交接确认；没有清单接口不虚构查询命令，不固定 sleep 后假称成功。Skill/同名 Agent 的真实来源、启用状态以当前宿主反馈为准；Codex `openai.yaml` 不替代 Claude 的触发语义。

文件与配置正确但当前会话目标未加载或无调用能力时，明确提示“请重启 Claude Code，并在同一项目重新调用 Skill”，附路径、原反馈及最小恢复信息。仅缺工具且原因未知时说明重启后仍需验证；其他故障单独报告。已经能调用时直接继续，不因刚补装而强制重启；恢复后不重复安装，同一缺口仍在则诊断，不循环要求重启。

完整开发检查五 Skill 和三个原生子角色；显式 Planner 由主会话执行，其余独立角色仅准备入口、当前角色及原生定义并直接返回调用者。主会话 Planner 直接决定交接、原样保存子角色报告并维护状态，三个子角色均不派生。旧 Planner 定义保留但不调用；旧工作流须先确认原 Planner 已停止并交接证据。缺工具、未加载、权限拒绝或生命周期未知时按[编排](../shared/orchestration.md#原生平级调用)停止受影响动作，不产生第二写入者。

warm 准备按入口守卫核对能力提示，失效只补查当前动作的必要输入，不重复加载整篇 bootstrap。主会话 Planner 可按共享契约交接有界独立 READY 波次或末 Task G6/G7 条件批次，同角色默认排队、满足全部条件才有限扩容；结果首次完整传输、后续固定指针及丢失恢复按[紧凑交接](../shared/orchestration.md#紧凑且精确的交接)。Reviewer 仍仅只读，Git 对象缓存由 Planner 提取/核验/恢复，blob 复用不能省略 Commit/路径/模式绑定。

文件安装、宿主加载、实际调用分列；需新会话时保留原任务与恢复信息。本文不证明真实 Claude 运行已通过。2026-09-12 核验来源：[Skills](https://code.claude.com/docs/en/skills)；2026-10-06 复核恢复入口（SendMessage）及目录监视和重启条件：[Sub-agents](https://code.claude.com/docs/en/sub-agents#write-subagent-files)。

## 复用与思考参数

同主会话固定三种职责，默认每角色一个实例，首次用实际 Agent 工具选择原生角色并保存返回的 agent ID；后续优先调用当前恢复接口。官方当前说明通过 `SendMessage` 的 `to` 指向该 ID/已确认名称，可恢复已完成的自定义子代理；重复 Agent 调用会新建实例。以本会话 Schema 为准，若旧版本提供其他恢复入口只使用其真实字段；没有恢复能力但旧动作已结束、证据已保全且下一动作获准时，可按[受控替换](../shared/agent-lifecycle.md#复用与受控替换)新建同职责实例，不重做旧动作。用户主动取消的实例不能由代理自动恢复；取消动作/Root A 不自动禁止用户新授权的独立 B，确认 A 停止并保全后可为 B 替换实例；取消角色后续执行或全部自动化则保持停止。取消范围和容量回收只按生命周期当前章节处理。来源：[Resume subagents](https://code.claude.com/docs/en/sub-agents#resume-subagents)。

首次确定参数、能力变化或明确不兼容时才读取[思考兼容](../shared/reasoning-compatibility.md)。确认宿主是否暴露 effort 控制及实际接入方，不能把 Anthropic API 字段直接加到 Agent 调用或原生定义；未暴露时只沿已验证可用的继承/默认路径，不改模型、用户配置或权限。
