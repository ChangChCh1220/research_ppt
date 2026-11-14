---
title: RetinaNet
type: entity
tags:
  - architecture
  - focal-loss
  - object-detection
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# RetinaNet

## 基本定位
**RetinaNet** 是由 FAIR 團隊（Lin et al., ICCV 2017）為了驗證 [[focal-loss|Focal Loss]] 所設計的經典單階段密集目標偵測器（見 [[2017-focal-loss]]）。在結構簡約的單階段架構下，首度在 COCO 基準上全面超越當時主流的兩階段偵測器（如 Faster R-CNN w/ FPN）。

---

## 核心網路架構
- **特徵金字塔骨幹（FPN Backbone）**：在 ResNet（或 ResNeXt）頂部構建多尺度特徵金字塔 $P_3 \sim P_7$（通道數 $C=256$），負責跨尺度目標特徵提取。
- **分類與回歸子網路（Classification & Box Subnets）**：
  - 分別由 4 個 $3\times 3$ 卷積層（ReLU 激活）組成；
  - 兩個子網路結構相同但權重不共享；
  - 每個空間位置生成 9 個錨框（Anchor，3 種尺度 $\times$ 3 種長寬比）。
- **先驗偏置初始化（Prior Initialization）**：
  分類子網路最後一層卷積偏置設定為 $b = -\log((1-\pi)/\pi)$（$\pi = 0.01$），搭配 Focal Loss（$\gamma=2.0, \alpha=0.25$）達成極端不平衡下的穩定收斂。

---

## 代表性指標（COCO test-dev）
- **RetinaNet-101-800**：達到 **39.1 AP**（超越 Faster R-CNN w/ FPN 36.2 AP 與 DSSD513 33.2 AP）。
- **RetinaNet-ResNeXt-101-800**：達到 **40.8 AP**。

---

## 參考來源
- [[2017-focal-loss]]
- [[focal-loss]]
