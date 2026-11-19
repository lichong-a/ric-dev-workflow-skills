# 工具与依赖最低要求

本页集中维护 DevFlow 安装、运行和源码验证的最低要求。读取所用场景对应的条目。业务仓库的技术栈和测试依赖由该仓库维护。

## 安装与运行

| 工具 | 最低要求 | 适用场景 |
| --- | --- | --- |
| Codex、Claude Code、ZCode | 暂无已核定的最低发行版本。当前宿主须能发现 Skill。完整研发还须能加载、调用原生子代理并返回结果。 | 选择实际使用的一个宿主，按 README 的宿主安装与加载步骤核验。 |
| `skills` 安装器 | 暂无已核定的最低发行版本。确认所选版本支持当前宿主、`add`、安装范围、来源及文档示例所需的选择参数。 | 通过 `npx skills` 安装时适用。 |
| Node.js、npm / `npx` | 下方已核验的安装器包清单要求 Node.js 22.20.0 或更新版本。所选 npm 须兼容该 Node.js 并提供 `npx`。 | 使用其他安装器发行版时，核对其 `package.json` 的 `engines.node`。 |
| Bash、文件工具或 PowerShell | 暂无已核定的最低发行版本。Bash 路径使用数组、`set -Eeuo pipefail`、`mkdir`、`cp` 和 `diff`。PowerShell 路径使用 `Copy-Item -LiteralPath -Recurse`。 | 仅在执行对应平台的普通复制命令时适用。 |

标准安装器的预检针对实际发行版的宿主目标与 canonical 目标。已核验的上游源码记录了两类清理行为及 AI Agent 环境中的自动非交互选择。 [安装实现源码](https://github.com/vercel-labs/skills/blob/d667282815248da03a08a18272b5d2eef9caf77c/src/installer.ts) · [调用与确认源码](https://github.com/vercel-labs/skills/blob/d667282815248da03a08a18272b5d2eef9caf77c/src/add.ts) · [同一源码的包清单](https://github.com/vercel-labs/skills/blob/d667282815248da03a08a18272b5d2eef9caf77c/package.json)

## 源码验证

| 工具 | 最低要求 | 适用场景 |
| --- | --- | --- |
| Bash 和文件工具 | 使用支持上表命令与选项的环境。 | 执行验证指南的 Bash 检查块。 |
| Git | 验证片段需要 Git 对象读取能力。运行独立 worktree 复验块时，最低为 Git 2.28，该块使用 `git init -b`。 | 对象验证需要指南所列的基线对象。[Git 上游命令文档](https://git-scm.com/docs/git-init/2.28.0) |
| Python | 3.11。检查器使用标准库 `tomllib`。 | 执行验证指南的 Python 检查片段。[Python 标准库文档](https://docs.python.org/3/library/tomllib.html) |
| PyYAML | 使用支持当前 Python 和平台的发行版，须提供 `yaml.SafeLoader` 和 `safe_dump(..., sort_keys=False)`。暂无独立核定的最低发行版本。 | YAML 检查与传输验证。 |
| Codex 自带的 `quick_validate.py` | 使用宿主随附的 Skill 验证器，脚本须可用所选解释器运行。 | 六个 Skill 的元数据检查。 |
| 原生宿主验证工具 | 仅在执行对应实机场景时需要。当前宿主须提供场景所需接口，例如 Codex CLI 的 `exec --json`。 | 记录实际工具反馈来核验加载、派发和终态。 |

历史验证报告记录实测环境。更新最低要求时应引用实际使用的命令、接口或上游声明。
