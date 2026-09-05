<div align="center">

# RIC DevFlow Skills

**把一次编码请求，升级成可审计、可接管、可交付的软件工程闭环。**

面向 Codex 的四角色软件交付 Skill：规划、独立审核、独立测试和受控实现。

![Codex Skills](https://img.shields.io/badge/Codex-Skills-111827?style=flat-square)
![Instruction Only](https://img.shields.io/badge/Architecture-Instruction--Only-2563EB?style=flat-square)
![Four Roles](https://img.shields.io/badge/Roles-4-7C3AED?style=flat-square)
![Schema Version](https://img.shields.io/badge/Schema-v1-059669?style=flat-square)

</div>

---

## 为什么是 DevFlow

普通 Agent 很擅长写代码，但复杂需求的真正难点往往不是“能不能写”，而是：

- 是否先弄清了既有仓库的真实入口、规则和历史包袱；
- 需求、测试、实现和审核是否绑定同一个版本与 Commit SHA；
- 谁可以修改什么，谁不能审核自己的工作；
- 环境、凭据或 live 服务缺失时，能否只阻塞受影响切片；
- 多轮审核是否会一次次释放新问题，导致成本和等待失控；
- 中途换 Agent 后，能否从持久证据恢复，而不是重新阅读整段对话。

RIC DevFlow Skills 把这些约束做成一套纯指令式工作流。它不增加新的 CLI、守护进程或状态机，而是让 Codex 在使用目标仓库原有 Git、测试、CI 和发布工具时，遵循清晰的角色边界、G0–G10 门禁和不可变证据规则。

## 一眼看懂

| 角色 | 默认调用方式 | 核心职责 | 写入边界 |
|---|---|---|---|
| `devflow-planner` | 可隐式调用，也可显式调用 | 需求接收、仓库接管、Spec、Task DAG、状态、调度、归因、合并与关闭 | 规划/状态产物和已过门禁的 Git 协调；不写生产代码 |
| `devflow-reviewer` | 仅显式调用或由 Planner 委派 | 独立审核基线、Spec、测试计划、代码和发布证据 | 只读；只返回 `APPROVE`、`REQUEST_CHANGES` 或 `BLOCKED` |
| `devflow-tester` | 仅显式调用或由 Planner 委派 | 测试计划、特征测试、集成/E2E/回归验证、缺陷证据 | 只写测试及自身证据；不改生产代码 |
| `devflow-implementer` | 仅显式调用或由 Planner 委派 | 在一个已批准 Task 和变更预算内完成最小完整实现 | 只处理获批范围；不改 Spec、不自审、不合并 |

```mermaid
flowchart LR
    U[用户需求] --> P[Planner<br/>预检、分期、Spec 与 DAG]
    P --> R1[Reviewer<br/>Spec / Test Review]
    R1 --> A[用户批准]
    A --> T[Tester<br/>计划与独立验证]
    T --> I[Implementer<br/>单 Task 实现]
    I --> R2[Reviewer<br/>Code Review]
    R2 --> P2[Planner<br/>按依赖顺序集成]
    P2 --> T2[Tester<br/>集成与 Smoke]
    T2 --> R3[Reviewer<br/>Release Review]
    R3 --> D[完成]
```

## 核心卖点

### 1. Brownfield 是一等公民

DevFlow 不假设项目从零开始。进入既有仓库时，它会先识别生效规则、真实目标分支、脏工作区、基线失败、相邻实现、契约和变更预算；续接半成品时，还会区分已接受、未验证、部分完成、Stub、冲突和未知工作。

### 2. 证据跟着版本和 SHA 走

Spec、用户批准、Review、Implementation Report 和 Test Report 都有明确身份。代码审核绑定精确的 `base_sha..head_sha`，测试绑定 `tested_sha`，发布审核绑定集成 SHA。分支名移动、行为变化或证据过期时，对应门禁会失效，而不是继续沿用一份“看起来通过”的旧结论。

### 3. 真正的职责隔离

Planner 不写生产代码，Reviewer 不修改被审核对象，Tester 不替 Implementer 修代码，Implementer 不改变需求也不合并。角色分离不是形式：它直接限制文件写入、批准权限和状态所有权。

### 4. 大需求先分期，再进入正式审核

首次 `SPEC_REVIEW` 前先做有界事实预检和规模判断。多个可独立交付价值、权限边界或环境边界会在 G2 前分期，同时保留完整能力清单、依赖与延期项；原子行为不会被机械拆碎。

### 5. 更小的上下文，更快的恢复

角色交接只传 Root Issue、动作、精确产物路径/版本、适用 SHA、当前 Finding/Defect、允许与保护路径、输出和停止条件。完整材料从持久产物读取；平台支持时默认使用最小上下文继承，例如 `fork_turns: "none"`。

### 6. Reviewer 循环可收敛

Reviewer 必须在一轮中完成全部适用维度，并一次性返回当时可发现的全部 P0/P1/P2 Finding。窄修正优先由同一 Reviewer 做 Delta 复审；同一对象连续两次 `REQUEST_CHANGES` 仍未收敛时，Planner 先停下来归因，而不是继续制造新版本。

### 7. 不把 Mock 当成上线证明

测试计划明确区分合成/Mock、本地、集成、live 和生产验证。缺少凭据或环境时输出精确的 `BLOCKED` 证据；可以独立验收的切片单独阻塞，低权威结果不能冒充完整验收。

## 快速开始

### 前置条件

- 已安装支持 Skills 与 Custom Agents 的 Codex；
- 本机有 Git；如使用下方 `gh repo clone` 命令，还需要已登录的 GitHub CLI；
- 如从私有仓库安装，当前 GitHub 身份必须拥有仓库读取权限；
- 目标项目仍应保留自己的 `AGENTS.md`、构建、测试、CI 和发布规则，DevFlow 会读取并映射它们，不会取代它们。

### 1. 获取仓库

```bash
gh repo clone lichong-a/ric-dev-workflow-skills
cd ric-dev-workflow-skills
```

也可以使用已配置凭据的 HTTPS 或 SSH：

```bash
git clone https://github.com/lichong-a/ric-dev-workflow-skills.git
```

### 2. 用户全局安装（推荐）

把五个目录链接到用户级 Skill 目录。四个角色会被发现，`_devflow_shared` 只作为共享资源库，不会成为第五个 Skill。

```bash
DEVFLOW_SOURCE_DIR="$(pwd)"
mkdir -p "${HOME}/.agents/skills"

for skill_name in \
  devflow-planner \
  devflow-reviewer \
  devflow-tester \
  devflow-implementer \
  _devflow_shared
do
  source_path="${DEVFLOW_SOURCE_DIR}/.agents/skills/${skill_name}"
  target_path="${HOME}/.agents/skills/${skill_name}"

  if [ -e "${target_path}" ] || [ -L "${target_path}" ]; then
    printf '保留已有路径，请先人工核对：%s\n' "${target_path}"
  else
    ln -s "${source_path}" "${target_path}"
  fi
done
```

然后安装 Custom Agent 配置。下面的命令不会覆盖同名文件；若提示已存在，请先使用 `diff -u` 审阅，再决定是否更新。

```bash
mkdir -p "${HOME}/.codex/agents"
cp --no-clobber .codex/agents/*.toml "${HOME}/.codex/agents/"
```

最后，把以下配置合并到 `${HOME}/.codex/config.toml`。如果已经存在 `[agents]` 段，请更新其中的键，不要重复追加同名 TOML 表。

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 6
```

仓库自带的推理强度配置偏向稳健交付：Planner 为 `max`、Reviewer 为 `high`、Tester 和 Implementer 为 `xhigh`；未硬编码具体模型，会沿用你的 Codex 模型配置。你可以按预算调整，但 Reviewer 建议至少保留 `high`，高风险变更不建议用低推理强度换速度。

> 符号链接依赖当前 clone 路径。移动或删除仓库前，应先更新用户目录中的链接。安装或更新后，建议重新启动 Codex 或新建任务，确保 Skills 与 Custom Agents 被重新加载。

### 3. 仅在单个项目中使用

如果不希望全局启用，可以只把本仓库 `.agents/skills/` 中的五个目录复制或链接到目标项目的 `.agents/skills/`，并把 `.codex/agents/*.toml` 合并到目标项目的 `.codex/agents/`。再将 `[agents]` 配置合并到目标项目的 `.codex/config.toml`。

不要覆盖目标项目已有的 `.agents`、`.codex` 或 `AGENTS.md`；逐项合并并保留更具体的项目规则。DevFlow 的运行期证据会写入目标仓库的 `.devflow/changes/<REQ-ID>/`。

## 怎么用

### 最常用：把开发请求交给 Planner

Planner 允许隐式调用。正常描述目标、约束和完成条件即可：

```text
请在当前项目增加订单导出能力，保持现有权限和 API 兼容；完成实现、测试和审核后交付。
```

也可以显式指定：

```text
使用 $devflow-planner 接管这个 Brownfield 需求。先核对仓库现状和既有流程，再完成分期、Spec、测试、实现与发布审核。
```

Planner 会先建立事实和证据，必要时才向你请求产品行为、敏感权限、生产操作或不可逆决策的确认。

### 独立审核

Reviewer 必须指定单一审核模式和精确对象：

```text
使用 $devflow-reviewer，以 CODE_REVIEW 模式审核 REQ-20260905-001 / TASK-003。
Spec 为 .devflow/changes/REQ-20260905-001/specs/spec-v2.md，
审核范围为 <base_sha>..<head_sha>，请一次性返回全部可发现的阻断 Finding。
```

支持的模式为 `BASELINE_REVIEW`、`SPEC_REVIEW`、`TEST_REVIEW`、`CODE_REVIEW` 和 `RELEASE_REVIEW`。

### 独立测试

```text
使用 $devflow-tester，为已批准的 Spec v2 创建测试计划。
请区分本地、集成和 live 验证边界，并把每条验收标准映射到稳定 Test Case ID。
```

### 单 Task 实现

```text
使用 $devflow-implementer，只实现 TASK-003。
严格遵守 Task 中的 base_sha、允许路径、保护路径、验收标准和变更预算；
完成后输出 Implementation Report，不要合并。
```

通常不需要手动逐个调用后三个角色；让 Planner 使用紧凑交接完成调度即可。显式调用更适合独立审计、测试补证或已经存在获批 Task 的场景。

## 产物与门禁

每个交付请求在目标仓库中使用独立目录：

```text
.devflow/changes/REQ-YYYYMMDD-NNN/
├── state.yaml
├── intake.md
├── repository-profile.md
├── root-issue.md
├── specs/
├── tasks/
├── tests/
├── reviews/
├── implementations/
├── defects/
└── reports/
```

规划草稿在首次正式送审前可以原地完善。一旦送交独立 Reviewer、产生审核结论或被 `state.yaml` 引用，就会冻结；被拒绝的版本也不能覆盖，修正必须发布新版本并保留 `supersedes` 谱系。

完整流程保留 G0–G10 门禁，覆盖需求、仓库基线、Spec、用户批准、测试计划、实现、代码审核、集成验证、发布审核和合并后 Smoke。FAST 模式可以压缩产物，但不会取消角色分离、必要批准或 SHA 绑定。

## 适用场景

特别适合：

- 已有代码、历史决策、脏工作区或半成品需要接管的 Brownfield 项目；
- 跨前后端、数据库、权限、基础设施或多个环境的功能；
- 对审计、兼容、证据、回滚和职责分离有要求的团队；
- 容易在多 Agent 协作中出现上下文膨胀、重复审核或范围漂移的任务；
- 希望保留 Codex 自主执行能力，同时让高风险动作受门禁约束的项目。

它不是所有任务都需要的仪式。纯解释、一次性文案、极小的无风险编辑或没有软件交付行为的请求，不应触发 Planner。DevFlow 也不会替代目标仓库自己的 CI、Issue 系统、发布审批或安全策略；语义等价的既有证据会被引用，而不是再维护一套竞争事实。

## 目录结构

```text
.
├── .agents/skills/
│   ├── devflow-planner/
│   ├── devflow-reviewer/
│   ├── devflow-tester/
│   ├── devflow-implementer/
│   └── _devflow_shared/
├── .codex/
│   ├── agents/
│   └── config.toml
├── .devflow/README.md
└── docs/
    ├── DEVFLOW_SKILLS_DESIGN.md
    └── DEVFLOW_SKILLS_VALIDATION.md
```

- 四个 `SKILL.md` 是聚焦的角色入口；
- `_devflow_shared` 保存共享契约、模板、参考和场景评测，但没有 `SKILL.md`；
- `.codex/agents/` 定义 Custom Agent 的推理强度、Sandbox 和角色约束；
- `.devflow/README.md` 说明目标项目中的运行期证据布局；
- `docs/` 保存完整设计与真实验证报告。

## 验证

当前版本已通过：

- 四个 Skill 的 Codex `quick_validate.py`：4/4；
- YAML 11/11、TOML 5/5 解析；
- 64 个 Markdown 文档、46 个本地相对链接检查，0 断链；
- 可发现的 Skill 恰好四个，且 `_devflow_shared` 不可发现；
- Reviewer 项目级与用户全局配置均为 `model_reasoning_effort = "high"`；
- `codex --strict-config doctor`：0 个配置失败；
- 两个隔离 Brownfield demo，包括分期、局部 live 阻塞、Finding 一次性披露、同 Reviewer 增量复审、送审冻结和协议映射。

你可以在 clone 后重新运行基础验证：

```bash
VALIDATOR_PATH="${CODEX_HOME:-${HOME}/.codex}/skills/.system/skill-creator/scripts/quick_validate.py"

for skill_name in \
  devflow-planner \
  devflow-reviewer \
  devflow-tester \
  devflow-implementer
do
  python3 "${VALIDATOR_PATH}" ".agents/skills/${skill_name}"
done

codex --strict-config doctor --summary --no-color --ascii
```

完整设计见 [DevFlow Skills 总体设计](docs/DEVFLOW_SKILLS_DESIGN.md)，实际命令、结果和演示边界见 [验证报告](docs/DEVFLOW_SKILLS_VALIDATION.md)。

## 安全与兼容承诺

- 保持 `schema_version: 1`，不要求迁移已有 `.devflow` 记录；
- 不硬编码目标分支、语言、框架、Issue 平台或具体模型；
- 不把 Token、Cookie、私钥、生产数据或未脱敏日志写入证据；
- 不用关闭规则、删除测试、无限重试或扩大超时制造假通过；
- 不自动进行生产发布、生产数据写入、付费操作或未授权的凭据访问；
- 不覆盖用户既有未提交修改，不用破坏性 Git 命令清理工作区。

## 当前状态

这是一个私有、纯指令式的 Codex Skill 包。当前版本已完成设计、静态门禁和隔离 Brownfield 行为演示；下一阶段最有价值的反馈，是在真实项目中持续记录审核轮次、交接大小、版本数量和端到端耗时，并据此做有证据的迭代。

如果你希望 Codex 不只是“交出代码”，而是交出一条经得起接管、复核和回滚的软件交付链路，这套 Skills 就是为此准备的。
