---
title: "基於有效樣本數的類別平衡損失（CVPR 2019）"
type: source
tags:
  - architecture
  - class-balanced-loss
  - focal-loss
  - loss-functions
  - long-tailed-learning
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# 基於有效樣本數的類別平衡損失

## 書目與基本資訊
- **論文標題**：*Class-Balanced Loss Based on Effective Number of Samples*
- **作者與機構**：Yin Cui, Menglin Jia, Tsung-Yi Lin, Yang Song, Serge Belongie（康乃爾大學 Cornell Univ.、Cornell Tech、Google Brain、Alphabet）
- **發表日期 / 編號**：CVPR 2019 / arXiv 2019-01-16｜`arXiv:1901.05555v1 [cs.CV]`
- **開源程式碼**：https://github.com/richardaecn/class-balanced-loss
- **原始來源檔案**：`raw/papers/2019-class-balanced-loss.pdf`

## 一句話總結（TL;DR）
[[class-balanced-loss|Class-Balanced Loss]] 建立了一個隨機覆蓋幾何理論框架，證明隨樣本數增加，新樣本與既有資料的特徵重疊（資訊邊際效益遞減），提出以 **有效樣本數（Effective Number of Samples）** $E_n = (1 - \beta^n)/(1 - \beta)$ 作為模型無關的重加權權重 $1/E_{n_y}$，套用至 Softmax CE、Sigmoid CE 與 Focal Loss，大幅提升長尾極端不平衡資料集上的辨識精度。

---

## 核心問題與動機
1. **長尾分佈（Long-Tailed Distribution）問題**：真實世界資料集多數類別（尾部）只有極少樣本，少數類別（頭部）佔據絕大多數樣本。
2. **傳統重加權（Re-weighting）與重採樣（Re-sampling）的缺陷**：
   - 過採樣（Over-sampling）重複少數樣本導致嚴重過擬合；
   - 欠採樣（Under-sampling）丟棄大量珍貴樣本；
   - 傳統以**逆類別頻率（Inverse Class Frequency, $1/n_i$）** 重加權在大型真實資料上容易使訓練極度不穩定甚至發散，實務上工程師常不得不使用啟發式的 $1/\sqrt{n_i}$。

---

## 關鍵數學架構與機制

### 1. 有效樣本數（Effective Number of Samples）
將特徵空間總體積建模為 $N$（原型空間容量），每個樣本佔據一個單位鄰域。當已有 $n-1$ 個樣本時，第 $n$ 個新樣本落在先前覆蓋範圍之外的期望機率為 $p = (N - E_{n-1})/N$。
解差分方程式可得有效樣本數的精確公式（命題 1）：
$$E_n = \frac{1 - \beta^n}{1 - \beta}, \quad \text{其中 } \beta = \frac{N - 1}{N} \in [0, 1)$$
- **極限性質**：
  - 當 $\beta = 0$（$N=1$，完全重疊），$E_n = 1$（無重加權）；
  - 當 $\beta \to 1$（$N \to \infty$，無任何重疊），$E_n \to n$（還原為傳統逆類別頻率 $1/n$）。

### 2. 類別平衡通用損失函數（CB Loss）
對於任何既有損失函數 $\mathcal{L}(\mathbf{p}, y)$，以真值類別 $y$ 的有效樣本數倒數進行縮放：
$$\text{CB}(\mathbf{p}, y) = \frac{1}{E_{n_y}} \mathcal{L}(\mathbf{p}, y) = \frac{1 - \beta}{1 - \beta^{n_y}} \mathcal{L}(\mathbf{p}, y)$$

### 3. 與 Focal Loss 的自然結合（CB Focal Loss）
將有效樣本數作為 $\alpha$-balanced Focal Loss 中的平衡權重 $\alpha_t = \frac{1 - \beta}{1 - \beta^{n_y}}$：
$$\text{CB}_{\text{focal}}(\mathbf{z}, y) = -\frac{1 - \beta}{1 - \beta^{n_y}} \sum_{i=1}^C (1 - p_i^t)^\gamma \log(p_i^t)$$
- 同時兼顧了 **樣本難度（透過 $(1-p_i^t)^\gamma$ 抑制簡單樣本）** 與 **類別頻率/資訊冗餘（透過 $1/E_{n_y}$ 平衡長尾）**。

---

## 主要實驗結果

### 1. 長尾 CIFAR-10 與 CIFAR-100（ResNet-32 錯誤率 %）
- **長尾 CIFAR-10（不平衡比例 200:1）**：
  - 標準 Softmax 錯誤率 34.32% $\to$ **CB Softmax 降至 31.11%**；
  - 標準 Focal 錯誤率 34.71% $\to$ **CB Focal ($\beta=0.9999, \gamma=1$) 降至 25.43%**（錯誤率大幅下降近 9%）。
- **長尾 CIFAR-100（不平衡比例 200:1）**：
  - 標準 Softmax 65.16% $\to$ **CB Focal ($\beta=0.9, \gamma=1$) 降至 63.77%**。

### 2. 大型真實長尾資料集驗證（iNaturalist 2017 & 2018，ResNet-50）
- **iNaturalist 2017（5,089 類，不平衡比 435:1）**：
  - Softmax Top-1 錯誤率 45.38% $\to$ **CB Focal 降至 41.92%**（輸入 $320\times320$ 時進一步降至 **38.16%**）。
- **iNaturalist 2018（8,142 類，不平衡比 500:1）**：
  - Softmax Top-1 錯誤率 42.86% $\to$ **CB Focal 降至 38.88%**（輸入 $320\times320$ 時達 **35.84%**）。

---

## 知識庫關聯
- **理論根基與傳承**：繼承並擴展了 [[2017-focal-loss|Focal Loss]] 的先驗偏置初始化與困難樣本加權哲學，提供了數學嚴謹的 $\alpha_t$ 設定準則。
- **關聯概念**：[[class-balanced-loss]]、[[focal-loss]]
