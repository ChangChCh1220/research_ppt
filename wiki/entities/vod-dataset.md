---
title: View-of-Delft（VoD）與 VoD-Fog 資料集
type: entity
tags:
  - evaluation
  - multimodal
  - autonomous-driving
  - 4d-radar
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# View-of-Delft（VoD）與 VoD-Fog 資料集

## 基本資訊
**View-of-Delft (VoD)** 是由台夫特理工大學（TU Delft）團隊建立的多模態自動駕駛感知資料集，重點在於城市複雜路況中的行人、自行車騎士與車輛 3D 目標偵測（見 [[2025-l4dr]]）。

- **資料規模**：8,693 幀同步資料（5,139 訓練幀 / 1,296 驗證幀 / 2,258 測試幀）。
- **感測器組合**：
  - 4D 毫米波雷達：5 掃描累積（5-scan accumulated）4D 雷達點雲；
  - LiDAR：單幀 64 線 LiDAR（Velodyne HDL-64E）；
  - 相機：高畫質前向 RGB 相機。
- **評測區域劃分**：
  - **Driving Corridor（行車廊道）**：自車前方主要車道範圍；
  - **Entire Area（全域範圍）**：包含周遭人行道、路邊停車等全部空間。

---

## 衍生基準：VoD-Fog（模擬濃霧基準）
在 [[2025-l4dr|L4DR]]（AAAI 2025）研究中，作者基於 VoD 驗證集構建了 **VoD-Fog**，用於評測感測器融合演算法在氣候衰減下的強健性：
- 依據光學散射模型對 LiDAR 點雲施加 5 種濃度的模擬大霧衰減：
  - 濃霧等級 $\mathcal{L} \in \{0, 1, 2, 3, 4\}$，霧濃度 $\alpha \in [0.00, 0.03, 0.06, 0.10, 0.20]$；
  - 4D 雷達保持真實毫米波穿透性不予衰減。
- **評測結果亮點**：
  在最嚴重的 Level 4 濃霧下，純 LiDAR 偵測 Moderate 3D mAP 從 70.3% 暴跌至 **8.92%**；而融合架構 L4DR 仍維持 **28.9%** 的高偵測率。

---

## 參考來源
- [[2025-l4dr]]
