---
schema_version: 2
root_issue_id: "REQ-YYYYMMDD-001"
---

# 独立证据账本

<!-- 下面注释仅说明记录格式；首条实际记录后移除此注释。历史完整记录不得改写。 -->
<!--
每条记录使用唯一显式锚点及标题，例如 REVIEW-001。
围栏外的 YAML 容器元数据：
  record_id: REVIEW-001
  kind: review
  author: 独立角色作者
  created_at: 带时区时间
  objects:
    - id: SPEC
      revision: 1
      path: .devflow/changes/REQ-YYYYMMDD-001/current.md
      commit_sha: 真实完整文档 Commit SHA
  supersedes: null

无提交身份时以 snapshot: {path, sha256} 替代 commit_sha，
不得使用占位 SHA 送审。objects 与载荷一起原样保留。
随后使用足够长的独立 fenced block 原样收录原报告载荷；
其 schema_version: 1、代码 SHA、Verdict、Finding、未知字段不改写。

状态事件、用户批准、适用性与 Finding 处置也各自独立记录，
作者如实标明；先完整追加再更新 state；同 ID 同载荷不重复。
命令须有 cwd/环境/复现条件/退出码/关键输出及未运行原因。
永久证据不只引用 .local。纠错追加新 ID + supersedes。
-->
