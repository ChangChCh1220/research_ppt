---
title: "CORENet：以 LiDAR 監督的跨模態 4D 雷達去噪網路（2025）"
type: source
tags:
  - multimodal
  - autonomous-driving
  - 4d-radar
  - point-cloud-denoising
  - cross-modal-supervision
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# CORENet：以 LiDAR 監督的跨模態 4D 雷達去噪網路

## 書目與基本資訊
- **論文標題**：*CORENet: Cross-Modal 4D Radar Denoising Network with LiDAR Supervision for Autonomous Driving*
- **作者與機構**：Fuyang Liu, Jilin Mei, Fangyuan Mao, Chen Min, Yan Xing, Yu Hu（中國科學院計算技術研究所 ICT-CAS、中國科學院大學 UCAS、北京控制工程研究所）
- **發表日期 / 編號**：2025-08-19｜`arXiv:2508.13485v1 [cs.CV]`
- **開源程式碼**：https://github.com/charlesuv/corenet.git
- **原始來源檔案**：`raw/papers/2025-corenet-cross-modal-4d-radar-denoising.md`

## 一句話總結（TL;DR）
[[corenet|CORENet]] 是一個可即插即用於體素（Voxel）3D 目標偵測器的 4D 毫米波雷達去噪框架：透過 **[[cross-modal-supervision|跨模態監督]]** 在訓練階段利用 LiDAR 點雲產生體素級真值遮罩（$M_{\text{gt}}$）來監督 **HMSD-Net** 分辨雜訊與有效點，並在推論階段完全脫離 LiDAR、實現純雷達高速推論。

---

## 核心問題與動機
1. **4D 成像雷達的優勢與痛點**：相較傳統 3D 雷達缺乏高度（elevation）資訊，4D 雷達同時量測距離（range）、方位角（azimuth）、俯仰角（elevation）與都卜勒速度（Doppler velocity），且在雨霧黑夜等惡劣天候比相機與 LiDAR 穩定；但其點雲高度稀疏且存在強烈雜訊。
2. **雜訊物理成因**：作者指出 4D 雷達的主要雜訊源自強目標周圍的**旁瓣能量洩漏（energy leakage in sidelobes）**，特別因垂直解析度低而沿垂直切線方向擴散。
3. **既有方法的侷限**：
   - 傳統點雲濾波器（[[4d-radar-denoising|ROR、SOR、MLS]]）可去除孤立離群點，但難以識別複雜空間分佈的大範圍雜訊群。
   - 現有深度去噪（Point-BERT、DNCNet）或 LiDAR-雷達融合/蒸餾（RadarNet、RadarDistill）大多針對低雜訊 3D 雷達或需要多模態輸入，尚未充分解決高雜訊 4D 雷達的去噪問題。

---

## 關鍵方法與技術細節

### 1. 整體雙分支架構
- **偵測分支（訓練＋推論）**：原始 4D 雷達點雲（含 $[x, y, z]$、訊號強度 $i_s$、徑向都卜勒速度 $v_D$）→ 體素化 $f_p \in \mathbb{R}^{N \times d}$ → **HMSD-Net** 輸出預測遮罩 $M_{\text{radar}}$ 與去噪特徵 → 投影至 BEV（$f_{\text{BEV}}$）→ 2D Backbone → Detection Head。
- **LiDAR 監督分支（僅訓練階段）**：LiDAR 點雲 $Q$ 經 **Mask Generator** 產生監督遮罩 $M_{\text{gt}}$。

### 2. HMSD-Net（Hierarchical Multi-Scale Denoising Network）
結合兩種互補的特徵提取分支，兼顧點雲拓撲與體素空間局部性：
- **HPNet（Hierarchical PointNet）**：多層降採樣（FPS + $k$-NN 聚類）與共享權重 MLP 特徵傳播＋跳躍串接，保留點之間的拓撲關係，輸出 $h_k \in \mathbb{R}^{N \times 128}$。
- **SConvNet（Sparse ConvNet）**：多階段子流形稀疏卷積（Submanifold Sparse Convolution）+ BN + ReLU，經上採樣對齊維度後串接得 $f_{\text{conv}} \in \mathbb{R}^{N \times 64}$。
- **Noise Predictor $P$**：以單層 MLP 對串接特徵進行通道預測，輸出每個體素屬於有效點雲的信心分數 $M_{\text{radar}} = P(h_k \oplus f_{\text{conv}}) \in [0, 1]$。

### 3. 基於 KDTree 的跨模態監督
- 對 LiDAR 點雲 $Q = \{q_j \in \mathbb{R}^3\}_{j=1}^M$ 建立週期性交替切分軸（$x \to y \to z$）的 KDTree。
- 對每個雷達點 $p_i$ 搜尋半徑 $\tau = 0.5\text{ m}$ 內的 LiDAR 鄰近點 $N(p_i)$：若 $|N(p_i)| \ge 1$ 則 $M_{\text{gt}}(i) = 1$，否則為 $0$。
- **總損失函數**：$L_{\text{total}} = \alpha L_{\text{cls}} + \beta L_{\text{reg}} + \gamma L_{\text{msk}}$，實驗設定 $(\alpha, \beta, \gamma) = (1, 2, 50)$，其中 $L_{\text{msk}}$ 為 Smooth $L_1$ Loss。

> [!NOTE] 論文文本中的細節差異記錄
> 論文 Sec. III-A 式 (3) 寫為 $L_{\text{cls}} = \text{BCE}(\cdot)$、$L_{\text{reg}} = \text{MSE}(\cdot)$，但 Sec. III-C 敘述「$L_{\text{cls}}$ utilizes focal loss. The $L_{\text{reg}}$ and $L_{\text{msk}}$ employ the Smooth L1 function」，兩處描述在原文中存在不一致，實作時應以開源程式碼為準。

---

## 主要實驗結果與限制

### 1. Dual-Radar 測試集主實驗（Table I，KITTI 風格 AP 平均）
評測於 [[dual-radar-dataset|Dual-Radar]] 資料集（Arbe Phoenix 4D 雷達，共 9,966 幀，5,100 訓練 / 2,400 測試 / 2,466 驗證；Car IoU=50%，Pedestrian/Cyclist IoU=25%）：
- **CasA-V 骨幹**：加入 CORENet 後，Car 3D AP 由 `14.81%` 提升至 **`49.46%`**（+34.65%），BEV AP 由 `16.87%` 提升至 **`54.38%`**（+37.51%）；全類別 Mean 3D AP 由 `4.96%` 升至 **`21.37%`**，Mean BEV AP 由 `5.66%` 升至 **`23.14%`**。
- **對比傳統去噪**：在 CasA-V 上大幅超越 `ROR`（Mean 3D/BEV `14.99% / 16.51%`）與 `SOR`（`16.62% / 18.26%`）。
- **跨架構泛化（Voxel R-CNN）**：加入 CORENet 後 Mean 3D AP 由 `11.76%` 升至 `19.44%`，Mean BEV AP 由 `13.17%` 升至 **`23.43%`**。

### 2. 消融實驗關鍵發現（Tables II–IV）
- **雙分支互補性**：僅用 HPNet（Mean 3D AP `26.64%`）或僅用 SConvNet（`24.92%`）皆低於兩者結合（`27.47%`）。
- **KDTree 匹配半徑 $\tau$**：$\tau = 0.5\text{ m}$ 最佳（`27.47%`），過小（`0.3 m` → `26.86%`）或過大（`0.7 m` → `25.81%`）均下降。
- **Noise Predictor MLP 層數**：**1 層 MLP 最佳**（`27.47%`），增至 2 層（`26.10%`）或 3 層（`26.37%`）反而因過度參數化導致過擬合而退步。
- *註*：Table II–IV 消融實驗的絕對 AP 數值（如 Car 3D AP `67.34%`）明顯高於 Table I 測試集（`49.46%`），推測消融可能是在驗證集或單一難度子集上進行，原文未明確說明分割差異。

### 3. 限制與未來工作
1. **訓練期仍強依賴高精度 LiDAR**：無法完全無監督或僅靠單模態訓練。
2. **小目標（行人、騎士）表現仍偏低**：在 Dual-Radar 測試集上 Pedestrian 3D AP 僅 `4.55%`（CasA-V+CORENet）、Cyclist 為 `10.11%`，仍是待突破瓶頸。

---

## 知識庫關聯
- **關聯概念**：[[4d-radar-denoising]]、[[cross-modal-supervision]]
- **關聯實體**：[[corenet|CORENet]]、[[dual-radar-dataset|Dual-Radar]]
