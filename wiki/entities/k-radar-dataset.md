---
title: K-Radar 資料集
type: entity
tags:
  - evaluation
  - multimodal
  - autonomous-driving
  - 4d-radar
  - 4d-radar-tensor
created: 2026-11-12
updated: 2026-11-12
sources_count: 2
status: active
---

# K-Radar 資料集

## 定義與規格
**K-Radar**（KAIST Radar Dataset，Paek et al., NeurIPS 2022）是自駕領域最具代表性的大規模多模態 **4D 雷達張量（4DRT）** 基準資料集（見 [[2022-k-radar]]、[[2023-enhanced-k-radar]]）。

- **開源資料庫**：https://github.com/kaist-avelab/k-radar
- **資料規模**：35,000 幀點雲/張量資料（訓練集 17,458 幀 / 測試集 17,536 幀；白天 21,820 幀 / 夜晚 13,174 幀），涵蓋 93,345 個 3D 邊界框與追蹤 ID。
- **全天候覆蓋**：包含 7 種真實駕駛天候——正常晴天（Normal）、陰天（Overcast）、濃霧（Fog）、雨天（Rain）、冰霰（Sleet）、輕雪（Light Snow）與大雪（Heavy Snow，降雪量 $>1\text{ cm/hr}$）。
- **感測器配置**：
  - 4D 毫米波雷達：`RETINA-4ST`（$64 \times 256 \times 107^\circ \times 37^\circ$，都卜勒-距離-方位角-俯仰角）；
  - LiDAR：Ouster OS2-64（240m）+ OS1-128（120m）；
  - 4 組立體相機：ZED2i（共 8 個 RGB 鏡頭）；
  - 定位：RTK-GPS 與雙 6 軸 IMU。

---

## 4DSRT 格式升級與密度標準
原始 4DRT 體積高達 12 TB；KAIST 團隊於 2023 年推出 **Enhanced K-Radar (4DSRT)**：
- 離線完成都卜勒壓縮、直角座標雙線性插值與 Top-$N\%$ 能量池化，形成 4D 稀疏張量 $(X, Y, Z, \text{PW})$，訓練速度提升 **17.1 倍**；
- 證實 **5% 密度** 為最優配置（$AP_{\text{3D}} = 47.9\%$，記憶體 380 MB）。

---

## 相關基線模型
- **RTNH（3D-SCB 稀疏卷積基線）**：利用 3D 子流形稀疏卷積提取 4D 空間特徵，在惡劣冰霰天候下以 $41.5\% AP_{\text{3D}}$ 擊敗 LiDAR PointPillars（$22.7\%$）。
- **L4DR**：於 K-Radar 上進行 LiDAR + 4DRT 融合，Sedan $AP_{\text{3D}}$ 達到 **53.5%**（見 [[2025-l4dr]]）。

---

## 參考來源
- [[2022-k-radar]]
- [[2023-enhanced-k-radar]]
- [[2025-l4dr]]
