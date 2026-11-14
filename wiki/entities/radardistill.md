---
title: RadarDistill
type: entity
tags:
  - multimodal
  - autonomous-driving
  - 3d-radar
  - cross-modal-supervision
  - knowledge-distillation
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# RadarDistill

## 基本定位
**RadarDistill** 是由韓國漢陽大學、KAIST 與首爾大學團隊於 CVPR 2024 提出的**跨模態特徵知識蒸餾框架**（見 [[2024-radardistill]]），旨在將 LiDAR 的高品質稠密 BEV 表徵傳遞至純雷達（PillarNet-18）目標偵測器中，推論期達成 100% 純雷達運作。

---

## 三大核心組件
1. **跨模態對齊（Cross-Modality Alignment, CMA）**：
   - 針對雷達非空 Pillar 僅約 LiDAR 10% 的極端稀疏性，透過雙向可變形卷積下採樣與轉置卷積上採樣，將低階雷達特徵進行膨脹與稠密化。
2. **基於啟動的特徵蒸餾（Activation-based Feature Distillation, AFD）**：
   - 將低階特徵空間劃分為活化區（$AR$）與非活化區（$IR$），藉由自適應面積權重 $\rho$ 解決前景與背景像素的不平衡。
3. **基於提案的特徵蒸餾（Proposal-based Feature Distillation, PFD）**：
   - 針對高階特徵，依據真值與雷達分類熱圖劃分 TP、FP、FN 三類提案區域；
   - 引入 **通道 Softmax 正規化**，跨越雷達與 LiDAR 特徵數值量級差異，消除梯度震盪。

---

## 代表性指標（nuScenes 純雷達測試集）
- **mAP**：**20.5%**（超越 PillarNet 基線 8.6%、KPConvPillars 4.9%）。
- **NDS**：**43.7%**（超越基線 34.7%）。
- **Car AP**：**54.0%**（超越純單目相機 FCOS3D 52.4% 與相機+雷達融合 CenterFusion 50.9%）。

---

## 參考來源
- [[2024-radardistill]]
- [[cross-modal-supervision]]
