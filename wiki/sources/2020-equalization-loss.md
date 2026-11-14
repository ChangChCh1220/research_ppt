---
title: "Equalization Loss for Long-Tailed Object Recognition"
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

# Equalization Loss for Long-Tailed Object Recognition (CVPR 2020)

- **作者 / 機構**：Jingru Tan, Changbao Wang, Buyu Li, Quanquan Li, Chen Ouyang, Chen Change Loy, Dahua Lin（商湯科技 SenseTime、香港中文大學 CUHK）
- **發表會議**：CVPR 2020
- **arXiv 連結**：https://arxiv.org/abs/2003.05176
- **原始檔案路徑**：`raw/papers/2020-equalization-loss.pdf`

---

## 一句話總結（TL;DR）
針對長尾目標識別與偵測中「常見頭部類別的正樣本在尾部類別上充當負樣本，產生壓倒性負梯度壓制尾部特徵學習」的根本痛點，提出**均衡損失（Equalization Loss, EQL）**，透過動態忽略背景中的前景干擾負梯度，保護少數類別權重不被抑制。

---

## 核心問題與動機
1. **長尾極端不平衡**：在真實世界資料集中，少數頭部類別擁有絕大多數標註，而尾部類別樣本極其稀缺（例如本研究中轎車與大型車 $21.9 : 1$ 的懸殊差距）。
2. **負梯度壓制效應（Negative Gradient Suppression）**：
   - 當訓練一個頭部類別的正樣本（如轎車）時，在多標籤 Softmax / Sigmoid 分類頭中，該樣本同時作為所有其他尾部類別（如卡車、巴士）的負樣本。
   - 由於頭部樣本數量過多，尾部類別的神經元權重會收到源源不絕的負梯度懲罰，導致分類權重範數（Weight Norm）大幅萎縮，推論時模型對尾部類別信心極低甚至直接漏檢。

---

## 關鍵方法與技術機制

### 1. 均衡損失數學定義
對於類別 $j$ 的 Sigmoid 交叉熵損失，EQL 引入權重調節因子 $w_j$：
$$L_{\text{EQL}} = -\sum_{j=1}^C \left[ y_j \log(p_j) + w_j (1 - y_j) \log(1 - p_j) \right]$$
其中調變權重 $w_j$ 定義為：
$$w_j = 1 - E(r) \cdot T_{\lambda}(f_j) \cdot (1 - y_j)$$

### 2. 核心門控機制
- **$E(r)$ 前景指示函數**：當前 Proposal 是否屬於某個真實前景目標。若是背景雜訊提案，則 $E(r) = 0$（正常計算背景負梯度）；若是另一個前景目標，則 $E(r) = 1$。
- **$T_{\lambda}(f_j)$ 尾部類別判別函數**：若類別 $j$ 的出現頻率 $f_j < \lambda$（低於指定頻率門檻），則視為尾部類別（$T_{\lambda}=1$）。
- **運作本質**：當一個 Proposal 是「轎車」時，它對「大型車」產生的負梯度被強制置零（$w_j = 0$），不再無情壓制大型車的權重！

---

## 對本研究之啟示與關聯
1. **直擊 $21.9 : 1$ 轎車與大車長尾失衡**：本論文訓練集 5,690 個轎車物件幀 vs 260 個大車幀。在傳統訓練下，5,690 幀轎車產生的負梯度對大車造成毀滅性壓制。
2. **Version08 損失改良指引**：在可靠度網路與下游 RTNH 偵測頭中，導入 EQL 門控機制，防止轎車樣本干擾大車類別學習。
