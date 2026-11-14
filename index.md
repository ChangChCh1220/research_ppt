# LLM 研究知識庫索引（Index）

> 本索引為內容導向的 Wiki 全域目錄。進行 Query 時請先讀取本頁定位相關筆記；每次 Ingest 或回寫 Query 結果後，務必同步更新本頁。

## 總覽（Overview）
- [[wiki/overview|LLM 研究知識庫總覽]] — 本知識庫的研究主線、技術版圖與演進綜述。（更新：2026-11-12｜來源數：11）

## 概念（Concepts）
- [[wiki/concepts/4d-radar-denoising|4D 毫米波雷達點雲去噪]] — 探討 4D 雷達旁瓣雜訊成因，對比傳統濾波（MLS/ROR/SOR）、點層級去噪（FAD）、張量稀疏化（4DSRT）與跨模態遮罩/特徵蒸餾。（更新：2026-11-12｜來源數：5）
- [[wiki/concepts/cross-modal-supervision|跨模態監督學習]] — 訓練期引入高精度輔助模態（如 LiDAR）提供遮罩或特徵蒸餾、推論期維持單模態部署的非對稱學習範式，並深入探討感測器物理失配與負面監督陷阱。（更新：2026-11-12｜來源數：3）
- [[wiki/concepts/lidar-radar-fusion|LiDAR 與雷達多模態融合]] — 車載推論期常態部署多感測器，早期 Pillar 雙向幾何傳播（MME）與後期多尺度門控（MSGF）在惡劣天候下的對決與抗衰減機制。（更新：2026-11-12｜來源數：3）
- [[wiki/concepts/4d-radar-tensor|4D 雷達張量與稀疏化]] — 深入解析 4DRT 原始高維張量（都卜勒-距離-方位-俯仰）及 4DSRT 離線預計算，證實 5% 最佳能量保留密度。（更新：2026-11-12｜來源數：2）
- [[wiki/concepts/pointnet-plus-plus|PointNet++ 與階層式點雲特徵學習]] — 集合抽象層（FPS 採樣 + 球查詢 + 局部 PointNet）與 MSG/MRG 隨機丟棄機制，以及在雷達去噪（HPNet、FAD）中的核心應用。（更新：2026-11-12｜來源數：3）
- [[wiki/concepts/focal-loss|Focal Loss（聚焦損失）]] — 單階段目標偵測中極端前景-背景不平衡的動態調變因子 $(1-p_t)^\gamma$ 與先驗偏置初始化 $\pi$。（更新：2026-11-12｜來源數：3）
- [[wiki/concepts/generalized-focal-loss|Generalized Focal Loss（廣義聚焦損失）]] — 將 Focal Loss 擴展至任意連續目標，包含品質聚焦損失（QFL）與分佈聚焦損失（DFL）。（更新：2026-11-12｜來源數：2）
- [[wiki/concepts/class-balanced-loss|類別平衡損失與有效樣本數]] — 基於隨機幾何覆蓋理論，以有效樣本數 $E_n = (1-\beta^n)/(1-\beta)$ 解決長尾分佈下的資訊邊際遞減與加權發散。（更新：2026-11-12｜來源數：2）
- [[wiki/concepts/adamw|AdamW 與解耦權重衰減]] — 證明自適應梯度優化器中 $L_2$ 正則化與權重衰減之數學不等價性，正規化權重衰減與餘弦退火熱重啟。（更新：2026-11-12｜來源數：1）

## 實體（Entities）
- [[wiki/entities/corenet|CORENet]] — 可即插即用於體素 3D 偵測器（CasA-V / Voxel R-CNN）的 4D 雷達去噪框架，核心為 HMSD-Net 與 KDTree 遮罩監督。（更新：2026-11-12｜來源數：1）
- [[wiki/entities/radardistill|RadarDistill]] — nuScenes 上藉由 CMA 稠密化、AFD 活化區分流與 PFD 通道正規化實現 LiDAR-to-Radar BEV 特徵蒸餾的框架。（更新：2026-11-12｜來源數：1）
- [[wiki/entities/l4dr|L4DR]] — 兩階段 LiDAR 與 4D 雷達惡劣天候融合架構（FAD 前景去噪 + MME 雙向特徵傳播 + $\{\text{IM}\}^2$ 骨幹 + MSGF 門控）。（更新：2026-11-12｜來源數：1）
- [[wiki/entities/retinanet|RetinaNet]] — FAIR 驗證 Focal Loss 的經典單階段密集目標偵測器，基於 ResNet-FPN 與先驗偏置初始化。（更新：2026-11-12｜來源數：1）
- [[wiki/entities/dual-radar-dataset|Dual-Radar 資料集]] — 包含 Arbe Phoenix 4D 雷達與 LiDAR 的自駕多模態資料集與 KITTI 風格評測基準設定。（更新：2026-11-12｜來源數：1）
- [[wiki/entities/k-radar-dataset|K-Radar 資料集]] — 包含 35K 幀 4D 雷達高維張量（4DRT）與雙 LiDAR、涵蓋 7 種惡劣天候的基準資料集與 4DSRT 格式。（更新：2026-11-12｜來源數：2）
- [[wiki/entities/vod-dataset|View-of-Delft（VoD）與 VoD-Fog 資料集]] — 包含 5 掃描累積 4D 雷達點雲與 64 線 LiDAR 之市區自駕資料集，以及 5 級模擬濃霧基準。（更新：2026-11-12｜來源數：1）

## 比較與分析（Comparisons）
- [[wiki/comparisons/cross-modal-supervision-vs-fusion|跨模態監督 vs. 推論期多模態融合橫向評比]] — 比較「訓練期借用 LiDAR、推論純雷達」與「推論期雙模態並行融合」在硬體成本、延遲、天候強健度與監督盲區上的權衡。（更新：2026-11-12｜來源數：5）
- [[wiki/comparisons/imbalanced-and-focal-loss-evolution.md|類別不平衡與聚焦損失演進史]] — 從 Focal Loss (2017) 到 Class-Balanced Loss (2019) 再到 Generalized Focal Loss (2020) 的數學泛化與長尾處理脈絡。（更新：2026-11-12｜來源數：3）
- [[wiki/comparisons/radar-representations-tensor-vs-pointcloud.md|4D 雷達張量 vs. 雷達點雲表徵對比]] — 4D 雷達點雲（RPC, 經硬體 CFAR 截斷）與 4D 雷達高維張量（4DRT / 4DSRT）在資訊保留、儲存負擔與運算效率之全方位對比。（更新：2026-11-12｜來源數：4）
- [[outputs/notes/2026-11-14-algorithmic-enhancements-guide|專題分析：演算法深入改良指引]] — 針對 Version06 功率侵蝕、長尾壓制、都卜勒混淆與 3D 高度脫節，融合 EQL v2、3D-DIoU、VoD 自車補償與平台曲線。（更新：2026-11-14｜來源數：5）
- [[outputs/notes/2026-11-14-supplementary-experiments-plan|專題分析：應該要補充的實驗清單]] — 落地 Table 4-6 六大優先級、15 序列盲測協議、物件級真值匹配（FN/FP）誤差歸因診斷與 8 大方法對比大表。（更新：2026-11-14｜來源數：5）

## 來源摘要（Sources）
- [[wiki/sources/2025-corenet|CORENet：以 LiDAR 監督的跨模態 4D 雷達去噪網路（2025）]] — Liu et al. (2025) 論文摘要，提出訓練期用 LiDAR 遮罩監督、推論期純 4D 雷達的 HMSD-Net 去噪框架。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2024-radardistill|RadarDistill：藉由 LiDAR 特徵知識蒸餾提升雷達 3D 目標偵測（2024 / 2025）]] — Bang et al. (CVPR 2024 / 2025) 論文摘要，提出 CMA 稠密化、AFD 活化區加權與 PFD 提案正規化蒸餾。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2025-l4dr|L4DR：惡劣天候強健的 LiDAR 與 4D 雷達融合 3D 目標偵測（2025）]] — Huang et al. (AAAI 2025) 論文摘要，提出 FAD 前景去噪、MME 雙向特徵交互、$\{\text{IM}\}^2$ 骨幹與 MSGF 門控融合。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2022-radar-occupancy-prediction|在 LiDAR 監督下保留長距離感知與穿透特性的雷達占用網格預測（2022）]] — Kung et al. (RA-L 2022) 論文摘要，提出雷達功率過濾、極座標滑動窗口推論與非對稱 Tversky 損失。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2022-k-radar|K-Radar：多樣天候條件下的 4D 雷達 3D 目標偵測基準資料集（2022）]] — Paek et al. (NeurIPS 2022) 論文摘要，發布 35K 幀 4DRT 資料集與 RTNH 稀疏卷積基線，證實高度維度重要性。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2023-enhanced-k-radar|Enhanced K-Radar：最佳密度縮減提升 4D 雷達張量偵測效能與可用性（2023）]] — Paek et al. (2023) 論文摘要，提出離線 4DSRT 加速 17.1 倍，證實 5% 能量密度最佳，推翻越稠密越好的直覺。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2017-pointnet-plus-plus|PointNet++：度量空間中點集的深層階層式特徵學習（2017）]] — Qi et al. (NeurIPS 2017) 論文摘要，提出集合抽象層（FPS + 球查詢 + 局部 PointNet）與 MSG/MRG 隨機丟棄機制。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2017-focal-loss|Focal Loss：密集目標偵測中的類別不平衡動態縮放損失（2017）]] — Lin et al. (ICCV 2017) 論文摘要，提出動態調變因子 $(1-p_t)^\gamma$、先驗偏置初始化 $\pi$ 與 RetinaNet 單階段架構。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2020-generalized-focal-loss|Generalized Focal Loss：密集目標偵測中的品質與分佈邊界框學習（2020）]] — Li et al. (NeurIPS 2020) 論文摘要，將 Focal Loss 推廣至任意連續區間，提出 QFL 與 DFL 統一框架。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2019-class-balanced-loss|基於有效樣本數的類別平衡損失（2019）]] — Cui et al. (CVPR 2019) 論文摘要，提出隨機幾何覆蓋理論、有效樣本數 $E_n = (1-\beta^n)/(1-\beta)$ 與 CB Focal Loss。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2019-adamw|解耦權重衰減正規化（AdamW / SGDW，2019）]] — Loshchilov & Hutter (ICLR 2019) 論文摘要，證明自適應梯度優化器中 $L_2$ 正則與權重衰減不等價，提出解耦更新與正規化權重衰減。（更新：2026-11-12｜來源數：1）
- [[wiki/sources/2020-equalization-loss|Equalization Loss：長尾目標識別中的負梯度均衡損失（2020）]] — Tan et al. (CVPR 2020) 論文摘要，提出忽略前景目標間負梯度之門控機制，解決轎車與大車 21.9:1 長尾失衡。（更新：2026-11-14｜來源數：1）
- [[wiki/sources/2021-equalization-loss-v2|Equalization Loss v2：長尾目標偵測的自適應梯度平衡（2021）]] — Tan et al. (CVPR 2021) 論文摘要，基於累積正負梯度比率之動態調節，實現無超參數長尾自適應平衡。（更新：2026-11-14｜來源數：1）
- [[wiki/sources/2020-distance-iou-loss|Distance-IoU 與 Complete-IoU 邊界框回歸損失（2020）]] — Zheng et al. (AAAI 2020) 論文摘要，引入歸一化中心點距離與長寬比一致性懲罰，改善 3D 框定位與高度回歸。（更新：2026-11-14｜來源數：1）
- [[wiki/sources/2022-voxel-4d-radar-detection|基於體素的 4D 雷達點雲 3D 目標偵測（2022 / 2023）]] — Li & Kong et al. (IEEE TVT 2023) 論文摘要，系統評估 4D 雷達點雲 0.4m 體素化、稀疏卷積與功率權重機制，為 RTNH 奠定參數基石。（更新：2026-11-14｜來源數：1）
- [[wiki/sources/2022-view-of-delft-4d-radar|View-of-Delft（VoD）：4D 雷達多類別 3D 目標偵測基準（2022）]] — Palffy et al. (IEEE RA-L 2022) 論文摘要，提供 5 掃描累積與多類別長距離雷達評測標準，為跨資料集泛化提供權威對照。（更新：2026-11-14｜來源數：1）
