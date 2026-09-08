# DevFlow Skill 包验证指南

本指南维护可复跑的方法，[验证报告](DEVFLOW_SKILLS_VALIDATION.md)只记录某次执行的结果。以下代码块是包维护者在本地复制执行的检查片段，不是安装后的工作流 CLI，也不执行 DevFlow 状态机。行为用例的原始输入及独立判定见[工作流评测](../.agents/skills/_devflow_shared/evals/workflow-cases.md)与[触发评测](../.agents/skills/_devflow_shared/evals/trigger-cases.md)。

## 前置条件与受检对象

- 在本源码仓库根目录执行，具备 Git、Bash、Python 3.11+、PyYAML 及随 Codex 提供的 `quick_validate.py`。缺依赖即报告未运行，不自动安装软件或修改全局配置。
- 第一段仅读取源码；第二段只在本段创建的临时目录进行安装/Git 操作，并在退出时清理。不会写用户 Skill/Agent 目录，不使用业务仓库或真实凭据。两段均显式拒绝 Python 优化模式，避免环境变量 PYTHONOPTIMIZE 使断言检查失效。
- 受检集合包括 Git 跟踪文件，以及 `.agents/skills`、三平台 agents、`docs` 中尚未提交的交付文件；不包含用户 `.zcode/plans/`。报告自身参与语法/链接检查，但不进入候选摘要，避免自引用。
- 源码身份记录完整 HEAD、受检文件的路径/模式/内容摘要和集合 SHA256；工作区候选不能仅用 HEAD 表示。保留输出中的文件清单与运行环境，比较两次运行必须先核对集合身份。

## 基础静态检查

此完整 Bash 块在任何检查失败时返回非零；不会因最后一个 Skill 成功而覆盖前面的失败。`DEVFLOW_VALIDATOR_PATH` 可指向宿主实际随附验证器，默认采用 Codex 用户目录。链接检查支持本包使用的行内相对链接、显式 HTML ID 与 Markdown 标题锚点；不访问远程链接，不宣称解析任意 Markdown 扩展。

```bash
(
set -Eeuo pipefail
python3 -B - <<'PY'
import hashlib, json, os, re, stat, subprocess, sys, tomllib
from pathlib import Path
from urllib.parse import unquote, urlsplit
import yaml

if sys.flags.optimize:
    raise SystemExit('Validation requires Python assertions; disable PYTHONOPTIMIZE for this invocation')
root = Path.cwd()
def git(*args):
    return subprocess.check_output(['git', *args], cwd=root)

assert root == Path(git('rev-parse', '--show-toplevel').decode().strip())
tracked = set(filter(None, git('ls-files', '-z').decode().split('\0')))
extras = set(filter(None, git('ls-files', '--others', '--exclude-standard', '-z').decode().split('\0')))
prefixes = ('.agents/skills/', '.claude/agents/', '.codex/agents/', '.zcode/agents/', 'docs/')
paths = sorted(tracked | {p for p in extras if p.startswith(prefixes)})
roles = ('planner', 'reviewer', 'tester', 'implementer')
validator = Path(os.environ.get('DEVFLOW_VALIDATOR_PATH',
    str(Path(os.environ.get('CODEX_HOME', str(Path.home() / '.codex'))) /
        'skills/.system/skill-creator/scripts/quick_validate.py')))
assert validator.is_file(), 'Missing bundled Skill validator'
for role in roles:
    subprocess.run([sys.executable, '-B', str(validator),
                    str(root / '.agents/skills' / ('devflow-' + role))], check=True)

class UniqueLoader(yaml.SafeLoader):
    pass
def unique_mapping(loader, node, deep=False):
    result = {}
    for key_node, value_node in node.value:
        key = loader.construct_object(key_node, deep=deep)
        if key in result:
            raise ValueError('Duplicate YAML key: ' + str(key))
        result[key] = loader.construct_object(value_node, deep=deep)
    return result
UniqueLoader.add_constructor(yaml.resolver.BaseResolver.DEFAULT_MAPPING_TAG, unique_mapping)

def prose(text):
    lines, marker, length = [], None, 0
    for line in text.splitlines():
        fence = re.match(r'^\s*(`{3,}|~{3,})(.*)$', line)
        if fence:
            run, tail = fence.groups()
            if marker is None:
                marker, length = run[0], len(run)
            elif run[0] == marker and len(run) >= length and not tail.strip():
                marker = None
            continue
        if marker is None:
            lines.append(line)
    assert marker is None, 'Unclosed fenced block'
    return '\n'.join(lines)

def anchors(text):
    body = prose(text)
    result = set(re.findall(r'<a\s+id=[\x22\x27]([^\x22\x27]+)', body))
    used = {}
    for heading in re.findall(r'^#{1,6}\s+(.+)$', body, re.M):
        heading = re.sub(r'\[([^]]+)\]\([^)]+\)', r'\1', heading)
        slug = re.sub(r'[^\w\- ]', '', heading.lower().strip()).replace(' ', '-')
        count = used.get(slug, 0)
        used[slug] = count + 1
        result.add(slug if count == 0 else slug + '-' + str(count))
    return result

counts = dict(yaml=0, toml=0, frontmatter=0, markdown=0, local_links=0, anchors=0)
manifest, fronts = [], {}
for name in paths:
    path = root / name
    mode = stat.S_IMODE(path.lstat().st_mode)
    if path.is_symlink():
        raw = os.readlink(path).encode()
        assert path.exists(), 'Broken symlink: ' + name
        entry = dict(path=name, mode='symlink', sha256=hashlib.sha256(raw).hexdigest())
    else:
        raw = path.read_bytes()
        entry = dict(path=name, mode=oct(mode), sha256=hashlib.sha256(raw).hexdigest())
    if name != 'docs/DEVFLOW_SKILLS_VALIDATION.md':
        manifest.append(entry)
    if path.is_symlink():
        continue
    if path.suffix not in ('.md', '.yaml', '.yml', '.toml'):
        continue
    text = raw.decode('utf-8')
    if path.suffix in ('.yaml', '.yml'):
        yaml.load(text, Loader=UniqueLoader)
        counts['yaml'] += 1
    if path.suffix == '.toml':
        tomllib.loads(text)
        counts['toml'] += 1
    if path.suffix == '.md':
        counts['markdown'] += 1
        front = re.match(r'^---\n(.*?)\n---(?:\n|$)', text, re.S)
        if front:
            fronts[name] = yaml.load(front.group(1), Loader=UniqueLoader)
            counts['frontmatter'] += 1
        for target in re.findall(r'\[[^\]\n]*\]\(([^)\n]+)\)', prose(text)):
            parsed = urlsplit(target.strip().strip('<>'))
            if parsed.scheme or parsed.netloc:
                continue
            destination = path.parent / unquote(parsed.path) if parsed.path else path
            assert destination.exists(), name + ': missing ' + target
            counts['local_links'] += 1
            if parsed.fragment:
                assert unquote(parsed.fragment) in anchors(destination.read_text()), name + ': missing anchor ' + target
                counts['anchors'] += 1

skills = {p.resolve() for p in (root / '.agents/skills').rglob('SKILL.md')}
expected = {(root / '.agents/skills' / ('devflow-' + r) / 'SKILL.md').resolve() for r in roles}
assert skills == expected
assert not (root / '.agents/skills/_devflow_shared/SKILL.md').exists()
assert (root / '.claude/skills').is_symlink()
assert os.readlink(root / '.claude/skills') == '../.agents/skills'
assert (root / '.claude/skills').resolve() == (root / '.agents/skills').resolve()
for host in ('claude', 'zcode'):
    native = root / ('.' + host) / 'agents'
    assert {p.name for p in native.iterdir()} == {'devflow-' + r + '.md' for r in roles}
    for role in roles:
        name = '.' + host + '/agents/devflow-' + role + '.md'
        assert not (root / name).is_symlink()
        front = fronts[name]
        assert front['name'] == 'devflow-' + role and front['model'] == 'inherit'
        expected_tools = {'Read', 'Grep', 'Glob'}
        if role != 'reviewer':
            expected_tools |= {'Bash', 'Edit', 'Write'}
        actual = front['tools']
        actual = set(x.strip() for x in actual.split(',')) if isinstance(actual, str) else set(actual)
        assert actual == expected_tools, name
        assert front.get('injectAgentsMd') is True if host == 'zcode' else 'injectAgentsMd' not in front
for role in roles:
    config = tomllib.loads((root / '.codex/agents' / ('devflow-' + role + '.toml')).read_text())
    assert config['name'] == 'devflow_' + role
    assert config['sandbox_mode'] == ('read-only' if role == 'reviewer' else 'workspace-write')
    policy = yaml.safe_load((root / '.agents/skills' / ('devflow-' + role) / 'agents/openai.yaml').read_text())
    assert policy['policy']['allow_implicit_invocation'] is (role == 'planner')
assert not any('/scripts/' in p or '/adapters/' in p for p in paths if p.startswith('.agents/'))
assert not any(Path(p).suffix in ('.py', '.js', '.ts', '.sh') for p in paths if p.startswith('.agents/'))
for name in ('package.json', 'pyproject.toml', 'Cargo.toml'):
    assert not (root / name).exists(), 'Unexpected package manifest: ' + name
subprocess.run(['git', 'diff', '--check'], check=True)
subprocess.run(['git', 'diff', '--cached', '--check'], check=True)
identity = hashlib.sha256(json.dumps(manifest, ensure_ascii=False, sort_keys=True).encode()).hexdigest()
print(json.dumps(dict(head=git('rev-parse', 'HEAD').decode().strip(), candidate_sha256=identity,
    python=sys.version.split()[0], pyyaml=yaml.__version__, git=git('--version').decode().strip(),
    counts=counts, files=manifest, native_runtime=False), ensure_ascii=False, indent=2))
PY
)
```

更新规则时另外对照变更前完整提交检查 `.codex`、`agents/openai.yaml` 和原模板的逐字差异；本轮若要求不变，任何差异都应失败，不以结构可解析代替兼容核对。

## 隔离安装与 Git 字节验证

以下块直接提取 README 的 Claude/ZCode 安装片段，只替换测试目标参数，在临时目录验证首次安装、同源重跑、普通文件冲突和失效链接。Git fixture 使用合成作者和本地提交，不改 Git 用户配置；其 SHA 仅是演示身份。代码只验证安装、Git 字节、测试执行组合和命令失败，不模拟角色批准或证明状态机行为。

```bash
(
set -Eeuo pipefail
python3 -B - <<'PY'
import json, os, re, subprocess, sys, tempfile
from pathlib import Path

if sys.flags.optimize:
    raise SystemExit('Validation requires Python assertions; disable PYTHONOPTIMIZE for this invocation')
source = Path.cwd()
readme = (source / 'README.md').read_text()
blocks = re.findall(r'```bash\n(.*?)\n```', readme, re.S)
install = next(b for b in blocks if 'devflow_sources=()' in b)
assert install.count('DEVFLOW_HOST=claude') == 1
target_assignment = 'DEVFLOW_INSTALL_DIR="${HOME}/.claude"'
assert [line.strip() for line in re.findall(r'^[ \t]*DEVFLOW_INSTALL_DIR=.*$', install, re.M)] == [target_assignment], 'Installer target assignment changed; review before executing fixture'
install = install.replace('DEVFLOW_HOST=claude', 'DEVFLOW_HOST="${DEVFLOW_TEST_HOST}"')
install = install.replace(target_assignment,
                          'DEVFLOW_INSTALL_DIR="${DEVFLOW_TEST_INSTALL_DIR}"')
assert 'DEVFLOW_INSTALL_DIR="${HOME}/.claude"' not in install
checks = []
def check(ok, label):
    assert ok, label
    checks.append(label)
def run(argv, cwd, ok=True, env=None):
    result = subprocess.run(argv, cwd=cwd, env=env, text=True, capture_output=True)
    if ok and result.returncode:
        raise AssertionError(str(argv) + '\n' + result.stdout + result.stderr)
    return result

with tempfile.TemporaryDirectory(prefix='devflow-guide-') as directory:
    temp = Path(directory)
    def targets(host, destination):
        result = [destination / 'agents' / ('devflow-' + r + '.md')
                  for r in ('planner', 'reviewer', 'tester', 'implementer')]
        if host == 'claude':
            result += [destination / 'skills' / name for name in
                       ('devflow-planner', 'devflow-reviewer', 'devflow-tester', 'devflow-implementer', '_devflow_shared')]
        return result
    def invoke(host, destination):
        env = dict(os.environ, DEVFLOW_TEST_HOST=host, DEVFLOW_TEST_INSTALL_DIR=str(destination))
        return run(['bash', '-c', install], source, ok=False, env=env)
    for host in ('claude', 'zcode'):
        destination = temp / host
        check(invoke(host, destination).returncode == 0, host + ': first install')
        links = targets(host, destination)
        check(all(p.is_symlink() and p.exists() for p in links), host + ': valid links')
        before = [os.readlink(p) for p in links]
        check(invoke(host, destination).returncode == 0 and before == [os.readlink(p) for p in links], host + ': repeat')
        for kind in ('file', 'broken-link'):
            conflict_root = temp / (host + '-' + kind)
            conflict = conflict_root / 'agents/devflow-tester.md'
            conflict.parent.mkdir(parents=True)
            if kind == 'file':
                conflict.write_text('user-owned synthetic content\n')
            else:
                conflict.symlink_to('missing-fixture-target')
            check(invoke(host, conflict_root).returncode != 0, host + ': reject ' + kind)
            preserved = conflict.read_text() == 'user-owned synthetic content\n' if kind == 'file' else os.readlink(conflict) == 'missing-fixture-target'
            check(preserved, host + ': preserve ' + kind)
            check(all(not p.exists() and not p.is_symlink() for p in targets(host, conflict_root) if p != conflict), host + ': no partial install ' + kind)

    repo = temp / 'git-fixture'
    repo.mkdir()
    def git(*args, ok=True):
        return run(['git', *args], repo, ok=ok)
    git('init', '-q', '--template=', '-b', 'fixture')
    def commit(paths, message, author):
        git('add', '--', *paths)
        git('-c', 'user.name=' + author, '-c', 'user.email=fixture@example.invalid',
            '-c', 'commit.gpgsign=false', '-c', 'core.hooksPath=/dev/null', 'commit', '-qm', message)
        return git('rev-parse', 'HEAD').stdout.strip()
    (repo / 'normalize.py').write_text('def normalize(value):\n    return value.strip()\n')
    (repo / 'notes.txt').write_text('Synthetic rename fixture\n')
    (repo / 'obsolete.txt').write_text('Synthetic deletion fixture\n')
    base = commit(['normalize.py', 'notes.txt', 'obsolete.txt'], 'fixture baseline', 'Fixture Baseline')
    (repo / 'normalize.py').write_text('def normalize(value):\n    return value.strip().casefold()\n')
    (repo / 'test_unit.py').write_text('import unittest\nfrom normalize import normalize\nclass Unit(unittest.TestCase):\n    def test_case(self):\n        self.assertEqual(normalize(" X "), "x")\n')
    implementation = commit(['normalize.py', 'test_unit.py'], 'fixture production candidate', 'Fixture Implementer')
    check('Ran 1 test' in run([sys.executable, '-B', '-m', 'unittest', 'discover', '-v'], repo).stderr, 'previous candidate unit test')
    (repo / 'test_contract.py').write_text('import unittest\nfrom normalize import normalize\ndef expected():\n    return "ss".upper()\nclass Contract(unittest.TestCase):\n    def test_unicode(self):\n        self.assertEqual(normalize(" ß "), expected())\n')
    failed_test = commit(['test_contract.py'], 'fixture faulty test helper', 'Fixture Tester')
    failed = run([sys.executable, '-B', '-m', 'unittest', 'discover', '-v'], repo, ok=False)
    check(failed.returncode != 0 and 'FAIL' in failed.stderr, 'faulty helper fails on actual candidate')
    check(git('show', implementation + ':test_contract.py', ok=False).returncode != 0, 'previous PASS lacks new test')
    (repo / 'test_contract.py').write_text('import unittest\nfrom normalize import normalize\nclass Contract(unittest.TestCase):\n    def test_unicode(self):\n        self.assertEqual(normalize(" ß "), "ss")\n')
    candidate = commit(['test_contract.py'], 'fixture repair test helper', 'Fixture Tester')
    check(git('rev-parse', candidate + '^').stdout.strip() == failed_test, 'test repair starts from failed candidate')
    check(git('diff', '--name-only', implementation, candidate).stdout.strip() == 'test_contract.py', 'Tester delta is test-only')
    check(git('merge-base', '--is-ancestor', implementation, candidate).returncode == 0, 'candidate ancestry')
    (repo / 'notes.txt').rename(repo / 'renamed.txt')
    (repo / 'obsolete.txt').unlink()
    outside = temp / 'external-fixture.txt'
    outside.write_text('Synthetic external marker\n')
    (repo / 'external-link').symlink_to(outside)
    transport = commit(['notes.txt', 'renamed.txt', 'obsolete.txt', 'external-link'], 'fixture transport shapes', 'Fixture Baseline')
    diff = git('diff', '--no-ext-diff', '--no-textconv', '-M', '--name-status', candidate, transport).stdout
    check('R100\t' in diff and 'D\tobsolete.txt' in diff, 'rename and delete preserved')
    check('120000 blob' in git('ls-tree', transport, 'external-link').stdout, 'symlink mode')

    (repo / 'normalize.py').write_text('raise RuntimeError("WORKTREE ONLY")\n')
    cache = temp / 'exact-candidate'
    cache.mkdir()
    for record in git('ls-tree', '-r', '-z', candidate).stdout.split('\0'):
        if not record:
            continue
        metadata, name = record.split('\t', 1)
        mode, kind, oid = metadata.split()
        check(mode == '100644' and kind == 'blob', 'expected candidate mode: ' + name)
        raw = subprocess.check_output(['git', 'cat-file', 'blob', oid], cwd=repo)
        destination = cache / name
        destination.parent.mkdir(parents=True, exist_ok=True)
        destination.write_bytes(raw)
        check(raw == subprocess.check_output(['git', 'show', candidate + ':' + name], cwd=repo), 'exact bytes: ' + name)
    result = run([sys.executable, '-B', '-m', 'unittest', 'discover', '-v'], cache)
    check('Ran 2 tests' in result.stderr and 'OK' in result.stderr, 'exact candidate tests')
    check('WORKTREE ONLY' not in (cache / 'normalize.py').read_text(), 'dirty worktree excluded')
    link_text = git('show', transport + ':external-link').stdout
    check(link_text == str(outside) and 'Synthetic external marker' not in link_text, 'symlink blob not followed')
    check(git('show', candidate + ':missing.txt', ok=False).returncode != 0, 'missing object fails')
    check(run([sys.executable, '-c', 'raise SystemExit(7)'], cache, ok=False).returncode == 7, 'nonzero exit preserved')
    check(git('show', candidate + ':normalize.py').stdout == (cache / 'normalize.py').read_text(), 'persistent Git source remains recoverable')
    print(json.dumps(dict(checks=len(checks), passed=checks, base_sha=base,
        implementation_sha=implementation, failed_test_sha=failed_test, tested_sha=candidate, transport_sha=transport,
        candidate_tests=2, native_runtime=False), ensure_ascii=False, indent=2))
check(not temp.exists(), 'temporary fixture removed')
print('Temporary fixture cleanup: PASS')
PY
)
```

此处 `implementation_sha` 实跑一条单元测试，`failed_test_sha` 上新增测试辅助函数引发真实失败，再从该失败点修正；`tested_sha` 的两条合成用例通过。`transport_sha` 只演示删除/重命名/符号链接，不声称执行过该 SHA 的测试。这些 Git/测试事实不构成角色 Gate。临时仓库清理后 SHA 不承诺可解析，演示可由本段重建。

## 独立行为评测与失败注入

1. 在隔离候选副本上按 WF-34–39 和 TRIGGER-15–18 准备原始事实。分别给独立评测者原始请求、角色入口和最少原始材料；不发送对应“判定”段、怀疑的问题或预期答案。
2. 记录评测者实际决策、必要调用和产物身份；对照稳定不变量逐项判定。依赖解锁、报告作者、旧 SHA 复用、派发与授权均检查可观察行为，不用固定措辞或关键字出现次数判通过。
3. 只在可丢弃副本注入重复 YAML 键、断开的本地链接或失效安装目标，执行上述完整命令块并断言非零；验证正常副本随后仍成功。不能修改真实源树或降低检查规则来通过。
4. 没有原生宿主执行时，主/子会话情境回答仅记为“决定级”。ZCode 的自动调用必须观察实际 Agent 调用目标及回传/继续事件；Codex 独立子代理、安装检查和模拟日志均不替代该证据。

没有角色调用工具、权限或原生客户端时如实标注未运行；可以继续静态与隔离 Git 检查，不能把未运行项计入成功数。评测不读取登录凭据，不为测试修改用户全局配置。

## 记录与清理

在报告中追加本轮日期、源码基线、候选摘要、命令与退出码、环境版本、运行/复用范围、失败与修正、限制及清理结果。保留必要脱敏输出或可恢复附件的定位；本源码仓库不提交演示 `.devflow/changes/`。第一段输出的文件清单可放稳定报告附件，不能只留不可再取得的临时路径；本指南足以重新计算同一文件集合的摘要。

历史报告保持原文；新报告只声明真实重跑范围。Schema、Codex 配置与用户既有文件的兼容性由逐字比较确认，不拿检查数量或文档长度推断性能改善。
