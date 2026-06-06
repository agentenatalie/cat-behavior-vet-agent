# Contributing

Thank you for improving Veterinary Behaviorist Agent. This project is a local-first agent workflow for evidence-based veterinary behavior support. Contributions should keep the project safe, citation-grounded, and easy to run in host-agent environments such as Claude Code and Codex.

## Good Contributions

- Better PubMed query buckets
- Better local retrieval ranking
- Tests for `scripts/search_corpus.py`, `scripts/fetch_oa.py`, and corpus validation
- Clearer setup docs for Claude Code, Codex, Zotero MCP, or PaperQA2
- Support for additional species or behavior domains
- Safer consult workflow language and escalation criteria

## Ground Rules

- Do not commit API keys, local Zotero exports, notes, annotations, PDFs, generated corpus text, or PaperQA2 indexes.
- Do not add fabricated citations.
- Do not add aversive, dominance-based, or punishment-first behavior advice.
- Keep host-agent mode as the default path. PaperQA2 and Zotero MCP should remain optional integrations.
- Treat medical claims carefully. Behavior change can be driven by pain or disease.

## Development Setup

```bash
git clone https://github.com/agentenatalie/cat-behavior-vet-agent.git
cd cat-behavior-vet-agent
```

Generate a local corpus if you need to test retrieval:

```bash
cd literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
cd ..
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

Run basic checks:

```bash
python3 -m py_compile scripts/search_corpus.py scripts/fetch_oa.py scripts/zotero_mcp_local.py literature/harvest_pubmed.py
bash -n scripts/consult.sh scripts/index.sh
python3 -m json.tool settings.json >/dev/null
python3 scripts/search_corpus.py "owner-directed aggression in cats" -n 3
```

Before opening a pull request, check that generated or private files are not staged:

```bash
git diff --cached --name-only | grep -E '(^\.env$|\.ris$|\.pdf$|papers/.*\.(txt|pdf)$|papers/manifest\.csv|\.pqa_index)' || true
```

## Pull Requests

Include:

- What changed
- Why it matters
- How you tested it
- Any safety, citation, or data-rights considerations

## 中文说明

欢迎贡献。这个项目的目标是让 Claude Code、Codex 等宿主 agent 以本地优先的方式执行循证兽医行为 consult。

贡献时请注意：

- 不要提交 API key、Zotero 本地库、笔记、注释、PDF、生成的摘要/全文语料或 PaperQA2 索引。
- 不要加入伪造引用。
- 不要加入惩罚优先、支配论或 aversive 行为建议。
- 默认路径应保持为 host-agent mode；PaperQA2 和 Zotero MCP 是可选增强。
- 医学相关表述要谨慎，行为变化可能来自疼痛或疾病。

提交 PR 时请写明改动内容、原因、测试方式，以及任何安全、引用或数据版权相关考虑。
