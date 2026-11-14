---
title: AdamW 與解耦權重衰減（AdamW & Decoupled Weight Decay）
type: concept
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

# AdamW 與解耦權重衰減（AdamW & Decoupled Weight Decay）

## 定義與核心洞察
**AdamW** 由 Loshchilov & Hutter (ICLR 2019) 提出（見 [[2019-adamw]]），指出了深度學習社群長年存在的關鍵誤區：**在自適應梯度優化器（如 Adam、RMSProp）中，$L_2$ 正則化與權重衰減（Weight Decay）在數學上完全不等價。**

主流框架（如舊版 TensorFlow、PyTorch）直接將 $L_2$ 懲罰項 $\lambda \theta$ 加至損失梯度中，導致歷史梯度較大（頻繁更新）的權重，其正則化衰減項被歷史二階矩 $\sqrt{v_t}$ 嚴重稀釋，破壞了正則化的初衷。

---

## 數學不對等性分析

### 1. 標準 SGD 下的等價性
在標準 SGD 更新式中：
$$\theta_{t+1} = \theta_t - \alpha (\nabla f_t(\theta_t) + \lambda'\theta_t) = (1 - \alpha\lambda')\theta_t - \alpha\nabla f_t(\theta_t)$$
只需設定 $L_2$ 係數 $\lambda' = \lambda / \alpha$，即可精確重現原始權重衰減。

### 2. Adam 自適應矩陣下的不等價
在 Adam 中，二階動量估計 $v_t$ 形成非純量對角預條件矩陣 $M_t = \text{diag}(\sqrt{\hat{v}_t} + \epsilon)^{-1}$：
- **傳統 $L_2$ 正則化（Adam with $L_2$）**：
  $$\theta_t \leftarrow \theta_{t-1} - \alpha \frac{\hat{m}_t(g + \lambda\theta)}{\sqrt{\hat{v}_t(g + \lambda\theta)} + \epsilon}$$
  正則化項被除以了 $\sqrt{v_t}$。因此，頻繁出現、梯度大的特徵權重所受的正則化程度遠小於稀疏權重。
- **解耦權重衰減（AdamW）**：
  $$\theta_t \leftarrow \theta_{t-1} - \eta_t \left(\alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon} + \lambda \theta_{t-1}\right)$$
  梯度適應與權重衰減完全解耦，所有權重均承受均勻、不受梯度歷史縮放的收縮懲罰。

---

## 關鍵技術特點與實務指南
1. **超參數解耦（Hyperparameter Decoupling）**：在傳統 Adam 中，調整學習率 $\alpha$ 必須連帶調整 $L_2$ 係數 $\lambda'$；AdamW 使得最優 $\alpha$ 與最優 $\lambda$ 彼此正交獨立，極大降低調參成本。
2. **正規化權重衰減（Normalized Weight Decay）**：
   $$\lambda = \lambda_{\text{norm}} \sqrt{\frac{b}{B T}}$$
   其中 $b$ 為批次大小，$B$ 為資料總數，$T$ 為總 Epoch 數。此公式可讓同一組最優 $\lambda_{\text{norm}}$（如 $0.025 \sim 0.05$）直接在不同規模資料集上泛化。
3. **餘弦退火與熱重啟（AdamWR）**：結合餘弦學習率排程與週期性熱重啟，進一步加速模型跳出局部極值。

---

## 在大語言模型（LLM）與深度學習中的地位
AdamW 的提出徹底解決了 Adam 在視覺與語言任務上泛化能力遜於帶動量 SGD 的歷史缺憾。如今，從 GPT、LLaMA、DeepSeek 等大語言模型的預訓練，到自駕 3D 目標偵測模型，AdamW 已成為現代深度學習最無可爭議的標準預設優化器。

---

## 參考來源
- [[2019-adamw]]
