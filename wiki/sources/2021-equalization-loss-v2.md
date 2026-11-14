---
title: "Equalization Loss v2: A New Gradient Balance Approach for Long-tailed Object Detection"
type: source
tags:
  - llm-research
  - post-training
  - object-detection
  - long-tailed
created: 2026-11-14
updated: 2026-11-14
sources_count: 1
status: active
---

# Equalization Loss v2: A New Gradient Balance Approach for Long-tailed Object Detection (CVPR 2021)

- **作者 / 機構**：Jingru Tan, Xiang Kang, Tong Lu, Buyu Li, Dahua Lin（南京大學、商湯科技、香港中文大學）
- **發表會議**：CVPR 2021
- **arXiv 連結**：https://arxiv.org/abs/2012.08548
- **原始檔案路徑**：`raw/papers/2021-equalization-loss-v2.pdf`

---

## 一句話總結（TL;DR）
克服 EQL v1 需要人工手動設定類別頻率門檻 $\lambda$ 的局限，提出**自適應梯度平衡機制（EQL v2）**，直接依據訓練過程中累積的正負梯度統計比率動態調變負梯度，實現全類別自動均衡優化。

---

## 核心演進與架構機制

### 1. EQL v1 的痛點
EQL v1 採用人工硬性劃分頭部與尾部類別（$f_j < \lambda$），在類別頻率連續變化的分佈中缺乏柔性，且難以自動適應不同訓練階段的動態梯度需求。

### 2. EQL v2 的梯度動態平衡公式
EQL v2 統計類別 $j$ 累積接收到的正梯度總量 $G_j^+$ 與負梯度總量 $G_j^-$：
$$g_j^{(t)} = \frac{\sum_{\tau=1}^t |\nabla_{z_j} L^+|}{\sum_{\tau=1}^t |\nabla_{z_j} L^-|}$$
調變權重 $q_j$ 定義為：
$$q_j = \frac{1}{1 + \exp\left(-\gamma (g_j - \mu)\right)}$$
當尾部類別累積負梯度過大而正梯度不足時（$g_j$ 極低），調變因子 $q_j$ 自動趨近於 0，動態抑制多餘的負梯度衝擊；當該類別正負梯度逐漸趨向均衡時，調變因子自適應放開。

---

## 對本研究之啟示與關聯
1. **無超參數自適應均衡**：本論文 Version08 規劃引入有界類別加權，但手動給予權重（如 2.0 或 3.0）容易引發訓練數值不穩定。
2. **EQL v2 梯度引導**：在弱監督可靠度模型與 RTNH 的分類訓練中，利用累積梯度比率自適應調節大車與轎車的反向傳播權重，兼顧平穩收斂與長尾保護。
