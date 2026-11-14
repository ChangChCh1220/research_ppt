---
title: "K-Radar：多樣天候條件下的 4D 雷達 3D 目標偵測基準資料集（NeurIPS 2022）"
type: source
tags:
  - evaluation
  - multimodal
  - autonomous-driving
  - 4d-radar
  - point-cloud-denoising
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# K-Radar：多樣天候條件下的 4D 雷達 3D 目標偵測基準資料集

## 書目與基本資訊
- **論文標題**：*K-Radar: 4D Radar Object Detection for Autonomous Driving in Various Weather Conditions*
- **作者與機構**：Dong-Hee Paek, Seung-Hyun Kong, Kevin Tirta Wijaya（韓國科學技術院 KAIST 交通移動研究所與機器人學程）
- **發表日期 / 編號**：NeurIPS 2022 (Datasets and Benchmarks Track)｜`arXiv:2206.08171v4 [cs.CV]`
- **開源資料庫與程式碼**：https://github.com/kaist-avelab/k-radar
- **原始來源檔案**：`raw/papers/2022-k-radar.pdf`

## 一句話總結（TL;DR）
[[k-radar-dataset|K-Radar]] 是自駕領域首個提供完整 **4D 雷達張量（4DRT：都卜勒、距離、方位角、俯仰角）** 的大型多模態資料集，涵蓋 35,000 幀資料與 93,345 個 3D 標註框，覆蓋霧、雨、霰、大雪等 7 種極端天候，並提出 **RTNH** 稀疏卷積基線網路，證實了 4D 雷達在雨雪冰霰中遠比 LiDAR 強健，且**高度（俯仰角）資訊**對 3D 空間目標定位不可或缺。

---

## 資料集規格與感測器配置

1. **4D 毫米波成像雷達（RETINA-4ST）**：
   - 安裝於前保險桿格柵（避免引擎蓋多次反射），FOV 為橫向 $107^\circ$、垂直 $37^\circ$（最大探測距離 $118\text{ m}$，10 FPS）。
   - **4DRT 原始張量維度**：$64(\text{Doppler}) \times 256(\text{Range}) \times 107(\text{Azimuth}) \times 37(\text{Elevation})$。
   - 解析度：都卜勒 $0.06\text{ m/s}$、距離 $0.46\text{ m}$、方位角 $1^\circ$、俯仰角 $1^\circ$。
2. **高精度雙 LiDAR 系統**：
   - 遠距 LiDAR：Ouster OS2-64（64 線，探測距離 $240\text{ m}$）。
   - 高解析 LiDAR：Ouster OS1-128（128 線，探測距離 $120\text{ m}$）。
3. **4 組立體相機（ZED2i）**：8 視角前/左/右/後立體相機（$1280 \times 720$，30 FPS）。
4. **定位系統**：RTK-GPS（誤差 $0.025\text{ m} + 1\text{ ppm}$）與雙 6 軸 IMU。

---

## 4DRT 處理流程與 RTNH 基線架構

### 1. 4D 雷達張量處理流程
- 傳統雷達點雲（RPC）經過 CFAR 門檻過濾，遺漏大量微弱但真實的環境結構；
- K-Radar 保留原始高維功率張量：沿都卜勒軸取平均得到 `3DRT-RAE`（$256 \times 37 \times 107$），經 3D 雙線性插值轉換為笛卡爾直角座標 `3DRT-XYZ`。
- **RTNH 稀疏化**：在感興趣區域（RoI）內保留**功率最高的 Top-10%** 測量值作為 3D 稀疏張量 $(X, Y, Z, \text{PW})$。

### 2. RTNH 網路架構
- 採用 **3D-SCB（3D 稀疏卷積塊）**：第 1 層為標準 3D 稀疏卷積（降採樣），其後接續 3D 子流形稀疏卷積（Submanifold Conv），稠密化後以 2D 轉置卷積上採樣並串接，輸出多尺度特徵圖。
- 偵測頭輸出分類與 8 維回歸參數（中心座標、尺寸、$\cos/\sin$ 偏航角），採 Focal Loss + Smooth L1 Loss 進行訓練。

---

## 關鍵實驗發現與量化數據

### 1. 高度資訊對 3D 偵測至關重要（Table 3 & 9）
- **RTNH（含高度 Elevation 3D-SCB）**：$AP_{\text{3D}} = \mathbf{47.44\%}, AP_{\text{BEV}} = \mathbf{58.39\%}$（GPU 記憶體僅需 **421 MB**）。
- **RTN（移除高度維度，ResNeXt-101 2D 卷積）**：$AP_{\text{3D}} = 40.12\%, AP_{\text{BEV}} = 50.67\%$（GPU 記憶體 520 MB）。
- **結論**：保留高度維度使 3D AP 顯著提升 **+7.32%**，同時記憶體消耗更少。

### 2. 惡劣天候下的感測器強健性對決（Table 4，Sedan IoU=0.3）

| 天候場景 | 4D 雷達 RTNH $AP_{\text{3D}}$ (%) | LiDAR PointPillars $AP_{\text{3D}}$ (%) | 4D 雷達表現分析 |
|---|---:|---:|---|
| **正常晴天（Normal）** | 49.9 | **52.3** | LiDAR 具備高解析度微幅領先 |
| **陰天（Overcast）** | **56.7** | 56.0 | 兩者持平 |
| **濃霧（Fog）** | **52.8** | 42.2 | LiDAR 衰減約 10%，雷達不受影響 |
| **雨天（Rain）** | 42.0 | **44.5** | 地面積水造成雷達多路徑反射 |
| **冰霰（Sleet）** | **41.5** | **22.7** | **LiDAR AP 暴跌 29.6%，雷達領先近 1 倍** |
| **輕雪（Light Snow）** | **50.6** | 40.6 | 雷達保持優異表現 |
| **大雪（Heavy Snow）** | **44.5** | **29.7** | **LiDAR 表面積雪結霜失效，雷達高出 14.8%** |

---

## 限制與後續修正
1. **儲存與計算瓶頸**：原始 4DRT 體積高達 **12 TB**，且訓練時每次 Iteration 都在線上進行座標轉換與 Top-10% 排序，導致訓練耗時。
2. **Top-10% 稀疏度的經驗設定**：K-Radar 中設定的 10% 屬於啟發式選取，後續被作者自身發表的 [[2023-enhanced-k-radar|Enhanced K-Radar]] 推翻（證實 5% 才是最佳平衡點）。
3. **視角受限**：雷達僅覆蓋前向 $107^\circ$，未達環視 $360^\circ$。
- **關聯實體**：[[k-radar-dataset]]、[[l4dr]]
- **關聯概念**：[[4d-radar-tensor]]、[[4d-radar-denoising]]
