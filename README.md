# tf-dna-ai — 转录因子–DNA 互作 × AI/分子模拟

追踪 **TF–DNA 结合与结合位点**方向的计算方法文献：结合/结合位点的预测、建模与分子机理，方法限定为深度学习等 AI、结构方法（AlphaFold / ESMFold / Rosetta）与分子动力学/模拟/对接。

对象用**互作中心词**驱动，但 PubMed 与预印本腿口径不同（本仓主题即 TF，裸 TF 只在 PubMed 用；预印本腿防越界）：

- **PubMed 腿** — `Transcription Factors` / `DNA-Binding Proteins` / `Zinc Fingers` 等 MeSH + `transcription factor*`、`TF binding`、`TFBS`、`DNA binding`、`DNA-binding protein*`、`protein-DNA binding` 等 tiab；
- **预印本腿**（arXiv/bioRxiv/medRxiv/chemRxiv）— 只收**锚定短语**：transcription factor binding (site)、TF binding/TFBS、DNA binding、DNA-binding protein/site、protein-DNA binding/interaction、zinc finger（补 MeSH）——**不收裸 `transcription factor`**，预印本引擎上裸 TF × AI 会把 enhancer/regulatory-element 类 ML 预后文拉进本腿。

两类腿都**不收** `gene regulation` / `enhancer` / `promoter` / cis-regulatory / regulatory element / 染色质可及性 / enhancer-promoter 等**调控基因组侧词**——它们会带入多组学+ML 预后、天然产物网络药理等噪声，且该侧已独立为 `3d-genome-ai`。

每周从 PubMed / arXiv / bioRxiv / medRxiv / chemRxiv 抓取最新元数据并提交回本仓库，同时开一条 Issue 汇总；本地用 Zotero 按 `_ids.txt` 批量导入人工筛选。

## 平台与量级

- **PubMed** — 主工作腿（TIAB + MeSH）。
- **bioRxiv** — 预印本主力（TF–DNA 计算预印本大多在此）。
- **arXiv** — 精确短语精查腿：周命中通常 0–5，无噪声；空属正常。
- **medRxiv / chemRxiv** — 本方向稀疏，常空，空属正常。

## 仓库结构

- `monitor.py` — 读取 `config.yaml`，逐平台检索，规范化后写入 `Discovery/`（合并 CSV + `_ids.txt`）与 `Archive/`（逐篇元数据 JSON），并生成 Issue 正文与标题。
- `config.yaml` — 检索配置：课题短名、时间窗口、每平台一条布尔检索式（对象 × 方法两段式）。PubMed 邮箱与 API key 通过环境变量注入，不写入文件。
- `.github/workflows/monitor.yml` — 每周一 09:23 UTC 自动运行，支持 `workflow_dispatch` 手动触发。

## 产出

```
Archive/                                # 逐篇完整元数据 JSON，只增不删，按年月归档
  {source}/{year}/{month}/{id}/{id}.json
Discovery/                              # 每次运行一份合并 CSV 与 _ids.txt，按抓取日归档
  {year}/{month}/{topic}_{date}.csv        # topic = tf-dna-ai（即 config.yaml 的 topic）
  {year}/{month}/{topic}_{date}_ids.txt
```

`source` 取值为 `pubmed`、`arxiv`、`biorxiv`、`medrxiv`、`chemrxiv`。

CSV 共 9 列：`source, id, doi, title, authors, journal, published_date, url, abstract`。`id` 为各平台主键（PubMed 为 PMID，预印本为 DOI），`doi` 为跨平台规范标识，`published_date` 统一为 ISO 日期 `YYYY-MM-DD`（PubMed 用 entrez date，预印本用 posting 日期，见下）。

`_ids.txt` 每行一个标识符，带类型前缀（`pmid:xxx`、`arXiv:xxx`，DOI 裸写），供 Zotero「按标识符添加」批量导入。

不做跨平台去重，也不判定是否已入库；重复与筛选由 Zotero 处理。当周无命中时，CSV 仅含表头。

## 日期口径（entrez date）

PubMed 的「发表日期」（DP）常残缺或滞后（ahead-of-print 无日期、只到年月、标引时滞），因此 PubMed 全线改用 **entrez date**（`[edat]`，即被 PubMed 收录的日期），搜索、归档、Issue 三处同源：

- **搜索**：查询窗口用 `[edat]` 过滤，抓取「本周新进 PubMed 的文献」，每篇只在收录周出现一次，无需重叠窗口与去重，`window_days: 7` 即可。
- **归档**：`Archive/pubmed/{year}/{month}/{id}/` 按 entrez date 归档，不因 DP 残缺落进 `unknown/`。
- **Issue**：日期列显示 entrez date（归一为 `YYYY-MM-DD`），并按它升序排序。

真实发表日期未丢弃——每篇完整元数据（含 DP）仍在 `Archive/*.json` 的 `data.source.pub_date` 中。预印本无标引时滞，仍用各自 posting 日期，不受影响。

## 密钥（PubMed）

PubMed 检索需要邮箱（必填）与 NCBI API key（可选），通过环境变量注入，不写入仓库：

- 本地：`export ENTREZ_EMAIL=you@example.com`，可选 `export NCBI_API_KEY=...`
- GitHub Actions：仓库 Settings → Secrets and variables → Actions，添加 `ENTREZ_EMAIL` 与 `NCBI_API_KEY` 两个 secret。

## 本地运行

```bash
pip install pyPaperFlow
export ENTREZ_EMAIL=you@example.com
python monitor.py --config config.yaml --out-dir . --issue-body /tmp/issue.md
```

调整时间窗口：`--window-days 1`，或修改 `config.yaml` 中的 `window_days`。检索口径细节见 `config.yaml` 内各平台注释。
