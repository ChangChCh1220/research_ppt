---
title: "Focal Loss：密集目標偵測中的類別不平衡動態縮放損失（ICCV 2017）"
type: source
tags:
  - architecture
  - focal-loss
  - object-detection
  - loss-functions
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# Focal Loss：密集目標偵測中的類別不平衡動態縮放損失

## 書目與基本資訊
- **論文標題**：*Focal Loss for Dense Object Detection*
- **作者與機構**：Tsung-Yi Lin, Priya Goyal, Ross Girshick, Kaiming He, Piotr Dollár（Facebook AI Research, FAIR）
- **發表日期 / 編號**：ICCV 2017 (Best Student Paper Award) / arXiv v2 2018-02-07｜`arXiv:1708.02002v2 [cs.CV]`
- **開源程式碼**：https://github.com/facebookresearch/Detectron
- **原始來源檔案**：`raw/papers/2017-focal-loss.pdf`

## 一句話總結（TL;DR）
[[focal-loss|Focal Loss]] 揭示了**極端前景-背景類別不平衡（約 1:1000）**是阻礙單階段密集偵測器超越兩階段偵測器的核心瓶頸，提出引入動態聚焦係數 $(1-p_t)^\gamma$ 的 Focal Loss，抑制大量容易分類的背景樣本梯度，並搭配簡約的 **RetinaNet** 單階段網路在 COCO 上一舉超越當時所有頂尖兩階段偵測器（如 Faster R-CNN w/ FPN）。

---

## 核心問題與動機
1. **單階段 vs. 兩階段偵測器的結構性失衡**：
   - 兩階段偵測器（Faster R-CNN）透過第一階段 RPN 將每張影像數萬個候選框快速收縮至 1k~2k，並在第二階段藉由固定正負樣本採樣比（1:3）或 OHEM 消除不平衡；
   - 單階段偵測器（SSD、YOLO）必須密集採樣 $\sim 100\text{k}$ 個候選位置。標準交叉熵（CE）下，大量易分類背景樣本（$p_t \gg 0.5$）累積的微小損失總和主導了總損失與梯度更新，導致訓練嚴重低效與模型退化。
2. **傳統平衡策略的盲點**：
   - $\alpha$-balanced CE 僅平衡正負樣本權重，但無法區分「簡單樣本（Easy Examples）」與「困難樣本（Hard Examples）」。

---

## 數學公式與機制

### 1. 標準交叉熵（CE）與 $\alpha$-balanced CE
定義真值類別 $y \in \{\pm 1\}$ 與預測機率 $p \in [0, 1]$：
$$p_t = \begin{cases} p, & \text{if } y = 1 \\ 1 - p, & \text{otherwise} \end{cases}, \qquad \text{CE}(p_t) = -\log(p_t)$$
$$\alpha\text{-balanced CE}(p_t) = -\alpha_t \log(p_t)$$

### 2. Focal Loss（FL）
引入動態調變因子 $(1 - p_t)^\gamma$（其中 $\gamma \ge 0$ 為可調節聚焦參數）：
$$\text{FL}(p_t) = -\alpha_t (1 - p_t)^\gamma \log(p_t)$$
- 當樣本分類錯誤且 $p_t$ 很小（困難樣本），調變因子接近 1，損失不受影響；
- 當樣本分類正確且 $p_t \to 1$（簡單樣本），調變因子趨近於 0，顯著抑制損失與反向傳播梯度。
- **最佳超參數**：$\gamma = 2.0, \alpha = 0.25$（此設定在 COCO 上較 $\alpha$-balanced CE 高出 **+2.9 AP**）。

### 3. 先驗偏置初始化（Prior Initialization, $\pi$）
針對單階段網路輸出層，將最後一層卷積偏置（Bias）初始化為：
$$b = -\log\left(\frac{1 - \pi}{\pi}\right)$$
設定 $\pi = 0.01$（使得訓練初始時前景預測機率約為 0.01）。此舉徹底防止了訓練初期數萬個背景樣本產生的巨大累計損失導致數值不穩定或發散。

---

## 主要實驗結果（COCO test-dev）

| 偵測器 | 骨幹架構 | AP (%) | $\text{AP}_{50}$ (%) | $\text{AP}_{75}$ (%) |
|---|---|---:|---:|---:|
| Faster R-CNN w/ FPN（兩階段） | ResNet-101 | 36.2 | 59.1 | 39.0 |
| DSSD513（單階段） | ResNet-101 | 33.2 | 53.3 | 35.2 |
| **RetinaNet-101-800（單階段）** | ResNet-101-FPN | **39.1** | **59.1** | **42.3** |
| **RetinaNet-101-800（單階段）** | ResNeXt-101-FPN | **40.8** | **61.1** | **44.1** |

- **對比 OHEM**：在相同條件下，Focal Loss 達到 **36.0 AP**，比最佳配置的 OHEM（32.8 AP）高出 **+3.2 AP**。

---

## 知識庫關聯
- **在 3D 感知領域的廣泛繼承**：
  - [[2025-corenet|CORENet]] 分類損失採用 Focal Loss；
  - [[2025-l4dr|L4DR]] 的 FAD 前景去噪採用 Focal Loss；
  - [[2022-k-radar|K-Radar]] 的 RTNH 偵測頭採用 Focal Loss。
- **後續演進**：
  - [[2019-class-balanced-loss|Class-Balanced Loss]]（引入有效樣本數 $E_n$ 定義 $\alpha_t$）；
  - [[2020-generalized-focal-loss|Generalized Focal Loss]]（將 FL 擴展至連續標籤與預測框分布）。
- **關聯概念**：[[focal-loss]]
- **關聯實體**：[[retinanet]]
