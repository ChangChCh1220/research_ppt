---
title: "研究筆記：K-Radar 4D 雷達張量全天候自動駕駛基準資料集"
date: 2026-11-13
topic: k-radar-dataset
author: Antigravity Agent
sources_consulted:
  - "[[wiki/sources/2022-k-radar]]"
  - "[[wiki/sources/2023-enhanced-k-radar]]"
  - "[[wiki/sources/2025-l4dr]]"
  - "[[wiki/entities/k-radar-dataset]]"
  - "[[wiki/concepts/4d-radar-tensor]]"
  - "[[wiki/comparisons/radar-representations-tensor-vs-pointcloud]]"
---

# 研究筆記：K-Radar 4D 雷達張量全天候自動駕駛基準資料集

> **規範遵循聲明**：本筆記正文與核心技術數據 **100% 嚴格依據知識庫 `wiki/` 既有內容撰寫**，所有引用均標註 `[[Wikilink]]`。Wiki 尚未涵蓋之邊界誠實標註 `[Wiki 尚無此部分資料]`。文末附上主動檢索之建議參考文獻與直接下載 PDF 連結。

---

## 一、資料集建立背景與核心痛點：為何需要 K-Radar？

在自動駕駛感知系統中，傳統光學感測器（相機與光達 LiDAR）雖然具備高空間解析度，但在惡劣氣候下極易失效（見 [[2022-k-radar]]）：
- **相機（RGB）**：在強光眩光、黑夜無照明、大雨與暴雪遮蔽鏡頭時無法精準測距；
- **光達（LiDAR）**：雷射光束（$\lambda = 850 \sim 1550\text{ nm}$）會被空氣中的雨滴、雪花強烈散射，且當冰霜、積雪堆積在感測器表面時會導致點雲密度急遽崩塌；
- **傳統毫米波雷達（3D Radar / 點雲 RPC）**：
  - 傳統車載雷達在韌體端經過 CFAR（恆虛警率）門檻過濾，輸出高度稀疏的點雲（RPC），遺失了大量微弱但真實的空間環境回波；
  - 既有雷達張量資料集（如 RADIATE 等）僅提供包含都卜勒、距離與方位角的 3D 雷達張量（3DRT），**缺乏垂直高度（俯仰角 Elevation）維度**，無法支援 3D 空間邊界框的高度與俯仰角預測。

為解決上述物理瓶頸，KAIST 團隊於 NeurIPS 2022 推出了 **[[k-radar-dataset|K-Radar]]**（Paek et al., 2022），成為自駕領域首個提供完整 **4D 雷達高維張量（4DRT）** 與全天候 3D 標註的旗艦級基準。

---

## 二、感測器套件與資料規格（Sensor Suite & Specs）

K-Radar 採集車輛配備了高防護等級（IP66+）的全天候感測陣列（見 [[2022-k-radar]]、[[wiki/entities/k-radar-dataset]]）：

1. **4D 毫米波成像雷達（RETINA-4ST）**：
   - 安裝於前保險桿格柵（避免引擎蓋與車頂多次反射干擾）；
   - FOV 為橫向 $107^\circ$、垂直 $37^\circ$，最大探測距離 $118\text{ m}$（10 FPS）；
   - **4DRT 原始高維張量維度**：$64(\text{Doppler}) \times 256(\text{Range}) \times 107(\text{Azimuth}) \times 37(\text{Elevation})$；
   - 物理解析度：都卜勒 $0.06\text{ m/s}$、距離 $0.46\text{ m}$、方位角 $1^\circ$、俯仰角 $1^\circ$。
2. **高精度雙 LiDAR 系統**：
   - 遠距 LiDAR：Ouster OS2-64（64 線，探測距離 $240\text{ m}$，131,072 點/幀）；
   - 高解析 LiDAR：Ouster OS1-128（128 線，探測距離 $120\text{ m}$，262,144 點/幀）。
3. **4 組立體相機（ZED2i）**：8 視角前/左/右/後立體相機（$1280 \times 720$，30 FPS）。
4. **高精定位與慣導**：RTK-GPS（誤差 $0.025\text{ m} + 1\text{ ppm}$）與雙 6 軸 IMU（100 FPS）。

### 資料規模與場景覆蓋
- **總幀數**：35,000 幀（58 個駕駛序列，訓練集 17,458 幀 / 測試集 17,536 幀；白天 21,820 幀 / 夜間 13,174 幀）；
- **3D 標註框**：93,345 個 3D 邊界框（含追蹤 ID），涵蓋 5 類物件：轎車（Sedan）、公車/卡車（Bus or Truck）、行人（Pedestrian）、自行車（Bicycle）、摩托車（Motorcycle）；
- **7 種天候條件**：正常晴天（Normal）、陰天（Overcast）、濃霧（Fog）、雨天（Rain）、冰霰（Sleet）、輕雪（Light Snow，$\le 1\text{ cm/hr}$）與大雪（Heavy Snow，$> 1\text{ cm/hr}$）；
- **多樣道路**：市區、高速公路、巷弄、郊區、大學校園、山路、停車場與路肩。

---

## 三、4D 雷達張量處理流程與 RTNH 基線架構

### 1. 4DRT 前處理管線（[[4d-radar-tensor]]）
1. 原始極座標 4DRT：沿都卜勒軸計算平均功率值，降維至三維空間張量 `3DRT-RAE`（$256 \times 37 \times 107$）；
2. 空間插值：藉由 3D 雙線性插值（Bilinear Interpolation）將極座標張量轉換為笛卡爾直角座標網格 `3DRT-XYZ`；
3. 稀疏化：在感興趣區域（RoI）內進行功率排序，保留最高能量點。

### 2. RTNH 基線網路（3D Sparse Convolution）
- 採用 **3D-SCB（3D 稀疏卷積塊）**：第 1 層為標準 3D 稀疏卷積，後續為 3D 子流形稀疏卷積（Submanifold Sparse Conv），維持空間幾何稀疏度；
- 稠密化與上採樣：透過 2D 轉置卷積（TransposeConv2D）將特徵上採樣並串接，輸出多尺度特徵；
- 偵測頭：以 $1\times 1$ 卷積預測分類分數與 8 維回歸參數（$x_c, y_c, z_c, x_l, y_l, z_l, \cos(\text{yaw}), \sin(\text{yaw})$），採 Focal Loss 與 Smooth $L_1$ 損失進行端到端優化。

---

## 四、關鍵實驗結論與演進修正

### 1. 核心結論 1：垂直高度（Elevation）對 3D 空間目標定位不可或缺
在 K-Radar 測試集上（Sedan IoU $> 0.3$，見 [[2022-k-radar]] Table 3 & 9）：
- **保留高度維度（RTNH，3D-SCB）**：$AP_{\text{3D}} = \mathbf{47.44\%}, AP_{\text{BEV}} = \mathbf{58.39\%}$，GPU 記憶體僅需 **421 MB**；
- **移除高度維度（RTN，ResNeXt-101 2D-DCB）**：$AP_{\text{3D}} = 40.12\%, AP_{\text{BEV}} = 50.67\%$，GPU 記憶體 520 MB；
- **結論**：保留高度資訊不僅使 3D AP 顯著提升 **+7.32%**，稀疏體素卷積更節省了 GPU 記憶體。

### 2. 核心結論 2：極端天候下的感測器優勢倒掛（4D 雷達顯著超越 LiDAR）
在不同天候條件下的 3D 偵測準確率對決（見 [[2022-k-radar]] Table 4）：
- **晴天正常天候**：LiDAR PointPillars 略勝（52.3% vs 49.9% 3D AP）；
- **濃霧（Fog）**：LiDAR AP 衰減至 42.2%，而 4D 雷達 RTNH 達到 **52.8%**；
- **冰霰（Sleet）**：LiDAR 光學散射與冰水混合使 AP 暴跌至 **22.7%**，4D 雷達達到 **41.5%**（領先近 1 倍）；
- **大雪（Heavy Snow）**：LiDAR 表面結霜覆雪失效降至 **29.7%**，4D 雷達仍穩定輸出 **44.5%**。

### 3. 重要演進與觀點修正：Enhanced K-Radar（4DSRT 與 5% 最佳密度）
> [!WARNING] 觀點修正：雷達張量並非越稠密越好
> - **原版 K-Radar (2022)** 啟發式地選擇保留 Top-10% 功率點，且在訓練時每個 Iteration 都在線上進行雙線性插值與排序，造成訓練極其緩慢（僅 0.47 iter/s），且 12 TB 原始資料無法上傳雲端；
> - **Enhanced K-Radar (2023)** 提出離線預計算的 **4DSRT**，訓練速度暴增 **17.1 倍**（達 8.04 iter/s）；
> - **密度消融實證**：保留 **5% 能量密度** 達到最高的 $AP_{\text{3D}}$（**47.9%**），且記憶體降至 380 MB。若強行推高至 15%~50%，記憶體暴增至 802 MB，而在冰霰天候下大量地面積冰雜訊被保留，導致 3D AP 從 39.2% 暴跌至 **22.2%**（見 [[2023-enhanced-k-radar]]）。

### 4. 在多模態融合中的標竿表現（L4DR）
[[2025-l4dr|L4DR]]（AAAI 2025）在 K-Radar 上驗證了推論期 LiDAR + 4DRT 融合，Sedan 3D AP 達到 **53.5%**，顯著超越 PointPillars（22.4%）、RTNH（37.4%）與先前最佳的 3D-LRF（45.2%）。

---

## 五、當前 Wiki 知識庫的研究缺口（Gaps in Wiki）

1. `[Wiki 尚無此部分資料]` **自動標註（Auto-Labeling）與半監督擴展機制**：
   - 知識庫目前收錄的 K-Radar 是完全依賴昂貴的人工手動標註；尚無利用 LiDAR 預訓練偵測器對 K-Radar 4DRT 進行自動化弱標註/無監督擴展的文獻。
2. `[Wiki 尚無此部分資料]` **跨感測器領域遷移（Domain Shift）**：
   - 作者在 2022 論文中坦承，K-Radar 與英國 RADIATE 等張量資料集存在雷達發射功率、頻率與解析度差異，跨資料集預訓練幾乎無法直接遷移；Wiki 尚無專門解決 4D 雷達感測器領域轉移（Sensor Domain Adaptation）的架構。
3. `[Wiki 尚無此部分資料]` **跨維度頻譜與影像融合（Cross-Dimension Fusion）**：
   - 既有 Wiki 中 K-Radar 的基準均為純雷達（RTNH）或 LiDAR-Radar 融合（L4DR），尚未收錄將原始 4DRT 頻譜與相機 RGB 影像直接進行跨維度注意力融合的最新架構。

---

## 六、建議參考文獻（待 Wiki 化）

為補齊上述第五節的缺口，精確檢索推薦以下 3 篇專門以 K-Radar 為核心評測基準、且**附帶直接下載 PDF 連結**的最新前沿論文：

1. **Auto-Labeling 4D Radar Tensors for 3D Object Detection**
   - **作者/年份**：IEEE IVS 2024（arXiv:2407.04709）
   - **可點擊下載 PDF**：[https://arxiv.org/pdf/2407.04709.pdf](https://arxiv.org/pdf/2407.04709.pdf)
   - **推薦理由（補齊缺口 1）**：利用 LiDAR 物件偵測網路（LODN）對 K-Radar 訓練集進行高精度自動化標註，解決 4DRT 手動標註極度耗時耗資的瓶頸。
2. **Multi-Teacher Knowledge Distillation for 4D Radar Object Detection**
   - **作者/年份**：2025 年預印本（arXiv:2502.06114）
   - **可點擊下載 PDF**：[https://arxiv.org/pdf/2502.06114.pdf](https://arxiv.org/pdf/2502.06114.pdf)
   - **推薦理由（補齊缺口 2）**：在 K-Radar 基準上採用多教師跨模態知識蒸餾，進一步突破 RTNH 的稀疏感知上限。
3. **RadarXFormer: Robust Object Detection via Cross-Dimension Fusion of 4D Radar Spectra and Images**
   - **作者/年份**：2026 年預印本（arXiv:2603.14822）
   - **可點擊下載 PDF**：[https://arxiv.org/pdf/2603.14822.pdf](https://arxiv.org/pdf/2603.14822.pdf)
   - **推薦理由（補齊缺口 3）**：在 K-Radar 上首次將 4D 原始雷達頻譜張量與 2D 相機影像進行跨維度 Transformer 融合。
