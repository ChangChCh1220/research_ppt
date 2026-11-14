# AGENTS.md — LLM 研究知識庫（LLM Wiki）維護規範

本文件是此知識庫的 **Schema（運作憲章）**，參考 Andrej Karpathy 的 *LLM Wiki* 模式，專為 **大型語言模型（LLM）與生成式 AI 研究** 設計。
在本專案中：**Obsidian（或編輯器）是 IDE，LLM Agent 是知識庫的維護者與編譯器，Wiki 是持續累積複利的程式碼庫。**

> 核心原則：**知識只編譯一次並持續維護，而非每次提問都從零重新檢索（RAG）。**
> 人類負責策展資料、提出好問題與判斷方向；LLM 負責閱讀、摘要、交叉引用、發現矛盾、維護一致性與簿記工作。

---

## 一、三層架構與目錄規範

```text
LLM_WIKI/
├── AGENTS.md              # [Schema] 本規範文件（隨著研究需求與使用者共同演進）
├── index.md               # [導航] 內容導向的全域目錄（查詢時優先讀取）
├── log.md                 # [日誌] 時間序的只增不減操作紀錄（Append-only）
│
├── raw/                   # [層 1：原始資料層] 不可變（Immutable）的真實來源
│   ├── papers/            #   論文檔案（PDF、arXiv Markdown、論文筆記原稿）
│   ├── articles/          #   技術部落格、官方文件、Web Clipper 擷取文章
│   ├── notes/             #   個人靈感、課堂/會議逐字稿、實驗紀錄原稿
│   └── assets/            #   本地下載的圖片與附件（供離線與視覺檢視）
│
└── wiki/                  # [層 2：Wiki 知識層] 由 LLM 全權建立與維護
    ├── overview.md        #   LLM 研究總覽、核心脈絡與演化地圖（Synthesis）
    ├── sources/           #   單篇來源的結構化摘要頁
    ├── concepts/          #   核心技術、演算法、方法論與理論概念頁
    ├── entities/          #   模型家族、研究機構、開源框架、評測基準（Benchmarks）
    └── comparisons/       #   橫向比較、技術路線對比與 Query 沉澱的專題分析

└── outputs/               # [層 3：應用輸出層] 依 Wiki 生成的研究筆記與簡報（不反向污染 raw/wiki）
    ├── notes/             #   嚴格僅依 wiki/ 撰寫的研究筆記（文末附可點擊下載之建議文獻）
    ├── slide-scripts/     #   依據筆記轉換的簡報逐頁腳本（Markdown）
    └── slides/            #   單檔 HTML 簡報（支援 LaTeX / KaTeX）
```

### 各層權責邊界
1. **`raw/`（只讀不改）**：這是唯一真實來源（Source of Truth）。LLM **絕對不可修改、刪除或覆寫** `raw/` 中的原始文件（若使用者在對話中直接貼入新論文或資料，LLM 先將其保存為 `raw/` 新檔案，該檔案寫入後即視為不可變）。
2. **`wiki/`（LLM 全權維護）**：所有摘要、概念頁、實體頁、比較分析與連結皆由 LLM 建立與更新，保持結構一致與交叉連結完整。
3. **`outputs/`（衍生應用輸出）**：存放依 `wiki/` 生成的研究筆記、逐頁簡報腳本與 HTML 簡報；僅在經使用者確認「回寫」時，才將洞見整理進 `wiki/comparisons/`。
4. **`AGENTS.md`（共同演進）**：當工作流程、分類或格式需要調整時，經與使用者確認後更新本文件。

---

## 二、格式與命名規範

### 1. 檔名與連結規則
- **檔名格式**：使用清晰、無歧義的英文或中英混合小寫短橫線命名（`kebab-case`），或一致的標準名稱（例如 `deepseek-r1.md`、`kv-cache.md`、`2024-rag-survey.md`），避免空白字元與特殊符號。
- **內部連結**：統一使用 Obsidian 相容的雙括號 Wikilinks，例如 `[[kv-cache]]` 或 `[[deepseek-r1|DeepSeek-R1]]`，以便 Obsidian Graph View 正確呈現知識網絡。
- **引用出處**：Wiki 頁面中的關鍵數據、主張或結論，必須以 Wikilink 標註來源頁面（例如：`見 [[2025-deepseek-r1-paper]]`）。

### 2. YAML Frontmatter（支援 Obsidian Dataview）
每個 `wiki/` 下的 Markdown 頁面都應包含標準 YAML frontmatter：

```yaml
---
title: 頁面標題
type: concept # 可選值: overview | source | concept | entity | comparison
tags:
  - llm-research
  - inference # 建議依研究領域標記（見下方標籤建議）
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources_count: 1 # 此頁面綜合了幾篇 raw 來源
status: active # 可選值: seed（草稿）| active（持續維護）| superseded（已被取代）
---
```

**建議領域標籤（Tags）**：
`architecture`（架構）、`pretraining`（預訓練）、`post-training`（微調/RLHF/DPO）、`reasoning`（推理/CoT/Test-time Compute）、`inference`（推論加速/量化/KV Cache）、`rag`（檢索增強）、`agents`（智慧體/工具調用）、`evaluation`（評測與基準）、`multimodal`（多模態）、`safety`（安全與對齊）、`autonomous-driving`（自駕感知/點雲）。

### 3. LaTeX 數學公式規範
- 全知識庫（`wiki/`、`outputs/notes/` 與 HTML 簡報）皆須支援標準 LaTeX 數學語法：行內公式使用 `$...$`，獨立區塊公式使用 `$$...$$`。

---

## 三、核心頁面範本

### A. 來源摘要頁（`wiki/sources/`）
- **書目與基本資訊**：作者 / 機構、發表年份、連結或 arXiv ID、原始檔案路徑（`raw/...`）
- **一句話總結（TL;DR）**
- **核心問題與動機**：想解決什麼痛點？
- **關鍵方法與技術細節**：核心架構、公式或演算法機制
- **主要實驗結果與限制**：關鍵數據、對比基線（Baselines）、作者承認或觀察到的限制
- **知識庫關聯與衝突**：與現有 `[[concepts]]` / `[[entities]]` 的連結；是否強化、補充或**推翻/矛盾**先前的觀點？

### B. 概念頁（`wiki/concepts/`）與實體頁（`wiki/entities/`）
- **定義與定位**：這個技術/模型/基準是什麼？解決什麼問題？
- **核心機制 / 演進脈絡**：依時間或技術演進整理（不要只堆砌事實，要寫出知識的演變）
- **代表性工作與實作**：關聯的模型、論文或工具（附上 Wikilinks）
- **優缺點與權衡（Trade-offs）**
- **矛盾與開放問題**：不同來源間存在分歧的觀點，或尚未解決的研究問題
- **參考來源**：條列所有貢獻到此頁面的 `wiki/sources/` 頁面

### C. 比較與專題分析頁（`wiki/comparisons/`）
- 用於橫向比較多種技術（如 `LoRA vs. QLoRA vs. Full Fine-tuning`）或從 Query 沉澱出的研究綜合報告。
- 建議包含：**比較維度總表（Markdown Table）**、**適用場景建議**、**關鍵差異分析**與**來源引用**。

---

## 四、三大核心操作流程（Operations）

### 1. Ingest（攝入新資料）
當使用者將新檔案放入 `raw/`，或直接在對話中貼上論文/資料並要求消化處理時，依以下步驟執行（預設採**互動式單篇攝入**；若使用者明確要求批次處理，則改為批次執行後彙總報告）：

1. **保存與讀取原始資料**：若為對話中直接貼上的來源，先依類別保存至 `raw/` 對應子目錄（如 `raw/papers/` 或 `raw/articles/`）；再完整閱讀 `raw/` 中的目標文件。
   - *圖片處理原則*：若 Markdown 內含本地圖片（如 `raw/assets/...`），先讀取全文掌握脈絡，再視需要單獨檢視關鍵圖片以補充圖表/架構圖資訊。
2. **提煉與對齊（互動模式）**：向使用者簡述核心收穫，確認重點方向或特別想追蹤的研究問題（若為明確的批次指令則可跳過確認直接編譯）。
3. **建立來源摘要頁**：在 `wiki/sources/` 建立對應的結構化摘要頁。
4. **編譯並更新知識網絡**：
   - 更新或新建相關的 `wiki/concepts/`、`wiki/entities/`、`wiki/comparisons/` 頁面。
   - 主動檢查新資料是否與既有 Wiki 頁面內容**矛盾或更新了舊結論**；若有，必須明確標註衝突來源與時間點（例如使用 `> [!WARNING] 觀點演進 / 衝突`）。
   - 若影響整體研究全貌，同步更新 `wiki/overview.md`。
   - 一份重要來源通常會觸及 3～15 個相關頁面，請務必維持雙向連結一致。
5. **更新 `index.md`**：新增或更新受影響頁面的條目、一行摘要與 `sources_count` / 更新日期。
6. **追加 `log.md`**：在 `log.md` 末尾加上一筆標準格式紀錄。

---

### 2. Query（查詢與知識沉澱）
當使用者針對 LLM 研究提問、要求比較或撰寫分析時：

1. **先讀索引**：優先讀取 `index.md`（與必要時的 `wiki/overview.md`）定位相關頁面，再深入讀取對應的 `wiki/` 頁面；只有在 Wiki 資訊不足或需核對精確細節時，才回頭查閱 `raw/`。
2. **附帶引用的回答**：整合跨頁面知識給出深入回答，所有關鍵論點皆以 `[[頁面名稱]]` 標明出處。若既有資料存在矛盾或資訊缺口，需誠實指出。
3. **支援多元輸出**：依使用者需求，可產出 Markdown 分析報告、比較表、Marp 簡報或資料視覺化圖表。
4. **知識回寫（File Back）**：
   - 好的回答不應該消失在對話紀錄中。若該次查詢產出了有價值的跨來源比較、新發現的關聯或研究綜述，主動詢問使用者是否將其沉澱為 `wiki/comparisons/`（或新的概念頁）。
   - 經確認寫入 `wiki/` 後，同步更新 `index.md` 並在 `log.md` 追加一筆 `query` 紀錄。

---

### 3. Lint（健康檢查與知識庫維護）
當使用者要求進行健康檢查（Lint）或定期維護時，全面檢視 `wiki/` 與 `index.md`：

1. **矛盾與過時檢查**：找出跨頁面間尚未標註的衝突說法，或已被較新論文推翻的舊宣稱（標記 `status: superseded` 或加註警告）。
2. **圖譜結構檢查**：
   - **孤島頁面（Orphans）**：沒有任何入站連結（Inbound links）的頁面。
   - **缺漏頁面（Missing Concepts）**：在多篇筆記中被 `[[連結]]` 或反覆提及、卻尚未建立獨立頁面的重要概念或實體。
   - **斷鏈與索引同步**：檢查 `index.md` 是否遺漏頁面，或列出了不存在的檔案。
3. **研究缺口建議**：指出目前知識庫中論據薄弱的地方，並提出具體的後續閱讀建議或可進行網頁搜尋補充的方向。
4. **紀錄結果**：修復核可的問題後，在 `log.md` 追加一筆 `lint` 紀錄。

---

### 4. Research-to-Slides（研究筆記與 HTML 簡報生成）
當使用者提出研究主題或問題，要求產出研究筆記並製作成簡報時，依以下三階段執行（同一主題共用 `YYYY-MM-DD-<topic-slug>` 檔名前綴）：

1. **產出研究筆記（`outputs/notes/`）**：
   - **嚴格僅用 Wiki 內容**：筆記正文與核心論述 **100% 僅能依據 `wiki/` 既有頁面撰寫**，並以 `[[頁面名稱]]` 標註來源；Wiki 尚未涵蓋之缺口須誠實標註 `[Wiki 尚無此部分資料]`，**絕對不可將外部搜尋內容寫入正文**。
   - **上網搜尋與可下載建議文獻**：主動上網搜尋該主題之相關論文與技術文章，統一列於筆記最末節「建議參考文獻（待 Wiki 化）」。每筆文獻須標明「推薦理由（可補齊哪個缺口）」，並**優先附上可直接點擊下載的 PDF 連結**（如 arXiv PDF 直連網址 `https://arxiv.org/pdf/<id>.pdf` 或官方 PDF 連結），以便使用者下載並匯入 `raw/` 進行 Wiki 化。
   - 支援標準 LaTeX 公式（`$...$` / `$$...$$`）。
2. **產出簡報逐頁腳本（`outputs/slide-scripts/`）**：
   - 嚴格依據該份 `outputs/notes/` 內容拆分為逐頁投影片大綱（含頁標題、精簡條列重點、圖表/公式視覺設計與講者口白備註），不可額外引入筆記以外的未入庫數據。
3. **產出 HTML 簡報（`outputs/slides/`）**：
   - 依逐頁腳本產出單檔自包含 `.html` 簡報，預設採用 **冰川極簡藍白（Glacial Tech Minimal）** 風格：
     - **視覺配色**：純白畫布（`#FFFFFF`）搭配極淡點陣底紋、冰藍卡片（`#F4F8FF` / `#EFF6FF`）、深海藍主文字（`#0F172A`）、科技藍強調色（`#2563EB`）與 `1px` 淺藍灰細邊框，搭配等寬字體數據標籤。
     - **版面與功能**：16:9 簡報畫布、內建 **KaTeX** 支援 `$...$` 與 `$$...$$` LaTeX 公式渲染、鍵盤 `←/→/Space` 翻頁、進度條與頁碼、`S` 鍵切換講者備註面板、`F` 全螢幕、以及 `@media print` 友善列印匯出 PDF 樣式。
4. **日誌紀錄**：產出完成後於 `log.md` 追加一筆 `query` 紀錄。

---

## 五、索引與日誌格式規範

### `index.md`（內容導向）
- 依分類維護：`總覽`、`概念 (Concepts)`、`實體 (Entities)`、`比較與分析 (Comparisons)`、`來源摘要 (Sources)`。
- 每筆條目格式建議：
  `- [[folder/filename|顯示名稱]] — 一行簡明摘要。（更新：YYYY-MM-DD｜來源數：N）`

### `log.md`（時間序，只增不減）
- **只可在檔案末尾追加（Append-only）**，不可修改或刪除歷史紀錄。
- 每筆紀錄的標題**必須**遵守以下嚴格前綴格式，以利指令工具解析（例如檢索最近 5 筆操作）：
  ```markdown
  ## [YYYY-MM-DD] <operation> | <簡短標題>
  - **操作類型**：ingest / query / lint / init
  - **涉及檔案**：`wiki/...`
  - **摘要**：簡述本次新增、更新或發現的重點。
  ```
  其中 `<operation>` 統一使用小寫：`init`、`ingest`、`query`、`lint`。

---

## 六、工作環境與工具搭配建議（Obsidian）

- **Obsidian Web Clipper**：用於快速將網頁技術文章轉為 Markdown 存入 `raw/articles/`。
- **本地圖片下載**：建議在 Obsidian 設定 `Files and links -> Attachment folder path` 為 `raw/assets/`，並為「Download attachments for current file」綁定快捷鍵（如 `Ctrl+Shift+D`），確保文章圖片能保存在本地供離線閱讀與檢視。
- **Graph View & Dataview**：利用標準 `[[wikilinks]]` 與 YAML frontmatter，可在 Obsidian 中視覺化研究知識網絡與建立動態研究清單。
- **Git 版本控制**：整個 Wiki 即為純文字與 Markdown 倉庫，每次大規模 Ingest 或 Lint 後建議提交一次 Git commit。
