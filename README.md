# LLM Wiki · 4D 毫米波雷達與自駕感知研究知識庫

> **大型語言模型（LLM）與生成式 AI 研究知識庫**  
> 參考 Andrej Karpathy 的 *LLM Wiki* 理念：**「知識只編譯一次並持續維護，而非每次提問都從零檢索。」**  
> 本專案以 **Obsidian 為 IDE，Agent 為知識編譯器，Wiki 為持續複利的代碼庫**。

---

## 📌 核心研究主線

- **碩士論文題目**：光達輔助弱監督之四維雷達可靠度估計與功率抑制（*LiDAR-Assisted Weakly Supervised Reliability Estimation and Power Suppression for 4D Imaging Radar*）
- **研究生**：張喬筑（國立陽明交通大學 土木工程研究所）
- **指導教授**：莊子毅 博士

---

## 🖥️ 互動式研究簡報門戶（Presentation Hub）

專案根目錄下的 [`index.html`](index.html) 整合了所有自包含 HTML 簡報，支援搜尋關鍵字與標籤過濾。所有簡報均採用 **冰川極簡藍白（Glacial Tech Minimal）** 風格，具備 **16:9 畫布、KaTeX LaTeX 公式即時渲染、鍵盤翻頁（`←` / `→` / `Space`）、講者口白備忘錄抽屜（按 `S` 鍵）與列印 PDF 支援**。

### 🌟 簡報目錄清單

| 編號 | 專題主題 | 投影片 | 相關連結 | 核心重點 |
| :---: | :--- | :---: | :---: | :--- |
| **00** | **碩士論文全景進度報告** | **24 頁** | [簡報](outputs/slides/2026-11-14-thesis-progress-and-future-work.html) · [筆記](outputs/notes/2026-11-14-thesis-progress-and-future-work.md) | 涵蓋三態弱監督、局部特徵網路、115組三種子結果、四大競爭假說與最小證據鏈之旗艦大匯總 |
| **01** | **弱監督教師與幾何標籤生成** | **8 頁** | [簡報](outputs/slides/2026-11-14-weak-supervision-teacher.html) · [筆記](outputs/notes/2026-11-14-weak-supervision-teacher.md) | 3D 物件框幾何映射、三態連續標籤（核心區/保護區/衰減區）、破除俯視 1,759 個負代理誤判 |
| **02** | **雷達原生特徵與可靠度網路** | **8 頁** | [簡報](outputs/slides/2026-11-14-radar-features-and-network.html) · [筆記](outputs/notes/2026-11-14-radar-features-and-network.md) | 中心 14 維 + 局部鄰域 10 維原生特徵、雙分支輕量 MLP、揭密移除顯式都卜勒使 AP 躍升至 68.46% |
| **03** | **功率抑制映射與約束式選模** | **8 頁** | [簡報](outputs/slides/2026-11-14-power-suppression-and-selection.html) | 原位功率調變原則（保持 15 萬點坐標）、連續軟抑制 W065、三道不可逾越死線與綜合目標 $J$ |
| **04** | **大型車衰退與四大競爭假說** | **8 頁** | [簡報](outputs/slides/2026-11-14-large-vehicle-diagnostics.html) | 21.9:1 長尾失衡、60–80m 遠距功率暴跌至 82.68%、轎車升大車跌矛盾、逐物件漏檢配對檢驗 |
| **05** | **研究限制與定稿最小證據鏈** | **8 頁** | [簡報](outputs/slides/2026-11-14-thesis-limitations-and-evidence-chain.html) | 誠實審視 7 大研究限制、Version08 平台滿額映射藍圖、表 4-6 六大優先級定稿最小證據鏈 |
| **06** | **演算法深入改良與前沿指引** | **8 頁** | [簡報](outputs/slides/2026-11-14-algorithmic-enhancements-guide.html) · [筆記](outputs/notes/2026-11-14-algorithmic-enhancements-guide.md) | 融合 EQL v2 動態梯度均衡、3D-DIoU 空間中心回歸、VoD 自車 IMU 都卜勒投影補償、快取式蒸餾 |
| **07** | **補充實驗清單與評測方案** | **8 頁** | [簡報](outputs/slides/2026-11-14-supplementary-experiments-plan.html) · [筆記](outputs/notes/2026-11-14-supplementary-experiments-plan.md) | Table 4-6 具體排程、15 序列盲測協議（防洩漏）、物件級 TP/LE/FN/FP 歸因、8 大方法橫評大表 |
| **08** | **四維雷達可靠度與功率抑制** | **10 頁** | [簡報](outputs/slides/2026-11-13-radar-reliability-power-suppression.html) | 軟式去噪核心概念、CORENet 體素連續信心、RadarDistill 軟蒸餾、Kung et al. 負面監督陷阱 |
| **09** | **K-Radar 張量自駕基準資料集** | **10 頁** | [簡報](outputs/slides/2026-11-13-k-radar-dataset.html) · [筆記](outputs/notes/2026-11-13-k-radar-dataset.md) | 全球首個全天候 4D 雷達張量（4DRT）資料集（35K 幀、7 種極端氣候）、RTNH 稀疏卷積與 4DSRT 5% |

---

## 📂 知識庫三層架構規範

```text
LLM_WIKI/
├── AGENTS.md              # [Schema] 知識庫運作憲章與維護規範
├── index.html             # [門戶] 互動式 HTML 簡報全景導覽門戶（支援 GitHub Pages）
├── index.md               # [導航] 內容導向的 Wiki 全域索引
├── log.md                 # [日誌] 時間序只增不減（Append-only）操作紀錄
│
├── raw/                   # [層 1：不可變原始資料]
│   ├── papers/            #   收錄 16 篇前沿論文原始 PDF
│   ├── notes/             #   論文原稿與實驗筆記（含 2026 碩論初稿 PDF）
│   ├── articles/          #   技術文章與 Web Clipper 擷取
│   └── assets/            #   圖表與架構圖附件
│
├── wiki/                  # [層 2：維護之知識層]
│   ├── overview.md        #   研究總覽與知識網絡全貌
│   ├── sources/           #   16 篇論文標準結構化摘要頁
│   ├── concepts/          #   9 大核心技術、演算法與損失函數概念頁
│   ├── entities/          #   7 大模型家族、資料集與開源框架
│   └── comparisons/       #   5 篇橫向評比與專題分析
│
└── outputs/               # [層 3：衍生應用輸出層]
    ├── notes/             #   嚴格僅依 Wiki 編寫之專題研究筆記
    ├── slide-scripts/     #   對應筆記之逐頁簡報腳本（含講者備忘稿）
    └── slides/            #   10 份單檔自包含 HTML 簡報（內建 KaTeX）
```

---

## 🌐 啟用 GitHub Pages 線上瀏覽

若您將本專案推送到 GitHub，可透過 GitHub Pages 免費發布為線上簡報網站：
1. 進入 GitHub 儲存庫頁面，點選 **Settings**。
2. 在左側選單點選 **Pages**。
3. 在 **Build and deployment** > **Branch** 選擇 `main`，資料夾選擇 `/ (root)`，並點選 **Save**。
4. 稍等約 1 分鐘後，即可透過 `https://<你的 GitHub 帳號>.github.io/<儲存庫名稱>/` 直接在瀏覽器中線上開啟全部 10 份研究簡報與全景門戶！

---

## 📜 操作日誌與版本控制

所有資料攝入（Ingest）、查詢沉澱（Query）與健康維護（Lint）均紀錄於 [`log.md`](log.md)，遵守嚴格的不可變追加原則。
