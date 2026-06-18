---
name: zotero-read-explainer
description: 对 Zotero 库中（或给定 PDF / DOI / arXiv 的）单篇论文做"决策型精读解析"：定位取数 → 渲染读图保公式 → 产出结构化中文精读报告（问题动机 / 方法与建模 / 仿真验证 / 主要公式一条不少 / 局限与工程价值评价）。当用户说"精读 / 深读 / 解析这篇论文""读懂这篇论文的方法和公式""给我一份精读 / 解析报告""评价 / 批判这篇论文""这篇 T1 帮我吃透"时，务必使用本 skill。它面向单篇高质量精读（公式保真、决策导向），区别于 zotero-analytical-writer 的 Obsidian 批量笔记，也区别于只做双语 / 中文通读、不下判断的 nature-reader。
---

# Zotero Read Explainer（单篇论文决策型精读）

把一篇论文"吃透"并落成一份可决策的中文精读报告。与本套件里其它 skill 的分工很关键，先判断要不要用我。

## 适用边界（先判断）

- ✅ **单篇要吃透**：讲清逻辑 / 方法 / 结论，**主要公式一条都不能少且要对**，并给出局限与对自身研究/工程的价值判断。
- ✅ 论文已在 Zotero（有 Item Key / 标题），或用户直接给 PDF / DOI / arXiv。
- ❌ 一次性**批量**过一整个 Zotero 分类、生成 Obsidian 笔记 → 用 `zotero-collection-manager` ＋ `zotero-analytical-writer`，不要用我。
- ❌ 只想"通读读懂外文全文"、暂时不需要下判断 → 先用 `nature-reader` 出通读稿，再回到本 skill 做精读判断。
- 默认**产出一份 Markdown 精读报告**（落到调用方指定的报告目录），不写 Obsidian、不依赖 Dataview。

## 1. 定位与取数

1. **定位条目**：论文已在 Zotero 时，用 Zotero MCP `zotero_search_items` → `zotero_get_item_metadata` / `zotero_get_item_children` 拿到元数据、附件 key、标签。
2. **拿到本地 PDF**：很多环境的 Zotero MCP **不是 local-file 模式**，`get_attachment_path` / `read_pdf_pages` 会失败（`file://` 报错）。兜底：读 Zotero 的 `prefs.js`，取 `extensions.zotero.dataDir` 作为数据目录，PDF 实体在 `<dataDir>/storage/<附件key>/*.pdf`。
3. **批注优先**：先取该条目的高亮 / 笔记（`zotero_get_annotations` / `zotero_get_item_children`）。**批注代表读者已经关注的重点，精读报告必须正面回应这些点**，否则等于没读到点子上。
4. **全文兜底**：`zotero_get_item_fulltext` 通常走本地 SQLite，可用来抽精确数字与原文措辞。

## 2. 读 PDF —— 公式保真是第一原则

**为什么**：精读不可替代的价值就在"公式不能少且要对"。纯文本抽取会丢符号、上下标、分式、约束 → 必须读**渲染后的版面**，不能只读文本流。

- **首选**：内置 Read 直接读 PDF（需 `poppler` / `pdftoppm` 在 PATH）。
- **兜底**（无 poppler 或渲染失败）：用 PyMuPDF 把每页渲染成 PNG 再读图：
  ```python
  import fitz, os, glob
  d = r"<dataDir>\storage\<附件key>"
  pdf = glob.glob(os.path.join(d, "*.pdf"))[0]
  out = os.path.join(os.environ["TEMP"], "read_<itemkey>"); os.makedirs(out, exist_ok=True)
  doc = fitz.open(pdf); m = fitz.Matrix(170/72.0, 170/72.0)   # ~170 dpi
  for i in range(doc.page_count):
      doc[i].get_pixmap(matrix=m).save(os.path.join(out, f"p{i+1:02d}.png"))
  ```
  再逐页 Read 这些 PNG。
- **数字交叉核对**：关键数值（百分比、参数表）另用 `pdfplumber` 抽纯文本核对，避免读图把 `13%` 看成 `1.3%`、把 `1539` 误读。
- **公式落 LaTeX**：行内 `$...$`、独立 `$$...$$`，保留下标 / 分式 / 约束条件。

## 3. 写精读报告

报告 9 段式结构见 `references/report-template.md`。要点：

- **公式宁多勿漏**：正文方法段内联关键式，末尾再汇成"关键公式清单"。
- **评价要平衡、决策导向**，不是为批判而批判：先摆出该领域的取舍优先级（例：车辆运动控制 **稳定性 > 驾驶性 > 经济性**），再据此判断论文的取舍是否合理；明确区分"**作者任务之外**（仿真论文不必苛求实车）"与"**我方落地必补**"。
- **术语 / 记号用表格**给出，并标**单位**。
- **关键结论附可跳转引用**：`zotero://open-pdf/library/items/<pdfKey>?page=<页码>`。
- **存疑要分级**：把"没读懂 / 影响结论"与"仅排版、系数级、不影响理解"分开写，不要把后者夸大成前者。

## 4. 终检

- 主要公式是否齐全、LaTeX 是否闭合（无未配对的 `$`）。
- 关键数值是否与原文文本核对过。
- 评价是否平衡（有取舍前提、有价值判断、不空泛苛责）。
- 是否回应了 Zotero 批注里读者已关注的点。

## 与其它 skill 的关系

- **上游可选**：`nature-reader`（通读稿）→ 本 skill（精读判断）。
- **取数可复用**：`zotero-data-fetcher` 的 `Raw_Data_Buffer` 可直接作本 skill 第 1 步的输入。
- **不要混用** `zotero-analytical-writer`：那条线产出 Obsidian **批量**笔记（公式靠 OCR、字段偏社科）；本 skill 产出**单篇决策型**精读报告（公式靠渲染读图、面向工程判断）。
