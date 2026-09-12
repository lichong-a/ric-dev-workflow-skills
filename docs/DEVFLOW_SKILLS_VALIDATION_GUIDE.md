# DevFlow Skill 包验证指南

本指南保存可复跑的维护者检查片段；[验证报告](DEVFLOW_SKILLS_VALIDATION.md)记录实际结果。片段不是安装后的 CLI 或流程编排器，不修改业务状态。运行期场景见[工作流评测](../skills/ric-devflow/references/evals/workflow-cases.md)与[触发评测](../skills/ric-devflow/references/evals/trigger-cases.md)。独立 Agent 评测必须给原始输入，事后按可观察决定及文件结果判定，不能给执行者 Oracle 或把关键词命中当行为通过。

## 环境、对象与历史边界

- 在源码仓库根执行，要求 Bash、Git、Python 3.11+、PyYAML 和 bundled `quick_validate.py`；每次 Python 命令设置 UTF-8，拒绝优化模式。缺依赖记录 BLOCKED，不自动安装全局工具。
- 如 bundled 验证器不在默认用户目录，在当前命令设置 `DEVFLOW_VALIDATOR_PATH` 为实际路径；不改用户配置。Node/标准安装器、PowerShell 与原生宿主未运行时单列，不以普通复制代替实机证据。
- 五 Skill 唯一发布源为 `skills/`。检查包括仍存在的跟踪文件及 `skills/`、`docs/` 新文件；删除的跟踪路径另列而不当作待读取文件。用户 `.zcode/plans/` 等未知未跟踪内容不纳入。
- 历史报告的前 63575 字节来自完整基线 `a72592ec1cb7ac5c67ec5454233d673a30000be5`，必须逐字保留。仅该原字节段内的链接按基线 Git 对象解析；报告新增段、当前文档及整个发布包的链接仍按当前文件检查。浅克隆缺该对象时应取得所需历史对象或报告 BLOCKED，不能豁免。
- 输出完整 HEAD、文件路径/类型/模式/字节摘要与集合摘要、真实命令和退出码。报告自身参与链接/历史保全检查，但不进入集合摘要，避免自引用。工作区摘要和测试 fixture Commit 均不能当产品 `tested_sha`。
- 第二段只在源码根下创建并精确清理本段的临时目录；不会写用户 Skill/Agent 目录。真实 bootstrap 的冲突、缺件、恢复与路由由独立 Agent 场景另验，不另写一套安装算法来“证明”指令正确。

## 完整静态、安装闭包与保持行为

复制执行以下完整 Bash 块。`DEVFLOW_CHECK_ROOT` 仅供下一段把相同检查定向到隔离副本；基线 Git 对象始终来自命令 cwd 的真实源码仓库。链接解析覆盖本包使用的行内 Markdown 链接、标题锚点与显式 HTML ID，不声称支持任意 Markdown 扩展或核验远程网页内容。

```bash
(
set -Eeuo pipefail
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B - <<'PY'
import hashlib, json, os, re, stat, subprocess, sys, tomllib
from pathlib import Path
from urllib.parse import unquote, urlsplit
import yaml

assert not sys.flags.optimize, 'disable PYTHONOPTIMIZE: checks require assertions'
origin = Path.cwd().resolve()
root = Path(os.environ.get('DEVFLOW_CHECK_ROOT', str(origin))).resolve()
base = 'a72592ec1cb7ac5c67ec5454233d673a30000be5'
report_name = 'docs/DEVFLOW_SKILLS_VALIDATION.md'
roles = ('planner', 'reviewer', 'tester', 'implementer')
names = {'ric-devflow', *('ric-devflow-' + r for r in roles)}
commands = []
def run(argv):
    proc = subprocess.run(argv, cwd=origin, capture_output=True, text=True)
    commands.append(dict(argv=argv, cwd=str(origin), exit_code=proc.returncode,
                         stdout=proc.stdout, stderr=proc.stderr))
    assert proc.returncode == 0, ('command failed', argv, proc.stdout, proc.stderr)
    return proc.stdout

def git_bytes(path):
    return subprocess.check_output(['git', 'show', base + ':' + path], cwd=origin)

head = run(['git', 'rev-parse', 'HEAD']).strip()
tracked = set(filter(None, run(['git', 'ls-files', '-z']).split('\0')))
if root == origin:
    extras = set(filter(None, run(['git', 'ls-files', '--others', '--exclude-standard', '-z']).split('\0')))
    requested = tracked | {p for p in extras if p.startswith(('skills/', 'docs/'))}
    deleted = sorted(p for p in tracked if not (root / p).exists() and not (root / p).is_symlink())
    paths = sorted(p for p in requested if (root / p).is_file() or (root / p).is_symlink())
else:
    paths = sorted(p.relative_to(root).as_posix() for p in root.rglob('*') if p.is_file() or p.is_symlink())
    deleted = []
assert {p.name for p in (root / 'skills').iterdir()} == names, 'unexpected Skill directory'
assert {p.parent.name for p in (root / 'skills').rglob('SKILL.md')} == names, 'extra or missing Skill'
assert not any(p.endswith('/SKILL.md') and not p.startswith('skills/') for p in paths), 'old Skill alias'
assert not any((root / p).is_symlink() for p in paths if p.startswith('skills/')), 'published symlink dependency'
assert not any(Path(p).suffix not in ('.md', '.yaml', '.toml') for p in paths if p.startswith('skills/')), 'non-instruction product file'
assert not any((root / p).exists() for p in ('package.json', 'pyproject.toml', 'Cargo.toml')), 'unexpected package manifest'

class UniqueLoader(yaml.SafeLoader):
    pass
def unique(loader, node, deep=False):
    value = {}
    for key_node, val_node in node.value:
        key = loader.construct_object(key_node, deep=deep)
        assert key not in value, 'duplicate YAML key: ' + str(key)
        value[key] = loader.construct_object(val_node, deep=deep)
    return value
UniqueLoader.add_constructor(yaml.resolver.BaseResolver.DEFAULT_MAPPING_TAG, unique)
def load_yaml(text):
    return yaml.load(text, Loader=UniqueLoader)

def prose(text):
    lines, fence = [], None
    for line in text.splitlines():
        match = re.match(r'^\s*(`{3,}|~{3,})(.*)$', line)
        if match:
            mark, tail = match.groups()
            if fence is None:
                fence = (mark[0], len(mark))
            elif mark[0] == fence[0] and len(mark) >= fence[1] and not tail.strip():
                fence = None
            continue
        if fence is None:
            lines.append(line)
    assert fence is None, 'unclosed fenced block'
    return '\n'.join(lines)

def anchors(text):
    body = prose(text)
    ids = set(re.findall(r'<a\s+id=[\x22\x27]([^\x22\x27]+)', body))
    used = {}
    for title in re.findall(r'^#{1,6}\s+(.+)$', body, re.M):
        title = re.sub(r'\[([^]]+)\]\([^)]+\)', r'\1', title)
        slug = re.sub(r'[^\w\- ]', '', title.lower().strip()).replace(' ', '-')
        count = used.get(slug, 0)
        used[slug] = count + 1
        ids.add(slug if not count else slug + '-' + str(count))
    return ids

link_re = re.compile(r'(?<!!)\[[^\]\n]*\]\(([^)\n]+)\)')
counts = dict(yaml=0, toml=0, frontmatter=0, markdown=0, current_links=0, historical_links=0)
manifest, fronts, edges = [], {}, {}
historical = git_bytes(report_name)
assert len(historical) == 63575
assert (root / report_name).read_bytes().startswith(historical), 'historical report prefix changed'

for name in paths:
    path = root / name
    raw = os.readlink(path).encode() if path.is_symlink() else path.read_bytes()
    mode = 'symlink' if path.is_symlink() else oct(stat.S_IMODE(path.stat().st_mode))
    if name != report_name:
        manifest.append(dict(path=name, mode=mode, sha256=hashlib.sha256(raw).hexdigest()))
    if path.is_symlink() or path.suffix not in ('.md', '.yaml', '.yml', '.toml'):
        continue
    text = raw.decode('utf-8')
    if path.suffix in ('.yaml', '.yml'):
        load_yaml(text)
        counts['yaml'] += 1
    elif path.suffix == '.toml':
        tomllib.loads(text)
        counts['toml'] += 1
    elif path.suffix == '.md':
        counts['markdown'] += 1
        front = re.match(r'^---\n(.*?)\n---(?:\n|$)', text, re.S)
        if front:
            fronts[name] = load_yaml(front.group(1))
            counts['frontmatter'] += 1
        pieces = [(False, text)] if name != report_name else [(True, historical.decode()), (False, raw[len(historical):].decode())]
        edges[name] = []
        for is_history, body in pieces:
            for match in link_re.finditer(prose(body)):
                target = match.group(1).strip().strip('<>')
                parsed = urlsplit(target)
                if parsed.scheme or parsed.netloc:
                    continue
                destination = (path.parent / unquote(parsed.path)).resolve() if parsed.path else path
                assert destination.is_relative_to(root), (name, 'link outside checked tree', target)
                relative = destination.relative_to(root).as_posix()
                if is_history:
                    # Exact old bytes are resolved against their preserved baseline only.
                    old_raw = git_bytes(relative)
                    if parsed.fragment:
                        assert unquote(parsed.fragment) in anchors(old_raw.decode()), (name, 'historical anchor', target)
                    counts['historical_links'] += 1
                else:
                    assert destination.is_file(), (name, 'missing link', target)
                    if name.startswith('skills/'):
                        assert relative.startswith('skills/'), (name, 'runtime dependency outside skills', target)
                    if parsed.fragment:
                        assert unquote(parsed.fragment) in anchors(destination.read_text()), (name, 'missing anchor', target)
                    edges[name].append(relative)
                    counts['current_links'] += 1

validator = Path(os.environ.get('DEVFLOW_VALIDATOR_PATH', str(Path(os.environ.get('CODEX_HOME', str(Path.home() / '.codex'))) / 'skills/.system/skill-creator/scripts/quick_validate.py')))
assert validator.is_file(), 'missing bundled validator'
for name in sorted(names):
    assert fronts['skills/' + name + '/SKILL.md']['name'] == name, 'frontmatter name mismatch'
    run([sys.executable, '-B', str(validator), str(root / 'skills' / name)])
    ui = load_yaml((root / 'skills' / name / 'agents/openai.yaml').read_text())
    assert ui['policy']['allow_implicit_invocation'] is (name == 'ric-devflow')
    assert 'RIC DevFlow' in ui['interface']['display_name']

closures = {}
for role in roles:
    seen, todo = set(), ['skills/ric-devflow-' + role + '/SKILL.md']
    while todo:
        current = todo.pop()
        if current not in seen:
            seen.add(current)
            todo.extend(edges.get(current, []))
    assert all(p.split('/')[1] in ('ric-devflow', 'ric-devflow-' + role) for p in seen), (role, 'unrelated role dependency')
    closures[role] = sorted(seen)
    config = tomllib.loads((root / 'skills/ric-devflow/assets/agents/codex' / ('ric-devflow-' + role + '.toml')).read_text())
    old = tomllib.loads(git_bytes('.codex/agents/devflow-' + role + '.toml').decode())
    assert config['name'] == 'ric_devflow_' + role
    for key in ('model', 'model_reasoning_effort', 'sandbox_mode'):
        assert config.get(key) == old.get(key), (role, key)
    for host, oldhost in (('claude-code', '.claude'), ('zcode', '.zcode')):
        p = 'skills/ric-devflow/assets/agents/' + host + '/ric-devflow-' + role + '.md'
        oldfront = load_yaml(git_bytes(oldhost + '/agents/devflow-' + role + '.md').decode().split('---', 2)[1])
        assert fronts[p]['name'] == 'ric-devflow-' + role
        for key in ('model', 'tools', 'injectAgentsMd'):
            assert fronts[p].get(key) == oldfront.get(key), (host, role, key)

template_prefix = '.agents/skills/_devflow_shared/templates/'
templates = run(['git', 'ls-tree', '-r', '--name-only', base, template_prefix]).splitlines()
for old in templates:
    new = root / 'skills/ric-devflow/assets/templates' / old[len(template_prefix):]
    assert new.read_bytes() == git_bytes(old), ('template bytes changed', old)
if root == origin:
    run(['git', 'diff', '--check'])
    run(['git', 'diff', '--cached', '--check'])
identity = hashlib.sha256(json.dumps(manifest, ensure_ascii=False, sort_keys=True).encode()).hexdigest()
print(json.dumps(dict(verdict='PASS', head=head, tested_sha=None, candidate_sha256=identity,
    python=sys.version.split()[0], pyyaml=yaml.__version__, baseline_sha=base,
    counts=counts, templates_byte_identical=len(templates), role_closures=closures,
    deleted_tracked_paths=deleted, files=manifest, commands=commands,
    native_runtime=False, boundary='source/isolated file diagnostic; no product Commit Gates'), ensure_ascii=False, indent=2))
PY
)
```

## README 普通复制与检查器负控

第二段读取第一段 Python 原文作为同一检查器，在各自临时副本中执行真实子进程并检查退出码。只替换 README 复制块的目标参数和“仅入口”目录列表，不重写复制逻辑；已有目标的非零退出是该示例明确规定的行为，不能当 bootstrap 幂等恢复结果。负控每例独立注入，干净对照先通过。

```bash
(
set -Eeuo pipefail
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B - <<'PY'
import hashlib, json, os, re, shlex, shutil, subprocess, sys, tempfile
from pathlib import Path
assert not sys.flags.optimize
root = Path.cwd()
guide = (root / 'docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md').read_text()
check_source = guide.split("python3 -B - <<'PY'\n", 1)[1].split('\nPY\n)', 1)[0]
readme = (root / 'README.md').read_text()
copy_block = next(b for b in re.findall(r'```bash\n(.*?)\n```', readme, re.S) if 'DEVFLOW_COPY_NAMES=' in b)
assignment = 'DEVFLOW_COPY_ROOT="/path/to/confirmed/skill-root"'
assert copy_block.count(assignment) == 1
roles = ('planner', 'reviewer', 'tester', 'implementer')
names = ['ric-devflow', *('ric-devflow-' + r for r in roles)]
commands = []
def run(argv, env=None):
    p = subprocess.run(argv, cwd=root, env=env, text=True, capture_output=True)
    commands.append(dict(argv=argv, cwd=str(root), exit_code=p.returncode, stdout=p.stdout, stderr=p.stderr))
    return p

def files(directory):
    return {p.relative_to(directory).as_posix(): ('link:' + os.readlink(p) if p.is_symlink() else hashlib.sha256(p.read_bytes()).hexdigest())
            for p in directory.rglob('*') if p.is_file() or p.is_symlink()}

with tempfile.TemporaryDirectory(prefix='.devflow-validation-', dir=root) as location:
    temp = Path(location)
    results = []
    for selected in (names, ['ric-devflow']):
        target = temp / ('copy-five' if len(selected) == 5 else 'copy-entry')
        block = copy_block.replace(assignment, 'DEVFLOW_COPY_ROOT=' + shlex.quote(str(target)))
        block = block.replace('DEVFLOW_COPY_NAMES=(' + ' '.join(names) + ')', 'DEVFLOW_COPY_NAMES=(' + ' '.join(selected) + ')')
        first = run(['bash', '-c', block])
        assert first.returncode == 0, first.stderr
        before = files(target)
        expected = {name + '/' + p: digest for name in selected for p, digest in files(root / 'skills' / name).items()}
        assert before == expected and not any(v.startswith('link:') for v in before.values())
        repeat = run(['bash', '-c', block])
        assert repeat.returncode != 0 and files(target) == before
        results.append(dict(case=target.name, first_exit=first.returncode, repeat_exit=repeat.returncode, files=len(before)))
    conflict = temp / 'copy-conflict'
    conflict.mkdir()
    (conflict / 'ric-devflow-tester').write_text('USER-CUSTOM\n')
    block = copy_block.replace(assignment, 'DEVFLOW_COPY_ROOT=' + shlex.quote(str(conflict)))
    before = files(conflict)
    rejected = run(['bash', '-c', block])
    assert rejected.returncode != 0 and files(conflict) == before
    results.append(dict(case='copy-conflict', exit_code=rejected.returncode, no_partial_write=True))
    # Source selection is identical to block one and includes new deliverable files.
    tracked = set(filter(None, subprocess.check_output(['git', 'ls-files', '-z']).decode().split('\0')))
    extras = set(filter(None, subprocess.check_output(['git', 'ls-files', '--others', '--exclude-standard', '-z']).decode().split('\0')))
    paths = sorted(p for p in tracked | {p for p in extras if p.startswith(('skills/', 'docs/'))} if (root / p).is_file())
    for case in ('control', 'duplicate-yaml', 'broken-toml', 'extra-skill', 'bad-link', 'bad-anchor', 'history-prefix', 'bad-frontmatter', 'validator-failure'):
        fixture = temp / case
        for name in paths:
            dst = fixture / name
            dst.parent.mkdir(parents=True, exist_ok=True)
            shutil.copyfile(root / name, dst)
        if case == 'duplicate-yaml':
            p = fixture / 'skills/ric-devflow/agents/openai.yaml'
            p.write_text(p.read_text() + '\npolicy: {}\n')
        elif case == 'broken-toml':
            p = fixture / 'skills/ric-devflow/assets/agents/codex/config.toml'
            p.write_text(p.read_text() + '\n[broken\n')
        elif case == 'extra-skill':
            p = fixture / 'skills/unexpected/SKILL.md'
            p.parent.mkdir()
            p.write_text('---\nname: unexpected\ndescription: fixture\n---\n')
        elif case in ('bad-link', 'bad-anchor'):
            p = fixture / 'docs/new-untracked-negative.md'
            p.write_text('[negative](does-not-exist.md)\n' if case == 'bad-link' else '[negative](DEVFLOW_SKILLS_DESIGN.md#missing-fixture-anchor)\n')
        elif case == 'history-prefix':
            p = fixture / 'docs/DEVFLOW_SKILLS_VALIDATION.md'
            p.write_bytes(b'!' + p.read_bytes()[1:])
        elif case == 'bad-frontmatter':
            p = fixture / 'skills/ric-devflow/SKILL.md'
            p.write_text(p.read_text().replace('name: ric-devflow\n', 'name: wrong-name\n', 1))
        elif case == 'validator-failure':
            p = fixture / 'skills/ric-devflow/SKILL.md'
            p.write_text(re.sub(r'^description:.*\n', '', p.read_text(), count=1, flags=re.M))
        env = dict(os.environ, DEVFLOW_CHECK_ROOT=str(fixture), PYTHONUTF8='1', PYTHONIOENCODING='utf-8')
        observed = run([sys.executable, '-B', '-c', check_source], env)
        assert (observed.returncode == 0) == (case == 'control'), (case, observed.stdout, observed.stderr)
        results.append(dict(case=case, exit_code=observed.returncode))
    print(json.dumps(dict(verdict='PASS', checks=results, commands=commands, native_runtime=False,
                         note='negative exits are expected test observations, not hidden failures'), ensure_ascii=False, indent=2))
assert not temp.exists()
print('Created temporary fixture cleanup: PASS')
PY
)
```

## 独立 Agent 的三批原始场景与增量复验

每批由 Root Planner 直接派发独立执行者，不传本节的判定说明；Tester 事后审核。先固定候选文件集与真实存在的隔离 Git 对象，原始输入和输出随报告保存。临时 Git 仅作测试数据，使用命令级合成作者，不提交源码仓库或修改用户 Git 配置。生成 fixture 时先留初始清单，真正处理请求的动作另记。

| 批次 | 给执行者的原始事实/请求 | 事后观察维度 |
|---|---|---|
| A 缺件 | 仅入口；四个单角色分别安装；Reviewer/Tester/Implementer最小闭包；必要传递文件缺失；已启动子角色被要求补装/派发；离线无可信源。原任务为保持权限/响应兼容的CSV导出 | 首次读取与下一动作、真实复制/读回、必要依赖、来源、角色上下文、原任务保全 |
| B 文件边界 | project/user入口分别链接到另一处可信source；范围未知；合成ZCode项目范围；同字节/定制/异版本/普通文件/断链；明确禁用及其他配置；部分安装后重复请求；离线完整/不完整源；合成获取成功/拒绝 | 逻辑写入清单、链接保留、冲突前后字节/mtime、全目标预检、读回/重复写入、合成与真实区分 |
| C 决定 | 开发/修复/续作/解释/建议/独立审核测试/文档/显式四角色；三宿主/未知/冲突身份；平级交接；文件/加载/调用/重载；无工具/权限；v1/v2、未提交测试、Oracle变化、冻结Task、VERIFIED/DONE各证据组合 | 真实选择与交接、权限/批准界限、唯一状态作者、保留历史、完整SHA绑定；不比对措辞 |
| 守卫 Delta | 入口存在但bootstrap缺失：正常、来源未知、现存入口字节冲突；四个单角色分别缺入口：正常、同名原生定义冲突、必需能力禁用 | 写前完整预检、真实缺件恢复、冲突/禁用时文件结果；新旧输入身份分别固定 |

实际文件子例不能只输出“将复制”。独立执行者可根据其决定执行逐文件复制，但不得另写通用 bootstrap 实现来替代被测指令。中断恢复区分预置的部分状态和本次真实故障：收集代码发生 KeyError 等错误必须保留原命令/退出码，修收集器后重新核对文件再恢复，不改候选或 Oracle。

真实拒绝测试须先证明底层拒绝发生：例如 chmod后重新读取mode并执行窄探针。若mode未变或探针成功，权限拒绝切片未运行；合成拒绝输入只能证明决定级响应。不能因为文件操作通过就声称权限沙箱、原生宿主加载/调用、远程获取或标准安装器已经通过。

候选变化后先核对精确Delta；未变契约/输入可引用原Tester结果，受影响缺件守卫重新执行对应真实fixture。原失败/阻塞记录保持原身份并保留，新证据单独发布。正式报告沿用原Schema和PASS/FAIL/BLOCKED；源码隔离结论与因无产品Commit/原生环境而未执行的门禁切片分开说明。

标准CLI说明另做负向跟随检查：在合成canonical目标放置用户定制目录，在另一例宿主目标放置已有真实symlink，记录全部目标的lstat/readlink与哨兵摘要；按README的首次安装前提在首个CLI调用前停止，并核对字节/类型/mtime未变。该检查不启动CLI，不能证明安装器实现安全或它已运行。实际原始输入、输出及本轮机械核验见[独立评测证据](DEVFLOW_SKILLS_EVAL_20260913.md)。
