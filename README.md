# Veterinary Behaviorist Agent

[English](README.md) | [中文](README.zh-CN.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/downloads/)
[![Local-first](https://img.shields.io/badge/Mode-local--first-lightgrey.svg)](#host-agent-mode)

Evidence-based veterinary behavior agent for cats and companion animals. It is designed for Claude Code, Codex, and other host agents that can read a skill file and run local commands.

The default mode uses the host agent itself as the reasoning model. You do not need to connect another LLM API. This repository provides the veterinary behavior prompt, local literature retrieval scripts, corpus generation tooling, and optional Zotero MCP / PaperQA2 integrations.

## What It Does

- Guides a host agent through a veterinary behavior consult workflow.
- Retrieves local evidence from a PubMed-derived cat behavior corpus.
- Supports cat stress, fear, aggression, elimination problems, pain or disease-related behavior change, and clinic handling questions.
- Encourages medical-first triage before behavioral interpretation.
- Requires citations from retrieved evidence. No fabricated PMID, DOI, or author-year citations.
- Supports optional Zotero MCP for local reference libraries, notes, annotations, and PDFs.
- Supports optional PaperQA2 mode when the user already has an OpenAI-compatible LLM API.

## Keywords

`veterinary-behavior`, `cat-behavior`, `animal-behavior`, `ai-agent`, `claude-code`, `codex`, `zotero`, `paperqa`, `pubmed`, `evidence-based`, `rag`, `local-first`, `veterinary-medicine`, `cats`

## Architecture

Default host-agent mode:

1. The user explicitly calls `/veterinary-behaviorist`.
2. Claude Code, Codex, or another host agent loads `skill/veterinary-behaviorist/SKILL.md`.
3. The host agent runs `scripts/search_corpus.py` to retrieve local evidence snippets.
4. If Zotero MCP is configured, the host agent can also search the local Zotero library.
5. The host agent writes the final answer using its own model and cites only retrieved sources.

Optional PaperQA2 mode:

1. The user configures an OpenAI-compatible API key.
2. PaperQA2 indexes the local corpus.
3. `scripts/consult.sh` asks PaperQA2 to retrieve and synthesize a cited answer.

## Repository Layout

```text
.
├── README.md
├── README.zh-CN.md
├── .env.example
├── .gitignore
├── settings.json
├── literature/
│   ├── README.md
│   ├── harvest_pubmed.py
│   └── cat-behavior.provenance.json
├── papers/
│   └── .gitkeep
├── scripts/
│   ├── search_corpus.py
│   ├── consult.sh
│   ├── fetch_oa.py
│   ├── index.sh
│   └── zotero_mcp_local.py
└── skill/
    └── veterinary-behaviorist/
        └── SKILL.md
```

Local corpus files are generated after setup:

```text
literature/cat-behavior.ris
papers/PMID*.abstract.txt
papers/PMID*.fulltext.txt
papers/PMID*.pdf
papers/manifest.csv
.pqa_index/
```

These files are for local retrieval and are not shipped with the repository.

## Data Policy

This repository does not include article full text, article abstracts, Zotero attachments, or Zotero notes.

The corpus is generated locally from:

- PubMed E-utilities: <https://www.ncbi.nlm.nih.gov/books/NBK25501/>
- Unpaywall API: <https://unpaywall.org/products/api>
- Europe PMC REST API: <https://europepmc.org/RestfulWebService>

`literature/cat-behavior.provenance.json` stores public bibliographic metadata: PMID, title, year, journal, and query source. Actual retrieval text is generated on the user's machine.

## Requirements

Default host-agent mode:

- Python 3.11+
- Claude Code, Codex, or another agent environment that can read local instructions and run shell commands

Optional integrations:

- Zotero 7+: <https://www.zotero.org/download/>
- Zotero MCP server: <https://pypi.org/project/zotero-mcp-server/>
- pipx: <https://pipx.pypa.io/stable/installation/>
- PaperQA2 / `paper-qa`: <https://github.com/Future-House/paper-qa>
- sentence-transformers: <https://www.sbert.net/docs/installation.html>
- LiteLLM provider configuration: <https://docs.litellm.ai/docs/providers>

## Quick Start

Clone the repository:

```bash
git clone https://github.com/agentenatalie/cat-behavior-vet-agent.git
cd cat-behavior-vet-agent
```

Generate the local literature corpus:

```bash
cd literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
cd ..
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

Test local retrieval:

```bash
python3 scripts/search_corpus.py "owner-directed aggression in cats" -n 5
python3 scripts/search_corpus.py "猫突然攻击主人，疼痛和 redirected aggression 怎么区分?" -n 5
```

## Install as a Claude Code or Codex Skill

Codex:

```bash
mkdir -p ~/.codex/skills
ln -s /path/to/cat-behavior-vet-agent/skill/veterinary-behaviorist ~/.codex/skills/veterinary-behaviorist
```

Claude Code:

```bash
mkdir -p ~/.claude/skills
ln -s /path/to/cat-behavior-vet-agent/skill/veterinary-behaviorist ~/.claude/skills/veterinary-behaviorist
```

Set the repository path:

```bash
export VET_AGENT_HOME=/path/to/cat-behavior-vet-agent
```

Put that line in your shell profile or configure the same environment variable in your agent runtime.

Call the skill explicitly:

```text
/veterinary-behaviorist
use the veterinary behaviorist skill for this case
consult the veterinary behaviorist agent
用兽医行为 skill 看一下这个 case
```

The skill is off by default. The host agent should not activate it just because a conversation mentions cats, dogs, behavior, aggression, stress, or anxiety.

## Host-Agent Mode

Host-agent mode is the default. It does not require an additional LLM API.

The host agent retrieves local evidence:

```bash
cd "$VET_AGENT_HOME"
python3 scripts/search_corpus.py "objective indicators of stress in cats" -n 10
python3 scripts/search_corpus.py "cat owner-directed aggression treatment" -n 10
```

`search_corpus.py` returns:

- Title and year
- DOI or PMID
- Corpus source
- Citation
- Matching snippet

The host agent then uses its own model to produce the consult answer.

## Behavior Contract for Host Agents

When the skill is activated, the host agent should:

1. Restate the case: species, age, sex/neuter status, behavior, triggers, timeline, and injury risk.
2. Start with medical-first triage: pain, skin disease, urinary disease, endocrine disease, neurologic disease, medication effects, cognitive decline.
3. Retrieve evidence with `scripts/search_corpus.py`.
4. Use Zotero MCP if available and relevant.
5. Classify aggression by motivation: fear/defensive, redirected, petting-induced, play, pain, territorial, predatory, or other supported categories.
6. Provide management, environmental modification, behavior modification, safety thresholds, and referral triggers.
7. Cite only retrieved evidence from local search, Zotero, or PaperQA2.
8. State uncertainty when evidence is abstract-only, weak, extrapolated, or missing.

Suggested answer format:

```text
Bottom line
Medical triage
Most likely diagnosis and differentials
Plan
Long-term living strategy
Evidence and citations
Limitations and escalation thresholds
```

## Generate or Refresh the Corpus

Generate PubMed RIS data:

```bash
cd /path/to/cat-behavior-vet-agent/literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
```

Fetch open-access full text when available and create `papers/manifest.csv`:

```bash
cd /path/to/cat-behavior-vet-agent
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

Fetch strategy:

1. Use an existing local `papers/PMID<pmid>.pdf` if present.
2. Try Unpaywall open-access PDF.
3. Try Europe PMC open-access full-text XML and convert it to text.
4. Use PubMed abstract text when no full text is available.

If you have lawful access to a PDF, place it in `papers/`:

```text
papers/PMID29099247.pdf
```

Then refresh the manifest:

```bash
python3 scripts/fetch_oa.py
```

## Optional: PaperQA2 Mode

PaperQA2 mode lets `pqa` retrieve and synthesize cited answers. Use it when you already have an OpenAI-compatible LLM API.

Install:

```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
pipx install "paper-qa>=5"
pipx inject paper-qa sentence-transformers
```

Configure:

```bash
cp .env.example .env
```

Edit `.env`:

```bash
PQA_API_KEY=your-openai-compatible-provider-key
UNPAYWALL_EMAIL=you@example.com
```

Index:

```bash
cd /path/to/cat-behavior-vet-agent
./scripts/index.sh
```

Ask:

```bash
./scripts/consult.sh "What are reliable objective indicators of stress in cats?"
```

Default PaperQA2 settings are in `settings.json`. The default model is `mimo-v2.5-pro` through an OpenAI-compatible endpoint. To use another provider, update `llm`, `summary_llm`, model names, model IDs, and API base values in `settings.json`. Keep `api_key` as `os.environ/PQA_API_KEY`.

## Optional: Zotero MCP

Zotero MCP lets the host agent search a local Zotero library, read metadata, inspect full text, and use notes or annotations.

Install:

```bash
pipx install zotero-mcp-server
```

Make sure Zotero 7+ is installed, running, and local API access is enabled.

If `localhost:23119` fails but `127.0.0.1:23119` works, use the launcher:

```bash
~/.local/pipx/venvs/zotero-mcp-server/bin/python /path/to/cat-behavior-vet-agent/scripts/zotero_mcp_local.py serve
```

Import the generated RIS file into Zotero:

```bash
cd /path/to/cat-behavior-vet-agent
curl -X POST http://127.0.0.1:23119/connector/import \
  -H "Content-Type: application/x-research-info-systems" \
  --data-binary @literature/cat-behavior.ris
```

Repeated imports can create duplicate Zotero items. Use Zotero's Duplicate Items view or import into a fresh collection.

## FAQ

### Do I need another LLM API?

No. The default host-agent mode uses Claude Code, Codex, or your existing host agent as the reasoning model.

### Why is there a PaperQA2 configuration?

PaperQA2 is an optional mode for users who want a separate literature-QA engine and already have an OpenAI-compatible API key.

### Why are papers not included?

Article abstracts, full text, and PDFs can have different redistribution rights. The repository includes reproducible retrieval scripts and public provenance metadata instead.

### Is this a veterinary diagnosis service?

No. It is an evidence-retrieval and reasoning aid. It cannot replace an in-person veterinarian or a board-certified veterinary behaviorist.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

Useful contribution areas:

- Better PubMed query buckets
- Higher-quality retrieval ranking
- Additional species or behavior domains
- More robust Zotero MCP setup docs
- Tests for corpus generation and local search

## Safety and Medical Disclaimer

This project is for educational and decision-support use. Behavior changes can be caused by pain, disease, medication effects, or environmental stressors. For injuries, escalating aggression, sudden behavior change, or welfare risk, seek in-person veterinary care.

## License

MIT. See [LICENSE](LICENSE).
