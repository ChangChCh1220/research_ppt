---
title: PointNet++ 與階層式點雲特徵學習（PointNet++ & Hierarchical Point Set Learning）
type: concept
tags:
  - architecture
  - 3d-point-cloud
  - pointnet-plus-plus
created: 2026-11-12
updated: 2026-11-12
sources_count: 3
status: active
---

# PointNet++ 與階層式點雲特徵學習

## 定義與定位
**PointNet++** 由 Qi et al. (Stanford, 2017) 提出（見 [[2017-pointnet-plus-plus]]），是處理無序 3D 點集的經典階層式深度神經網路架構。它克服了第一代 PointNet 僅具單一層級全域特徵提取、無法捕捉局部度量空間結構的根本缺陷，在點雲領域奠定了類似卷積神經網路（CNN）「由局部至全域、多尺度漸進抽取」的階層式表徵典範。

---

## 核心架構模組

### 1. 集合抽象層（Set Abstraction, SA Level）
由三階段管線構成，將點集進行下採樣並整合局部鄰域：
1. **FPS 採樣（Farthest Point Sampling）**：依據度量距離迭代挑選空間覆蓋最均勻的 $N'$ 個質心點。
2. **度量球查詢（Metric Ball Query）**：在半徑 $r$ 內收集最多 $K$ 個鄰近點。相較於 $k$-NN，球查詢確保了幾何感受野在實體空間尺度的一致性。
3. **局部 PointNet（PointNet Layer）**：將鄰近點轉化為相對於質心的相對座標後，以共享權重 MLP 與 Max Pooling 聚合為質心的高階幾何特徵。

### 2. 適應非均勻密度的聚合機制
- **多尺度群組化（Multi-Scale Grouping, MSG）**：在每個質心同時採用多個不同半徑提取特徵；訓練時搭配 **隨機輸入丟棄（Random Input Dropout, DP）**，使網路在稠密點與稀疏點區域皆具備極高的強健性。
- **多解析度群組化（Multi-Resolution Grouping, MRG）**：串接下層抽象特徵與原始點雲局部特徵，降低運算複雜度。

### 3. 特徵傳播層（Feature Propagation, FP Level）
利用反距離加權（Inverse-distance weighted）的 $k$-NN 空間插值與 Skip Connection 串接，將特徵上採樣還原至所有原始點，實現逐點分類與分割。

---

## 在 4D 毫米波雷達感知中的核心地位
4D 毫米波雷達點雲同時具備極端稀疏、非均勻分佈與雜訊嚴重的物理特性，PointNet++ 的階層式架構在此領域發揮了關鍵作用：
1. **作為體素偵測器的點拓撲補償**：在 [[corenet|CORENet]]（見 [[2025-corenet]]）中，純體素稀疏卷積（SConvNet）容易損失點拓撲細節，作者以 PointNet++ 為基礎構建 **HPNet（Hierarchical PointNet）**，透過 FPS 降採樣與 MLP 傳播，使車輛與小目標偵測 AP 大幅提升。
2. **作為雷達前景去噪分割器**：在 [[l4dr|L4DR]]（見 [[2025-l4dr]]）中，作者以 PointNet++ 構建 **FAD（Foreground-Aware Denoising）** 模組，在早期融合前準確預測雷達點的前景機率，過濾多路徑反射雜訊。

---

## 參考來源
- [[2017-pointnet-plus-plus]]
- [[2025-corenet]]
- [[2025-l4dr]]
