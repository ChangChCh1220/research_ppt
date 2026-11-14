---
title: Dual-Radar 資料集
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

# Dual-Radar 資料集

## 定義與定位
**Dual-Radar**（Zhang et al., `arXiv:2310.07602`）是一個針對自動駕駛場景的多模態感知資料集，特色在於同時提供高雜訊挑戰性的 **4D 成像毫米波雷達**（包含 **Arbe Phoenix** 4D radar）與高精度 **LiDAR** 點雲，涵蓋日間、夜間、雨天與黑暗隧道等多樣駕駛環境（見 [[2025-corenet]]）。

---

## 在 [[corenet|CORENet]] 實驗中的使用設定
- **感測器資料**：採用資料集中的 **Arbe Phoenix** 4D 雷達與配對 LiDAR 點雲。
- **資料集劃分（共 9,966 幀）**：
  - 訓練集（Training）：`5,100` 幀
  - 驗證集（Validation）：`2,466` 幀
  - 測試集（Testing）：`2,400` 幀
- **評測指標（KITTI 官方標準）**：
  - 指標形式：同時計算 **3D AP** 與 **BEV（Bird's-Eye View）AP**，並對 Easy / Moderate / Hard 三個難度等級取平均。
  - **IoU 閾值**：
    - `Car`：`50%`
    - `Pedestrian` / `Cyclist`：`25%`

---

## 基準觀察（來自 [[2025-corenet]]）
- 資料集具有顯著的 4D 雷達雜訊水準，使得未經去噪的純體素偵測器（如 `CasA-T` Mean 3D AP 僅 `2.78%`、`CasA-V` 僅 `4.96%`）表現大幅受限；即便搭配傳統統計濾波（`SOR`），Mean 3D AP 亦僅 `16.62%`。
- 對行人（Pedestrian）與騎士（Cyclist）等小目標而言，原始雷達點雲極度稀疏，所有基線模型與去噪後模型的 AP 仍顯著低於車輛類別（最高 `10.11%` Cyclist / `4.55%` Pedestrian 3D AP），顯示該資料集在小目標偵測上仍極具挑戰性。

---

## 參考來源
- [[2025-corenet]]（引用文獻 [4]: *Dual radar: A multi-modal dataset with dual 4d radar for autonomous driving*, arXiv:2310.07602）
