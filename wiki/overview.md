---
title: LLM 研究知識庫總覽
type: overview
tags:
  - llm-research
created: 2026-11-12
updated: 2026-11-12
sources_count: 11
status: active
---

# LLM 研究知識庫總覽

本頁面是整個研究知識庫的演化地圖（Synthesis）。隨著 `raw/` 來源持續攝入，LLM 會在每次重大 Ingest 後更新本頁，串聯各研究子領域的核心脈絡、主流共識、關鍵爭鳴與未解研究缺口。

---

## 核心研究版圖與知識聚類

### 1. 多模態感知與自駕 3D 點雲（Multimodal Perception & 3D Sensing）
- **4D 毫米波雷達表徵**：
  - [[4d-radar-tensor|4D 雷達張量（4DRT / 4DSRT）]] vs. 傳統 CFAR 截斷點雲（RPC）。
  - 核心突破：[[k-radar-dataset|K-Radar]] 證明高度維度對 3D 偵測不可或缺；[[2023-enhanced-k-radar|Enhanced K-Radar]] 證實 **5% 能量密度** 達到最高精度且訓練速度提升 17.1 倍（推翻越稠密越好的直覺）。
- **去噪與特徵學習**：
  - [[4d-radar-denoising|4D 雷達去噪技術路線]]：從傳統幾何/統計濾波（MLS/ROR/SOR），演進至點層級語意去噪（[[l4dr|L4DR]] 中的 FAD）、高階 BEV 特徵蒸餾（[[radardistill|RadarDistill]] 中的 CMA/AFD/PFD），以及體素幾何遮罩監督（[[corenet|CORENet]]）。
  - 基礎骨幹：[[pointnet-plus-plus|PointNet++]] 集合抽象層（FPS + 球查詢 + 局部 PointNet）為 irregular 點雲拓撲提供核心特徵提取能力。
- **跨模態學習範式對比**：
  - 專題分析：[[cross-modal-supervision-vs-fusion|跨模態監督 vs. 推論期多模態融合橫向評比]]。
  - **訓練期跨模態監督**（[[cross-modal-supervision|Cross-Modal Supervision]]）：[[corenet|CORENet]]、[[radardistill|RadarDistill]]、Kung et al. (RA-L 2022) 達成推論端 100% 純雷達部署；但需注意 LiDAR 的短視距與無穿透性產生的負面監督（需透過功率過濾與極座標滑動窗口克服）。
  - **推論期多模態融合**（[[lidar-radar-fusion|LiDAR-Radar Multi-Modal Fusion]]）：[[l4dr|L4DR]] 透過早期雙向 Pillar 幾何傳播（MME）與後期多尺度門控（MSGF），在 Level 4 濃霧與大雪中保護感知系統免於 LiDAR 崩潰。

### 2. 目標偵測與極端不平衡損失函數（Loss Functions & Imbalanced Learning）
- 專題演進：[[imbalanced-and-focal-loss-evolution|類別不平衡與聚焦損失演進史]]。
- **經典基準**：[[focal-loss|Focal Loss (2017)]] 引入 $(1-p_t)^\gamma$ 抑制簡單背景樣本，搭配 [[retinanet|RetinaNet]] 突破單階段偵測瓶頸。
- **長尾資訊容量**：[[class-balanced-loss|Class-Balanced Loss (2019)]] 建立隨機幾何覆蓋理論，以有效樣本數 $E_n = (1-\beta^n)/(1-\beta)$ 為 Focal Loss 提供動態 $\alpha_t$ 加權。
- **連續目標統一**：[[generalized-focal-loss|Generalized Focal Loss (2020)]] 突破二元離散標籤限制，將 Focal Loss 擴展至任意連續區間，統一了品質-分類聯合學習（QFL）與連續分佈邊界框回歸（DFL）。

### 3. 基礎優化器與模型正則化（Optimization Foundations）
- **核心基石**：[[adamw|AdamW 與解耦權重衰減（Decoupled Weight Decay）]] 揭示了在自適應梯度優化器（Adam）中 $L_2$ 正則化被二階動量稀釋的數學不等價性；解耦後的 AdamW 改善泛化曲面並在超參數空間實現正交調參，奠定了當代 Transformer、LLM 預訓練與 3D 視覺骨幹的預設優化器地位。

---

## 當前演進摘要與重要共識 / 爭鳴

> [!TIP] 關鍵共識（Consensus）
> 1. **LiDAR 監督下的雷達去噪與特徵蒸餾**（[[2025-corenet|CORENet]], [[2024-radardistill|RadarDistill]]）已證實可大幅拉近純雷達與光學感測器的差距（Car 3D AP 逼近 50%~54%），推論端零 LiDAR 成本。
> 2. **極端天候下的感測器倒掛**：在冰霰（Sleet）與重度濃霧（Fog L4）下，純 LiDAR 效能會劇降 30%~80%（見 [[2022-k-radar|K-Radar]]、[[2025-l4dr|L4DR]]），毫米波雷達的全天候物理穿透特性在惡劣天氣下具備無可替代的安全性。
> 3. **解耦優化**：不論在損失調變（Focal / GFL）或權重正則化（AdamW），將耦合參數（正負比例、LR 與 Weight Decay）進行正交解耦皆顯著提升泛化穩定度。

> [!WARNING] 當前研究缺口與爭鳴（Research Gaps & Discrepancies）
> 1. **極稀疏小目標（行人、自行車）偵測依然低落**：在 Dual-Radar 與 nuScenes 上，純雷達模型在行人與自行車的 3D AP 普遍仍在 $1\% \sim 10\%$ 徘徊（CORENet 行人 3D AP 僅 4.55%），亟需時序多幀累積或更高解析度張量特徵支援。
> 2. **推論融合的即時性代價**：如 L4DR 引入並行模態骨幹與門控機制，推論延遲達 96 ms（約 10.4 FPS），剛好逼近車端 10 Hz 採樣極限，高算力與低延遲之間仍存在顯著權衡。
> 3. **LiDAR 負面監督問題**：LiDAR 無法穿透且無法探測細長物體，直接蒸餾易引發雷達在遠距與遮擋區的幻覺，需要更精細的物理門檻過濾。
