---
title: "RadarDistill：藉由 LiDAR 特徵知識蒸餾提升雷達 3D 目標偵測（CVPR 2024 / 2025）"
type: source
tags:
  - multimodal
  - autonomous-driving
  - 3d-radar
  - cross-modal-supervision
  - knowledge-distillation
created: 2026-11-12
updated: 2026-11-12
sources_count: 1
status: active
---

# RadarDistill：藉由 LiDAR 特徵知識蒸餾提升雷達 3D 目標偵測

## 書目與基本資訊
- **論文標題**：*RadarDistill: Boosting Radar-based Object Detection Performance via Knowledge Distillation from LiDAR Features*
- **作者與機構**：Geonho Bang, Kwangjin Choi, Jisong Kim, Dongsuk Kum, Jun Won Choi（韓國漢陽大學 Hanyang Univ.、KAIST、首爾大學 SNU）
- **發表日期 / 編號**：CVPR 2024 / arXiv v3 2025-02-26｜`arXiv:2403.05061v3 [cs.CV]`
- **原始來源檔案**：`raw/papers/2024-radardistill.pdf`

## 一句話總結（TL;DR）
[[radardistill|RadarDistill]] 是一個跨模態知識蒸餾（KD）框架：針對雷達點雲極度稀疏（低階特徵非空 Pillar 僅 LiDAR 的約 10%）與特徵幅值失衡的物理限制，提出 **CMA（特徵稠密化對齊）**、**AFD（區域加權低階特徵蒸餾）** 與 **PFD（通道正規化提案區域高階蒸餾）**，在訓練期將 LiDAR 的豐富 BEV 表徵蒸餾至純雷達偵測器（[[radardistill|PillarNet-18]]），推論時完全無需 LiDAR。

---

## 核心問題與動機
1. **純雷達偵測痛點**：雷達點雲空間解析度低、多路徑反射雜訊多，純雷達 3D 偵測長期受限（例如在 nuScenes 上，LiDAR 偵測 mAP 超過 60%，而雷達基線僅個位數）。
2. **直接進行 LiDAR-to-Radar 特徵蒸餾的失效原因**：
   - **Pillar 極端稀疏不對稱**：雷達低階 BEV 特徵 $F^{(l)}_{\text{rdr}}$ 中非空 Pillar 平均數量僅為 LiDAR 特徵 $F^{(l)}_{\text{ldr}}$ 的約 **10%**；若強行進行像素級（Pixel-wise）全特徵對齊，會迫使雷達在大量空 Pillar 上擬合雜訊。
   - **高階特徵幅值尺度嚴重失衡**：雷達與 LiDAR 高階語意特徵存在巨大數值量級差異，直接計算均方誤差（MSE）會導致蒸餾梯度不穩定。

---

## 關鍵方法與架構

### 1. 骨幹網路與繼承策略（Inheriting Strategy）
- 教師（LiDAR）與學生（Radar）均採用 **PillarNet-18**（以 ResNet-18 為骨幹之 Pillar 架構，權重不共享）。
- 學生雷達骨幹的權重先直接繼承預訓練完成的 LiDAR 骨幹權重，顯著加速訓練收斂。
- 低階與高階 BEV 特徵抽取：
  $$F^{(l)}_{\text{mod}} = \text{SparseEnc}(F^{\text{2D}}_{\text{mod}}), \quad F^{(h)}_{\text{mod}} = \text{DenseEnc}(F^{(l)}_{\text{mod}}), \quad \text{mod} \in \{\text{ldr}, \text{rdr}\}$$

### 2. 跨模態對齊模組（Cross-Modality Alignment, CMA）
- 在低階雷達特徵 $F^{(l)}_{\text{rdr}}$ 進入蒸餾前進行膨脹與稠密化（Densification）。
- 由 **Down Block**（可變形卷積 Deformable Conv 下採樣 + ConvNeXt V2 Block）、**Up Block**（2D 轉置卷積上採樣）與 **Aggregation Module**（通道串接 + $1\times 1$ Conv）組成。
- 透過兩階段下採樣與上採樣（$H/8 \to H/16 \to H/32 \to H/16 \to H/8$），輸出兩組稠密化後的低階雷達 BEV 特徵 $F^{(l_1)}_{\text{rdr}}$ 與 $F^{(l_2)}_{\text{rdr}}$。

### 3. 基於啟動的特徵蒸餾（Activation-based Feature Distillation, AFD）
針對低階特徵，依據非零啟動值分割為活化區（Active Region, $AR$）與非活化區（Inactive Region, $IR$）：
$$AR^{(l_n)} = (M^{(l_n)}_{\text{rdr}} = 1) \land (M^{(l)}_{\text{ldr}} = 1), \quad IR^{(l_n)} = (M^{(l_n)}_{\text{rdr}} = 1) \land (M^{(l)}_{\text{ldr}} = 0)$$
引入動態面積加權係數 $\rho^{(l_n)} = N_{AR^{(l_n)}} / N_{IR^{(l_n)}}$ 平衡兩區梯度貢獻：
$$W^{(l_n)}_{\text{sep},i,j} = \begin{cases} \alpha, & (i,j) \in AR^{(l_n)} \\ \rho^{(l_n)} \cdot \beta, & (i,j) \in IR^{(l_n)} \\ 0, & \text{otherwise} \end{cases}$$
$$L_{\text{AFD}} = \frac{1}{2}\sum_{n=1}^{2} \sum_{c,i,j} W^{(l_n)}_{\text{sep},i,j} \left(F^{(l)}_{\text{ldr},c,i,j} - F^{(l_n)}_{\text{rdr},c,i,j}\right)^2$$
實驗設定 $\alpha = 3\times 10^{-4}, \beta = 5\times 10^{-5}$。

### 4. 基於提案的特徵蒸餾（Proposal-based Feature Distillation, PFD）
針對高階 DenseEnc 特徵，利用雷達分類熱圖 $H^{\text{cls}}_{\text{rdr}}$ 與 GT 高斯熱圖 $H^{\text{cls}}_{\text{GT}}$（閾值 $\sigma = 0.1$）劃分三類關鍵區域：
- **TP**（真陽性區）、**FN**（假陰性區，即雷達遺漏目標）、**FP**（假陽性區，即雷達雜訊偽檢）。
- 區域加權權重（$\lambda_1 = 5, \lambda_2 = 1$）：
  $$W_{\text{proposal},i,j} = \begin{cases} \frac{\lambda_1}{N_{\text{TP}} + N_{\text{FN}}}, & (i,j) \in (\text{TP} \cup \text{FN}) \\ \frac{\lambda_2}{N_{\text{FP}}}, & (i,j) \in \text{FP} \\ 0, & \text{otherwise} \end{cases}$$
- **通道 Softmax 正規化**：解決模態特徵尺度差距問題：
  $$S^{(h_m)}_{\text{mod},c,i,j} = \frac{\exp(F^{(h_m)}_{\text{mod},c,i,j})}{\sum_{k=1}^C \exp(F^{(h_m)}_{\text{mod},k,i,j})}$$
  $$L_{\text{PFD}} = \frac{1}{2}\sum_{m=1}^{2} \sum_{c,i,j} W_{\text{proposal},i,j} \left| S^{(h_m)}_{\text{ldr},c,i,j} - S^{(h_m)}_{\text{rdr},c,i,j} \right|$$

### 5. 總損失函數
$$L_{\text{total}} = L_{\text{det}} + \gamma L_{\text{AFD}} + \delta L_{\text{PFD}}, \quad (\gamma = 5, \delta = 25)$$

---

## 主要實驗結果（nuScenes 測試集）

| 模型 / 模態 | mAP (%) | NDS (%) | mATE (m) | mASE | mAOE (rad) | mAVE (m/s) |
|---|---:|---:|---:|---:|---:|---:|
| Radar-PointGNN（純雷達） | 0.5 | 3.4 | 1.024 | 0.859 | 0.897 | 1.020 |
| KPConvPillars（純雷達） | 4.9 | 13.9 | 0.823 | 0.428 | 0.607 | 2.081 |
| PillarNet* 基線（純雷達） | 8.6 | 34.7 | 0.532 | 0.283 | 0.615 | 0.438 |
| **RadarDistill（純雷達）** | **20.5** | **43.7** | **0.461** | **0.263** | **0.525** | **0.336** |

- **類別亮點**：Car AP 達 **54.0%**（超越純相機 FCOS3D 52.4%、CenterNet 53.6%、以及相機+雷達融合 CenterFusion 50.9%），拖車（Trailer）達 29.5%（原 6.5%）、路障（Barrier）達 42.3%（原 8.1%）。
- **消融亮點**：
  - CMA 使 NDS 從 27.3 升至 29.3；若移除 CMA，整體 NDS 下滑 4.0。
  - PFD 中通道 Softmax 正規化使 Car AP 從 39.9% 巨幅躍升至 **46.1%**（NDS 31.8 → 34.7）。
- **相機-雷達融合（C+R）增益**：將經由 RadarDistill 預訓練/蒸餾後的雷達編碼器接入 BEVFusion，C+R 偵測 mAP 由 38.3% 提升至 **39.6%**（Car AP 65.9% → 67.7%）。

---

## 限制與知識庫關聯
- **限制**：未針對多幀雷達堆疊作進一步的時間維度蒸餾；在小型目標（自行車 Bicycle 僅 0.9% AP、行人 Pedestrian 9.2% AP）上受限於原始 3D 雷達點雲密度，仍有大幅提升空間。
- **與 [[2025-corenet|CORENet]] 差異**：
  - CORENet 是針對 **4D 雷達**，利用 KDTree 生成二元體素遮罩在輸入端進行**去噪（Denoising）**；
  - RadarDistill 針對 **3D 雷達**（nuScenes 5 雷達），在神經網路 **BEV 特徵層級進行知識蒸餾（Knowledge Distillation）**。
- **關聯概念**：[[cross-modal-supervision]]、[[4d-radar-denoising]]
- **關聯實體**：[[radardistill]]
