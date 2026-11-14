---
title: 跨模態監督 vs. 推論期多模態融合橫向評比（Cross-Modal Supervision vs. Multi-Modal Fusion）
type: comparison
tags:
  - multimodal
  - autonomous-driving
  - cross-modal-supervision
  - lidar-radar-fusion
created: 2026-11-12
updated: 2026-11-12
sources_count: 5
status: active
---

# 跨模態監督 vs. 推論期多模態融合橫向評比

## 綜述與核心哲學差異
在自動駕駛雷達與光學感測器（LiDAR / 相機）的研究中，存在兩大截然不同的技術路線：
1. **跨模態監督 / 知識蒸餾（Cross-Modal Supervision / Knowledge Distillation）**：以 [[corenet|CORENet]]、[[radardistill|RadarDistill]] 與 Kung et al. (RA-L 2022) 為代表。核心理念是**「訓練期藉助多模態教師，推論期回歸單一雷達輕量部署」**。
2. **推論期多模態融合（Runtime Multi-Modal Fusion）**：以 [[l4dr|L4DR]]、RadarNet、BEVFusion 為代表。核心理念是**「車端常態部署多感測器，透過演算法動態互補物理盲區」**。

---

## 橫向維度對比總表

| 比較維度 | 跨模態監督（CORENet / RadarDistill） | 推論期多模態融合（L4DR / RadarNet） |
|---|---|---|
| **推論期感測器需求** | **僅需 4D / 3D 雷達**（100% Radar-Only） | **同時需 LiDAR + 4D 雷達** |
| **車載硬體成本** | **極低**（省去量產級高線數 LiDAR） | **高**（需配備全套車載 LiDAR 與校正系統） |
| **推論計算延遲與功耗** | **輕量低延遲**（如 CORENet 僅多單層 MLP） | **較高**（如 L4DR 延遲 96 ms，約 10.4 FPS） |
| **晴天幾何精度極限** | 中等偏上（受限於雷達原始波長與角解析度） | **極高**（直接利用 LiDAR 緻密點雲） |
| **極端天候（暴雪/濃霧）防護** | **極高**（雷達天然穿透冰雪，無光學表面遮蔽） | **極高**（需仰賴動態門控抑制受損 LiDAR） |
| **訓練資料採集門檻** | 高（需時間同步的雷達-LiDAR 成對資料集） | 高（同左） |
| **監督/融合層級** | 體素遮罩（Smooth L1）或 BEV 特徵蒸餾（AFD/PFD） | 點/柱級早期交互（MME）+ 特徵門控（MSGF） |

---

## 關鍵技術權衡與適用場景分析

### 1. 跨模態監督的適用情境
- **適用場景**：量產乘用車（ADAS L2+ / L3）、成本敏感型機器人、需超遠距（$>150\text{ m}$）與穿透感知之場景。
- **代表性進展**：
  - **CORENet**（見 [[2025-corenet]]）：在 Dual-Radar 上將 CasA-V Car 3D AP 從 14.81% 提至 **49.46%**。
  - **RadarDistill**（見 [[2024-radardistill]]）：在 nuScenes 上 Car AP 達 **54.0%**，超越純單目相機與早期融合模型。
- **潛在陷阱**：LiDAR 物理盲區（短距視距、無法穿透）會對雷達產生負面監督，需施加雷達功率門檻或極座標滑動窗口（見 [[2022-radar-occupancy-prediction]]）。

### 2. 推論期多模態融合的適用情境
- **適用場景**：L4/L5 級無人自駕計程車（Robotaxi）、全天候無人採礦與幹線物流，追求極限安全性與容錯冗餘。
- **代表性進展**：
  - **L4DR**（見 [[2025-l4dr]]）：在濃霧（Level 4）下，純 LiDAR AP 崩跌至 8.92%，而融合系統仍能提供 **28.9%** 的高精度保障；在真實冰霰天候下以 **46.2%** 3D AP 大幅領先純 LiDAR（22.7%）。

---

## 參考來源
- [[2025-corenet]]
- [[2024-radardistill]]
- [[2025-l4dr]]
- [[2022-radar-occupancy-prediction]]
