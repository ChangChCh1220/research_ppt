# 專題研究筆記：如何進一步做得更好——演算法深入改良與前沿架構指引

- **論文出處**：張喬筑碩士論文《光達輔助弱監督之四維雷達可靠度估計與功率抑制》（指導教授：莊子毅 博士，NYCU）
- **主題定位**：針對現行版本四大痛點（功率折損、21.9:1 長尾壓制、都卜勒混淆、3D 高度回歸脫節），融合前沿文獻提出具體可行的五大演算法突破方向。
- **引用新進文獻**：
  - [[wiki/sources/2020-equalization-loss|Equalization Loss (EQL, CVPR 2020)]]
  - [[wiki/sources/2021-equalization-loss-v2|Equalization Loss v2 (EQL v2, CVPR 2021)]]
  - [[wiki/sources/2020-distance-iou-loss|Distance-IoU / Complete-IoU (AAAI 2020)]]
  - [[wiki/sources/2022-voxel-4d-radar-detection|Voxel-based 4D Radar Detection (IEEE TVT 2023)]]
  - [[wiki/sources/2022-view-of-delft-4d-radar|View-of-Delft 4D Radar Benchmark (IEEE RA-L 2022)]]

---

## 一、現有研究四大瓶頸之精確診斷

1. **功率抑制過度折損車體**：連續軟式 W065 公式 $P' = P[0.65 + 0.35s]$ 雖然保證最低 65%，但對可信度 $s \in [0.5, 0.9]$ 的真實車體點依然產生 10%–20% 的非必要削弱，使弱回波跌破體素激活閾值。
2. **$21.9 : 1$ 極端長尾負梯度壓制**：訓練集 5,690 個轎車物件幀在分類頭反向傳播時，持續對 260 個大型車神經元輸出負梯度，導致大型車分類權重範數萎縮。
3. **徑向都卜勒之自車運動混淆**：原始雷達都卜勒未經自車運動補償，路側靜止護欄因角度關係帶有假性徑向速度，在有限訓練預算下干擾了空間幾何學習。
4. **俯視指標與 3D 高度定位脫節**：Version06 先導結果顯示轎車 BEV AP 大增 $+8.76\%$，但 3D AP 僅增 $+0.63\%$，且兩類平均微降 $-0.22\%$，說明純 Smooth L1 框回歸無法有效約束高度與三維中心偏差。

---

## 二、五大前瞻演算法改良方案

### 改良一：Version08 平台式滿額映射曲線（Platform Curve）
$$w(s) = \alpha + (1 - \alpha)\text{clip}\left(\frac{s}{\tau}, 0, 1\right)$$
- **參數設定建議**：門檻 $\tau = 0.20 \sim 0.35$，下限底線 $\alpha = 0.25 \sim 0.35$。
- **物理效益**：只要點的可靠度分數達到 $\tau$（中等可信），即刻享有 **$100\%$ 滿額無損功率**；只有低於 $\tau$ 的低分點才進行線性壓制。徹底消除對大型車與遠距微弱有效回波的能量侵蝕。

### 改良二：引入 EQL / EQL v2 動態梯度均衡機制
- **原理**：借鑒 Tan et al. (CVPR 2020, 2021)，在多類別分類頭中引入門控權重 $w_j$：
  $$L_{\text{cls}} = -\sum_{j \in \{\text{Sedan}, \text{Truck}\}} \left[ y_j \log(p_j) + w_j (1 - y_j) \log(1 - p_j) \right]$$
- **動態策略（EQL v2）**：統計大車累計正負梯度比率 $g_{\text{truck}} = \frac{\sum |\nabla^+|}{\sum |\nabla^-|}$。當轎車樣本過多時，動態將轎車 Proposal 對大車產生的負梯度降權（$w_{\text{truck}} \to 0$），阻止大車權重被淹沒。

### 改良三：下游 RTNH 引入 3D-DIoU / 3D-CIoU 空間回歸損失
- **原理**：借鑒 Zheng et al. (AAAI 2020)，將傳統 Smooth L1 升級為三維中心距離懲罰：
  $$\mathcal{L}_{\text{3D-DIoU}} = 1 - \text{IoU}_{\text{3D}} + \frac{\rho^2(\mathbf{c}_{\text{pred}}, \mathbf{c}_{\text{gt}})}{d_{\text{box}}^2}$$
- **效益**：直接對三維中心點空間偏差 $(\Delta x, \Delta y, \Delta z)$ 施加歸一化懲罰，特別是抑制 $z$ 軸高度估計的漂移，確保俯視框定位精度切實轉化為 3D 體積交並比增益。

### 改良四：自車運動補償（Ego-Motion Compensation）與動態特徵恢復
- **原理**：參考 Palffy et al. (VoD, 2022)，利用車載 IMU/CAN 總線的自車縱向速度 $v_x$ 與橫擺角速度 $\omega_z$，補償靜態幾何的徑向分量：
  $$v_{\text{comp}} = v_{\text{meas}} - (v_x \cos\theta - \omega_z y)$$
- **效益**：消除靜態背景的偽徑向速度，使靜止護欄的都卜勒精確歸零，重新解鎖都卜勒特徵對動態車輛的辨識威力。

### 改良五：體素空間連續性先驗與快取式特徵蒸餾
- **體素特徵分析**：依據 Li & Kong et al. (IEEE TVT 2023)，大車長達 9.5 米，在 0.4 米體素下覆蓋 20 個以上體素網格。可在體素特徵編碼（VFE）階段引入長向連續性先驗，對連續相鄰非空體素進行特徵增強。
- **快取式特徵蒸餾**：針對本機 16GB VRAM 限制，先離線快取光達教師網路（如 SECOND-LiDAR）的中間特徵圖至硬碟，訓練雷達學生模型時直接讀取快取計算 MSE 蒸餾損失，免去同時在顯存中加載兩個龐大骨幹的負擔。
