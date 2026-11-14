# 操作日誌（Log）

> 本檔案為**只增不減（Append-only）**的時間序紀錄，用於追蹤知識庫的演進。
> 所有紀錄標題皆須符合 `## [YYYY-MM-DD] <operation> | <標題>` 格式，以便快速檢索（如 `Select-String "^## \[" log.md | Select-Object -Last 5` 或 `grep "^## \[" log.md | tail -5`）。

## [2026-11-12] init | 初始化 LLM 研究知識庫結構
- **操作類型**：init
- **涉及檔案**：`AGENTS.md`, `index.md`, `log.md`, `wiki/overview.md`, `raw/README.md`
- **摘要**：依據 Karpathy LLM Wiki 三層架構建立中文 Schema（`AGENTS.md`）、索引、日誌、`raw/` 原始資料區與 `wiki/` 知識層目錄骨架。

## [2026-11-12] ingest | CORENet: Cross-Modal 4D Radar Denoising Network (arXiv:2508.13485v1)
- **操作類型**：ingest
- **涉及檔案**：`raw/papers/2025-corenet-cross-modal-4d-radar-denoising.md`, `wiki/sources/2025-corenet.md`, `wiki/concepts/4d-radar-denoising.md`, `wiki/concepts/cross-modal-supervision.md`, `wiki/entities/corenet.md`, `wiki/entities/dual-radar-dataset.md`, `wiki/overview.md`, `index.md`
- **摘要**：將使用者貼上的 CORENet 論文保存至 `raw/papers/`，建立來源摘要頁與 4 個關聯概念/實體頁（4D 雷達去噪、跨模態監督、CORENet、Dual-Radar），並同步更新知識庫總覽與索引。

## [2026-11-12] ingest | 批次攝入 10 篇 3D/4D 感知、損失函數與優化器核心論文
- **操作類型**：ingest
- **涉及檔案**：`wiki/sources/2024-radardistill.md`, `wiki/sources/2025-l4dr.md`, `wiki/sources/2022-radar-occupancy-prediction.md`, `wiki/sources/2022-k-radar.md`, `wiki/sources/2023-enhanced-k-radar.md`, `wiki/sources/2017-pointnet-plus-plus.md`, `wiki/sources/2017-focal-loss.md`, `wiki/sources/2020-generalized-focal-loss.md`, `wiki/sources/2019-class-balanced-loss.md`, `wiki/sources/2019-adamw.md`, `wiki/concepts/focal-loss.md`, `wiki/concepts/generalized-focal-loss.md`, `wiki/concepts/class-balanced-loss.md`, `wiki/concepts/adamw.md`, `wiki/concepts/pointnet-plus-plus.md`, `wiki/concepts/4d-radar-tensor.md`, `wiki/concepts/lidar-radar-fusion.md`, `wiki/concepts/4d-radar-denoising.md`, `wiki/concepts/cross-modal-supervision.md`, `wiki/entities/k-radar-dataset.md`, `wiki/entities/vod-dataset.md`, `wiki/entities/radardistill.md`, `wiki/entities/l4dr.md`, `wiki/entities/retinanet.md`, `wiki/comparisons/cross-modal-supervision-vs-fusion.md`, `wiki/comparisons/imbalanced-and-focal-loss-evolution.md`, `wiki/comparisons/radar-representations-tensor-vs-pointcloud.md`, `wiki/overview.md`, `index.md`
- **摘要**：完成 10 篇新論文的完整 PDF 深度解析與結構化建檔，將知識庫擴充為 11 篇來源、9 大核心概念、7 大實體與 3 篇橫向專題比較（跨模態監督 vs. 推論期融合、損失函數演進史、4DRT 張量 vs. 點雲表徵），全面更新總覽與索引目錄。

## [2026-11-13] query | 光達輔助弱監督之四維雷達可靠度估計與軟式去噪研究與簡報
- **操作類型**：query
- **涉及檔案**：`outputs/notes/2026-11-13-radar-reliability-power-suppression.md`, `outputs/slide-scripts/2026-11-13-radar-reliability-power-suppression.md`, `outputs/slides/2026-11-13-radar-reliability-power-suppression.html`
- **摘要**：針對「光達輔助弱監督之四維雷達可靠度估計與功率抑制（軟式去噪）」主題，嚴格依據 Wiki 既有內容產出研究筆記（誠實標記缺口並附 4 篇可點擊下載之 arXiv 建議文獻）、逐頁簡報腳本，以及冰川極簡藍白風格之單檔 HTML 簡報（支援 KaTeX、講者備忘錄與列印 PDF）。

## [2026-11-13] query | K-Radar 資料集全天候感知與 4DRT 基準研究與簡報
- **操作類型**：query
- **涉及檔案**：`outputs/notes/2026-11-13-k-radar-dataset.md`, `outputs/slide-scripts/2026-11-13-k-radar-dataset.md`, `outputs/slides/2026-11-13-k-radar-dataset.html`
- **摘要**：針對「K-Radar 4D 雷達張量全天候自駕基準資料集」主題，嚴格依據 Wiki 既有內容產出研究筆記（涵蓋感測器規格、4DRT 管線、RTNH 稀疏卷積、高度價值實證、惡劣天氣倒掛與 5% 最佳密度，誠實標記缺口並附 3 篇可點擊下載之 arXiv 建議文獻）、逐頁簡報腳本，以及冰川極簡藍白風格之單檔 HTML 簡報（支援 KaTeX、講者備忘錄與列印 PDF）。

## [2026-11-13] init | 建立專案根目錄簡報導覽門戶 (index.html)
- **操作類型**：init
- **涉及檔案**：`index.html`
- **摘要**：在專案根目錄設計同為「冰川極簡藍白（Glacial Tech Minimal）」風格的導覽首頁 `index.html`，具備統計看板、即時關鍵字搜尋與分類過濾，可一鍵直達每份 HTML 簡報、研究筆記與講者腳本。

## [2026-11-13] lint | 修復 K-Radar 簡報 HTML JavaScript 語法錯誤
- **操作類型**：lint
- **涉及檔案**：`outputs/slides/2026-11-13-k-radar-dataset.html`
- **摘要**：修復投影片 JavaScript 模板字串中因未跳脫反引號（`3DRT-XYZ` 改為 `<code>3DRT-XYZ</code>`）導致的 `SyntaxError: Invalid or unexpected token` 語法錯誤，經 Node DOM mock 驗證 10 頁投影片全數正確載入與渲染。

## [2026-11-14] query | 碩士論文研究進度、實驗診斷與未來工作全景簡報 (24 頁完整旗艦 Deck)
- **操作類型**：query
- **涉及檔案**：`raw/notes/2026-master-thesis-draft-chang.pdf`, `outputs/notes/2026-11-14-thesis-progress-and-future-work.md`, `outputs/slide-scripts/2026-11-14-thesis-progress-and-future-work.md`, `outputs/slides/2026-11-14-thesis-progress-and-future-work.html`, `index.html`
- **摘要**：依據張喬筑碩士論文初稿（交大土木莊子毅教授指導）全篇 40 頁完整內容，歸納 7 大核心模組、製作共 24 頁自包含 HTML 旗艦簡報。完整涵蓋軟式去噪物理本質、三態弱監督標籤（核心區/保護區/距離衰減）、局部 MLP 原生特徵網路、115 組三種子開發結果（移除都卜勒 68.46% AP）、破除俯視 1,759 個負代理的 3D 側視核對、21.9:1 樣本失衡與大車退步 4 大競爭假說、Version08 平台式映射與表 4-6 定稿最小證據鏈；並同步建立研究筆記、逐頁講者備忘腳本，更新根目錄導覽門戶。

## [2026-11-14] query | 碩士論文分章核心概念深度專題簡報矩陣 (5 份獨立 HTML 簡報)
- **操作類型**：query
- **涉及檔案**：`outputs/slides/2026-11-14-weak-supervision-teacher.html`, `outputs/slides/2026-11-14-radar-features-and-network.html`, `outputs/slides/2026-11-14-power-suppression-and-selection.html`, `outputs/slides/2026-11-14-large-vehicle-diagnostics.html`, `outputs/slides/2026-11-14-thesis-limitations-and-evidence-chain.html`, `outputs/notes/...`, `outputs/slide-scripts/...`, `index.html`
- **摘要**：響應使用者「將每個概念分開寫成多份簡報」之要求，將碩士論文初稿拆解為 5 大專題簡報（每份 8 頁獨立 HTML 簡報 + 研究筆記 + 講者腳本）：1. 物件感知弱監督教師與幾何標籤；2. 雷達原生特徵與局部可靠度網路；3. 功率抑制映射機制與約束選模；4. 大型車遠距衰退、失敗診斷與四大假說；5. 研究限制、演算法改良與定稿最小證據鏈。全數通過 Node.js 語法驗證並部署至首頁導覽門戶。


## [2026-11-14] ingest | 下載並編譯 5 篇長尾損失、空間幾何與 4D 雷達基準前沿文獻
- **操作類型**：ingest
- **涉及檔案**：`raw/papers/2020-equalization-loss.pdf`, `raw/papers/2021-equalization-loss-v2.pdf`, `raw/papers/2020-distance-iou-loss.pdf`, `raw/papers/2022-voxel-4d-radar-detection.pdf`, `raw/papers/2022-view-of-delft-4d-radar.pdf`, `wiki/sources/2020-equalization-loss.md`, `wiki/sources/2021-equalization-loss-v2.md`, `wiki/sources/2020-distance-iou-loss.md`, `wiki/sources/2022-voxel-4d-radar-detection.md`, `wiki/sources/2022-view-of-delft-4d-radar.md`, `index.md`
- **摘要**：針對碩士論文目前 Version06 四大核心瓶頸（長尾壓制、3D 高度回歸脫節、體素特徵斷裂、都卜勒偽動態干擾），完整下載 5 篇經典頂會與頂刊論文至 `raw/papers/`，建立 5 份標準結構化 Wiki 摘要並更新全域索引：1. Equalization Loss (EQL, CVPR 2020)；2. Equalization Loss v2 (EQL v2, CVPR 2021)；3. Distance-IoU / Complete-IoU Loss (AAAI 2020)；4. Voxel-based 4D Radar Detection (IEEE TVT 2023)；5. View-of-Delft 4D Radar Benchmark (IEEE RA-L 2022)。

## [2026-11-14] query | 演算法深入改良指引與應該補充的實驗——兩份全新專題簡報與全庫文獻深化
- **操作類型**：query
- **涉及檔案**：`outputs/notes/2026-11-14-algorithmic-enhancements-guide.md`, `outputs/slide-scripts/2026-11-14-algorithmic-enhancements-guide.md`, `outputs/slides/2026-11-14-algorithmic-enhancements-guide.html`, `outputs/notes/2026-11-14-supplementary-experiments-plan.md`, `outputs/slide-scripts/2026-11-14-supplementary-experiments-plan.md`, `outputs/slides/2026-11-14-supplementary-experiments-plan.html`, `outputs/slides/2026-11-14-large-vehicle-diagnostics.html`, `outputs/slides/2026-11-14-thesis-limitations-and-evidence-chain.html`, `index.html`, `index.md`
- **摘要**：深入研討並產出兩份全新 8 頁專題簡報 Deck：
  1. **演算法深入改良指引（Deck 06）**：跳脫微調參數，融合 EQL v2 動態梯度均衡解決 21.9:1 長尾壓制、3D-DIoU 中心空間約束解決高度漂移、VoD 車載 IMU 都卜勒投影補償消除路側偽動態、Version08 平台滿額映射曲線與 TVT 2023 體素連續性先驗，並設計 16GB 顯存友善之快取式知識蒸餾。
  2. **應該要補充的實驗清單（Deck 07）**：全面落地 Table 4-6 六大優先級實驗（40ep 收斂、平台曲線、教師消融、下游泛化、EQL、IMU）、建立 15 序列無洩漏場景協議（十字路口/夜雨/高速盲測）、物件級真值匹配（TP/LE/FN/FP）誤差歸因診斷矩陣，以及 8 大經典與前沿方法橫向對比大表。
  同時將新文獻理論深度反哺融入專題 04（大車診斷）與專題 05（限制與藍圖），更新專案首頁 `index.html` 擴展為 10 份簡報樞紐。全數通過 Node.js 語法與 KaTeX 渲染校驗。


## [2026-11-14] init | 本地專案初始化 Git 儲存庫並推送到 GitHub 遠端
- **操作類型**：init
- **涉及檔案**：`.gitignore`, `README.md`, 全專案 102 個檔案
- **摘要**：建立 `.gitignore` 與專業 `README.md`，將全專案 10 份自包含 HTML 簡報、Wiki 知識層與 16 篇論文原始 PDF 暫存並完成初次 Commit（`01d607b`），成功連結並推送到 GitHub 遠端儲存庫 `https://github.com/ChangChCh1220/research_ppt.git`（主分支 `main`），達成全庫雲端備份與版本控制。
