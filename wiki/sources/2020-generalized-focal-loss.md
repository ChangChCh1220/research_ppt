---
title: "Generalized Focal Loss：密集目標偵測中的品質與分佈邊界框學習（NeurIPS 2020）"
type: source
tags:
  - architecture
  - focal-loss
  - generalized-focal-loss
  - object-detection
  - loss-functions
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# Generalized Focal Loss：密集目標偵測中的品質與分佈邊界框學習

## 書目與基本資訊
- **論文標題**：*Generalized Focal Loss: Learning Qualified and Distributed Bounding Boxes for Dense Object Detection*
- **作者與機構**：Xiang Li, Wenhai Wang, Lijun Wu, Shuo Chen, Xiaolin Hu, Jun Li, Jinhui Tang, Jian Yang（南京理工大學 PCALab、Momenta、南京大學、微軟亞洲研究院 MSRA、清華大學）
- **發表日期 / 編號**：NeurIPS 2020｜`arXiv:2006.04388v1 [cs.CV]`
- **開源程式碼**：https://github.com/implus/GFocal
- **原始來源檔案**：`raw/papers/2020-generalized-focal-loss.pdf`

## 一句話總結（TL;DR）
[[generalized-focal-loss|Generalized Focal Loss (GFL)]] 針對密集目標偵測中「分類分數與定位品質分離導致 NMS 排序失真」以及「邊界框回歸採用狄拉克 $\delta$ 分佈無法表達邊界模糊性」的痛點，提出 **品質聚焦損失（QFL）** 與 **分佈聚焦損失（DFL）**，將經典 Focal Loss 從離散二元標籤 $\{0, 1\}$ 推廣至任意連續區間 $[y_l, y_r]$，在不增加推論負擔下顯著提升偵測精度。

---

## 核心問題與動機
1. **訓練與推論的品質指標不一致（Inconsistency in Quality Estimation）**：
   - 現有單階段偵測器（如 FCOS、ATSS）將分類分支與定位品質分支（Centerness 或 IoU 預測）獨立訓練，但推論時直接將兩者相乘作為 NMS 排序依據；
   - 品質分支**僅在正樣本上受監督**，負樣本區域從未被抑制，導致背景噪點容易預測出極高的品質分數（如 $>0.9$），在 NMS 中誤殺真實前景。
2. **僵硬的邊界框狄拉克分佈（Inflexible Bounding Box Representation）**：
   - 傳統邊界框回歸將座標建模為狄拉克脈衝分佈 $\delta(x - y)$，無法表達真實世界中遮擋、陰影、模糊等造成的邊界不確定性（Uncertainty）；高斯分佈又過於對稱且難以與 IoU 損失結合。

---

## 關鍵方法與數學統一框架

### 1. 品質聚焦損失（Quality Focal Loss, QFL）
將分類向量與定位品質（IoU）結合成聯合表徵，標籤為連續值 $y \in [0, 1]$（負樣本 $y=0$；正樣本 $y$ 為預測框與真值框之 IoU）：
$$\text{QFL}(\sigma) = -|y - \sigma|^\beta \big((1 - y)\log(1 - \sigma) + y\log(\sigma)\big)$$
設定 $\beta = 2$。全域極小值嚴格落在 $\sigma = y$。

### 2. 分佈聚焦損失（Distribution Focal Loss, DFL）
將邊界框座標建模為一般機率分佈 $P(x)$，離散化為 $\{y_0, y_1, \dots, y_n\}$（區間間隔 $\Delta = 1$），透過 Softmax 輸出 $\mathcal{S}_i = P(y_i)$，預測座標為期待值 $\hat{y} = \sum_{i=0}^n \mathcal{S}_i y_i$。
DFL 迫使機率快速聚焦在距離真值 $y$ 最近的兩個離散相鄰值 $y_i, y_{i+1}$（$y_i \le y \le y_{i+1}$）：
$$\text{DFL}(\mathcal{S}_i, \mathcal{S}_{i+1}) = -\big((y_{i+1} - y)\log(\mathcal{S}_i) + (y - y_i)\log(\mathcal{S}_{i+1})\big)$$
極小值為 $\mathcal{S}_i = \frac{y_{i+1}-y}{y_{i+1}-y_i}, \mathcal{S}_{i+1} = \frac{y-y_i}{y_{i+1}-y_i}$，數學上嚴格保證 $\hat{y} = y$。

### 3. 統一的 Generalized Focal Loss（GFL）
對於任意連續區間 $[y_l, y_r]$ 與兩端機率 $p_{y_l}, p_{y_r}$（預測值 $\hat{y} = y_l p_{y_l} + y_r p_{y_r}$）：
$$\text{GFL}(p_{y_l}, p_{y_r}) = -|y - \hat{y}|^\beta \big((y_r - y)\log(p_{y_l}) + (y - y_l)\log(p_{y_r})\big)$$
- **Focal Loss (FL)** 只是 GFL 的特例：$y \in \{0, 1\}, y_l = 0, y_r = 1, \beta = \gamma$；
- **QFL** 是 $y_l = 0, y_r = 1$ 的特例；
- **DFL** 是 $\beta = 0, y_l = y_i, y_r = y_{i+1}$ 的特例。

---

## 主要實驗結果（COCO 基準）
- **消融實驗（ATSS ResNet-50 骨幹，`minival`）**：
  - ATSS 基線：$39.2$ AP（19.4 FPS）
  - +QFL：$39.9$ AP（+0.7 AP，19.4 FPS）
  - +DFL：$39.5$ AP（+0.3 AP，19.4 FPS）
  - **+QFL+DFL (完整 GFL)**：**$40.2$ AP**（**+1.0 AP**，維持相同 **19.4 FPS**，零推論延遲）。
- **SOTA 對比（COCO test-dev）**：
  - GFL (ResNet-101)：**45.0 AP**（超越 ATSS R-101 43.6 AP、FCOS R-101 41.5 AP、RetinaNet R-101 39.1 AP）。
  - GFL (ResNeXt-101-32x4d-DCN)：達到 **48.2 AP**。

---

## 知識庫關聯
- **推廣與演進來源**：[[2017-focal-loss]]
- **關聯概念**：[[generalized-focal-loss]]、[[focal-loss]]
