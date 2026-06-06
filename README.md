# Veterinary Behaviorist Agent

循证猫/伴侣动物行为 consult agent。适合接入 Claude Code、Codex 或其他支持本地命令/skill 的 agent 环境，用来回答猫应激、攻击、排泄、恐惧、疼痛/疾病相关行为变化等问题。

默认用法是 **host-agent mode**：Claude Code/Codex 自己就是推理模型。本项目只提供行为专科 prompt、本地文献检索脚本和可选 Zotero/PaperQA2 工具，不要求再接入另一个 LLM API。

## 工作方式

默认流程：

1. 用户显式调用 `/veterinary-behaviorist`。
2. 宿主 agent 读取 `skill/veterinary-behaviorist/SKILL.md`，按兽医行为专科流程工作。
3. 宿主 agent 用 `scripts/search_corpus.py` 检索本地文献片段。
4. 如果配置了 Zotero MCP，宿主 agent 可继续读取本地 Zotero 元数据、全文、笔记和注释。
5. 宿主 agent 用自己的模型生成回答，并只引用实际检索到的文献。

可选流程：

- 已经有 OpenAI-compatible LLM API 时，可以启用 PaperQA2，让 `scripts/consult.sh` 直接生成带引用的文献 QA。
- 没有额外 LLM API 时，不用 PaperQA2 QA，直接用 `search_corpus.py` 检索证据，由 Claude Code/Codex 自己回答。

## 文献语料

仓库不自带论文全文、摘要正文或 Zotero 附件。运行安装步骤后，脚本会从 PubMed、Unpaywall 和 Europe PMC 在本地生成检索语料。

这样做是因为摘要和全文的版权状态因期刊和文章而异，不同使用者也可能有自己的机构权限、Zotero 库和 PDF 来源。

仓库内的 `literature/cat-behavior.provenance.json` 只记录可公开引用的检索元数据：PMID、标题、年份、期刊和查询来源。实际问答语料由使用者在本机生成。

## 目录结构

```text
.
├── README.md
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

这些文件只供本地检索使用。

## 组件和下载链接

默认 host-agent mode 只需要：

- Python 3.11+：<https://www.python.org/downloads/>
- Claude Code、Codex 或其他能读取 skill 并运行本地命令的 agent 环境

文献获取用到的公开服务：

- NCBI E-utilities：<https://www.ncbi.nlm.nih.gov/books/NBK25501/>
- Unpaywall API：<https://unpaywall.org/products/api>
- Europe PMC REST API：<https://europepmc.org/RestfulWebService>

可选组件：

- Zotero 7+：<https://www.zotero.org/download/>
- Zotero MCP server：<https://pypi.org/project/zotero-mcp-server/>
- pipx：<https://pipx.pypa.io/stable/installation/>
- PaperQA2 / `paper-qa`：<https://github.com/Future-House/paper-qa>
- sentence-transformers：<https://www.sbert.net/docs/installation.html>
- LiteLLM provider 配置参考：<https://docs.litellm.ai/docs/providers>

## 快速接入 Claude Code / Codex

进入仓库根目录：

```bash
cd /path/to/vet-agent
```

生成本地文献语料：

```bash
cd literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
cd ..
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

安装 skill。

Codex：

```bash
mkdir -p ~/.codex/skills
ln -s /path/to/vet-agent/skill/veterinary-behaviorist ~/.codex/skills/veterinary-behaviorist
```

Claude Code：

```bash
mkdir -p ~/.claude/skills
ln -s /path/to/vet-agent/skill/veterinary-behaviorist ~/.claude/skills/veterinary-behaviorist
```

设置仓库路径：

```bash
export VET_AGENT_HOME=/path/to/vet-agent
```

建议把这行放进 shell profile，或在 agent 运行环境里配置同名环境变量。Skill 里不写死本机路径；宿主 agent 从 `VET_AGENT_HOME` 找仓库根目录。

调用：

```text
/veterinary-behaviorist
用兽医行为 skill 看一下这个 case
consult 兽医行为医生 agent
call the veterinary behaviorist
```

这个 skill 默认关闭。不要因为对话里出现“猫、狗、行为、攻击、应激、焦虑”等词就自动启用，必须显式调用。

## Host-Agent Mode

这是默认模式，不需要额外 LLM API。

宿主 agent 调用本地检索：

```bash
cd "$VET_AGENT_HOME"
python3 scripts/search_corpus.py "猫突然攻击主人，疼痛和 redirected aggression 怎么区分?"
python3 scripts/search_corpus.py "objective indicators of stress in cats" -n 10
```

`search_corpus.py` 会输出：

- 文献标题和年份
- DOI 或 PMID
- 语料来源
- citation
- 命中的文本片段

宿主 agent 读取这些片段后，用自己的模型完成医学分诊、动机诊断、方案和引用整理。

如果没有生成本地语料，先运行：

```bash
cd "$VET_AGENT_HOME"
cd literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
cd ..
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

## 给宿主 Agent 的执行规则

处理 case 时按这个顺序：

1. 确认物种、年龄、性别/绝育、行为、触发因素、时间线、伤害风险。
2. 先做医疗优先分诊：疼痛、皮肤病、泌尿、内分泌、神经、药物、认知退化等。
3. 用本地检索脚本查证据：

```bash
cd "$VET_AGENT_HOME"
python3 scripts/search_corpus.py "focused evidence question" -n 8
```

4. 如果 Zotero MCP 可用，再用 Zotero 搜索本地库，读取元数据、全文、笔记或注释。
5. 按动机分类行为问题。猫攻击要区分 fear/defensive、redirected、petting-induced、play、pain、territorial、predatory 等。
6. 方案包含环境管理、行为改变、风险控制、何时需要兽医行为专科或当面兽医。
7. 引用只来自 `search_corpus.py` 输出、PaperQA2 输出或 Zotero item。没有证据就明确说是临床推理，不要伪造 PMID、DOI、作者年份。

推荐输出结构：

```text
结论
医疗优先分诊
最可能诊断和鉴别
处理方案
长期相处策略
证据和引用
局限和升级条件
```

## 生成本地文献语料

先用 PubMed E-utilities 生成 RIS。这个步骤会把摘要写入本地 RIS 文件。

```bash
cd /path/to/vet-agent/literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
```

输出：

```text
literature/cat-behavior.ris
literature/cat-behavior.provenance.json
```

再抓取开放获取全文，并为本地检索生成 manifest：

```bash
cd /path/to/vet-agent
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

输出：

```text
papers/PMID*.abstract.txt
papers/PMID*.fulltext.txt
papers/PMID*.pdf
papers/manifest.csv
```

抓取策略：

1. 如果 `papers/PMID<pmid>.pdf` 已存在且大小正常，使用现有 PDF。
2. 尝试 Unpaywall 开放获取 PDF。
3. 尝试 Europe PMC 开放获取 full-text XML，转为纯文本。
4. 找不到全文时，使用 PubMed 摘要生成 `PMID*.abstract.txt`。

如果有机构权限获取的 PDF，可以手动放进 `papers/`：

```text
papers/PMID29099247.pdf
```

再运行：

```bash
python3 scripts/fetch_oa.py
```

脚本会优先使用已有 PDF，并重写 `papers/manifest.csv`。

## 可选：PaperQA2 Mode

PaperQA2 mode 会让 `pqa` 自己完成文献检索和回答生成，适合已经有 OpenAI-compatible LLM API 的使用者。

安装：

```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
pipx install "paper-qa>=5"
pipx inject paper-qa sentence-transformers
```

确认 `pqa` 可用：

```bash
pqa --help
```

配置环境变量：

```bash
cp .env.example .env
```

编辑 `.env`：

```bash
PQA_API_KEY=your-openai-compatible-provider-key
UNPAYWALL_EMAIL=you@example.com
```

`UNPAYWALL_EMAIL` 用于 Unpaywall API 联系邮箱。建议使用可联系到你的邮箱。

本项目默认 PaperQA2 配置：

- LLM：`mimo-v2.5-pro`
- API base：`https://token-plan-sgp.xiaomimimo.com/v1`
- API key 环境变量：`PQA_API_KEY`
- embedding：`st-multi-qa-MiniLM-L6-cos-v1`
- embedding 在本地跑，不调用外部 embedding API

要换模型或 provider，改 `settings.json`：

- `llm`
- `summary_llm`
- `llm_config.model_list[0].model_name`
- `llm_config.model_list[0].litellm_params.model`
- `llm_config.model_list[0].litellm_params.api_base`
- `summary_llm_config` 里的同名字段

`api_key` 建议继续用 `os.environ/PQA_API_KEY`。

建索引：

```bash
cd /path/to/vet-agent
./scripts/index.sh
```

问答：

```bash
cd /path/to/vet-agent
./scripts/consult.sh "猫的应激行为有哪些可靠的客观指标?"
./scripts/consult.sh "redirected aggression 的触发因素和攻击对象规律?"
./scripts/consult.sh "猫突然开始攻击主人，应该优先排查哪些医学原因?"
```

`consult.sh` 会读取 `.env`，检查 `PQA_API_KEY`，设置 `OPENAI_API_KEY`，再调用 `pqa --settings settings ask`。

## Zotero MCP 可选配置

Zotero MCP 用于搜索本地 Zotero 文献库、读取元数据、全文、笔记和注释。Host-agent mode 可以在 MCP 可用时同时使用本地语料检索和 Zotero。

安装：

```bash
pipx install zotero-mcp-server
```

确认命令：

```bash
zotero-mcp --help
```

Zotero 设置：

1. 安装并打开 Zotero 7+。
2. 打开 Zotero 本地 API。通常在 Settings / Advanced 相关设置里。
3. 保持 Zotero 运行。

如果本机 `localhost:23119` 返回 503，但 `127.0.0.1:23119` 正常，可以用启动器：

```bash
~/.local/pipx/venvs/zotero-mcp-server/bin/python /path/to/vet-agent/scripts/zotero_mcp_local.py serve
```

把 MCP server 注册到 Codex 或 Claude Code 时，命令使用上面这一行即可。注册完成后，agent 可以调用 Zotero MCP 工具搜索本地文献库。

导入 RIS 到 Zotero：

```bash
cd /path/to/vet-agent
curl -X POST http://127.0.0.1:23119/connector/import \
  -H "Content-Type: application/x-research-info-systems" \
  --data-binary @literature/cat-behavior.ris
```

重复导入会产生重复条目。需要去 Zotero 的 Duplicate Items 里合并，或导入到新 collection。

## 常见问题

`papers/manifest.csv` 不存在

先生成本地语料：

```bash
cd literature && NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
cd ..
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

`literature/cat-behavior.ris` 不存在

运行：

```bash
cd literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
```

`pqa: command not found`

只有 PaperQA2 mode 需要 `pqa`。安装后确认 pipx 的 bin 目录在 PATH：

```bash
python3 -m pipx ensurepath
export PATH="$HOME/.local/bin:$PATH"
```

`ERROR: 还没设置 PQA_API_KEY`

只有 PaperQA2 mode 需要 `PQA_API_KEY`。确认 `.env` 存在，并且不是占位值：

```bash
cat .env
```

Zotero MCP 连不上

确认 Zotero 正在运行、本地 API 已开启。再检查：

```bash
curl http://127.0.0.1:23119/api/
```

如果 `localhost` 失败、`127.0.0.1` 成功，使用 `scripts/zotero_mcp_local.py` 启动 MCP。

## 更新语料

修改 `literature/harvest_pubmed.py` 里的 `BUCKETS` 查询后重跑：

```bash
cd literature
NCBI_EMAIL=you@example.com python3 harvest_pubmed.py
cd ..
UNPAYWALL_EMAIL=you@example.com python3 scripts/fetch_oa.py
```

更新后检查：

```bash
python3 - <<'PY'
import csv
from pathlib import Path
root = Path(".")
missing = []
with (root / "papers/manifest.csv").open(newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):
        if not (root / "papers" / row["file_location"]).exists():
            missing.append(row["file_location"])
print("missing", len(missing))
for name in missing:
    print(name)
PY
```

确认无缺失后即可用于 `search_corpus.py`。PaperQA2 mode 还需要重跑：

```bash
./scripts/index.sh
```
