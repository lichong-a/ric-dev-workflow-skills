# Root Issue — <REQ-ID>：<标题>

版本：`1`
状态：`DRAFT`

```yaml
project_context: "GREENFIELD"
work_type: "feature"
delivery_mode: "STANDARD"
risk_level: "medium"
```

## 背景与问题

<当前情况及用户可见问题。>

## 目标与用户价值

<期望结果及其价值。>

## 范围

- <纳入的行为>

## 非目标

- <明确排除的行为>

## 必须保持的行为

- <不得改变的公开或隐含行为>

## 约束与依赖

- <仓库、授权、环境、顺序或外部约束>

## 风险与回滚方向

| 风险 | 预防/检测 | 回滚或恢复 |
|---|---|---|
| <风险> | <控制> | <动作> |

## 变更预算

```yaml
change_radius: "local"
allowed_paths: []
protected_paths: []
public_contract_changes: "none"
database_changes: "none"
dependency_changes: "forbidden"
framework_upgrade: "forbidden"
build_system_changes: "forbidden"
broad_rename_or_move: "forbidden"
formatting_scope: "touched_lines_or_files_only"
generated_files: "generator_only"
refactor_policy: "required_only"
max_parallel_writers_per_area: 1
```

## 完成标准

- [ ] <可观察的验收结果>
- [ ] 所有适用门禁都有当前有效且正确绑定的证据。
- [ ] 目标分支冒烟测试通过，且不存在剩余阻塞项。
