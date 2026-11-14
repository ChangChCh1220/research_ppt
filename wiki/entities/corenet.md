---
title: CORENet
type: entity
tags:
  - multimodal
  - autonomous-driving
  - 4d-radar
  - point-cloud-denoising
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# CORENet

## 基本定位
**CORENet**（*Cross-Modal 4D Radar Denoising Network with LiDAR Supervision*）是由中國科學院計算技術研究所（ICT-CAS）等團隊於 2025 年提出的即插即用（plug-and-play）**4D 雷達點雲去噪與 3D 目標偵測增強框架**（見 [[2025-corenet]]）。

- **開源實作**：https://github.com/charlesuv/corenet.git（基於 `OpenPCDet` 與 PyTorch 1.8.1）
- **核心設計哲學**：訓練期利用 LiDAR 做 [[cross-modal-supervision|跨模態監督]]，推論期維持 **100% 純 4D 雷達運作**。

---

## 核心元件與資料流
1. **輸入特徵**：4D 雷達體素點雲 $f_p = \{p_1, \dots, p_N\}$，包含 3D 座標 $[x, y, z]$、訊號強度 $i_s$ 與徑向都卜勒速度 $v_D$。
2. **HMSD-Net（階層式多尺度去噪網路）**：
   - **HPNet（Hierarchical PointNet）**：FPS 降採樣 + $k$-NN 聚類 + 共享權重 MLP 與跳躍串接，擷取點間拓撲特徵 $h_k \in \mathbb{R}^{N \times 128}$。
   - **SConvNet（Sparse ConvNet）**：多階段子流形稀疏卷積 + BN + ReLU，經上採樣對齊後輸出空間特徵 $f_{\text{conv}} \in \mathbb{R}^{N \times 64}$。
   - **Noise Predictor $P$**：單層 MLP（消融證實 1 層優於 2/3 層），輸入 $h_k \oplus f_{\text{conv}}$ 並預測每個體素的有效信心分數 $M_{\text{radar}} \in [0, 1]$。
3. **Mask Generator（僅訓練期）**：
   - 使用 KDTree 在 LiDAR 點雲中以半徑 $\tau = 0.5\text{ m}$ 進行最近鄰配對，生成二元監督遮罩 $M_{\text{gt}} \in \{0, 1\}$。
4. **相容偵測骨幹**：
   - 可無縫接入體素架構的 3D 偵測器（如 `CasA-V`、`Voxel R-CNN`），將 SConvNet 特徵投影為 BEV 表示 $f_{\text{BEV}}$ 後交由既有 2D Backbone 與 Detection Head 處理。

---

## 代表性表現（[[dual-radar-dataset|Dual-Radar]] 測試集）
- **CasA-V + CORENet**：Car 3D AP `49.46%`（原 `14.81%`）、Mean 3D AP `21.37%`（原 `4.96%`）、Mean BEV AP `23.14%`（原 `5.66%`）。
- **Voxel R-CNN + CORENet**：Car 3D AP `46.48%`（原 `35.15%`）、Mean 3D AP `19.44%`（原 `11.76%`）、Mean BEV AP `23.43%`（原 `13.17%`）。
- **關鍵超參數**：$\alpha = 1, \beta = 2, \gamma = 50$，KDTree 距離閾值 $\tau = 0.5\text{ m}$，訓練 80 epochs，lr = `0.003`。

---

## 關聯頁面
- **來源**：[[2025-corenet]]
- **核心概念**：[[4d-radar-denoising]]、[[cross-modal-supervision]]
- **評測基準**：[[dual-radar-dataset|Dual-Radar]]
