# Zotero Skills

> **把 Zotero 文献库接到 AI 助理：从搜索 / 入库 / 批注 / 管集合，到按分类批量精读、自动生成 Obsidian 中文精读笔记，一条流水线跑完。**

[简体中文](#简体中文) | [English](#english)

---

<a id="简体中文"></a>

## 简体中文

### 概览

把 AI 编程助理（Claude Code / Codex CLI / Gemini CLI / Cursor 等）接到 Zotero 文献库的一体化技能包。围绕"搜索 → 入库 → 分类批处理 → 抓数据 → 写精读"完整链路提供 6 个 skill，可独立调用，也可串联成精读工作流。

### Skills 一览

| Skill | 角色 | 核心能力 |
|---|---|---|
| `zotero-library-manager` | 文献库 CRUD 中枢 | 双 API 架构（本地 23119 读 + Web `api.zotero.org` 写）；自动健康检查与降级；全文搜索、标签筛选、集合浏览；新增期刊 / 书 / 会议 / 报告 / 学位论文 / 网页（含完整元数据）；编辑元数据 / 管理标签 / 批量移动；删除项目、笔记、集合 + 查重 |
| `zotero-collection-manager` | 分类批处理调度器 | 读取并维护 `_ProcessLog_进度记录.md`；自动跳过已完成 / 已跳过条目；按篇串行调度，处理完一篇立即写日志（断点续跑）；把抓取与写作下放给下游 skill |
| `zotero-data-fetcher` | 单篇论文语料采集 | 先读 Zotero 数据目录与数据库；优先批注 → 次取全文缓存 → 兜底本地 PDF；保持原始语言，本步骤不翻译不总结 |
| `zotero-analytical-writer` | 中文精读笔记生成 | Frontmatter 字段高度提炼（不机械抄摘要）；研究区 / 数据源 / 方法 / 核心变量精确提取；公式提取防乱码 + OCR 兜底；过滤作者单位、基金号、投稿规范等学术噪音；按 `templates/论文精读模板.md` 写入 Obsidian |
| `zotero-read-explainer` | 单篇决策型精读 | 定位取数（Zotero MCP，附件路径取不到时回落 `prefs.js` 的 `dataDir`）；渲染读图保公式（内置 Read + poppler 首选 / PyMuPDF 兜底 + pdfplumber 核数）；产出 9 段式中文精读报告（方法 / 主要公式一条不少 / 平衡且决策导向的评价 / 对自身工程价值）；面向**单篇**，输出 Markdown 报告，不写 Obsidian |
| `zotero-full-translater` | 全文翻译式通俗解析 | 正文**每部分每段含义完整讲清**（拿不准就直译）；**全部公式一条不删**；引言给"概述 + 参考文献现状表"；只留主要图表；附符号参数表（带单位）+ 术语表；通篇标题序列；文末列不确定项。比精读更饱满、比逐句对照更通俗 |

### 推荐工作流

```
zotero-collection-manager  ──┐
   （挑出该分类下未处理论文，串行调度）
                             ▼
                       zotero-data-fetcher
                       （抓单篇元数据/批注/全文）
                             ▼
                     zotero-analytical-writer
                     （生成中文精读笔记，写入 Obsidian）

zotero-library-manager  随时可用（搜索 / 新增 / 批注 / 集合管理）
zotero-read-explainer   单篇精读随时可用（可选先经 nature-reader 通读 → 输出 Markdown 精读报告）
zotero-full-translater  全文翻译式通俗解析随时可用（每段讲透 + 全公式 + 引言文献表）
```

### 前置要求

- **Python 3.10+**
- **pyzotero**：`pip install pyzotero`
- **Zotero 桌面端**（可选；启用后 `zotero-library-manager` 自动走本地 API 加速读取，`zotero-data-fetcher` 也依赖本地数据目录）

### API 凭证

1. **API Key** — 到 [https://www.zotero.org/settings/keys](https://www.zotero.org/settings/keys) 生成，勾选 `Allow library access` 和 `Allow write access`
2. **Library ID** — 同一页面 "Your userID for use in API calls" 处获取

### 配置

把 `config.example.json` 复制为 `config.json`，填入凭证：

```bash
cp config.example.json config.json
```

```json
{
  "zotero_api_key": "YOUR_API_KEY",
  "zotero_library_id": "YOUR_LIBRARY_ID",
  "zotero_library_type": "user",
  "collections": {
    "my_collection": "COLLECTION_KEY"
  }
}
```

> **注意**：`config.json` 已在 `.gitignore` 中，不会被推送。切勿提交含 API Key 的文件。

### 安装

通过 marketplace 安装：

```bash
claude plugin marketplace add Teller-Lu/zotero-skills
claude plugin install zotero-skills@zotero-skills --scope user
```

默认装到 user scope（当前 OS 用户全局可用、跨所有 project）。只想装在当前 project 加 `--scope project`。

> **从 `cp -r` / `git clone` 老方式迁移过来的？** 新版 `SKILL.md` 已经按 marketplace 规范放在 `skills/<name>/SKILL.md`，旧的 `cp -r zotero-skills/ ~/.claude/skills/zotero-skills/` 布局不会加载（Claude Code 只扫一层）。请先 `rm -rf ~/.claude/skills/zotero-skills` 再走上面的 marketplace 安装。

### 非 Claude CLI 适配

本套件以 Claude Code 为基准开发，其他 AI 助理同样适用：

| CLI | 加载方式 |
|---|---|
| Claude Code | 放入 `~/.claude/skills/` 或 `.claude/skills/`，自动加载 |
| Codex CLI | 用 `-C` 把 `SKILL.md` 作上下文文件传入，或写进任务 prompt |
| Gemini CLI | 把 `SKILL.md` 加进系统提示或项目上下文 |
| Cursor / Windsurf | 把 `SKILL.md` 加进 `.cursor/rules` 或对应规则文件 |
| 其他工具 | 把 `SKILL.md` 相关段落贴进系统提示 |

### 双 API 架构（`zotero-library-manager` 核心）

Zotero 暴露两套 API，本 skill 自动路由：

| | 本地 API（`localhost:23119`） | Web API（`api.zotero.org`） |
|---|---|---|
| 访问条件 | 需 Zotero 桌面端运行 | 任何环境 |
| 读 | ✅ 快、支持全文搜索 | ✅ 标准查询 |
| 写 | ❌ 不支持（返回 `501`） | ✅ 完整 CRUD |
| 速率 | 无限制 | ~50 请求 / 10 秒 |
| 鉴权 | `Zotero-Allowed-Request: true` 头 | `Zotero-API-Key: <key>` 头 |

**健康检查与自动降级**：`ZoteroDualClient` 初始化时调 `check_local_api()` 轻量 GET `localhost:23119`。

- Zotero 桌面端未跑 → 读自动降级到 Web API，无需改配置
- 性能略降（Web 延迟），所有操作仍工作

```python
from zotero_client import ZoteroDualClient
dual = ZoteroDualClient()
# dual.local_available → True 桌面端在线；False 否

results = dual.search("flood adaptation")              # 优先本地 API
dual.create_note("ITEM_KEY", "Section", "Notes...")    # 一律 Web API
```

**重要提醒**：

- 本地 API **不支持写** — 所有增 / 改 / 删走 Web API
- 用 Zotero MCP 连接器并设 `ZOTERO_LOCAL=true` 时，写工具会失败 → 改用 `zotero_client.py`

### 可用函数

来自 `scripts/zotero_client.py`：

| 函数 | 说明 |
|---|---|
| `get_client()` | 配置好的 `pyzotero.Zotero` Web API 实例 |
| `get_collection(name)` | 按显示名从 `config.json` 查集合 key |
| `add_note(zot, item_key, content)` | 给文献项目挂一条子笔记 |
| `check_duplicate(zot, title, doi)` | 按标题或 DOI 查重 |
| `check_local_api(timeout)` | 测桌面端本地 API 是否可达 |
| `ZoteroDualClient` | 双 API 包装器——本地读、Web 写、自动降级 |
| `safe_api_call(func)` | 自动处理速率限制回退的 API 调用包装 |

### 使用示例

**ZoteroDualClient（推荐）**

```python
import sys
sys.path.insert(0, r"~/.claude/skills/zotero-library-manager/scripts")
from zotero_client import ZoteroDualClient

dual = ZoteroDualClient()
print(f"本地 API 可用：{dual.local_available}")

results = dual.search("flood risk perception")
dual.create_note("I3P2J58S", "Section 2", "<p>讨论 PMT 框架。</p>")
```

**新增期刊论文**

```python
from pyzotero import zotero
zot = zotero.Zotero("YOUR_LIBRARY_ID", "user", "YOUR_API_KEY")

template = zot.item_template("journalArticle")
template["title"] = "Deep Learning for NLP: A Survey"
template["creators"] = [{"creatorType": "author", "firstName": "Jane", "lastName": "Doe"}]
template["publicationTitle"] = "Nature Machine Intelligence"
template["date"] = "2025"
template["DOI"] = "10.1038/s42256-025-00001-1"
zot.create_items([template])
```

**按标签搜索**

```python
items = zot.items(tag="machine-learning", limit=25)
for item in items:
    print(f"{item['data']['title']} — {item['data'].get('date', 'n.d.')}")
```

### 项目结构

```
zotero-skills/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── zotero-library-manager/    # 双 API CRUD 中枢
│   │   └── SKILL.md
│   ├── zotero-collection-manager/ # 分类批处理 + 断点续跑
│   │   └── SKILL.md
│   ├── zotero-data-fetcher/       # 单篇论文元数据/批注/全文抓取
│   │   └── SKILL.md
│   ├── zotero-analytical-writer/  # 中文精读笔记生成
│   │   └── SKILL.md
│   ├── zotero-read-explainer/     # 单篇决策型精读（渲染读图保公式 → Markdown 报告）
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── report-template.md
│   └── zotero-full-translater/    # 全文翻译式通俗解析（每段讲透 + 全公式 + 文献表）
│       ├── SKILL.md
│       └── references/
│           └── output-spec.md
├── templates/
│   └── 论文精读模板.md             # 精读模板（被 analytical-writer 引用）
├── scripts/
│   ├── zotero_client.py           # ZoteroDualClient + 工具函数
│   └── add_literature.py          # 批量导入脚本
├── config.example.json
├── config.json                    # 已 gitignore
└── README.md
```

### 环境说明

仓库默认按 Windows 路径习惯编写，保留了开发环境的默认目录（例如 `D:\ResearchVault\note`）。换机器或换仓库时，建议通过命令行参数覆盖这些默认值，避免硬编码依赖。

---

<a id="english"></a>

## English

### Overview

A unified skill bundle that connects AI coding assistants (Claude Code / Codex CLI / Gemini CLI / Cursor) to your Zotero library. Covers the full "search → ingest → batch classify → fetch → close-reading" pipeline through 6 skills — usable standalone or chained into a deep-reading workflow.

### Skills at a Glance

| Skill | Role | Capabilities |
|---|---|---|
| `zotero-library-manager` | Library CRUD hub | Dual-API (local `23119` reads + web `api.zotero.org` writes); auto health-check & fallback; full-text search, tag filtering, collection browsing; create journal article / book / conference paper / report / thesis / webpage with full metadata; edit metadata, manage tags, batch move; delete items, notes, collections + duplicate detection |
| `zotero-collection-manager` | Batch orchestrator | Reads/maintains `_ProcessLog_进度记录.md`; auto-skips completed/skipped entries; serially schedules one paper at a time and writes the log immediately after each (resumable); delegates fetching and writing to downstream skills |
| `zotero-data-fetcher` | Per-paper data ingest | Reads Zotero data directory + database first; prefers annotations → full-text cache → local PDF fallback; preserves original language (no translation/summarization at this stage) |
| `zotero-analytical-writer` | Chinese close-reading notes | Highly distilled frontmatter (no mechanical abstract copy); precise extraction of study area / data source / method / core variables; formula extraction with anti-garble + OCR fallback; filters out author affiliations, grant IDs, submission boilerplate; writes Obsidian notes via `templates/论文精读模板.md` |
| `zotero-read-explainer` | Single-paper decision-grade deep read | Locate & fetch (Zotero MCP; falls back to `prefs.js` `dataDir` when attachment paths are unavailable); formula-faithful page reading (built-in Read + poppler preferred / PyMuPDF fallback + pdfplumber number-checking); produces a 9-section Chinese deep-read report (method / no-formula-left-behind / balanced decision-oriented critique / value to your own engineering); single-paper focused, outputs Markdown, not Obsidian |
| `zotero-full-translater` | Full-text plain-language translation+explanation | Covers **every section & paragraph in full** (translate verbatim when unsure); **keeps every formula, deletes none**; intro becomes "overview + cited-literature status table"; only main figures/tables; symbol/parameter table (with units) + glossary; full heading structure; trailing uncertainty list. Fuller than a deep-read report, more readable than line-by-line bilingual |

### Recommended Workflow

```
zotero-collection-manager  ──┐
   (picks unprocessed papers, schedules serially)
                             ▼
                       zotero-data-fetcher
                       (fetch single-paper metadata/annotations/fulltext)
                             ▼
                     zotero-analytical-writer
                     (generate Chinese close-reading note → Obsidian)

zotero-library-manager  available anytime (search / add / annotate / collections)
zotero-read-explainer   single-paper deep read, anytime (optionally after nature-reader → outputs a Markdown report)
zotero-full-translater  full-text plain-language translation+explanation, anytime (every paragraph + every formula + cited-lit table)
```

### Prerequisites

- **Python 3.10+**
- **pyzotero**: `pip install pyzotero`
- **Zotero desktop app** (optional; when running, `zotero-library-manager` reads via fast local API and `zotero-data-fetcher` relies on the local data directory)

### API Credentials

1. **API Key** — Generate at [https://www.zotero.org/settings/keys](https://www.zotero.org/settings/keys). Enable `Allow library access` and `Allow write access`.
2. **Library ID** — Same page, under "Your userID for use in API calls".

### Configuration

Copy `config.example.json` to `config.json` and fill in credentials:

```bash
cp config.example.json config.json
```

```json
{
  "zotero_api_key": "YOUR_API_KEY",
  "zotero_library_id": "YOUR_LIBRARY_ID",
  "zotero_library_type": "user",
  "collections": {
    "my_collection": "COLLECTION_KEY"
  }
}
```

> **Note**: `config.json` is gitignored. Never commit credentials.

### Installation

Via marketplace:

```bash
claude plugin marketplace add Teller-Lu/zotero-skills
claude plugin install zotero-skills@zotero-skills --scope user
```

Defaults to user scope (OS-user global, across all projects). Use `--scope project` to install only in the current project.

> **Migrating from a previous `cp -r` / `git clone` install?** `SKILL.md` now lives at `skills/<name>/SKILL.md` for marketplace auto-discovery. The old `cp -r zotero-skills/ ~/.claude/skills/zotero-skills/` layout no longer loads (Claude Code scans only one level deep). Remove the old copy (`rm -rf ~/.claude/skills/zotero-skills`) and use the marketplace install above.

### Non-Claude CLI Adaptation

Developed for Claude Code, but works with any AI assistant:

| CLI | How to load |
|---|---|
| Claude Code | Place in `~/.claude/skills/` or `.claude/skills/` — auto-loaded |
| Codex CLI | Pass `SKILL.md` as context via `-C`, or include in task prompt |
| Gemini CLI | Include `SKILL.md` in system prompt or project context |
| Cursor / Windsurf | Add `SKILL.md` to `.cursor/rules` or equivalent rules file |
| Any other tool | Paste relevant `SKILL.md` sections into your system prompt |

### Dual-API Architecture (core of `zotero-library-manager`)

Zotero exposes two APIs with different capabilities; this skill routes automatically.

| | Local API (`localhost:23119`) | Web API (`api.zotero.org`) |
|---|---|---|
| Access | Requires Zotero desktop running | Anywhere |
| Read | ✅ Fast, full-text search | ✅ Standard queries |
| Write | ❌ Not supported (`501`) | ✅ Full CRUD |
| Rate limit | None | ~50 req / 10 sec |
| Auth | `Zotero-Allowed-Request: true` header | `Zotero-API-Key: <key>` header |

**Health check & auto-fallback**: On init, `ZoteroDualClient` calls `check_local_api()` — a lightweight GET to `localhost:23119`.

- Zotero desktop not running → reads fall back to Web API automatically, no config change
- Performance degrades slightly (web latency), all operations still work

```python
from zotero_client import ZoteroDualClient
dual = ZoteroDualClient()
# dual.local_available → True if desktop running

results = dual.search("flood adaptation")              # local API if available
dual.create_note("ITEM_KEY", "Section", "Notes...")    # always web API
```

**Important notes**:

- Local API **does not support writes** — all create/update/delete go through Web API
- If you use a Zotero MCP connector with `ZOTERO_LOCAL=true`, its write tools will fail — use `zotero_client.py` for writes

### Available Functions

From `scripts/zotero_client.py`:

| Function | Description |
|---|---|
| `get_client()` | Configured `pyzotero.Zotero` Web API instance |
| `get_collection(name)` | Find collection key by display name from `config.json` |
| `add_note(zot, item_key, content)` | Attach a child note to a library item |
| `check_duplicate(zot, title, doi)` | Check if item with given title or DOI exists |
| `check_local_api(timeout)` | Test if Zotero desktop local API is reachable |
| `ZoteroDualClient` | Dual-API wrapper — local reads, web writes, auto-fallback |
| `safe_api_call(func)` | API call wrapper with automatic rate-limit backoff |

### Usage Examples

**ZoteroDualClient (recommended)**

```python
import sys
sys.path.insert(0, r"~/.claude/skills/zotero-library-manager/scripts")
from zotero_client import ZoteroDualClient

dual = ZoteroDualClient()
print(f"Local API available: {dual.local_available}")

results = dual.search("flood risk perception")
dual.create_note("I3P2J58S", "Section 2", "<p>Discusses PMT framework.</p>")
```

**Create a journal article**

```python
from pyzotero import zotero
zot = zotero.Zotero("YOUR_LIBRARY_ID", "user", "YOUR_API_KEY")

template = zot.item_template("journalArticle")
template["title"] = "Deep Learning for NLP: A Survey"
template["creators"] = [{"creatorType": "author", "firstName": "Jane", "lastName": "Doe"}]
template["publicationTitle"] = "Nature Machine Intelligence"
template["date"] = "2025"
template["DOI"] = "10.1038/s42256-025-00001-1"
zot.create_items([template])
```

**Search by tag**

```python
items = zot.items(tag="machine-learning", limit=25)
for item in items:
    print(f"{item['data']['title']} — {item['data'].get('date', 'n.d.')}")
```

### Project Structure

```
zotero-skills/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── zotero-library-manager/    # Dual-API CRUD hub
│   │   └── SKILL.md
│   ├── zotero-collection-manager/ # Batch classification + resume
│   │   └── SKILL.md
│   ├── zotero-data-fetcher/       # Single-paper metadata/annotations/fulltext
│   │   └── SKILL.md
│   ├── zotero-analytical-writer/  # Chinese close-reading note generator
│   │   └── SKILL.md
│   ├── zotero-read-explainer/     # Single-paper deep read (formula-faithful page reading → Markdown report)
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── report-template.md
│   └── zotero-full-translater/    # Full-text plain-language translation+explanation (every paragraph + every formula)
│       ├── SKILL.md
│       └── references/
│           └── output-spec.md
├── templates/
│   └── 论文精读模板.md             # Close-reading template (referenced by analytical-writer)
├── scripts/
│   ├── zotero_client.py           # ZoteroDualClient + helpers
│   └── add_literature.py          # Batch import script
├── config.example.json
├── config.json                    # gitignored
└── README.md
```

### Environment Notes

The repo defaults to Windows path conventions and keeps the development environment's defaults (e.g. `D:\ResearchVault\note`). When using on a different machine or repo, override these via CLI args rather than relying on hard-coded defaults.

---

## License

MIT

## Credits

- `zotero-library-manager` derives from [WenyuChiou's Zotero skill](https://github.com/WenyuChiou/ai-research-skills) (dual-API architecture).
- `zotero-collection-manager` / `zotero-data-fetcher` / `zotero-analytical-writer` / `zotero-read-explainer` / `zotero-full-translater` and the close-reading templates are by Teller-Lu (cheneternity).
