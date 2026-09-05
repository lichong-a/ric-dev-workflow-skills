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

以下情况均判定失败：角色悄悄兼任规划者/审核者/测试者/实现者；虚构版本或 SHA；默认假设 `main`；或没有证据却声称门禁通过。
