---
title: L4DR
type: entity
tags:
  - multimodal
  - autonomous-driving
  - 4d-radar
  - lidar-radar-fusion
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# L4DR

## 基本定位
**L4DR**（*LiDAR-4DRadar Fusion for Weather-Robust 3D Object Detection*）是由廈門大學等團隊於 AAAI 2025 提出的**兩階段 LiDAR 與 4D 毫米波雷達推論期感測器融合框架**（見 [[2025-l4dr]]），旨在解決惡劣氣候下 LiDAR 嚴重衰退與 4D 雷達雜訊過大難以融合的兩大難題。

- **開源實作**：https://github.com/ylwhxht/L4DR

---

## 兩階段融合架構
1. **第一階段：3D 資料融合編碼器**
   - **FAD（前景感知去噪）**：以 [[pointnet-plus-plus|PointNet++]] 預測雷達點的前景置信度，過濾多路徑雜訊點（門檻 $\tau = 0.2$）；
   - **MME（多模態編碼器）**：在同一柱狀空間（Pillar）內進行雙向交叉特徵傳播，將雷達的速度、RCS 與 LiDAR 的反射率幾何資訊無縫互補。
2. **第二階段：$\{\text{IM}\}^2$ 骨幹與 MSGF 融合**
   - **$\{\text{IM}\}^2$ 並行骨幹**：保持純 LiDAR、純雷達與跨模態融合三條特徵流獨立抽取；
   - **MSGF（多尺度門控融合）**：利用融合特徵計算動態門控權重，在濃霧與大雪等氣候下自動抑制退化模態的無效特徵。

---

## 代表性指標
- **K-Radar 資料集（真實雨雪霧天候）**：Sedan $AP_{\text{3D}}$ 達到 **53.5%**（超越先前最佳 3D-LRF 45.2%）。
- **VoD-Fog（Level 4 極端濃霧）**：Moderate 3D mAP 達到 **28.9%**（相較純 LiDAR 8.92% 提升超過 3 倍）。

---

## 參考來源
- [[2025-l4dr]]
- [[lidar-radar-fusion]]
- [[4d-radar-denoising]]
