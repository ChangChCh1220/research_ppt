---
title: "在 LiDAR 監督下保留長距離感知與穿透特性的雷達占用網格預測（RA-L 2022）"
type: source
tags:
  - multimodal
  - autonomous-driving
  - cross-modal-supervision
  - occupancy-grid
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# 在 LiDAR 監督下保留長距離感知與穿透特性的雷達占用網格預測

## 書目與基本資訊
- **論文標題**：*Radar Occupancy Prediction with Lidar Supervision while Preserving Long-Range Sensing and Penetrating Capabilities*
- **作者與機構**：Pou-Chun Kung（龔伯均）、Chieh-Chih Wang（王傑智）、Wen-Chieh Lin（林文杰）（國立陽明交通大學 NYCU、工研院 ITRI）
- **發表日期 / 編號**：IEEE Robotics and Automation Letters (RA-L), Accepted Jan 2022｜`arXiv:2112.04282v3 [cs.RO]`
- **原始來源檔案**：`raw/papers/2022-radar-occupancy-prediction-lidar-supervision.pdf`

## 一句話總結（TL;DR）
本論文揭示並解決了使用 LiDAR 監督雷達學習占用網格（Occupancy Grid）時的兩大物理衝突（LiDAR 視距短導致長距離動態物體遺漏、以及雷達具穿透性但 LiDAR 無法穿透導致遮擋物後方訊號被錯誤懲罰），提出 **雷達功率閾值過濾（Power Thresholding）**、**極座標滑動窗口推論（Polar Sliding Window Inference）** 與 **非對稱 Tversky 損失**，在僅有近距離 LiDAR 標註下成功預測超遠距離（$160\text{ m} \sim 500\text{ m}$）與遮擋物後方的物體。

---

## 核心問題與動機（感測器物理失配）
1. **感知距離不對稱（Range Discrepancy）**：
   - 旋轉式 FMCW 毫米波雷達探測距離達 $163\text{ m} \sim 500\text{ m}$；
   - 常用 LiDAR（如 Velodyne HDL-32 / VLS-128）的有效反射點 99% 落在 $50\text{ m} \sim 100\text{ m}$ 以內。若直接用 LiDAR 作為真值，會導致模型將雷達在 $100\text{ m}$ 外探測到的真實物體當成假陽性（FP）壓制。
2. **穿透與物理成像特性差異（Penetration Discrepancy）**：
   - **雷達看不到但 LiDAR 看得到**：細樹枝、草叢等高頻小目標會產生 LiDAR 反射，但雷達回波極弱；若強制雷達學習，會迫使神經網路在雷達雜訊區虛構預測（Hallucination）。
   - **雷達穿透得到但 LiDAR 視距被擋**：毫米波雷達波長較長，能穿透樹籬、雨霧或前車底盤探測遮擋後方車輛；LiDAR 屬於嚴格視距（Line-of-sight）感測器，遮擋後方全為無點雲空缺，導致 LiDAR 監督訊號錯誤懲罰了雷達珍貴的穿透信號。

---

## 關鍵方法與架構

### 1. 訓練資料前處理（Radar Power Filtering）
- 將去除地面後的 LiDAR 點雲投影至 BEV 作為占用網格真值。
- **過濾雷達不可見目標**：剔除對應雷達影像功率強度低於閾值 $p$（Oxford 設 $0.08$，自採資料集設 $0.12$）的 LiDAR 測量點，避免強迫網路在雷達空白區域過度擬合。

### 2. 極座標空間表徵（Polar Representation）
- 傳統笛卡爾直角座標（Cartesian）下，雷達遠距離波束橫向發散，形變嚴重；
- 在**極座標空間（方位角 $\times$ 距離）**下，遠距與近距的物體僅發生方位角維度的尺度縮放（Scale change），卷積神經網路（CNN, 本文使用 U-Net）在極座標下具備極佳的遠距泛化能力。

### 3. 極座標滑動窗口推論（Polar Sliding Window Inference）
- 模型僅在近距離 LiDAR 真值範圍（如 $52.5\text{ m}$）內訓練。
- 推論時，將固定大小的極座標窗口沿「距離軸（Range axis）」以步長 $S = 10.5\text{ m}$ 向外滑動：
  - **擴展超遠距動態目標**：無需依靠靜態子地圖（Submap）累積，即可偵測遠達 $160\text{ m} \sim 500\text{ m}$ 的**運動車輛與建築**。
  - **恢復穿透感知能力**：原本被前方遮擋的目標，在滑動窗口推進後變成了該窗口內的第一個有效回波，使網路能正常預測出被遮擋物體的占用狀態。

### 4. 非對稱 Tversky 損失函數
因遠距離雷達回波自然衰減，為防止模型保守預測而產生過多假陰性（FN），採用 Tversky Loss 提高召回權重：
$$\text{Tversky Loss} = 1 - \frac{\text{TP}}{\text{TP} + \alpha \text{FP} + \beta \text{FN}}, \quad (\alpha + \beta = 1)$$
遠距離推論時設定 $\alpha = 0.4, \beta = 0.6$，大幅緩解超視距漏檢。

---

## 主要實驗結果

### 1. 極座標 vs. 笛卡爾座標遠距離泛化（IoU 比較）
僅在近距離 $n=0$（$0 \sim 17.5\text{ m}$）訓練，並推論至遠距離區域 $n=1 \sim 5$（自採資料集，最遠 $105\text{ m}$）：
- **極座標表徵**：$n=0$ 為 `0.42`，遠距離 $n=1 \sim 5$ 分別為 `0.26 / 0.13 / 0.14 / 0.13 / 0.17`。
- **直角座標表徵**：$n=0$ 為 `0.35`，遠距離驟降至 `0.16 / 0.04 / 0.02 / 0.03 / 0.04`。
- **核心結論**：極座標表徵在未見過的遠距離區域平均獲得 **4.2 倍更高的 IoU**。

### 2. 下游雷達里程計改善（Oxford RobotCar，NDT-based Radar Odometry）
- 未使用占用預測前處理：平移誤差 `0.0783 m/frame`，旋轉誤差 `0.0748 deg/frame`。
- 使用本方法預測之占用網格後：平移誤差降至 **`0.0528 m/frame`（-32.6%）**，旋轉誤差降至 **`0.0692 deg/frame`（-7.5%）**。

---

## 知識庫關聯與啟發
- **對跨模態監督的重要警示**：後續所有使用 LiDAR 監督雷達的架構（如 [[2025-corenet|CORENet]] 用 KDTree 生成二元遮罩、[[2024-radardistill|RadarDistill]] 進行 BEV 特徵蒸餾），都必須意識到 **LiDAR 的物理盲區（長距離衰減、無穿透力、細微結構反光）會反向污染雷達監督訊號**，需透過門檻過濾或距離解耦。
- **關聯概念**：[[cross-modal-supervision]]、[[4d-radar-denoising]]
