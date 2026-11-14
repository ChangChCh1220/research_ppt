---
title: "解耦權重衰減正規化（AdamW / SGDW，ICLR 2019）"
type: source
tags:
  - architecture
  - pretraining
  - optimization
  - adamw
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# 解耦權重衰減正規化（AdamW / SGDW）

## 書目與基本資訊
- **論文標題**：*Decoupled Weight Decay Regularization*
- **作者與機構**：Ilya Loshchilov, Frank Hutter（德國弗萊堡大學 University of Freiburg）
- **發表日期 / 編號**：ICLR 2019 (Oral) / arXiv v3 2019-01-04｜`arXiv:1711.05101v3 [cs.LG]`
- **開源程式碼**：https://github.com/loshchil/AdamW-and-SGDW
- **原始來源檔案**：`raw/papers/2019-adamw-decoupled-weight-decay.pdf`

## 一句話總結（TL;DR）
[[adamw|AdamW]] 揭示了在 Adam 等自適應梯度優化器中 **$L_2$ 正則化與權重衰減（Weight Decay）在數學上完全不等價** 的事實：主流深度學習框架誤將 $L_2$ 正則化當作權重衰減，導致歷史梯度較大的權重未獲得足夠的衰減正則。透過將權重衰減從梯度自適應更新中**解耦（Decoupled）**，AdamW 在 CIFAR-10 與 ImageNet 上降低了 15% 測試誤差，徹底縮小了 Adam 與帶動量 SGD 在泛化能力上的差距，並成為當代大語言模型（LLM）預訓練的標準優化器。

---

## 核心問題與數學不對等性證明

### 1. 標準 SGD 下的等價性（命題 1）
對於標準 SGD（學習率 $\alpha$），在目標函數加入 $L_2$ 正則項 $\frac{\lambda'}{2}\|\theta\|_2^2$ 產生的梯度更新：
$$\theta_{t+1} = \theta_t - \alpha \left(\nabla f_t(\theta_t) + \lambda'\theta_t\right) = (1 - \alpha\lambda')\theta_t - \alpha \nabla f_t(\theta_t)$$
這與原始權重衰減（Hanson & Pratt, 1988）$\theta_{t+1} = (1 - \lambda)\theta_t - \alpha\nabla f_t(\theta_t)$ 在令 $\lambda' = \frac{\lambda}{\alpha}$ 時**完全等價**。

### 2. 自適應梯度演算法下的根本不等價（命題 2 & 3）
對於具備非純量預條件矩陣 $M_t \neq kI$（如 Adam、AdaGrad、RMSProp）的優化器：
- **Adam 搭配 $L_2$ 正則化（傳統寫法）**：
  $$g_t \leftarrow \nabla f_t(\theta_{t-1}) + \lambda \theta_{t-1}$$
  $$\theta_t \leftarrow \theta_{t-1} - \alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$
  正則化梯度 $\lambda \theta_{t-1}$ 被除以了歷史二階矩 $\sqrt{\hat{v}_t}$。這意味著：**歷史梯度特別大、頻繁更新的權重，所受到的相對權重衰減反而被大幅縮小！**
- **AdamW（解耦權重衰減）**：
  $$g_t \leftarrow \nabla f_t(\theta_{t-1})$$
  $$\theta_t \leftarrow \theta_{t-1} - \eta_t \left(\alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon} + \lambda \theta_{t-1}\right)$$
  只有原始損失梯度經過二階矩適應；所有權重均以一致的衰減率 $\lambda$ 進行等比例縮減。

---

## 關鍵演算法與超參數排程

### 1. 超參數解耦效應
在傳統 Adam ($L_2$) 中，最佳學習率 $\alpha$ 與正則化係數 $\lambda$ 呈現對角線耦合，難以單獨微調；解耦為 AdamW 後，最佳 $\alpha$ 與 $\lambda$ 在超參數空間呈現獨立的正交軸對齊網格，調參穩定性大幅提高。

### 2. 正規化權重衰減（Normalized Weight Decay）
為使最優 $\lambda$ 不隨訓練總步數 $T$、批次大小 $b$ 與樣本總數 $B$ 劇烈變動，建議採用正規化縮放公式：
$$\lambda = \lambda_{\text{norm}} \sqrt{\frac{b}{B T}}$$
實驗證明最佳 $\lambda_{\text{norm}}$（如 $0.025 \sim 0.05$）可直接跨資料集（CIFAR-10 與長度 24 倍的 ImageNet32x32）無痛轉移。

### 3. 餘弦退火與熱重啟（AdamWR）
搭配排程倍率 $\eta_t = 0.5 + 0.5\cos(\pi T_{\text{cur}} / T_i)$，在第 1 次重啟時即可加速達到收斂標準，訓練速度較一般 Adam 提升達 **10 倍**。

---

## 主要實驗結果
- **測試誤差相對降低 15%**：在 CIFAR-10（Top-1）與 ImageNet32x32（Top-5）的多種訓練時長（100 至 1800 epochs）下，AdamW 普遍較傳統 Adam 取得 15% 的相對測試誤差降低。
- **泛化本質而非單純加速**：在相同訓練損失（Training Loss）下，AdamW 取得明顯更低的測試誤差，證明其解耦正則化改善了泛化曲面。
- **神經科學與 NLP 廣泛驗證**：在 Transformer 語言模型基準（Radford et al., 2018）與腦電圖分類（Völker et al., 2018）中，AdamW 皆顯著擊敗傳統 Adam。

---

## 知識庫關聯
- **在 LLM 與 3D 視覺中的通用地位**：從現代 Transformer（GPT、LLaMA、DeepSeek）到 3D 感知網路，AdamW 已成為標準優化器配置。
- **關聯概念**：[[adamw]]
