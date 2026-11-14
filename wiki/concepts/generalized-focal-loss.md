---
title: Generalized Focal Loss（廣義聚焦損失）
type: concept
tags:
  - architecture
  - focal-loss
  - generalized-focal-loss
  - object-detection
  - loss-functions
created: 2026-11-12
updated: 2026-11-12
sources_count: 2
status: active
---

# Generalized Focal Loss（廣義聚焦損失）

## 定義與定位
**Generalized Focal Loss (GFL)** 由 Li et al. (2020) 提出（見 [[2020-generalized-focal-loss]]），是 [[focal-loss|Focal Loss]] 向**連續標籤區間**與**機率分佈建模**的數學廣義化。它解決了單階段密集偵測器中兩個深層矛盾：
1. **訓練與推論的品質估計不一致**（獨立訓練的 Centerness/IoU 分支在推論時與分類分數相乘排序，導致未受抑制的背景負樣本誤排前列）；
2. **狄拉克脈衝框回歸過於僵硬**（傳統座標回歸假設邊界絕對清晰，無法表達遮擋與模糊邊界的真實不確定性）。

---

## 兩大核心分支

### 1. 品質聚焦損失（Quality Focal Loss, QFL）
- **聯合標籤表徵**：將分類向量的正樣本標籤直接設定為該預測框與真值框之連續真實 IoU $y \in (0, 1]$，負樣本為 $y=0$。
- **公式**：
  $$\text{QFL}(\sigma) = -|y - \sigma|^\beta \big((1 - y)\log(1 - \sigma) + y\log(\sigma)\big), \quad (\beta=2)$$
- **優勢**：在全域（包含所有負樣本）上進行端到端聯合訓練，徹底消除單獨品質預測分支產生的假陽性高分噪點。

### 2. 分佈聚焦損失（Distribution Focal Loss, DFL）
- **一般分佈建模**：將邊界框各邊偏移量離散化為 $n+1$ 個單位區間，以 Softmax 輸出機率分佈，期待值為最終座標：
  $$\hat{y} = \sum_{i=0}^n \mathcal{S}_i y_i$$
- **公式**：迫使機率迅速聚焦於最鄰近真值 $y$ 的兩個離散點 $y_i, y_{i+1}$：
  $$\text{DFL}(\mathcal{S}_i, \mathcal{S}_{i+1}) = -\big((y_{i+1} - y)\log(\mathcal{S}_i) + (y - y_i)\log(\mathcal{S}_{i+1})\big)$$

---

## 統一數學理論
GFL 將上述概念統一為在任意連續區間 $[y_l, y_r]$ 上的損失函數：
$$\text{GFL}(p_{y_l}, p_{y_r}) = -|y - \hat{y}|^\beta \big((y_r - y)\log(p_{y_l}) + (y - y_l)\log(p_{y_r})\big)$$
- 當 $y \in \{0, 1\}, y_l=0, y_r=1, \beta=\gamma$ 時，精確還原為原始 [[focal-loss|Focal Loss]]；
- 當 $y_l=0, y_r=1, y \in [0, 1]$ 時，為 QFL；
- 當 $\beta=0, y_l=y_i, y_r=y_{i+1}$ 時，為 DFL。

---

## 參考來源
- [[2020-generalized-focal-loss]]
- [[2017-focal-loss]]
