---
title: 類別平衡損失與有效樣本數（Class-Balanced Loss & Effective Number of Samples）
type: concept
tags:
  - architecture
  - class-balanced-loss
  - focal-loss
  - loss-functions
  - long-tailed-learning
created: 2026-11-12
updated: 2026-11-12
sources_count: 2
status: active
---

# 類別平衡損失與有效樣本數

## 定義與定位
**Class-Balanced Loss (CB Loss)** 由 Cui et al. (CVPR 2019) 提出（見 [[2019-class-balanced-loss]]），建立了一套基於隨機幾何覆蓋的數學框架，量化指出隨著類別中訓練樣本數 $n$ 的增加，新加入樣本與既有資料特徵重疊的機率增大，其提供的新資訊邊際效益呈幾何級數遞減。

CB Loss 提出以 **有效樣本數（Effective Number of Samples）** $E_n$ 的倒數作為損失重加權權重，解決了傳統以逆類別頻率（$1/n$）重加權在大型長尾資料集上訓練發散或難以收斂的難題。

---

## 核心數學公式
$$E_n = \frac{1 - \beta^n}{1 - \beta}, \quad \text{其中 } \beta = \frac{N - 1}{N} \in [0, 1)$$
- $N$ 代表原型特徵空間的總容量；
- **極限狀態**：
  - 當 $\beta \to 0$ 時，$E_n \to 1$（無重加權，標準交叉熵）；
  - 當 $\beta \to 1$ 時，$E_n \to n$（還原為傳統逆類別頻率 $1/n$）；
  - 實務上超參數 $\beta$ 常取 $0.9, 0.99, 0.999, 0.9999$。

### 類別平衡通用加權
$$\text{CB}(\mathbf{p}, y) = \frac{1}{E_{n_y}} \mathcal{L}(\mathbf{p}, y) = \frac{1 - \beta}{1 - \beta^{n_y}} \mathcal{L}(\mathbf{p}, y)$$

---

## 與 Focal Loss 的整合（CB Focal Loss）
將有效樣本數引入 [[focal-loss|Focal Loss]]，形成完整的長尾密集學習損失：
$$\text{CB}_{\text{focal}}(\mathbf{z}, y) = -\frac{1 - \beta}{1 - \beta^{n_y}} \sum_{i=1}^C (1 - p_i^t)^\gamma \log(p_i^t)$$
- **兩級調控**：$1/E_{n_y}$ 在**類別層級**抵消長尾分佈的頻率不平衡；$(1 - p_i^t)^\gamma$ 在**實例層級**過濾易分類樣本，專注困難樣本。
- 在 iNaturalist 2017/2018 與長尾 CIFAR 上相較基準 Cross-Entropy 帶來 4%~9% 的錯誤率降低。

---

## 參考來源
- [[2019-class-balanced-loss]]
- [[2017-focal-loss]]
