---
title: "Distance-IoU Loss: Faster and Better Learning for Bounding Box Regression"
type: source
tags:
  - llm-research
  - architecture
  - object-detection
  - loss-functions
created: 2026-11-14
updated: 2026-11-14
sources_count: 1
status: active
---

# Distance-IoU Loss: Faster and Better Learning for Bounding Box Regression (AAAI 2020)

- **作者 / 機構**：Zhaohui Zheng, Ping Wang, Wei Liu, Jinze Li, Rongguang Ye, Dongwei Ren（天津大學）
- **發表會議**：AAAI 2020
- **arXiv 連結**：https://arxiv.org/abs/1911.08287
- **原始檔案路徑**：`raw/papers/2020-distance-iou-loss.pdf`

---

## 一句話總結（TL;DR）
指出傳統 IoU 與 GIoU 損失在預測框與真值框無交集或長寬不一致時無法有效衡量中心點距離，提出**距離交並比損失（Distance-IoU, DIoU）**與**完全交並比損失（Complete-IoU, CIoU）**，將中心點歸一化歐氏距離與長寬比一致性納入邊界框回歸損失。

---

## 核心問題與數學公式

### 1. 傳統 IoU 與 GIoU 的局限
- **IoU**：若兩框無交集，$\text{IoU}=0$，梯度完全消失；
- **GIoU**：透過最小閉包區域緩解梯度消失，但在兩框水平或垂直並列時，閉包區域退化，收斂速度極慢，無法直接最小化兩框中心距離。

### 2. Distance-IoU (DIoU) 損失定義
$$\mathcal{L}_{\text{DIoU}} = 1 - \text{IoU} + \frac{\rho^2(\mathbf{b}, \mathbf{b}^{gt})}{c^2}$$
- $\mathbf{b}$ 與 $\mathbf{b}^{gt}$ 分別為預測框與真值框的三維中心坐標；
- $\rho(\cdot)$ 為歐氏距離；
- $c$ 為覆蓋兩框的最小包圍閉包（Minimum Enclosing Box）的對角線長度。

### 3. Complete-IoU (CIoU) 損失定義
$$\mathcal{L}_{\text{CIoU}} = 1 - \text{IoU} + \frac{\rho^2(\mathbf{b}, \mathbf{b}^{gt})}{c^2} + \alpha v$$
其中 $v$ 衡量長寬比的一致性：
$$v = \frac{4}{\pi^2} \left( \arctan\frac{w^{gt}}{h^{gt}} - \arctan\frac{w}{h} \right)^2, \quad \alpha = \frac{v}{(1 - \text{IoU}) + v}$$

---

## 對本研究之啟示與關聯
1. **破解「俯視提升未轉化為 3D 提升」之痛點**：本論文 Version06 歷史 RTNH 結果顯示，轎車 BEV AP 大增 $+8.76\%$，但 3D AP 僅增加 $+0.63\%$，且兩類平均微降 $-0.22\%$。
2. **中心與高度回歸改進**：雷達點雲在 $z$ 軸高度維度本就較為稀疏，RTNH 現行的 Smooth L1 框回歸容易在中心偏差上產生振盪。若在 RTNH 回歸損失中引入 **3D-DIoU / 3D-CIoU**，直接約束三維中心距離與長寬高比例，能顯著提升三維交並比（3D IoU）門檻下的真陽性配對率。
