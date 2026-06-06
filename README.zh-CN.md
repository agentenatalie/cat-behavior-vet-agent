<h1 align="center">Veterinary Behaviorist Agent</h1>

<p align="center">
  面向 Claude Code、Codex 和本地优先 AI agent 的循证猫行为 consult 工具。
</p>

<p align="center">
  <a href="README.md">English</a> · <a href="README.zh-CN.md">中文</a>
</p>

<p align="center">
  <a href="LICENSE"><img alt="License: CC BY-NC-ND 4.0" src="https://img.shields.io/badge/License-CC_BY--NC--ND_4.0-lightgrey.svg"></a>
  <a href="https://www.python.org/downloads/"><img alt="Python 3.11+" src="https://img.shields.io/badge/Python-3.11%2B-blue.svg"></a>
  <a href="#原生-agent-模式"><img alt="Local-first mode" src="https://img.shields.io/badge/Mode-local--first-lightgrey.svg"></a>
</p>

循证猫/伴侣动物行为 consult agent。适合接入 Claude Code、Codex，或其他能读取 skill 文件并运行本地命令的 agent 环境。

默认模式是 **原生 Agent 模式**：Claude Code/Codex 自己就是推理模型，不需要额外接入另一个 LLM API。本仓库提供兽医行为专科 prompt、本地文献检索脚本、语料生成工具，以及可选的 Zotero MCP / PaperQA2 集成。

## 它能做什么

- 引导当前 agent 按兽医行为 consult 流程工作。
- 从 PubMed 派生的本地猫行为语料中检索证据。
- 支持猫应激、恐惧、攻击、排泄问题、疼痛/疾病相关行为变化、就诊处理等问题。
- 强制先做医学优先分诊，再解释行为动机。
- 要求引用来自真实检索结果，不伪造 PMID、DOI 或作者年份。
- 可选接入 Zotero MCP，读取本地文献库、笔记、注释和 PDF。
- 可选接入 PaperQA2，适合已经有 OpenAI-compatible LLM API 的用户。

## 架构

默认原生 Agent 模式：

1. 用户显式调用 `/veterinary-behaviorist`。
2. Claude Code、Codex 或其他兼容 agent 读取 `skill/veterinary-behaviorist/SKILL.md`。
3. 当前 agent 运行 `scripts/search_corpus.py` 检索本地文献片段。
4. 如果配置了 Zotero MCP，当前 agent 可以继续搜索本地 Zotero 文献库。
5. 当前 agent 用自己的模型生成最终回答，并且只引用检索到的来源。

可选 PaperQA2 mode：

1. 用户配置 OpenAI-compatible API key。
2. PaperQA2 索引本地语料。
3. `scripts/consult.sh` 调用 PaperQA2 检索并生成带引用的回答。

## 仓库结构

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

运行后会在本地生成：

```text
literature/cat-behavior.ris
papers/PMID*.abstract.txt
papers/PMID*.fulltext.txt
papers/PMID*.pdf
papers/manifest.csv
.pqa_index/
```

这些文件只供本地检索使用，不随仓库发布。

## 数据说明

仓库不包含论文全文、摘要正文、Zotero 附件或 Zotero 笔记。

语料由用户在本地从以下来源生成：

- PubMed E-utilities：<https://www.ncbi.nlm.nih.gov/books/NBK25501/>
- Unpaywall API：<https://unpaywall.org/products/api>
- Europe PMC REST API：<https://europepmc.org/RestfulWebService>

`literature/cat-behavior.provenance.json` 只保存公开可引用的书目信息：PMID、标题、年份、期刊和查询来源。真正用于问答的文本由使用者在本机生成。

## 环境要求

默认原生 Agent 模式：

- Python 3.11+
- Claude Code、Codex，或其他能读取本地指令并运行 shell 命令的 agent 环境

可选集成：

- Zotero 7+：<https://www.zotero.org/download/>
- Zotero MCP server：<https://pypi.org/project/zotero-mcp-server/>
- pipx：<https://pipx.pypa.io/stable/installation/>
- PaperQA2 / `paper-qa`：<https://github.com/Future-House/paper-qa>
- sentence-transformers：<https://www.sbert.net/docs/installation.html>
- LiteLLM provider 配置：<https://docs.litellm.ai/docs/providers>

## 快速开始

克隆仓库：

```bash
git clone https://github.com/agentenatalie/cat-behavior-vet-agent.git
cd cat-behavior-vet-agent
```

生成本地文献语料：

```bash
cd literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
cd ..
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

测试本地检索：

```bash
python3 scripts/search_corpus.py "owner-directed aggression in cats" -n 5
python3 scripts/search_corpus.py "猫突然攻击主人，疼痛和 redirected aggression 怎么区分?" -n 5
```

## 安装为 Claude Code / Codex Skill

Codex：

```bash
mkdir -p ~/.codex/skills
ln -s /path/to/cat-behavior-vet-agent/skill/veterinary-behaviorist ~/.codex/skills/veterinary-behaviorist
```

Claude Code：

```bash
mkdir -p ~/.claude/skills
ln -s /path/to/cat-behavior-vet-agent/skill/veterinary-behaviorist ~/.claude/skills/veterinary-behaviorist
```

设置仓库路径：

```bash
export VET_AGENT_HOME=/path/to/cat-behavior-vet-agent
```

建议把这行放进 shell profile，或在 agent 运行环境中配置同名环境变量。

显式调用：

```text
/veterinary-behaviorist
use the veterinary behaviorist skill for this case
consult the veterinary behaviorist agent
用兽医行为 skill 看一下这个 case
```

这个 skill 默认关闭。不要因为对话里出现“猫、狗、行为、攻击、应激、焦虑”等词就自动启用。

## 原生 Agent 模式

这是默认模式，不需要额外 LLM API。

当前 agent 检索本地证据：

```bash
cd "$VET_AGENT_HOME"
python3 scripts/search_corpus.py "objective indicators of stress in cats" -n 10
python3 scripts/search_corpus.py "cat owner-directed aggression treatment" -n 10
```

`search_corpus.py` 会输出：

- 标题和年份
- DOI 或 PMID
- 语料来源
- citation
- 命中的文本片段

当前 agent 读取这些片段后，用自己的模型完成 consult 回答。

## 给当前 Agent 的行为约定

skill 被激活后，当前 agent 应该：

1. 复述 case：物种、年龄、性别/绝育、行为、触发因素、时间线、伤害风险。
2. 先做医学优先分诊：疼痛、皮肤病、泌尿系统疾病、内分泌疾病、神经问题、药物影响、认知退化。
3. 用 `scripts/search_corpus.py` 检索证据。
4. Zotero MCP 可用且相关时，继续使用 Zotero 搜索本地库。
5. 按动机分类攻击：fear/defensive、redirected、petting-induced、play、pain、territorial、predatory 等。
6. 给出管理、环境调整、行为改变、安全阈值和转诊条件。
7. 只引用本地检索、Zotero 或 PaperQA2 返回的证据。
8. 证据只有摘要、证据弱、来自外推或没有证据时，要明确说明不确定性。

推荐回答结构：

```text
结论
医学分诊
最可能诊断和鉴别
处理方案
长期相处策略
证据和引用
局限和升级条件
```

## 生成或刷新语料

生成 PubMed RIS：

```bash
cd /path/to/cat-behavior-vet-agent/literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
```

抓取开放获取全文并生成 `papers/manifest.csv`：

```bash
cd /path/to/cat-behavior-vet-agent
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

抓取策略：

1. 如果本地已有 `papers/PMID<pmid>.pdf`，优先使用。
2. 尝试 Unpaywall 开放获取 PDF。
3. 尝试 Europe PMC 开放获取 full-text XML，并转成文本。
4. 找不到全文时，使用 PubMed 摘要。

如果你有合法获取的 PDF，可以放到：

```text
papers/PMID29099247.pdf
```

然后刷新 manifest：

```bash
python3 scripts/fetch_oa.py
```

## 可选：PaperQA2 Mode

PaperQA2 mode 会让 `pqa` 自己检索并生成带引用回答。适合已经有 OpenAI-compatible LLM API 的用户。

安装：

```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
pipx install "paper-qa>=5"
pipx inject paper-qa sentence-transformers
```

配置：

```bash
cp .env.example .env
```

编辑 `.env`：

```bash
PQA_API_KEY=your-openai-compatible-provider-key
UNPAYWALL_EMAIL=you@example.com
```

索引：

```bash
cd /path/to/cat-behavior-vet-agent
./scripts/index.sh
```

提问：

```bash
./scripts/consult.sh "What are reliable objective indicators of stress in cats?"
```

默认 PaperQA2 配置在 `settings.json`。默认模型是通过 OpenAI-compatible endpoint 调用的 `mimo-v2.5-pro`。要换 provider，修改 `settings.json` 中的 `llm`、`summary_llm`、model name、model ID 和 API base。建议保留 `api_key` 为 `os.environ/PQA_API_KEY`。

## 可选：Zotero MCP

Zotero MCP 让当前 agent 搜索本地 Zotero 文献库、读取元数据、查看全文，以及使用笔记和注释。

安装：

```bash
pipx install zotero-mcp-server
```

确认 Zotero 7+ 已安装、正在运行，并且本地 API 已开启。

如果 `localhost:23119` 失败但 `127.0.0.1:23119` 正常，可以使用启动器：

```bash
~/.local/pipx/venvs/zotero-mcp-server/bin/python /path/to/cat-behavior-vet-agent/scripts/zotero_mcp_local.py serve
```

把生成的 RIS 导入 Zotero：

```bash
cd /path/to/cat-behavior-vet-agent
curl -X POST http://127.0.0.1:23119/connector/import \
  -H "Content-Type: application/x-research-info-systems" \
  --data-binary @literature/cat-behavior.ris
```

重复导入会产生重复条目。可以使用 Zotero 的 Duplicate Items 视图合并，或导入到新 collection。

## FAQ

### 我需要另一个 LLM API 吗？

不需要。默认的原生 Agent 模式会使用 Claude Code、Codex 或你正在接入的 agent 作为推理模型。

### 为什么还有 PaperQA2 配置？

PaperQA2 是可选模式。它适合想要独立文献 QA 引擎，并且已经有 OpenAI-compatible API key 的用户。

### 为什么仓库不带论文？

文章摘要、全文和 PDF 的再分发权利不同。仓库提供可复现的抓取脚本和公开 provenance 元数据。

### 这是兽医诊断服务吗？

不是。它是证据检索和推理辅助工具，不能替代线下兽医或兽医行为专科医生。

## 贡献

见 [CONTRIBUTING.md](CONTRIBUTING.md)。

适合贡献的方向：

- 更好的 PubMed query buckets
- 更好的本地检索排序
- 其他物种或行为领域
- 更完整的 Zotero MCP 文档
- 语料生成和本地检索测试

## 安全和医学免责声明

本项目只用于教育和决策辅助。行为变化可能来自疼痛、疾病、药物影响或环境压力。涉及受伤、攻击升级、突然行为变化或福利风险时，应寻求线下兽医帮助。

## License

本项目使用 [Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International](LICENSE)。

你可以在非商业场景下署名分享未修改版本。商业使用和分发修改版本需要另行获得许可。

因为该许可证包含 NonCommercial 和 NoDerivatives 限制，本项目是 public-source 项目，不是 OSI 意义上的 open-source 项目。
