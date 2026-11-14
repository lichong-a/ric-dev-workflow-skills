# DevFlow Skill 包验证指南

本指南保存可复跑的维护者检查片段；[验证报告](DEVFLOW_SKILLS_VALIDATION.md)记录实际结果。片段不是安装后的 CLI 或流程编排器，不修改业务状态。运行期场景见[工作流评测](../skills/ric-devflow/references/evals/workflow-cases.md)与[触发评测](../skills/ric-devflow/references/evals/trigger-cases.md)。独立 Agent 评测必须给原始输入，事后按可观察决定及文件结果判定，不能给执行者 Oracle 或把关键词命中当行为通过。

## 环境、对象与历史边界

- 在源码仓库根执行，要求 Bash、Git、Python 3.11+、PyYAML 和 bundled `quick_validate.py`；每次 Python 命令设置 UTF-8，拒绝优化模式。缺依赖记录 BLOCKED，不自动安装全局工具。
- 如 bundled 验证器不在默认用户目录，在当前命令设置 `DEVFLOW_VALIDATOR_PATH` 为实际路径；不改用户配置。Node/标准安装器、PowerShell 与原生宿主未运行时单列，不以普通复制代替实机证据。
- 六 Skill 唯一发布源为 `skills/`，包含入口与五业务角色，三平台各四个原生子角色。检查包括仍存在的跟踪文件及 `skills/`、`docs/` 新文件；删除的跟踪路径另列而不当作待读取文件。用户 `.zcode/plans/` 等未知未跟踪内容不纳入。
- 历史报告的前 63575 字节来自完整基线 `a72592ec1cb7ac5c67ec5454233d673a30000be5`，必须逐字保留。仅该原字节段内的链接按基线 Git 对象解析；报告新增段、当前文档及整个发布包的链接仍按当前文件检查。浅克隆缺该对象时应取得所需历史对象或报告 BLOCKED，不能豁免。
- 原有 `ffe2e7ff6897539614bdddd055fe676865144b59` 报告前缀检查继续保留；同时对 `458b1778986b0f008aaa46b805af6f7baf6e6880` 和 `8e2059f29e86011f625316c524650b74aa5b95d0` 中整份已发布报告逐字检查，仅允许其后追加。新增 Researcher 不进入旧三个原生角色的历史元数据比较，而有独立的类型、ID、模型继承、只读工具和报告字段断言。
- 输出完整 HEAD、文件路径/类型/模式/字节摘要与集合摘要、真实命令和退出码。报告自身参与链接/历史保全检查，但不进入集合摘要，避免自引用。工作区摘要和测试 fixture Commit 均不能当产品 `tested_sha`。
- v1 模板保持历史基线字节；Compact 模板允许路径说明注释更新，state 仅调整仓库根与 current 的相对路径默认值，其余载荷保持兼容。模板兼容检查不证明宿主已按路径规则生成产物。
- 检查片段的原始工具返回用于会话内诊断；写入 `.devflow` 前由记录作者按[持久文件路径](../skills/ric-devflow/references/contracts/artifact-lifecycle.md#持久文件路径)处理 cwd、参数与输出中的本机定位并注明规范化方式，不直接把机器路径诊断保存为正式证据。
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
roles = ('planner', 'reviewer', 'tester', 'implementer', 'researcher')
child_roles = ('reviewer', 'tester', 'implementer', 'researcher')
historical_child_roles = ('reviewer', 'tester', 'implementer')
maintenance_base = '458b1778986b0f008aaa46b805af6f7baf6e6880'
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
previous_report = subprocess.check_output(['git', 'show', 'ffe2e7ff6897539614bdddd055fe676865144b59:' + report_name], cwd=origin)
assert (root / report_name).read_bytes().startswith(previous_report), 'previous published report changed'
maintenance_report = subprocess.check_output(['git', 'show', maintenance_base + ':' + report_name], cwd=origin)
assert (root / report_name).read_bytes().startswith(maintenance_report), 'published maintenance report changed'

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
    assert ui['policy']['allow_implicit_invocation'] is (name == 'ric-devflow'), (name, 'implicit invocation policy')
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

# Five business Skills and four native child roles. Planner runs in the main session.
agent_root = root / 'skills/ric-devflow/assets/agents'
for host, suffix in (('codex', '.toml'), ('claude-code', '.md'), ('zcode', '.md')):
    expected = {'ric-devflow-' + r + suffix for r in child_roles}
    if host == 'codex':
        expected.add('config.toml')
    assert {p.name for p in (agent_root / host).iterdir()} == expected, (host, 'native role set')
for role in child_roles:
    config = tomllib.loads((root / 'skills/ric-devflow/assets/agents/codex' / ('ric-devflow-' + role + '.toml')).read_text())
    assert config['name'] == 'ric_devflow_' + role, (role, 'native ID')
    assert 'model_reasoning_effort' not in config, (role, 'fixed reasoning effort')
    if role in historical_child_roles:
        old = tomllib.loads(git_bytes('.codex/agents/devflow-' + role + '.toml').decode())
        for key in ('model', 'sandbox_mode'):
            assert config.get(key) == old.get(key), (role, key)
    else:
        assert 'model' not in config, 'researcher model inheritance'
        assert config['sandbox_mode'] == 'read-only', 'researcher read-only sandbox'
    for host, oldhost in (('claude-code', '.claude'), ('zcode', '.zcode')):
        p = 'skills/ric-devflow/assets/agents/' + host + '/ric-devflow-' + role + '.md'
        assert fronts[p]['name'] == 'ric-devflow-' + role, (host, role, 'native ID')
        if role in historical_child_roles:
            oldfront = load_yaml(git_bytes(oldhost + '/agents/devflow-' + role + '.md').decode().split('---', 2)[1])
            for key in ('model', 'tools', 'injectAgentsMd'):
                assert fronts[p].get(key) == oldfront.get(key), (host, role, key)
        else:
            assert fronts[p]['model'] == 'inherit', (host, 'researcher model inheritance')
            tools = fronts[p]['tools']
            assert isinstance(tools, str), (host, 'researcher tools type')
            assert [t.strip() for t in tools.split(',')] == ['Read', 'Grep', 'Glob', 'WebSearch', 'WebFetch'], (host, 'researcher tool allowlist')
            if host == 'zcode':
                assert fronts[p]['injectAgentsMd'] is True, 'researcher ZCode instruction injection'

research = load_yaml((root / 'skills/ric-devflow/assets/templates/research-report.yaml').read_text())
research_fields = {'schema_version', 'research_id', 'supersedes', 'author', 'created_at',
                   'root_issue_id', 'task_id', 'question', 'status', 'sources', 'facts',
                   'inferences', 'recommendations', 'unresolved', 'not_run'}
assert set(research) == research_fields, 'research report fields'
assert type(research['schema_version']) is int and research['schema_version'] == 1, 'research report schema'
assert re.fullmatch(r'RESEARCH-\d{3}', research['research_id']), 'research report ID'
assert research['status'] in ('COMPLETE', 'PARTIAL', 'BLOCKED'), 'research report status'
for key in ('author', 'created_at', 'question'):
    assert isinstance(research[key], str), ('research report string', key)
for key in ('supersedes', 'root_issue_id', 'task_id'):
    assert research[key] is None or isinstance(research[key], str), ('research report nullable ID', key)
for key in ('sources', 'facts', 'inferences', 'recommendations', 'unresolved', 'not_run'):
    assert isinstance(research[key], list), ('research report array', key)

template_prefix = '.agents/skills/_devflow_shared/templates/'
templates = run(['git', 'ls-tree', '-r', '--name-only', base, template_prefix]).splitlines()
templates_byte_identical = 0
compact_templates_compatible = 0
for old in templates:
    relative = old[len(template_prefix):]
    new = root / 'skills/ric-devflow/assets/templates' / relative
    original = git_bytes(old)
    if not relative.startswith('compact/'):
        assert new.read_bytes() == original, ('v1 template bytes changed', old)
        templates_byte_identical += 1
        continue
    if relative == 'compact/state.yaml':
        expected = load_yaml(original.decode())
        expected['repository']['root'] = '.'
        expected['artifacts']['current'] = '.devflow/changes/REQ-YYYYMMDD-001/current.md'
        assert load_yaml(new.read_text()) == expected, ('compact state payload changed', relative)
    else:
        def without_comments(text):
            return [line.rstrip() for line in re.sub(r'<!--.*?-->', '', text, flags=re.S).splitlines() if line.strip()]
        assert without_comments(new.read_text()) == without_comments(original.decode()), ('compact document payload changed', relative)
    compact_templates_compatible += 1
assert templates_byte_identical == 16 and compact_templates_compatible == 4, 'historical template inventory'
if root == origin:
    run(['git', 'diff', '--check'])
    run(['git', 'diff', '--cached', '--check'])
identity = hashlib.sha256(json.dumps(manifest, ensure_ascii=False, sort_keys=True).encode()).hexdigest()
print(json.dumps(dict(verdict='PASS', head=head, tested_sha=None, candidate_sha256=identity,
    python=sys.version.split()[0], pyyaml=yaml.__version__, baseline_sha=base,
    counts=counts, templates_byte_identical=templates_byte_identical,
    compact_templates_compatible=compact_templates_compatible, role_closures=closures,
    business_roles=list(roles), native_child_roles=list(child_roles),
    historical_child_roles_compared=list(historical_child_roles), research_report_fields=sorted(research),
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
roles = ('planner', 'reviewer', 'tester', 'implementer', 'researcher')
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
    selections = [names, ['ric-devflow'], *(['ric-devflow', 'ric-devflow-' + role] for role in roles)]
    for selected in selections:
        label = 'copy-six' if len(selected) == 6 else ('copy-entry' if len(selected) == 1 else 'copy-' + selected[1])
        target = temp / label
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
    expected_errors = {
        'control': None, 'duplicate-yaml': 'duplicate YAML key', 'broken-toml': 'TOMLDecodeError',
        'extra-skill': 'unexpected Skill directory', 'bad-link': 'missing link', 'bad-anchor': 'missing anchor',
        'history-prefix': 'historical report prefix changed', 'bad-frontmatter': 'frontmatter name mismatch',
        'validator-failure': 'command failed', 'planner-native': 'native role set', 'fixed-effort': 'fixed reasoning effort',
        'missing-researcher': 'extra or missing Skill', 'implicit-researcher': 'implicit invocation policy',
        'researcher-codex-id': 'native ID', 'researcher-codex-model': 'researcher model inheritance',
        'researcher-codex-write': 'researcher read-only sandbox', 'researcher-claude-tools': 'researcher tool allowlist',
        'researcher-zcode-model': 'researcher model inheritance', 'research-report-fields': 'research report fields',
        'research-report-schema': 'research report schema', 'research-report-status': 'research report status',
        'v1-template-bytes': 'v1 template bytes changed', 'compact-payload': 'compact document payload changed',
        'recent-history-prefix': 'published maintenance report changed', 'role-closure-backlink': 'unrelated role dependency',
        'product-script': 'non-instruction product file', 'package-manifest': 'unexpected package manifest',
        'product-symlink': 'published symlink dependency',
    }
    for case, expected_error in expected_errors.items():
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
        elif case == 'planner-native':
            p = fixture / 'skills/ric-devflow/assets/agents/codex/ric-devflow-planner.toml'
            p.write_text('name = "ric_devflow_planner"\n')
        elif case == 'fixed-effort':
            p = fixture / 'skills/ric-devflow/assets/agents/codex/ric-devflow-tester.toml'
            p.write_text('model_reasoning_effort = "xhigh"\n' + p.read_text())
        elif case == 'missing-researcher':
            (fixture / 'skills/ric-devflow-researcher/SKILL.md').unlink()
        elif case == 'implicit-researcher':
            p = fixture / 'skills/ric-devflow-researcher/agents/openai.yaml'
            p.write_text(p.read_text().replace('allow_implicit_invocation: false', 'allow_implicit_invocation: true', 1))
        elif case.startswith('researcher-codex-'):
            p = fixture / 'skills/ric-devflow/assets/agents/codex/ric-devflow-researcher.toml'
            text = p.read_text()
            if case == 'researcher-codex-id':
                text = text.replace('name = "ric_devflow_researcher"', 'name = "wrong"', 1)
            elif case == 'researcher-codex-model':
                text = 'model = "unapproved-model"\n' + text
            else:
                text = text.replace('sandbox_mode = "read-only"', 'sandbox_mode = "workspace-write"', 1)
            p.write_text(text)
        elif case == 'researcher-claude-tools':
            p = fixture / 'skills/ric-devflow/assets/agents/claude-code/ric-devflow-researcher.md'
            p.write_text(p.read_text().replace('tools: Read, Grep, Glob, WebSearch, WebFetch', 'tools: Read, Grep, Glob, WebSearch, WebFetch, Bash', 1))
        elif case == 'researcher-zcode-model':
            p = fixture / 'skills/ric-devflow/assets/agents/zcode/ric-devflow-researcher.md'
            p.write_text(p.read_text().replace('model: inherit', 'model: unapproved-model', 1))
        elif case.startswith('research-report-'):
            p = fixture / 'skills/ric-devflow/assets/templates/research-report.yaml'
            text = p.read_text()
            if case == 'research-report-fields':
                text = text.replace('not_run: []\n', '', 1)
            elif case == 'research-report-schema':
                text = text.replace('schema_version: 1', 'schema_version: 2', 1)
            else:
                text = text.replace('status: "BLOCKED"', 'status: "PASS"', 1)
            p.write_text(text)
        elif case == 'v1-template-bytes':
            p = fixture / 'skills/ric-devflow/assets/templates/test-report.yaml'
            p.write_text(p.read_text() + '\n# changed historical bytes\n')
        elif case == 'compact-payload':
            p = fixture / 'skills/ric-devflow/assets/templates/compact/test-plan.md'
            p.write_text(p.read_text().replace('revision: 1', 'revision: 2', 1))
        elif case == 'recent-history-prefix':
            p = fixture / 'docs/DEVFLOW_SKILLS_VALIDATION.md'
            old = subprocess.check_output(['git', 'show', '458b1778986b0f008aaa46b805af6f7baf6e6880:docs/DEVFLOW_SKILLS_VALIDATION.md'])
            raw = p.read_bytes()
            p.write_bytes(raw[:len(old) - 1] + b'!' + raw[len(old):])
        elif case == 'role-closure-backlink':
            p = fixture / 'skills/ric-devflow/references/shared/research.md'
            p.write_text(p.read_text() + '\n[unrelated role](../../../ric-devflow-researcher/SKILL.md)\n')
        elif case == 'product-script':
            (fixture / 'skills/ric-devflow/runner.py').write_text('print("unapproved")\n')
        elif case == 'package-manifest':
            (fixture / 'package.json').write_text('{}\n')
        elif case == 'product-symlink':
            (fixture / 'skills/ric-devflow/alias.md').symlink_to('SKILL.md')
        env = dict(os.environ, DEVFLOW_CHECK_ROOT=str(fixture), PYTHONUTF8='1', PYTHONIOENCODING='utf-8')
        observed = run([sys.executable, '-B', '-c', check_source], env)
        assert (observed.returncode == 0) == (case == 'control'), (case, observed.stdout, observed.stderr)
        if expected_error is not None:
            assert expected_error in observed.stderr, (case, 'wrong rejection reason', observed.stderr)
        results.append(dict(case=case, exit_code=observed.returncode, expected_error=expected_error))
    print(json.dumps(dict(verdict='PASS', checks=results, commands=commands, native_runtime=False,
                         note='negative exits are expected test observations, not hidden failures'), ensure_ascii=False, indent=2))
assert not temp.exists()
print('Created temporary fixture cleanup: PASS')
PY
)
```

## 独立 Agent 的三批原始场景与增量复验

维护者评测只传原始情境与候选指令，不传判定；事后逐项核对动作轨迹与结果。若在完整 DevFlow 内执行，由主会话 Planner 复用固定 Tester，不为每批另建执行者；独立 Skill 维护评测与原生工作流调用分列，不能拿通用评测代理冒充已加载角色。先固定候选文件集与真实存在的隔离 Git 对象，原始输入和输出随报告保存。临时 Git 仅作测试数据，使用命令级合成作者，不提交源码仓库或修改用户 Git 配置。生成 fixture 时先留初始清单，真正处理请求的动作另记。

| 批次 | 给执行者的原始事实/请求 | 事后观察维度 |
|---|---|---|
| A 缺件 | 仅入口；五个单角色分别安装；Reviewer/Tester/Implementer/Researcher最小闭包；必要传递文件缺失；已启动子角色被要求补装/派发；离线无可信源。原任务为保持权限/响应兼容的CSV导出 | 首次读取与下一动作、真实复制/读回、必要依赖、来源、角色上下文、原任务保全 |
| B 文件边界 | project/user入口分别链接到另一处可信source；范围未知；合成ZCode项目范围；同字节/定制/异版本/普通文件/断链；明确禁用及其他配置；部分安装后重复请求；离线完整/不完整源；合成获取成功/拒绝 | 逻辑写入清单、链接保留、冲突前后字节/mtime、全目标预检、读回/重复写入、合成与真实区分 |
| C 决定 | 开发/修复/续作/解释/建议/独立审核测试/文档/显式四角色；三宿主/未知/冲突身份；平级交接；文件/加载/调用/重载；无工具/权限；v1/v2、未提交测试、Oracle变化、冻结Task、VERIFIED/DONE各证据组合 | 真实选择与交接、权限/批准界限、唯一状态作者、保留历史、完整SHA绑定；不比对措辞 |
| 守卫 Delta | 入口存在但bootstrap缺失：正常、来源未知、现存入口字节冲突；四个单角色分别缺入口：正常、同名原生定义冲突、必需能力禁用 | 写前完整预检、真实缺件恢复、冲突/禁用时文件结果；新旧输入身份分别固定 |
| 加载与恢复 Delta | WF-51–57 的原始输入：三宿主项目/用户补装、复用零写入、后续角色切换、无清单/无工具、批准不足、独立角色/子角色、分类故障、重启后成功/仍失败 | 按需真实交接或决定级下一动作、明确重启提示及恢复信息、错误归因、停止受影响动作、无重复安装/业务动作/探测、单角色证据范围；决定级与实机分列 |
| 固定角色与参数 Delta | WF-58–70：逻辑职责/实例复用与替换、额度 1/2/外部占用、安全回收、取消作用范围、有限扩容、未知生命周期、旧 Planner 迁移及思考兼容 | 主会话唯一 Planner，四种子职责下实际 ID 的创建/继续/回收/替换及读取轨迹；有效模型/参数、重试次数、完整载荷和文件结果；模拟动作与原生运行分别记录 |
| 研究与安全并行 Delta | WF-71–78、TRIGGER-35–38、PERF-09–11：明确缺口/无缺口、恶意材料/断网、来源身份/原载荷、冲突连通分量/同父目录/跨流依赖、四默认实例/一个临时实例、DB事务迁移与输出隔离 | 原始请求和 Oracle 分别固定；真实读取/决定/完整载荷/副作用及占用轨迹，不用静态文本或合成占用证明原生调度；未加载Researcher与其他宿主如实未运行 |

实际文件子例不能只输出“将复制”。独立执行者可根据其决定执行逐文件复制，但不得另写通用 bootstrap 实现来替代被测指令。中断恢复区分预置的部分状态和本次真实故障：收集代码发生 KeyError 等错误必须保留原命令/退出码，修收集器后重新核对文件再恢复，不改候选或 Oracle。

加载与恢复 Delta 只向独立执行者提供对应“原始输入”、当前 Skill/必要参考及具体宿主反馈，不提供“判定”段。没有真实宿主工具时明确以合成反馈作决定级评测，保存原始输入/回答和受检文件摘要，事后按可观察动作判定，不比对精确措辞或用关键词计数冒充行为通过。实机另需当前宿主版本、会话身份、原生工具目标、启动/返回与重启后反馈；缺任一项就记录对应切片未运行，通用评测代理的回答不证明原生角色已加载。

真实拒绝测试须先证明底层拒绝发生：例如 chmod后重新读取mode并执行窄探针。若mode未变或探针成功，权限拒绝切片未运行；合成拒绝输入只能证明决定级响应。不能因为文件操作通过就声称权限沙箱、原生宿主加载/调用、远程获取或标准安装器已经通过。

候选变化后先核对精确Delta；未变契约/输入可引用原Tester结果，受影响缺件守卫重新执行对应真实fixture。原失败/阻塞记录保持原身份并保留，新证据单独发布。正式报告沿用原Schema和PASS/FAIL/BLOCKED；源码隔离结论与因无产品Commit/原生环境而未执行的门禁切片分开说明。

标准CLI说明另做负向跟随检查：在合成canonical目标放置用户定制目录，在另一例宿主目标放置已有真实symlink，记录全部目标的lstat/readlink与哨兵摘要；按README的首次安装前提在首个CLI调用前停止，并核对字节/类型/mtime未变。该检查不启动CLI，不能证明安装器实现安全或它已运行。实际原始输入、输出及本轮机械核验见[独立评测证据](DEVFLOW_SKILLS_EVAL_20260913.md)。

## 单功能分支与 detached worktree 的 Git 机制复验

以下独立片段仅依赖 Python 标准库和 Git。在系统临时目录创建合成仓库，使用命令环境中的合成作者，不修改源码仓库、用户 Git 配置或已安装 Skill；所有 fixture 自动清理。原始命令、退出码和候选 bundle 保留在打印出的专用临时证据目录。它验证 Git 机制与不变量，**不证明原生角色已经派发、审核已批准、实际渐进加载或 G0–G10 已通过**。角色串行交接、冲突作者处理是合成动作；真实角色是否遵循规则仍需独立原始场景及读取轨迹。

片段中的所有 merge（包括在 Task 工作区内启动合并）模拟 Planner 协调，冲突文件内容写入模拟原作者；fixture helper 组合这些动作不授予作者合并权。

```bash
(
set -Eeuo pipefail
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B - <<'PY'
import hashlib, json, os, subprocess, sys, tempfile
from pathlib import Path

assert not sys.flags.optimize, 'disable PYTHONOPTIMIZE: checks require assertions'
source = Path.cwd()
source_head = subprocess.check_output(['git', 'rev-parse', 'HEAD'], text=True).strip()
guide = source / 'docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md'
evidence = Path(tempfile.mkdtemp(prefix='devflow-worktree-evidence-'))
print('Evidence directory:', evidence, flush=True)
env = {k: v for k, v in os.environ.items() if not k.startswith('GIT_')}
env.update(GIT_CONFIG_NOSYSTEM='1', GIT_CONFIG_GLOBAL=os.devnull,
           GIT_AUTHOR_NAME='DevFlow Fixture', GIT_AUTHOR_EMAIL='fixture@example.invalid',
           GIT_COMMITTER_NAME='DevFlow Fixture', GIT_COMMITTER_EMAIL='fixture@example.invalid',
           GIT_TERMINAL_PROMPT='0')
commands, cases = [], []
result = dict(source_head=source_head, guide_sha256=hashlib.sha256(guide.read_bytes()).hexdigest(),
              cases=cases, commands=commands, verdict='FAIL', native_roles=False)

def git(cwd, *args, success=True):
    argv = ['git', '-c', 'core.hooksPath=' + os.devnull, '-c', 'commit.gpgSign=false',
            '-c', 'rerere.enabled=false', *args]
    p = subprocess.run(argv, cwd=cwd, env=env, text=True, capture_output=True)
    commands.append(dict(cwd=str(cwd), argv=argv, exit_code=p.returncode,
                         stdout=p.stdout, stderr=p.stderr))
    assert (p.returncode == 0) == success, (argv, p.returncode, p.stdout, p.stderr)
    return p.stdout.strip()

def commit(cwd, name, content, message):
    (cwd / name).write_text(content, encoding='utf-8')
    git(cwd, 'add', '--', name)
    git(cwd, 'commit', '-m', message)
    return git(cwd, 'rev-parse', 'HEAD')

def passed(name, **observations):
    cases.append(dict(case=name, verdict='PASS', **observations))

try:
    with tempfile.TemporaryDirectory(prefix='devflow-worktree-fixture-') as location:
        temp = Path(location)
        repo, integration = temp / 'repo', temp / 'integration'
        repo.mkdir()
        git(repo, 'init', '-b', 'main')
        base = commit(repo, 'app.txt', 'base\n', 'fixture baseline')
        (repo / 'app.txt').write_text('USER DIRTY\n')
        (repo / 'user-untracked.txt').write_text('KEEP\n')
        protected = {p.name: p.read_bytes() for p in (repo / 'app.txt', repo / 'user-untracked.txt')}
        feature = 'feature/REQ-WORKTREE-CHECK'
        git(repo, 'branch', feature, base)
        git(repo, 'worktree', 'add', str(integration), feature)
        branches = lambda: git(repo, 'for-each-ref', '--format=%(refname:short)', 'refs/heads').splitlines()
        expected = sorted(['main', feature])
        assert branches() == expected
        passed('one-feature-branch', branches=branches())

        # Three independent tasks start at one exact SHA; no task refs are created.
        tasks = [temp / ('task-' + name) for name in ('a', 'b', 'c')]
        for task in tasks:
            git(repo, 'worktree', 'add', '--detach', str(task), base)
            assert git(task, 'rev-parse', 'HEAD') == base
            git(task, 'symbolic-ref', '-q', 'HEAD', success=False)
        assert branches() == expected
        passed('parallel-detached-tasks', task_count=len(tasks), base_sha=base)

        a, b, c = tasks
        task_directory = git(a, 'rev-parse', '--absolute-git-dir')
        first = commit(a, 'app.txt', 'implementation\n', 'fixture implementation')
        tests = commit(a, 'test.txt', 'independent tests\n', 'fixture tester handoff')
        repaired = commit(a, 'app.txt', 'implementation repaired\n', 'fixture author repair')
        assert task_directory == git(a, 'rev-parse', '--absolute-git-dir')
        git(a, 'symbolic-ref', '-q', 'HEAD', success=False)
        assert branches() == expected
        passed('same-task-serial-handoff', candidate_shas=[first, tests, repaired])

        git(integration, 'merge', '--ff-only', repaired)
        assert git(integration, 'rev-parse', 'HEAD') == repaired
        passed('fast-forward-integration', head_sha=repaired)
        parallel = commit(b, 'parallel.txt', 'parallel work\n', 'fixture parallel task')
        git(integration, 'merge', '--no-edit', parallel)
        merged = git(integration, 'rev-parse', 'HEAD')
        assert len(git(integration, 'show', '-s', '--format=%P', 'HEAD').split()) == 2
        git(integration, 'merge-base', '--is-ancestor', repaired, merged)
        git(integration, 'merge-base', '--is-ancestor', parallel, merged)
        passed('parallel-merge-integration', head_sha=merged)

        conflict = commit(c, 'app.txt', 'conflicting task\n', 'fixture conflicting candidate')
        git(integration, 'merge', '--no-edit', conflict, success=False)
        assert git(integration, 'diff', '--name-only', '--diff-filter=U') == 'app.txt'
        git(integration, 'merge', '--abort')
        assert git(integration, 'rev-parse', 'HEAD') == merged
        git(c, 'merge', '--no-edit', merged, success=False)
        assert git(c, 'diff', '--name-only', '--diff-filter=U') == 'app.txt'
        resolved = commit(c, 'app.txt', 'author reconciled both tasks\n', 'fixture author resolution')
        assert resolved != conflict
        git(c, 'merge-base', '--is-ancestor', conflict, resolved)
        git(c, 'merge-base', '--is-ancestor', merged, resolved)
        git(integration, 'merge', '--ff-only', resolved)
        passed('conflict-returned-to-author', old_sha=conflict, replacement_sha=resolved)

        # Resume the registered path without creating a new branch or worktree.
        assert git(integration, 'branch', '--show-current') == feature
        assert git(integration, 'rev-parse', 'HEAD') == resolved
        assert branches() == expected
        passed('existing-feature-reuse', integration_branch=feature)
        assert all((repo / name).read_bytes() == raw for name, raw in protected.items())
        assert git(repo, 'rev-parse', 'HEAD') == base
        passed('dirty-original-checkout-preserved')

        scratch = a / 'unfinished.txt'
        scratch.write_text('unfinished task evidence\n')
        git(repo, 'worktree', 'remove', str(a), success=False)
        assert scratch.read_text() == 'unfinished task evidence\n'
        assert git(a, 'rev-parse', 'HEAD') == repaired
        scratch.unlink()  # Only this known synthetic file is removed after the rejection.
        git(repo, 'worktree', 'lock', '--reason', 'fixture active writer', str(a))
        git(repo, 'worktree', 'remove', str(a), success=False)
        assert a.exists()
        git(repo, 'worktree', 'unlock', str(a))
        passed('resume-and-reject-dirty-or-locked-cleanup', resumed_sha=repaired)

        published = [first, tests, repaired, parallel, conflict, resolved]
        for sha in published:
            git(integration, 'merge-base', '--is-ancestor', sha, resolved)
        for task in tasks:
            assert not git(task, 'status', '--porcelain')
            git(repo, 'worktree', 'remove', str(task))
        for sha in published:
            git(repo, 'cat-file', '-e', sha + '^{commit}')
        passed('published-candidates-survive-cleanup', published_shas=published)

        squash_task = temp / 'task-squash'
        git(repo, 'worktree', 'add', '--detach', str(squash_task), resolved)
        candidate = commit(squash_task, 'squash.txt', 'squash candidate\n', 'fixture squash candidate')
        candidate_object = git(squash_task, 'cat-file', 'commit', candidate)
        candidate_tree = git(squash_task, 'rev-parse', candidate + '^{tree}')
        git(integration, 'merge', '--squash', candidate)
        git(integration, 'commit', '-m', 'fixture repository squash policy')
        git(integration, 'merge-base', '--is-ancestor', candidate, 'HEAD', success=False)
        bundle = evidence / 'published-candidate.bundle'
        git(squash_task, 'bundle', 'create', str(bundle), 'HEAD')
        git(repo, 'bundle', 'verify', str(bundle))
        restored = temp / 'restored'
        restored.mkdir()
        git(restored, 'init', '-b', 'restore')
        git(restored, 'fetch', str(bundle), 'HEAD')
        assert git(restored, 'rev-parse', 'FETCH_HEAD') == candidate
        assert git(restored, 'cat-file', 'commit', candidate) == candidate_object
        assert git(restored, 'rev-parse', candidate + '^{tree}') == candidate_tree
        # Recovery is proved in an empty object database before the original is removed.
        assert not git(squash_task, 'status', '--porcelain')
        git(repo, 'worktree', 'remove', str(squash_task))
        assert branches() == expected
        assert all((repo / name).read_bytes() == raw for name, raw in protected.items())
        passed('squash-bundle-recovery', candidate_sha=candidate,
               bundle_sha256=hashlib.sha256(bundle.read_bytes()).hexdigest(), branches=branches())
    assert not temp.exists()
    result.update(verdict='PASS', fixture_cleanup=True, case_count=len(cases))
finally:
    (evidence / 'result.json').write_text(json.dumps(result, ensure_ascii=False, indent=2) + '\n')
print(json.dumps(dict(verdict=result['verdict'], case_count=len(cases),
                     evidence=str(evidence), native_roles=False), ensure_ascii=False, indent=2))
PY
)
```

渐进加载另按正常安装、单角色直接调用、代码审核、测试计划和缺件恢复采集实际读取记录：标记每次读取的触发动作及材料身份，确认未触发的其他平台、迁移和发布材料未被加载。安装闭包的存在性/摘要检查不计为正文读取；文档中的路由词检查只能补充静态证据。

## 有限性能 A/B 与行为复验

本节是本地维护者复跑说明，不新增产品 CLI、脚本、CI、Gate 或报告字段。沿用上面三个完整检查块及其全部断言；新增行为的原始情境与独立 Oracle 见[性能行为场景](../skills/ric-devflow/references/evals/performance-cases.md)。仅看规则关键词、文件体积、模型叙述或文件存在，均不能证明行为或提速。

### 受检对象与权限

1. 固定 A 的完整源码 Commit，B 的实际代码 Commit；没有 B 提交授权时保全基线 Commit、逐文件类型/模式/内容清单与集合摘要，明确为未提交候选诊断，不能用摘要替代正式 `tested_sha`。既有报告不得改绑 SHA。
2. 静态/闭包/负控/Git 检查分别对 A、B 运行，保留每块真实命令、退出码、关键结果和必要附件。历史报告维持完整原字节前缀，本轮报告只追加；其自身不进入候选集合摘要，避免自引用。模板、原生身份、模型/权限/隐式调用元数据的保持检查与候选行为检查分开。
3. 测试计划需独立 TEST_REVIEW，明确 fixture、工具、等待/取消预算和写范围。Root Planner 唯一调度原生角色；执行者不派生。原生定义未动态加载候选时，声明测量的是当前宿主执行指定来源指令的动作，不声称安装、自动发现或其他宿主实机已验证。
4. fixture、短原始请求和临时测量代码仅在隔离临时目录或当前 Root `.local`；正式复现材料、输入身份、失败/BLOCKED/not_run 与必要原始回执另作持久附件。不得使用真实用户 Skill 安装目录、业务数据、共享数据库、常驻服务或全局配置。文档中的 fixture Git Commit 只作测试数据身份。

### 三种最小 fixture、四种动作

| fixture | 相同业务输入和真实动作 | 采样边界 |
|---|---|---|
| PREP | 两臂分别提供完整且预置匹配的源码/逻辑发现位置、宿主身份、范围、可信来源身份；原请求仅核验测试者准备状态。首次新鲜原生会话执行，随后同会话同输入再次执行 | COLD 与 WARM 分别记录；不因 miss 自动授权安装/联网/建 Root；完整性检查字节与正文读取字节区分 |
| GATES | 小型合成 Git 仓库、一条 stdlib unittest、固定 Spec/plan/数据、最后一个 Task 的给定前序证据；请求真实增量并处理完整验证。命令例：`python3 -m unittest discover -s tests -p 'test_*.py'` | 记录真实派发数、命令数及两份原模板报告。两臂收到相同请求，旧版可能已有有效复用；不能预设 B 更快。失败或身份变化另测，不混入正常性能样本 |
| WAVE | 三个 READY Task 各自拥有独立 AC 和 `alpha`/`beta`/`gamma` 路径，在私有 detached worktree 将对应 `expected.txt` 原字节写入 `actual.txt` 并实际断言相同。无共享契约/构建/数据库/端口/输出 | 一个调用只做一个 Task；记录三次真实调用及起止。由受测规则决定调度，不在输入中泄露判定；旧 A 按固定历史规则观察，新 B 默认每职责一个可复用实例，独立 READY Task 的隔离、资源与收益条件成立时才临时扩容，遵守两 Task/每职责二/总四与宿主更严上限。记录实例复用轨迹，未观察到执行重叠时如实报告 |

fixture 预置过程、实际角色动作与最终验证分别记录。角色不能把给定的历史批准或预置产物声称为本次已真实完成。所有实际命令都采集退出码；取消只结束本样本子进程并等待退出，不触碰用户服务。源码/fixture 清单改变使相关样本失效，不能将不同 B 的结果拼为同一候选。

### 十二对采样与共同采集

四类 COLD、WARM、GATES、WAVE 各三对，共十二对、二十四动作样本。第一/三对 A→B，第二对 B→A；COLD/WARM 在同一臂同一会话连续进行。每对 A/B 顺序运行，使用相同原始请求、业务 fixture 字节、角色 ID、公开模型/推理档位、权限、资源与采集方式。隐藏宿主上下文或模型身份不可观测时明确受限，不把未知写成一致。

两臂共用只含测量要求的约定，不能包含 Oracle、期望分支、缓存算法或调度答案。每个原生执行者独占一个采集输出目录，该目录与 PREP_ROOT、受测源码和业务 fixture 分开。临时 wrapper 只能执行给定命令并采集结果，不能替角色实现工作流。

- Planner 记录真实 dispatch/followup 工具及返回回执、起止时间和精度；执行者记录动作起止及实际命令/读取。只用已有真实时钟和工具接口，不编造 token 或 cache API。
- 保存精确 argv/完整 shell 文本、cwd、exit、stdout/stderr 原始字节、timeout/取消及进程耗时。`time.perf_counter_ns()` 可包裹真实 subprocess；宿主支持时 `Date.now()` 可包裹真实 await 工具调用。不同层级时间分别报告，不相减推算模型耗时。
- 读取记录包含相对路径、实际范围、来源身份、返回边界及完整性。未截断 UTF-8 返回可计精确字节；文件 stat 大小不能冒充实际读取，进程 stdout 字节不能冒充宿主模型上下文。截断或不可见指标写 null/不可测与原因。
- setup、执行与测量开销分别标明；日志成功路径保留命令、exit、身份、覆盖和关键结果，失败及必要复现材料持久保存。原始回执无法保存时说明缺口，不重新执行成功命令补造过去回执。
- WAVE 必须采集真实 dispatch 重叠区间，内部动作重叠另算；动作短而未观察到重叠时只报告事实，不插入 sleep 人为制造加速。
- 每场景每臂列全部三个样本、median、min/max 和配对差值；不混合场景得出全局收益，不给 p95 或由指令字节外推 token/成本。操作数相同、无改善或变慢均保留。一次明显回归最多一次有假设的针对性重跑，保留首轮，不能扩大 timeout 或无限重试。

### 行为、保持与失败证据

性能正常样本之外，执行性能场景 PERF-01—PERF-08 的失败/边界变体。原始请求与 Oracle 分开发送；真正负责动作的角色只读前者，Tester 事后对照文件结果、工具回执、报告完整载荷、状态序列与调用身份。传输与状态恢复由 Root Planner 亲自操作隔离 fixture，不能通过第二 Planner 或自写编排器代替。

每项缺少必要原生调用或源对象即标该切片 BLOCKED，不能以合成输入替代 live 宿主证据。基线同命令失败只有相关性证据充足才标历史失败；候选新增失败、测试收集器失败和环境失败分别保留，并由 Planner 归因。收集器解析异常时保全底层命令结果，从现存完整原文恢复，不为获得漂亮输出重复成功工作。

正式结果仍使用原 Test/Defect/Implementation Report 载荷；未提交源码只能给诊断结论，不能宣布 G5—G9。恢复能力必须通过持久源/fixture bundle 及实读证明，删除 `.local` 后仅缓存可重建不等于丢失正式历史。

## Codex CLI 生命周期实机验证

仅在明确授权的源码维护验证中使用可用 CLI 和隔离 fixture；这不授权业务流程在角色不可用时换客户端绕过权限，也不把 CLI 结果当桌面或其他平台结果。先记录实际 `codex --version`、`codex --help`、`codex exec --help` 与工具 Schema。候选六 Skill/四 Codex 定义仅复制到本轮隔离项目，绑定字节摘要；不写真实用户安装，不修改既有 .codex/ 或全局配置。历史对照始终按其固定SHA与原清单，不把旧五/三对象记为新六/四实机结果。

CLI 可能把新 fixture 的项目信任记录自动写入用户配置；不能仅凭 `-c` 属于进程覆盖就声称没有持久副作用。本轮环境即使传入进程级 fixture trust_level，仍出现自动信任记录。后续先核验隔离/授权与配置保全方案；没有已确认的无外部写入路径或相应授权时，停止该实机切片并说明条件，不反复试建。已获授权的自建 fixture 若产生自动记录，只精确清理可证明本轮产生的项，保留其他当前字节，不整份回滚用户配置；其他指纹变化单独报告，不猜测来源，不通过更改 model、审批或用户禁用设置解决。

沿用已配置模型、认证与权限；配额只使用当前进程 `-c agents.max_concurrent_threads_per_session=1` 或 `=2`，需要验证其他占用/扩容时另记录该隔离进程的实际上限。可用时用 `--no-daemon` 避免共享后台服务，`exec --json` 采集真实事件。不得用忽略权限/规则、绕过审批、伪造原生角色选择或另写调度器让场景“通过”。认证或调用失败时保留脱敏原反馈并有限诊断，不打印凭据、安装全局工具或伪造成功。

先以必要的最小角色交接核对候选是否真的加载及调用；报告角色定义可读、宿主可用、实际选择的原生角色和返回实例分别取证。只读到了指令或 CLI 能启动不等于候选原生角色已执行。真实不支持的字段/故障注入不反复猜测；标该切片 BLOCKED/not_run，可另列模拟证据。

| 实机场景 | 必须保存的可观察结果 |
|---|---|
| 同 Root 多次使用三个角色 | 实际角色选择、实例 ID、继续接口、完整结果；替换原因与旧新绑定 |
| 配额 1、2，完成线程仍保留 | 真实额度/拒绝、结果保全、释放接口与回执、后续角色是否仅执行一次 |
| 其他实例占额度 | 不触碰无关实例；只回收自有已保全闲置者，或提前报告无法推进 |
| 宿主仅创建不能恢复 | 实际能力确实如此时才计实机；旧动作结束后替换下一动作，不能用 Prompt 隐藏恢复工具模拟该能力 |
| 用户取消 A 后明确授权 B | 真实用户/宿主取消事件、停止与副作用保全；A 不重启，B 身份独立并正确执行。没有安全取消注入方式则 not_run |
| 结果丢失、超时、生命周期未知 | 真实已启动动作与后续取证、结果恢复和动作计数；无法安全注入时 not_run，不以文字假设替代 |
| 两个独立 READY Task | 真实调度重叠或串行结果、打开实例峰值、工作区/输出隔离与临时实例回收；没有测量不宣称加速 |

每个切片保留相对 fixture 位置、候选身份、命令和退出码、必要脱敏事件与文件结果；私密原始日志仅临时保留，规范化/脱敏方式明确记录。失败、权限/环境限制与未运行分别列出。旧决定级原载荷不改写，新报告追加；实机通过只限有真实调用记录的宿主/入口/场景。Claude Code、ZCode 未执行时继续标未运行，三平台整体验收保持未关闭。


<a id="原生交接与原120秒回执修复回归"></a>

## 目标完成、显式时限与原生交接回归

本段保护历史整份报告和原 Git 机制检查，保留全部既有复制/检查器负控及原发送职责正反例；本段另加整份 8e2059 已发布报告前缀与独立损坏负控，前三个完整 Bash 块保持原文。场景为确定性输入与事后判定边界，不代表候选原生行为；不提取成产品 runner/CLI。

默认按完整目标、原载荷、必要来源与实际验证结果判定。无明确截止时，完成晚于历史显式 SLA 示例界点仍可 PASS，持续推进未完成或工具单次等待超时继续等待；取消、拒绝、权限、真实阻塞和既有无进展条件仍停止。只有用户明确时限、业务 SLA 或外部硬截止才验证原截止、真实时钟与剩余预算；宿主最短等待不支持时如实限制，不请求越界等待，不把工具超时推导为总截止。下列旧 timing 输入明确是历史 SLA 示例，通用判定器不硬编码其时长。

原始业务输入与 Oracle 分别固定。实际 Researcher 只接收相同原问题/材料、必要固定来源、真实权限与候选指令，不接收判定器、Oracle 或整个测试目录。Planner 取得完整 cold 研究报告后才向同一实际实例交付窄范围 followup；正常推进就分段等待，不设替代默认总时间。角色选取、完整报告/来源/read-log、文件监控与未暴露遥测分别核验，不推算缓存、Token 收益或内部时长。历史失败/报告保留原字节与原绑定。

```bash
(
set -Eeuo pipefail
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B - <<'PY'
import ast, hashlib, json, re, subprocess, sys
from decimal import Decimal, InvalidOperation
from pathlib import Path
assert not sys.flags.optimize
root = Path.cwd()
base = '5750c71bdf73c51ce98589f848a0cbc9ca109146'
commands = []
def git_bytes(path):
    argv = ['git', 'show', base + ':' + path]
    p = subprocess.run(argv, cwd=root, capture_output=True, timeout=120)
    commands.append(dict(argv=argv, exit_code=p.returncode,
                         stdout_bytes=len(p.stdout),
                         stdout_sha256=hashlib.sha256(p.stdout).hexdigest(),
                         stderr=p.stderr.decode('utf-8', errors='replace')))
    assert p.returncode == 0, (argv, p.stderr)
    return p.stdout

report_name = 'docs/DEVFLOW_SKILLS_VALIDATION.md'
published = git_bytes(report_name)
current_report = (root / report_name).read_bytes()
def preserve_report(candidate):
    assert candidate.startswith(published), 'whole5750 published report prefix changed'
preserve_report(current_report)
damaged = published[:-1] + bytes([published[-1] ^ 1])
try:
    preserve_report(damaged)
except AssertionError as error:
    assert 'whole5750' in str(error)
else:
    raise AssertionError('whole5750 prefix negative control was accepted')

guide_name = 'docs/DEVFLOW_SKILLS_VALIDATION_GUIDE.md'
blocks = lambda raw: re.findall(chr(96) * 3 + r'bash\n(.*?)\n' + chr(96) * 3, raw.decode('utf-8'), re.S)
old_blocks = blocks(git_bytes(guide_name))
new_blocks = blocks((root / guide_name).read_bytes())
assert len(old_blocks) == 3
assert new_blocks[:3] == old_blocks[:3], 'original three complete guide blocks changed'
# Preserve the original blocks and every original mutation/rejection reason.
legacy_guide = subprocess.check_output(['git', 'show', '8e2059f29e86011f625316c524650b74aa5b95d0:' + guide_name])
def copy_controls(block):
    body = block.split("python3 -B - <<'PY'\n", 1)[1].rsplit('\nPY\n)', 1)[0]
    return next(ast.literal_eval(n.value) for n in ast.walk(ast.parse(body))
                if isinstance(n, ast.Assign) and isinstance(n.targets[0], ast.Name)
                and n.targets[0].id == 'expected_errors')
prior_controls = copy_controls(blocks(legacy_guide)[1])
current_controls = copy_controls(new_blocks[1])
assert all(current_controls.get(k) == v for k, v in prior_controls.items()), 'existing copy negative weakened'
assert current_controls == prior_controls, 'original negative inventory changed'
whole8e = subprocess.check_output(['git', 'show', '8e2059f29e86011f625316c524650b74aa5b95d0:' + report_name])
def preserve_whole8e(candidate):
    assert candidate.startswith(whole8e), 'whole8e2059 published report changed'
preserve_whole8e(current_report)
try:
    preserve_whole8e(whole8e[:-1] + bytes([whole8e[-1] ^ 1]))
except AssertionError as error:
    assert 'whole8e2059' in str(error)
else:
    raise AssertionError('whole8e2059 negative control accepted')



SEMANTIC_INPUTS = [{'case_id': 'S01', 'actual_role': 'ric_devflow_tester', 'recipient': 'ric_devflow_planner', 'message': '职责：Planner｜测试计划进度。已核对Tester交接。'}, {'case_id': 'S02', 'actual_role': 'ric_devflow_tester', 'recipient': 'ric_devflow_planner', 'message': '职责：测试者｜测试计划进度；接收：规划者｜采纳测试事实。'}, {'case_id': 'S03', 'actual_role': 'ric_devflow_researcher', 'host_nickname': 'Planner', 'recipient': 'ric_devflow_planner', 'message': '职责：研究者｜限定来源调查；接收：规划者｜采纳来源事实。'}, {'case_id': 'S04', 'actual_role': 'ric_devflow_reviewer', 'recipient': 'ric_devflow_tester', 'message': '职责：审核者｜计划审核；接收：测试者｜查看独立审核。'}, {'case_id': 'S05', 'actual_role': 'ric_devflow_implementer', 'recipient': 'ric_devflow_planner', 'message': '职责：测试者｜实现完成；接收：规划者｜合并候选。'}, {'case_id': 'S06', 'actual_role': 'ric_devflow_researcher', 'recipient': 'ric_devflow_planner', 'message': '已完成，请采纳。'}, {'case_id': 'S07', 'actual_role': 'ric_devflow_planner', 'recipient': 'ric_devflow_researcher', 'message': '职责：规划者｜研究交接；接收：研究者｜核验限定事实。'}]
SEMANTIC_ORACLE = {'S01': 'FAIL', 'S02': 'PASS', 'S03': 'PASS', 'S04': 'PASS', 'S05': 'FAIL', 'S06': 'BLOCKED', 'S07': 'PASS'}
TIMING_INPUTS = [{'case_id': 'T01', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '119', 'upper_seconds': '119.999', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T02', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '120', 'upper_seconds': '120', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T03', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '120.001', 'upper_seconds': '120.001', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T04', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '107', 'upper_seconds': '122', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T05', 'dispatch_seconds': None, 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '119', 'upper_seconds': '119.999', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T06', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': False, 'status': 'progress', 'lower_seconds': '119', 'upper_seconds': '119.8', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T07', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': False, 'status': 'report_text', 'lower_seconds': None, 'upper_seconds': None, 'report_created_seconds': '1', 'saved_seconds': '130', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T08', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '180', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '119', 'upper_seconds': '119.999', 'transport_recovery': True, 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T09', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'B', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '119', 'upper_seconds': '119.999', 'transport_recovery': True, 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T10', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'write_report', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '119', 'upper_seconds': '119.999', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T11', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '119', 'upper_seconds': None, 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T12', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '95', 'upper_seconds': '96', 'saved_seconds': '130', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T13', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': False, 'status': 'progress', 'lower_seconds': '121', 'upper_seconds': '121', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T14', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': True, 'status': 'completed', 'lower_seconds': '122', 'upper_seconds': '107', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}, {'case_id': 'T15', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'original_action_id': 'A', 'action_id': 'A', 'next_action': 'wait', 'terminal_receipt': False, 'status': 'completed', 'lower_seconds': '100', 'upper_seconds': '101', 'report_created_seconds': '99', 'deadline_source': 'business_sla', 'context': 'Legacy explicitly supplied SLA example; not a default action policy'}]
TIMING_ORACLE = {'T01': 'PASS', 'T02': 'PASS', 'T03': 'FAIL', 'T04': 'BLOCKED', 'T05': 'BLOCKED', 'T06': 'BLOCKED', 'T07': 'BLOCKED', 'T08': 'FAIL', 'T09': 'FAIL', 'T10': 'FAIL', 'T11': 'BLOCKED', 'T12': 'PASS', 'T13': 'FAIL', 'T14': 'BLOCKED', 'T15': 'BLOCKED'}
roles = {'Planner': 'ric_devflow_planner', '规划者': 'ric_devflow_planner',
         '审核者': 'ric_devflow_reviewer', '测试者': 'ric_devflow_tester',
         '实现者': 'ric_devflow_implementer', '研究者': 'ric_devflow_researcher'}
def sender_result(item):
    first = item['message'].splitlines()[0]
    sender = re.search(r'^职责：([^｜；]+)｜', first)
    if sender is None:
        return 'BLOCKED'
    if roles.get(sender.group(1)) != item['actual_role']:
        return 'FAIL'
    receiver = re.search(r'接收：([^｜；]+)｜', first)
    if receiver is None or roles.get(receiver.group(1)) != item['recipient']:
        return 'FAIL'
    return 'PASS'

def timing_result(item):
    keys = ('dispatch_seconds', 'budget_seconds', 'deadline_seconds',
            'lower_seconds', 'upper_seconds')
    try:
        values = {key: Decimal(item[key]) for key in keys if item.get(key) is not None}
    except (InvalidOperation, TypeError):
        return 'BLOCKED'
    if any(key not in values for key in ('dispatch_seconds', 'budget_seconds', 'deadline_seconds')):
        return 'BLOCKED'
    if item.get('deadline_source') not in ('user', 'business_sla', 'external_hard_cutoff'):
        return 'BLOCKED'
    if not all(v.is_finite() for v in values.values()) or values['budget_seconds'] <= 0:
        return 'BLOCKED'
    if values['deadline_seconds'] != values['dispatch_seconds'] + values['budget_seconds']:
        return 'FAIL'
    if item['action_id'] != item['original_action_id'] or item['next_action'] != 'wait':
        return 'FAIL'
    if any(key not in values for key in ('lower_seconds', 'upper_seconds')):
        return 'BLOCKED'
    lower = values['lower_seconds'] - values['dispatch_seconds']
    upper = values['upper_seconds'] - values['dispatch_seconds']
    if lower < 0 or upper < lower:
        return 'BLOCKED'
    if lower > values['budget_seconds']:
        return 'FAIL'
    if not item['terminal_receipt'] or item['status'] != 'completed':
        return 'BLOCKED'
    if upper > values['budget_seconds']:
        return 'BLOCKED'
    return 'PASS'

results = []
for group, inputs, expected, classify in (
        ('sender', SEMANTIC_INPUTS, SEMANTIC_ORACLE, sender_result),
        ('timing', TIMING_INPUTS, TIMING_ORACLE, timing_result)):
    assert {item['case_id'] for item in inputs} == set(expected), (group, 'coverage mismatch')
    for item in inputs:
        observed = classify(item)
        assert observed == expected[item['case_id']], (item['case_id'], observed, expected[item['case_id']])
        results.append(dict(case=item['case_id'], group=group,
                            observed=observed, expected=expected[item['case_id']],
                            negative_control=expected[item['case_id']] != 'PASS'))
# Reference decisions for fixed synthetic traces, not native-agent execution.
def goal_result(item):
    if type(item.get('terminal')) is not bool:
        return 'BLOCKED'
    if item.get('replayed_unknown_effect') or item.get('wrong_instance'):
        return 'FAIL'
    if item.get('cancelled'):
        return 'STOP_CANCELLED'
    if item.get('permission_denied') or item.get('explicit_refusal'):
        return 'BLOCKED'
    if item.get('no_progress_rule_reached'):
        return 'STOP_NO_PROGRESS'
    if item.get('real_blocker') or not item.get('identity_verified', False):
        return 'BLOCKED'
    explicit = item.get('explicit_deadline')
    if explicit is not None:
        if explicit.get('deadline_source') not in ('user', 'business_sla', 'external_hard_cutoff'):
            return 'BLOCKED'
        try:
            clock_values = [Decimal(explicit[key]) for key in ('dispatch_seconds', 'budget_seconds',
                            'deadline_seconds', 'lower_seconds', 'upper_seconds')]
        except (KeyError, InvalidOperation, TypeError):
            return 'BLOCKED'
        if not all(value.is_finite() for value in clock_values):
            return 'BLOCKED'
        timing = dict(explicit, original_action_id='GOAL-A', action_id='GOAL-A',
                      next_action='wait', terminal_receipt=item.get('terminal', False),
                      status='completed' if item.get('terminal') else 'progress')
        verdict = timing_result(timing)
        if verdict == 'FAIL':
            return 'FAIL'
        if item.get('terminal') and verdict != 'PASS':
            return 'BLOCKED'
    if not item.get('terminal'):
        return 'CONTINUE' if item.get('progressing', True) else 'BLOCKED'
    if item.get('payload', 'missing') == 'malformed':
        return 'FAIL'
    if (item.get('payload', 'missing') != 'complete'
            or not item.get('sources_verified', False)
            or not item.get('required_goals')
            or not set(item['required_goals']).issubset(item.get('achieved_goals', []))):
        return 'BLOCKED'
    return 'PASS'


def wait_result(item):
    """A tool timeout is local; only an explicit deadline constrains the action."""
    if item.get('deadline_source') is None:
        if any(item.get(key) is not None for key in ('deadline_seconds', 'budget_seconds')):
            return 'FAIL_UNAUTHORIZED_DEFAULT'
        return 'CONTINUE'
    if item['deadline_source'] not in ('user', 'business_sla', 'external_hard_cutoff'):
        return 'BLOCKED'
    try:
        now, deadline, minimum, requested = (Decimal(item[key]) for key in
            ('now_seconds', 'deadline_seconds', 'host_min_wait_seconds', 'requested_wait_seconds'))
    except (KeyError, InvalidOperation, TypeError):
        return 'BLOCKED'
    if not all(value.is_finite() for value in (now, deadline, minimum, requested)):
        return 'BLOCKED'
    if minimum <= 0 or requested < minimum:
        return 'BLOCKED_WAIT_CAPABILITY'
    remaining = deadline - now
    if remaining <= 0:
        return 'STOP_EXPLICIT_DEADLINE'
    if remaining < minimum or requested > remaining:
        return 'BLOCKED_WAIT_CAPABILITY'
    return 'CONTINUE'


GOAL_INPUTS = [
    {'case_id': 'G01', 'terminal': True, 'observed_seconds': '250', 'label': 'No deadline; complete after historical SLA example boundary'},
    {'case_id': 'G02', 'terminal': False, 'progressing': True, 'observed_seconds': '250', 'payload': 'incomplete', 'achieved_goals': ['answer']},
    {'case_id': 'G03', 'terminal': False, 'progressing': True, 'tool_wait_timeout': True},
    {'case_id': 'G04', 'terminal': True, 'payload': 'missing'},
    {'case_id': 'G05', 'terminal': True, 'payload': 'malformed'},
    {'case_id': 'G06', 'terminal': True, 'achieved_goals': ['answer']},
    {'case_id': 'G07', 'terminal': True, 'sources_verified': False},
    {'case_id': 'G08', 'terminal': True, 'identity_verified': False},
    {'case_id': 'G09', 'terminal': False, 'cancelled': True, 'progressing': True},
    {'case_id': 'G10', 'terminal': False, 'permission_denied': True},
    {'case_id': 'G11', 'terminal': False, 'explicit_refusal': True},
    {'case_id': 'G12', 'terminal': False, 'no_progress_rule_reached': True},
    {'case_id': 'G13', 'terminal': False, 'real_blocker': True},
    {'case_id': 'G14', 'terminal': True, 'replayed_unknown_effect': True},
    {'case_id': 'G15', 'terminal': True, 'wrong_instance': True},
    {'case_id': 'G16', 'terminal': True},
    {'case_id': 'G17', 'terminal': True, 'explicit_deadline': {'deadline_source': 'user', 'dispatch_seconds': '10', 'budget_seconds': '37.5', 'deadline_seconds': '47.5', 'lower_seconds': '47.499', 'upper_seconds': '47.499'}},
    {'case_id': 'G18', 'terminal': True, 'explicit_deadline': {'deadline_source': 'business_sla', 'dispatch_seconds': '10', 'budget_seconds': '37.5', 'deadline_seconds': '47.5', 'lower_seconds': '47.5', 'upper_seconds': '47.5'}},
    {'case_id': 'G19', 'terminal': True, 'explicit_deadline': {'deadline_source': 'external_hard_cutoff', 'dispatch_seconds': '10', 'budget_seconds': '37.5', 'deadline_seconds': '47.5', 'lower_seconds': '47.501', 'upper_seconds': '47.501'}},
    {'case_id': 'G20', 'terminal': True, 'explicit_deadline': {'deadline_source': 'user', 'dispatch_seconds': '10', 'budget_seconds': '37.5', 'deadline_seconds': '47.5', 'lower_seconds': '47', 'upper_seconds': '48'}},
    {'case_id': 'G21', 'terminal': True, 'explicit_deadline': {'deadline_source': 'user', 'dispatch_seconds': None, 'budget_seconds': '37.5', 'deadline_seconds': '47.5', 'lower_seconds': '47', 'upper_seconds': '47'}},
    {'case_id': 'G22', 'terminal': False, 'tool_wait_timeout': True, 'progressing': True, 'explicit_deadline': {'deadline_source': 'user', 'dispatch_seconds': '10', 'budget_seconds': '37.5', 'deadline_seconds': '47.5', 'lower_seconds': '20', 'upper_seconds': '21'}},
    {'case_id': 'G23', 'terminal': False, 'explicit_deadline': {'deadline_source': 'user', 'dispatch_seconds': '10', 'budget_seconds': '37.5', 'deadline_seconds': '47.5', 'lower_seconds': '48', 'upper_seconds': '48'}},
]
GOAL_ORACLE = {'G01': 'PASS', 'G02': 'CONTINUE', 'G03': 'CONTINUE', 'G04': 'BLOCKED',
    'G05': 'FAIL', 'G06': 'BLOCKED', 'G07': 'BLOCKED', 'G08': 'BLOCKED',
    'G09': 'STOP_CANCELLED', 'G10': 'BLOCKED', 'G11': 'BLOCKED', 'G12': 'STOP_NO_PROGRESS',
    'G13': 'BLOCKED', 'G14': 'FAIL', 'G15': 'FAIL', 'G16': 'PASS', 'G17': 'PASS',
    'G18': 'PASS', 'G19': 'FAIL', 'G20': 'BLOCKED', 'G21': 'BLOCKED',
    'G22': 'CONTINUE', 'G23': 'FAIL'}
WAIT_INPUTS = [
    {'case_id': 'Q01'},
    {'case_id': 'Q02', 'tool_wait_timeout': True, 'observed_seconds': '10000'},
    {'case_id': 'Q03', 'budget_seconds': '60', 'deadline_seconds': '60'},
    {'case_id': 'Q04', 'deadline_source': 'user', 'now_seconds': '34', 'deadline_seconds': '40', 'host_min_wait_seconds': '10', 'requested_wait_seconds': '10'},
    {'case_id': 'Q05', 'deadline_source': 'business_sla', 'now_seconds': '20', 'deadline_seconds': '40', 'host_min_wait_seconds': '10', 'requested_wait_seconds': '10'},
    {'case_id': 'Q06', 'deadline_source': 'external_hard_cutoff', 'now_seconds': '30', 'deadline_seconds': '40', 'host_min_wait_seconds': '10', 'requested_wait_seconds': '10'},
    {'case_id': 'Q07', 'deadline_source': 'user', 'now_seconds': '31', 'deadline_seconds': '40', 'host_min_wait_seconds': '10', 'requested_wait_seconds': '9'},
    {'case_id': 'Q08', 'deadline_source': 'user', 'now_seconds': '40', 'deadline_seconds': '40', 'host_min_wait_seconds': '10', 'requested_wait_seconds': '10'},
    {'case_id': 'Q09', 'deadline_source': 'user', 'now_seconds': 'NaN', 'deadline_seconds': '40', 'host_min_wait_seconds': '10', 'requested_wait_seconds': '10'},
]
WAIT_ORACLE = {'Q01': 'CONTINUE', 'Q02': 'CONTINUE', 'Q03': 'FAIL_UNAUTHORIZED_DEFAULT',
    'Q04': 'BLOCKED_WAIT_CAPABILITY', 'Q05': 'CONTINUE', 'Q06': 'CONTINUE',
    'Q07': 'BLOCKED_WAIT_CAPABILITY', 'Q08': 'STOP_EXPLICIT_DEADLINE', 'Q09': 'BLOCKED'}
# Every fixture supplies explicit known identity/source/goal evidence. The
# classifier itself does not manufacture missing evidence from defaults.
GOAL_BASE = dict(identity_verified=True, sources_verified=True, payload='complete',
                 required_goals=['answer', 'sources'], achieved_goals=['answer', 'sources'])
GOAL_INPUTS = [dict(GOAL_BASE, **item) for item in GOAL_INPUTS]
for case, missing in (('G24', 'identity_verified'), ('G25', 'sources_verified'),
                      ('G26', 'payload'), ('G27', 'required_goals')):
    row = dict(GOAL_BASE, terminal=True, case_id=case)
    row.pop(missing)
    GOAL_INPUTS.append(row)
    GOAL_ORACLE[case] = 'BLOCKED'
GOAL_INPUTS.append(dict(GOAL_BASE, case_id='G28', terminal='completed'))
GOAL_ORACLE['G28'] = 'BLOCKED'
for group, inputs, expected, classify in (
        ('goal', GOAL_INPUTS, GOAL_ORACLE, goal_result),
        ('tool-wait', WAIT_INPUTS, WAIT_ORACLE, wait_result)):
    assert {item['case_id'] for item in inputs} == set(expected), (group, 'coverage mismatch')
    for item in inputs:
        observed = classify(item)
        assert observed == expected[item['case_id']], (item['case_id'], observed, expected[item['case_id']])
        results.append(dict(case=item['case_id'], group=group, observed=observed,
                            expected=expected[item['case_id']]))
for value in ('0', '121', '10000', None):
    assert goal_result(dict(GOAL_BASE, terminal=True, observed_seconds=value)) == 'PASS'
    assert goal_result(dict(GOAL_BASE, terminal=False, progressing=True, observed_seconds=value)) == 'CONTINUE'
sequence = [dict(terminal=False, tool_wait_timeout=True), dict(terminal=False, progressing=True),
            dict(terminal=False, tool_wait_timeout=True), dict(terminal=True)]
assert [goal_result(dict(GOAL_BASE, **item)) for item in sequence] == ['CONTINUE', 'CONTINUE', 'CONTINUE', 'PASS']

print(json.dumps(dict(verdict='PASS', layer='deterministic fixture/oracle and source-history checks',
    tested_sha=None, base_sha=base, native_runtime=False,
    report_prefix_bytes=len(published), original_guide_blocks_unchanged=3, prior_copy_controls_preserved=len(prior_controls), whole8e_negative_control="rejected", goal_completion_cases=len(GOAL_INPUTS), wait_cases=len(WAIT_INPUTS),
    whole5750_negative_control='rejected',
    cases=results, commands=commands,
    boundary='No native calls or model/cache/OS/network claim; synthetic decisions do not prove native behavior. Legacy timing inputs are explicit SLA examples only; goal completion has no default total deadline.'),
    ensure_ascii=False, indent=2))
PY
)
```


## 独立READY波次保持与公开报告传输

普通独立 READY 波次可有界派发后等待未完成集合；无明确截止也正常推进，不要求 deadline 字段；成员可保留诊断 dispatch_seconds，不能因此拒绝，也不能据此生成总预算/截止。 已提供且可比较的派发起点须先核对此前尚未完成成员的明确截止，再区分自身是否有总时限；缺少可选诊断起点不补造字段，已完成成员不再约束后续无截止成员。明确时限仅约束适用成员；原并发、依赖、实际资源隔离、同 Task 作者串行及全部退出码/清理不变。固定旧时长仅作明确 SLA 输入，新增无截止、混合截止、配额/共享资源/替换违规负控。

直接原始 UTF8 YAML 与原有可选外层包装均严格解析、拒绝重复键并验证原 schema/类型/ID；包装再验证编码、字节大小及摘要，不改变 decoded 载荷或搬移 not_run。完整性不证明实际作者或业务完成；显示占位与不透明持久参数不能证明送达损坏，不解密或编码秘密绕过宿主拒绝。实际调用另取完整原件与真实回执。

```bash
(
set -Eeuo pipefail
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B - <<'PY'
import base64, binascii, copy, hashlib, json, re, sys
from decimal import Decimal, InvalidOperation
from pathlib import Path
import yaml
assert not sys.flags.optimize
WAVE_INPUTS = [{'case_id': 'W01', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W02', 'mode': 'exclusive_single_action', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W03', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': False, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W04', 'mode': 'ordinary_ready_wave', 'approved_wave': False, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W05', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': False, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W06', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': False, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W07', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': None, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W08', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 3, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W09', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': True, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W10', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'write_report', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W11', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '122', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W12', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W13', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 2, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W14', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 3, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W15', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': False, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W16', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': True, 'completed_before_wait': []}, {'case_id': 'W17', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W18', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '121', 'budget_seconds': '120', 'deadline_seconds': '241', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W19', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': None, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'wait_deadline_seconds': '120', 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W20', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '120', 'deadline_seconds': '120', 'deadline_source': 'business_sla'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '120', 'deadline_seconds': '122', 'deadline_source': 'business_sla'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['B'], 'wait_target': ['B'], 'wait_deadline_seconds': '122', 'unrelated_interleaving': False, 'completed_before_wait': ['A']}, {'case_id': 'W21', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W22', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': False, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W23', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 5, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W24', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'same_task_author_overlap': True}, {'case_id': 'W25', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'generated_default_deadline': True}, {'case_id': 'W26', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'dispatch_replacement', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W27', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['B'], 'wait_target': ['B'], 'unrelated_interleaving': False, 'completed_before_wait': ['A']}, {'case_id': 'W28', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W29', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0'}, {'alias': 'B', 'dispatch_seconds': '2'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W30', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W31', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'budget_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W32', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W33', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2', 'budget_seconds': '40'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W34', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2', 'deadline_seconds': '40'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W35', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '39'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W36', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '40'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W37', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '41'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W38', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W39', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0'}, {'alias': 'B', 'dispatch_seconds': '41'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W40', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A'}, {'alias': 'B'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W41', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '41'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['B'], 'wait_target': ['B'], 'unrelated_interleaving': False, 'completed_before_wait': ['A']}, {'case_id': 'W42', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': True}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W43', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': 'NaN'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W44', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': 'Infinity'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W45', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': 'not-comparable'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W46', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '-1'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W47', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '39', 'budget_seconds': '40'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W48', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '39', 'deadline_seconds': '40'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W49', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'deadline_source': 'user', 'budget_seconds': '10', 'deadline_seconds': '12'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W50', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': None}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W51', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': True, 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}, {'case_id': 'W52', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': False}, {'case_id': 'W53', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'dispatch_seconds': '0'}, {'alias': 'B', 'dispatch_seconds': '10000'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': []}, {'case_id': 'W54', 'mode': 'ordinary_ready_wave', 'approved_wave': True, 'ready_evidence_verified': True, 'dependencies_verified': True, 'resource_isolation_verified': True, 'capacity_verified': True, 'dispatch_benefit_verified': True, 'host_limit_respected': True, 'task_count': 2, 'max_role_instances_after': 2, 'total_open_instances_after': 4, 'temporary_instances_after': 1, 'actions': [{'alias': 'A', 'deadline_source': 'user', 'dispatch_seconds': '0', 'budget_seconds': '40', 'deadline_seconds': '40'}, {'alias': 'B', 'dispatch_seconds': '2', 'deadline_source': 'unknown'}], 'b_dispatched_before_a_terminal': True, 'policy_requires_a_terminal_before_b': False, 'next_action': 'wait_outstanding', 'outstanding': ['A', 'B'], 'wait_target': ['A', 'B'], 'unrelated_interleaving': False, 'completed_before_wait': [], 'wait_deadline_seconds': '40'}]
WAVE_ORACLE = {'W01': 'PASS', 'W02': 'FAIL', 'W03': 'FAIL', 'W04': 'FAIL', 'W05': 'FAIL', 'W06': 'FAIL', 'W07': 'BLOCKED', 'W08': 'FAIL', 'W09': 'FAIL', 'W10': 'FAIL', 'W11': 'FAIL', 'W12': 'FAIL', 'W13': 'FAIL', 'W14': 'FAIL', 'W15': 'FAIL', 'W16': 'FAIL', 'W17': 'FAIL', 'W18': 'FAIL', 'W19': 'BLOCKED', 'W20': 'PASS', 'W21': 'PASS', 'W22': 'FAIL', 'W23': 'FAIL', 'W24': 'FAIL', 'W25': 'FAIL', 'W26': 'FAIL', 'W27': 'PASS', 'W28': 'PASS', 'W29': 'PASS', 'W30': 'PASS', 'W31': 'FAIL', 'W32': 'FAIL', 'W33': 'FAIL', 'W34': 'FAIL', 'W35': 'PASS', 'W36': 'FAIL', 'W37': 'FAIL', 'W38': 'PASS', 'W39': 'PASS', 'W40': 'PASS', 'W41': 'PASS', 'W42': 'BLOCKED', 'W43': 'BLOCKED', 'W44': 'BLOCKED', 'W45': 'BLOCKED', 'W46': 'BLOCKED', 'W47': 'FAIL', 'W48': 'FAIL', 'W49': 'BLOCKED', 'W50': 'BLOCKED', 'W51': 'BLOCKED', 'W52': 'BLOCKED', 'W53': 'PASS', 'W54': 'BLOCKED'}
TRANSPORT_ORACLE = {'P01': 'PASS', 'P02': 'PASS', 'P03': 'FAIL', 'P04': 'FAIL', 'P05': 'FAIL', 'P06': 'FAIL', 'P07': 'FAIL', 'P08': 'FAIL', 'P09': 'FAIL', 'P10': 'FAIL', 'P11': 'FAIL', 'P12': 'FAIL', 'P13': 'FAIL', 'P14': 'FAIL'}
def wave_result(item):
    actions = item['actions']
    if item['mode'] == 'exclusive_single_action' and len(actions) > 1:
        return 'FAIL'
    if item['mode'] != 'ordinary_ready_wave':
        return 'BLOCKED'
    for key in ('approved_wave', 'ready_evidence_verified', 'dependencies_verified',
                'resource_isolation_verified', 'capacity_verified',
                'dispatch_benefit_verified', 'host_limit_respected'):
        if item.get(key) is None:
            return 'BLOCKED'
        if item[key] is not True:
            return 'FAIL'
    for key, upper in (('task_count', 2), ('max_role_instances_after', 2),
                       ('total_open_instances_after', 4), ('temporary_instances_after', 1)):
        if type(item[key]) is not int:
            return 'BLOCKED'
        if item[key] < 0 or item[key] > upper:
            return 'FAIL'
    if item['policy_requires_a_terminal_before_b']:
        return 'FAIL'
    if item['next_action'] != 'wait_outstanding' or item['unrelated_interleaving']:
        return 'FAIL'
    aliases = [action['alias'] for action in actions]
    if len(aliases) != len(set(aliases)) or item['task_count'] != len(actions):
        return 'FAIL'
    pending, complete = set(item['outstanding']), set(item['completed_before_wait'])
    if pending & complete or pending | complete != set(aliases):
        return 'FAIL'
    if set(item['wait_target']) != pending or not pending:
        return 'FAIL'
    if item.get('same_task_author_overlap') or item.get('generated_default_deadline'):
        return 'FAIL'
    deadlines = {}
    previous_start = None
    try:
        for action in actions:
            source = action.get('deadline_source')
            start = None
            # A supplied comparable start constrains scheduling relative to any
            # earlier unfinished explicit deadline, even without an own limit.
            if 'dispatch_seconds' in action:
                if isinstance(action['dispatch_seconds'], bool):
                    return 'BLOCKED'
                start = Decimal(action['dispatch_seconds'])
                if not start.is_finite():
                    return 'BLOCKED'
                if previous_start is not None and start < previous_start:
                    return 'BLOCKED'
                if any(start >= value for alias, value in deadlines.items() if alias in pending):
                    return 'FAIL'
                previous_start = start
            if source is None:
                # Missing starts remain optional; no own budget/deadline is inferred.
                if any(key in action for key in ('budget_seconds', 'deadline_seconds')):
                    return 'FAIL'
                continue
            if source not in ('user', 'business_sla', 'external_hard_cutoff'):
                return 'BLOCKED'
            if start is None:
                return 'BLOCKED'
            if any(isinstance(action[key], bool) for key in ('budget_seconds', 'deadline_seconds')):
                return 'BLOCKED'
            budget, deadline = (Decimal(action[key]) for key in ('budget_seconds', 'deadline_seconds'))
            if not all(value.is_finite() for value in (budget, deadline)) or budget <= 0:
                return 'BLOCKED'
            if deadline != start + budget:
                return 'FAIL'
            deadlines[action['alias']] = deadline
        constrained = [deadlines[alias] for alias in pending if alias in deadlines]
        if constrained:
            if isinstance(item['wait_deadline_seconds'], bool):
                return 'BLOCKED'
            wait_end = Decimal(item['wait_deadline_seconds'])
            if not wait_end.is_finite():
                return 'BLOCKED'
            if wait_end > min(constrained):
                return 'FAIL'
        elif item.get('wait_deadline_seconds') is not None:
            return 'FAIL'
    except (InvalidOperation, KeyError, TypeError):
        return 'BLOCKED'
    return 'PASS'

class UniqueLoader(yaml.SafeLoader):
    pass
def unique(loader, node, deep=False):
    result = {}
    for key, value in loader.construct_pairs(node, deep=deep):
        if key in result:
            raise ValueError('duplicate YAML key')
        result[key] = value
    return result
UniqueLoader.add_constructor(yaml.resolver.BaseResolver.DEFAULT_MAPPING_TAG, unique)
template = yaml.safe_load(Path('skills/ric-devflow/assets/templates/research-report.yaml').read_text())
required_payload_fields = set(template)
def validate_payload(blob, expected_id=None):
    if not isinstance(blob, bytes):
        raise ValueError('raw payload must be original UTF8 bytes')
    value = yaml.load(blob.decode('utf-8'), Loader=UniqueLoader)
    if not isinstance(value, dict) or set(value) != required_payload_fields:
        raise ValueError('decoded schema field set')
    if type(value['schema_version']) is not int or value['schema_version'] != 1:
        raise ValueError('decoded schema version')
    for key in ('author', 'created_at', 'question', 'research_id', 'status'):
        if not isinstance(value[key], str):
            raise ValueError('decoded string type')
    if not re.fullmatch(r'RESEARCH-\d{3}', value['research_id']):
        raise ValueError('decoded research ID')
    if expected_id is not None and value['research_id'] != expected_id:
        raise ValueError('decoded report identity')
    if value['status'] not in ('COMPLETE', 'PARTIAL', 'BLOCKED'):
        raise ValueError('decoded report status')
    for key in ('supersedes', 'root_issue_id', 'task_id'):
        if value[key] is not None and not isinstance(value[key], str):
            raise ValueError('decoded nullable ID type')
    for key in ('sources', 'facts', 'inferences', 'recommendations', 'unresolved', 'not_run'):
        if not isinstance(value[key], list):
            raise ValueError('decoded array type')
    return blob

def decode_report(wrapper):
    required = {'transport_version', 'encoding', 'content_type', 'payload_size_bytes',
                'payload_sha256', 'report_id', 'payload', 'read_log'}
    if set(wrapper) != required:
        raise ValueError('transport field set')
    if type(wrapper['transport_version']) is not int or wrapper['transport_version'] != 1:
        raise ValueError('transport version')
    if wrapper['encoding'] not in ('utf-8', 'base64') or wrapper['content_type'] != 'application/yaml; charset=utf-8':
        raise ValueError('transport encoding/content type')
    if type(wrapper['payload_size_bytes']) is not int or wrapper['payload_size_bytes'] < 0:
        raise ValueError('transport size type')
    if not all(isinstance(wrapper[key], str) for key in ('payload_sha256', 'report_id', 'payload', 'read_log')):
        raise ValueError('transport string type')
    if not re.fullmatch('[0-9a-f]{64}', wrapper['payload_sha256']):
        raise ValueError('transport digest format')
    if not re.fullmatch(r'RESEARCH-\d{3}', wrapper['report_id']):
        raise ValueError('transport report ID')
    blob = (wrapper['payload'].encode('utf-8') if wrapper['encoding'] == 'utf-8'
            else base64.b64decode(wrapper['payload'], validate=True))
    if len(blob) != wrapper['payload_size_bytes'] or hashlib.sha256(blob).hexdigest() != wrapper['payload_sha256']:
        raise ValueError('decoded byte size/digest mismatch')
    return validate_payload(blob, wrapper['report_id'])

payload = copy.deepcopy(template)
payload.update(research_id='RESEARCH-901', author='Synthetic Tester fixture',
    created_at='2026-11-14T00:00:00+00:00', question='Synthetic public transport fixture only',
    status='BLOCKED', unresolved=['Synthetic fixture; no business research performed.'])
original = yaml.safe_dump(payload, allow_unicode=True, sort_keys=False).encode()
def wrap(blob, encoding='base64'):
    return dict(transport_version=1, encoding=encoding,
        content_type='application/yaml; charset=utf-8', payload_size_bytes=len(blob),
        payload_sha256=hashlib.sha256(blob).hexdigest(), report_id='RESEARCH-901',
        payload=base64.b64encode(blob).decode() if encoding == 'base64' else blob.decode('utf-8'),
        read_log='Synthetic fixture only; no native action.')
transport_inputs = [dict(case_id='P01', wrapper=wrap(original, 'utf-8')),
                    dict(case_id='P02', wrapper=wrap(original))]
for case, key, value in (
        ('P03', 'payload_sha256', '0' * 64), ('P04', 'payload_size_bytes', len(original) + 1),
        ('P05', 'encoding', 'rot13'), ('P06', 'payload', '!invalid'),
        ('P08', 'report_id', 'RESEARCH-902'), ('P10', 'payload_size_bytes', True),
        ('P11', 'unexpected', 'extra')):
    item = wrap(original)
    item[key] = value
    transport_inputs.append(dict(case_id=case, wrapper=item))
transport_inputs.append(dict(case_id='P07', wrapper=wrap(b'\xff')))
missing = {key: value for key, value in payload.items() if key != 'not_run'}
transport_inputs.append(dict(case_id='P09', wrapper=wrap(yaml.safe_dump(missing, allow_unicode=True, sort_keys=False).encode())))
transport_inputs.append(dict(case_id='P12', wrapper=wrap(original + b'\nstatus: COMPLETE\n')))
no_log = wrap(original)
del no_log['read_log']
transport_inputs.append(dict(case_id='P13', wrapper=no_log))
bad_schema = copy.deepcopy(payload)
bad_schema['schema_version'] = True
transport_inputs.append(dict(case_id='P14', wrapper=wrap(yaml.safe_dump(bad_schema, allow_unicode=True, sort_keys=False).encode())))
transport_inputs.sort(key=lambda item: item['case_id'])
results = []
assert {item['case_id'] for item in WAVE_INPUTS} == set(WAVE_ORACLE)
for item in WAVE_INPUTS:
    actual = wave_result(item)
    assert actual == WAVE_ORACLE[item['case_id']], (item['case_id'], actual)
    results.append(dict(case=item['case_id'], group='wave', observed=actual,
                        expected=WAVE_ORACLE[item['case_id']]))
assert {item['case_id'] for item in transport_inputs} == set(TRANSPORT_ORACLE)
for item in transport_inputs:
    try:
        blob = decode_report(item['wrapper'])
        actual = 'PASS'
        assert blob == original, 'accepted payload bytes changed'
    except (ValueError, UnicodeError, binascii.Error, yaml.YAMLError):
        actual = 'FAIL'
    assert actual == TRANSPORT_ORACLE[item['case_id']], (item['case_id'], actual)
    results.append(dict(case=item['case_id'], group='transport', observed=actual,
                        expected=TRANSPORT_ORACLE[item['case_id']]))
identity = lambda value: hashlib.sha256(json.dumps(value, ensure_ascii=False, sort_keys=True).encode()).hexdigest()
raw_inputs = [
    ('U01', original, 'PASS'),
    ('U02', b'\xff', 'FAIL'),
    ('U03', yaml.safe_dump(missing).encode(), 'FAIL'),
    ('U04', original + b'\nstatus: COMPLETE\n', 'FAIL'),
    ('U05', yaml.safe_dump(dict(payload, root_issue_id=True)).encode(), 'FAIL'),
    ('U06', yaml.safe_dump(dict(payload, schema_version=True)).encode(), 'FAIL'),
    ('U07', yaml.safe_dump(dict(payload, facts='not-an-array')).encode(), 'FAIL'),
    ('U08', yaml.safe_dump(dict(payload, research_id='wrong')).encode(), 'FAIL'),
]
for case, raw, expected in raw_inputs:
    try:
        assert validate_payload(raw) == original
        actual = 'PASS'
    except (ValueError, UnicodeError, yaml.YAMLError):
        actual = 'FAIL'
    assert actual == expected, (case, actual, expected)
    results.append(dict(case=case, group='raw-UTF8-transport', observed=actual, expected=expected))

print(json.dumps(dict(verdict='PASS', tested_sha=None, native_runtime=False,
    layer='deterministic ordinary-wave/exclusive-window and public report transport fixtures',
    cases=results, wave_input_sha256=identity(WAVE_INPUTS),
    transport_input_sha256=identity(transport_inputs),
    boundary='No actual dispatch, concurrency or performance proof; wrapper checksum is transfer integrity, not author identity. Legacy explicit-SLA examples and deadline-free goal cases remain separate. Opaque message logs are not plaintext integrity evidence.'),
    ensure_ascii=False, indent=2))
PY
)
```


## 实际出站参数的发送前负控

在已有不可读消息覆盖内，核验准备实际提交的目标及message正文，而不是只证明磁盘原件正确。空白、唯一脱敏占位、未解析指针/身份、错误目标/范围、原件摘要不符或预检后参数改变均须在业务派发前拦截。合法完整可读的精确指针允许，不强迫重复原文。绑定实际参数UTF8字节摘要与原件路径/版本/来源/权限，提交同一参数组，变化重验；单动作和普通波次每次派发均适用，不改变既有等待与容量。

已发出后才确认缺损的原动作保留已发送/中断失败及原身份；有明确时限才保留原截止/已用时限预算，允许的控制恢复不重放业务；不同post-change动作须独立明确授权，不覆盖旧失败。下列负控只使用实际字符串和隔离文件，不调用任何原生角色或业务工具。准备参数预检不证明实际送达；显示脱敏或不透明日志表示不可与明文直接比较，也不证明损坏。实际消息缺损才恢复原件，未知项保留可观察性限制。禁止编码控制消息或秘密以绕过拒绝。

```bash
(
set -Eeuo pipefail
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 -B - <<'PY'
import copy, hashlib, json, re, tempfile
from pathlib import Path
INPUTS = [{'case_id': 'H01', 'actual_argument': '', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H02', 'actual_argument': ' \n\t', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H03', 'actual_argument': '{{SECRET_SYNTHETIC_HANDOFF}}', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H04', 'actual_argument': 'Read __UNRESOLVED_POINTER__.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H05', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': False, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H06', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': False, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H07', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H08', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'tester-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H09', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': True, 'phase': 'pre_dispatch', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H10', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'already_sent_recovery', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H11', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'already_sent_recovery', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '240', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H12', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'new_post_change', 'original_action': 'FAILED-OLD-201', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'INTERRUPTED_FAILED', 'business_replay': False, 'new_post_change_authorized': True, 'deadline_source': 'business_sla'}, {'case_id': 'H13', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'already_sent_recovery', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'original_deadline': '120', 'current_deadline': '120', 'old_failure_lifecycle': 'INTERRUPTED_FAILED', 'business_replay': True, 'new_post_change_authorized': False, 'deadline_source': 'business_sla'}, {'case_id': 'H14', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'already_sent_recovery', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'old_failure_lifecycle': 'RUNNING_OR_UNKNOWN', 'business_replay': False, 'new_post_change_authorized': False}, {'case_id': 'H15', 'actual_argument': 'Read bound-original-input.json', 'actual_target': 'researcher-fixture', 'bound_target': 'researcher-fixture', 'identity_complete': True, 'pointer_exists': True, 'pointer_digest_matches': True, 'argument_changed_after_preflight': False, 'phase': 'already_sent_recovery', 'original_action': 'BOUND-A', 'current_action': 'BOUND-A', 'old_failure_lifecycle': 'INTERRUPTED_FAILED', 'business_replay': False, 'new_post_change_authorized': False}]
ORACLE = {'H01': 'BLOCKED_BEFORE_DISPATCH', 'H02': 'BLOCKED_BEFORE_DISPATCH', 'H03': 'BLOCKED_BEFORE_DISPATCH', 'H04': 'BLOCKED_BEFORE_DISPATCH', 'H05': 'BLOCKED_BEFORE_DISPATCH', 'H06': 'BLOCKED_BEFORE_DISPATCH', 'H07': 'READY_TO_DISPATCH', 'H08': 'BLOCKED_BEFORE_DISPATCH', 'H09': 'BLOCKED_BEFORE_DISPATCH', 'H10': 'CONTROL_RECOVERY_ONLY', 'H11': 'BLOCKED_BEFORE_DISPATCH', 'H12': 'READY_NEW_ACTION', 'H13': 'BLOCKED_BEFORE_DISPATCH', 'H14': 'CONTROL_RECOVERY_ONLY', 'H15': 'BLOCKED_BEFORE_DISPATCH'}
def preflight(actual, expected_target, target, approved_argument_sha, pointer_root,
              pointer_sha, expected_control, item):
    if not isinstance(actual, str) or not actual.strip():
        return 'BLOCKED_BEFORE_DISPATCH'
    text = actual.strip()
    if re.fullmatch(r'\{\{(?:SECRET_[^{}]+|REDACTED)\}\}', text):
        return 'BLOCKED_BEFORE_DISPATCH'
    if re.search(r'__(?:PLANNER|UNRESOLVED|TEMPLATE)[A-Z0-9_]*__', text):
        return 'BLOCKED_BEFORE_DISPATCH'
    if hashlib.sha256(actual.encode()).hexdigest() != approved_argument_sha or target != expected_target:
        return 'BLOCKED_BEFORE_DISPATCH'
    match = re.fullmatch(r'Read ([A-Za-z0-9_.-]+)', text)
    if match is None or match.group(1) != 'bound-original-input.json':
        return 'BLOCKED_BEFORE_DISPATCH'
    pointer = pointer_root / match.group(1)
    if not pointer.is_file() or hashlib.sha256(pointer.read_bytes()).hexdigest() != pointer_sha:
        return 'BLOCKED_BEFORE_DISPATCH'
    control = json.loads(pointer.read_text())
    required = ('root_issue_id', 'task_id', 'action_id', 'source_identity', 'scope')
    if any(not control.get(key) for key in required) or control != expected_control:
        return 'BLOCKED_BEFORE_DISPATCH'
    if item['phase'] == 'already_sent_recovery':
        if (item['current_action'] != item['original_action'] or
            (item.get('deadline_source') is not None and item.get('current_deadline') != item.get('original_deadline')) or
            item['old_failure_lifecycle'] == 'INTERRUPTED_FAILED' or item['business_replay']):
            return 'BLOCKED_BEFORE_DISPATCH'
        return 'CONTROL_RECOVERY_ONLY'
    if item['phase'] == 'new_post_change':
        if not item['new_post_change_authorized'] or item['current_action'] == item['original_action']:
            return 'BLOCKED_BEFORE_DISPATCH'
        return 'READY_NEW_ACTION'
    return 'READY_TO_DISPATCH'

results = []
assert {x['case_id'] for x in INPUTS} == set(ORACLE)
for item in INPUTS:
    frozen_item = copy.deepcopy(item)
    expected_control = dict(root_issue_id='SYNTHETIC-ROOT', task_id='SYNTHETIC-TASK',
        action_id='BOUND-A', source_identity='fixed-content-fixture', scope=dict(read=['public-fixture'], write=[]))
    control = copy.deepcopy(expected_control)
    if not item['identity_complete']:
        control['source_identity'] = None
    with tempfile.TemporaryDirectory(prefix='devflow-outgoing-') as directory:
        root = Path(directory)
        pointer = root / 'bound-original-input.json'
        if item['pointer_exists']:
            pointer.write_text(json.dumps(control, ensure_ascii=False, sort_keys=True))
        expected_digest = hashlib.sha256(json.dumps(control, ensure_ascii=False, sort_keys=True).encode()).hexdigest()
        if not item['pointer_digest_matches']:
            expected_digest = '0' * 64
        actual = item['actual_argument']
        approved_sha = hashlib.sha256(actual.encode()).hexdigest()
        if item['argument_changed_after_preflight']:
            actual += '\nUnapproved change after preflight.'
        result = preflight(actual, item['bound_target'], item['actual_target'],
            approved_sha, root, expected_digest, expected_control, item)
        assert result == ORACLE[item['case_id']], (item['case_id'], result)
        assert item == frozen_item, 'old identity/applicable deadline/failure observation mutated'
        results.append(dict(case=item['case_id'], result=result,
            expected=ORACLE[item['case_id']],
            actual_argument_sha256=hashlib.sha256(actual.encode()).hexdigest(),
            business_dispatchable=result in ('READY_TO_DISPATCH', 'READY_NEW_ACTION'),
            actual_dispatches=0))
    assert not root.exists()
def logged_body_integrity(item):
    # Only authoritative original plaintext can be compared with original bytes.
    # An opaque or redacted log remains an observation limit even when an
    # independent acknowledgment confirms receipt of the request pointer.
    if item['representation'] != 'authoritative_original_plaintext':
        return 'UNKNOWN'
    return ('VERIFIED_BODY_INTEGRITY' if item['original_body'] == item['expected_body']
            else 'BODY_MISMATCH')

for case, item, expected in (
        ('L01', dict(representation='display_redacted', recipient_ack=False), 'UNKNOWN'),
        ('L02', dict(representation='opaque_persisted', recipient_ack=False), 'UNKNOWN'),
        ('L03', dict(representation='opaque_persisted', recipient_ack=True), 'UNKNOWN'),
        ('L04', dict(representation='authoritative_original_plaintext', original_body='Read input.yaml', expected_body='Read input.yaml'), 'VERIFIED_BODY_INTEGRITY'),
        ('L05', dict(representation='authoritative_original_plaintext', original_body='different', expected_body='Read input.yaml'), 'BODY_MISMATCH')):
    observed = logged_body_integrity(item)
    assert observed == expected, (case, observed, expected)
    results.append(dict(case=case, group='message-observability', result=observed, expected=expected))

print(json.dumps(dict(verdict='PASS', tested_sha=None, native_runtime=False,
    layer='actual argument text and isolated real pointer preflight fixtures',
    cases=results, cleanup=True, actual_dispatches=0,
    boundary='Prepared plaintext preflight is distinct from actual delivery. Display placeholders and opaque persisted representations are not comparable plaintext or proof of corruption; do not decrypt them. Already-sent/interrupted failures remain preserved; no actual tool or business call occurs.'),
    ensure_ascii=False, indent=2))
PY
)
```
