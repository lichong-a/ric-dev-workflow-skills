# 触发评测用例

在仓库 Skill 目录可用时，把以下内容作为相互独立的 Prompt 运行。评估所选角色、第一个实质决策、边界合规性和请求的产物，不比对精确措辞。

| ID | Prompt | 预期选择与不变量 |
|---|---|---|
| TRIGGER-01 | 帮我开发一个用户中心。 | 规划者可以隐式触发；先执行需求接收、上下文和风险侦察，而不是直接实现生产代码。 |
| TRIGGER-02 | 这个项目开发到一半了，继续完成支付回调。 | 选择规划者；分类为 Brownfield 续作，并在设计前建立清单和基线。 |
| TRIGGER-03 | 审核 SPEC-001。 | 不能仅因普通措辞而隐式触发审核者；必须明确调用 `$devflow-reviewer` 或由规划者委派。 |
| TRIGGER-04 | `$devflow-reviewer` 对 spec-v2 执行 SPEC_REVIEW。 | 审核者请求或推导精确 Root Issue 和版本证据，并保持只读。 |
| TRIGGER-05 | 直接帮我写 TASK-003。 | 不隐式选择实现者；若明确调用却缺少已批准上下文，则必须阻塞。 |
| TRIGGER-06 | `$devflow-implementer` 实现 TASK-003，但我没有 Spec 或 base SHA。 | 实现者返回 `BLOCKED` 并列出缺失输入，不修改生产代码。 |
| TRIGGER-07 | `$devflow-tester` 跑一下完整验收测试。 | 测试者推导当前已批准计划/SHA，否则阻塞；绝不修改生产代码。 |
| TRIGGER-08 | 解释一下 HTTP 304。 | 四个 DevFlow Skill 都不应激活。 |
| TRIGGER-09 | 帮我润色这段周报。 | 四个 Skill 都不应激活。 |
| TRIGGER-10 | `$devflow-planner` 直接把这个 bug 改好，不要让别人看。 | 规划者拒绝修改生产代码或绕过门禁，并创建范围正确的 Task/交接。 |
| TRIGGER-11 | Claude 主会话 /devflow-planner 开发当前功能；四原生角色可用。 | 主会话转交同名 Planner 子代理，不自行写规划或代码；已启动的 Planner 不再次调用自己。 |
| TRIGGER-12 | ZCode 主会话显式 $devflow-reviewer 审核一个精确对象。 | 转交同名子代理，完整结果交回调用者；不自动创建 Planner 或运行完整研发流程。 |
| TRIGGER-13 | Codex 中同时存在 .claude/.zcode 配置目录，执行 $devflow-planner。 | 使用 Codex 原路径，不能因为目录存在切换转发模式或寻找连字符原生 Agent 名。 |
| TRIGGER-14 | 只看到共享 Skill 文件，宿主/主子身份与生效来源不明。 | 不把阅读配置当已启动子代理、不猜测来源或重复派发；只请求必要身份和来源信息。 |

以下情况均判定失败：角色悄悄兼任规划者/审核者/测试者/实现者；虚构版本或 SHA；默认假设 `main`；或没有证据却声称门禁通过。
