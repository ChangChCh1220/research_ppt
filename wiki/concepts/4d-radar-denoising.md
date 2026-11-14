---
title: 4D 毫米波雷達點雲去噪（4D Radar Denoising）
type: concept
tags:
  - multimodal
  - autonomous-driving
  - 4d-radar
  - point-cloud-denoising
created: 2026-11-12
updated: 2026-11-12
sources_count: 5
status: active
---

# 4D 毫米波雷達點雲去噪（4D Radar Denoising）

## 定義與定位
**4D 成像毫米波雷達（4D Imaging Radar）** 可同時量測目標的距離（range）、方位角（azimuth）、俯仰角（elevation）與徑向都卜勒速度（Doppler velocity），克服了傳統 3D 雷達缺乏高度資訊的限制，且在雨、霧、暗夜、冰雪等惡劣環境下比相機與 LiDAR 更具強健性（見 [[2025-corenet]]、[[2022-k-radar]]）。

然而，4D 雷達點雲同時具有**高度稀疏（sparse）**與**強烈雜訊（noisy）**兩大缺陷，會顯著拉高 3D 目標偵測的誤檢率（false detection rate）。**4D 雷達去噪**的核心任務即是在保留真實目標結構點雲的同時，抑制雜訊點或雜訊特徵。

---

## 雜訊成因與物理特性
根據雷達訊號成像原理與各文獻分析：
1. **旁瓣能量洩漏（Sidelobe Energy Leakage）**：受限於雷達系統固有解析度，強反射目標周圍會產生旁瓣能量洩漏，形成緊鄰真實目標的偽點雲群（見 [[2025-corenet]]）。
2. **垂直方向散佈**：由於雷達在垂直方向（elevation）解析度較低，雜訊點容易沿垂直切線方向大量分布，使孤立點統計方法難以精準切割。
3. **多路徑反射（Multipath Clutter）**：車體、護欄與地面積水引發的二次/多次反射，即使經過 CFAR 門檻後仍殘留大量偽前景點（見 [[2025-l4dr]]、[[2024-radardistill]]）。
4. **感測器物理失配雜訊**：在跨模態監督中，細小樹枝與草叢會造成 LiDAR 反射但無雷達回波，強制學習會誘發模型在雷達雜訊區產生幻覺（見 [[2022-radar-occupancy-prediction]]）。

---

## 技術路線與演進脈絡

| 技術路線 | 代表方法 | 核心機制 | 優點 | 侷限性 |
|---|---|---|---|---|
| **經典幾何優化** | Moving Least Squares (MLS)、Low-Rank Learning | 局部平面投影與自相似性重建 | 無需大量標註資料 | 原為低雜訊曲面設計，計算迭代複雜，難應付高度不規則雷達雜訊 |
| **訊號統計濾波** | ROR（Radius Outlier Removal）、SOR（Statistical Outlier Removal） | 依固定半徑鄰居數或距離統計分布剔除離群點 | 實作簡單、可有效消除孤立離群點 | 面對具空間群聚性的大範圍旁瓣雜訊效果有限 |
| **點層級語意去噪** | FAD（[[2025-l4dr\|L4DR]]） | 以 [[pointnet-plus-plus\|PointNet++]] 預測雷達點前景機率並設定門檻過濾（$\tau=0.2$） | 去除多路徑雜訊，防止早期融合污染 LiDAR 點 | 需點級別前景真值監督 |
| **高維張量稀疏化** | 4DSRT（[[2023-enhanced-k-radar\|Enhanced K-Radar]]） | 離線 Top-5% 最佳能量池化與都卜勒降維 | 訓練加速 17.1 倍，徹底去除底層底噪 | 需存取未經 CFAR 的原始 4DRT 張量 |
| **跨模態幾何遮罩去噪** | [[corenet\|CORENet]]（HMSD-Net + LiDAR Mask） | 訓練期以 LiDAR 幾何建立體素信心遮罩監督；推論期純雷達運作 | 結合點拓撲與稀疏體素特徵，推論不依賴 LiDAR | 訓練期仍需空間對齊的高精度 LiDAR 點雲 |
| **BEV 特徵層級蒸餾去噪** | [[radardistill\|RadarDistill]]（CMA + AFD + PFD） | 膨脹稠密化雷達特徵，並分區約束高低階 BEV 表徵貼近 LiDAR | 跨越雷達空 Pillar 稀疏障礙，大幅提升 AP | 需高階神經網路蒸餾流程 |

---

## 優缺點與權衡（Trade-offs）
- **去噪 vs. 保留稀疏小目標**：4D 雷達點雲本就稀疏，在行人（Pedestrian）與騎士（Cyclist）等小目標上有效點極少；過度激進的濾波容易誤刪關鍵前景點（[[2025-corenet]] 在 Dual-Radar 上 Pedestrian 3D AP 僅約 4.55%）。
- **空間卷積 vs. 點拓撲保留**：僅用體素稀疏卷積（SConvNet）能擷取局部網格空間資訊，但會損失點與點之間的拓撲關聯；結合 PointNet 式拓撲聚合（HPNet）能提升小目標與整體表現（見 [[2025-corenet]]、[[2017-pointnet-plus-plus]]）。
- **密度並非越高越好**：Enhanced K-Radar 證實保留 **Top-5% 能量** 即可達到最高偵測精度（47.9% 3D AP）；將密度推至 50% 會保留大量地面积雪冰霜散射雜訊，使冰霰天候 AP 崩跌 17%（見 [[2023-enhanced-k-radar]]）。

---

## 參考來源
- [[2025-corenet]]
- [[2024-radardistill]]
- [[2025-l4dr]]
- [[2022-k-radar]]
- [[2023-enhanced-k-radar]]
- [[2022-radar-occupancy-prediction]]
- [[2017-pointnet-plus-plus]]
