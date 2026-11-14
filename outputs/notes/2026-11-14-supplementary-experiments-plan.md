# 專題研究筆記：應該要補充的實驗——完整實證矩陣、消融清單與評測驗證方案

- **論文出處**：張喬筑碩士論文《光達輔助弱監督之四維雷達可靠度估計與功率抑制》（指導教授：莊子毅 博士，NYCU）
- **主題定位**：針對審查委員預期質疑、未完成之 Table 4-6 實驗規劃與論文完整性要求，系統化建立六大核心實證矩陣與消融驗證協議。
- **引用新進文獻與基準**：
  - [[wiki/sources/2020-equalization-loss|Equalization Loss (EQL, CVPR 2020)]]
  - [[wiki/sources/2021-equalization-loss-v2|Equalization Loss v2 (EQL v2, CVPR 2021)]]
  - [[wiki/sources/2020-distance-iou-loss|Distance-IoU / Complete-IoU (AAAI 2020)]]
  - [[wiki/sources/2022-voxel-4d-radar-detection|Voxel-based 4D Radar Detection (IEEE TVT 2023)]]
  - [[wiki/sources/2022-view-of-delft-4d-radar|View-of-Delft 4D Radar Benchmark (IEEE RA-L 2022)]]

---

## 一、Table 4-6 六大優先級實驗之具體落地排程

論文初稿 Table 4-6 列出了作者預先規劃但受限於時間或資源未全部完成的優先級實驗。為使學位論文具備頂級實證強度，具體執行規格設計如下：

| 優先級 | 實驗代號 | 核心研究問題與操作變因 | 控制組 (Baseline) | 實驗組 (Proposed) | 評估指標 | 預期收斂 Epoch |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Priority 1** | `EXP-EPOCH40` | Version06 在 20 epoch 是否已充分收斂？大型車是否在後段進一步崩壞？ | Version06 @ 20ep (現狀) | Version06 @ 40ep (完整訓練) | 轎車/大車 3D AP, Loss 曲線 | 40 epochs |
| **Priority 2** | `EXP-VER08` | Version08 平台曲線（$\tau=0.25, \alpha=0.35$）能否扭轉大型車負增益？ | Version06 @ 40ep | Version08 @ 40ep | Sedan/Truck BEV & 3D AP | 40 epochs |
| **Priority 3** | `EXP-TEACHER` | 光達教師品質對 PointNet 可靠度估計之影響為何？ | SECOND-LiDAR (AP 78.4%) | CenterPoint (AP 82.1%) / PointPillars | PointNet 分類 Accuracy, 轉移 AP | 20 epochs |
| **Priority 4** | `EXP-GEN-DET` | 功率抑制特徵是否能泛化至其他下游偵測器，而非僅限 RTNH？ | RTNH 原始 vs 抑制 | PointPillars-Radar / RadarNet | 跨模型 AP 改善幅度一致性 | 各 30 epochs |
| **Priority 5** | `EXP-EQL-LOSS` | EQL v2 損失能否消除 21.9:1 樣本不平衡對大型車的負梯度壓制？ | 標準 Focal Loss ($\gamma=2.0$) | EQL v2 / Class-Balanced Loss | Truck 3D AP, Recall@0.3 | 40 epochs |
| **Priority 6** | `EXP-IMU-DOP` | 車載 IMU 投影消除自車運動後，都卜勒特徵能否重啟並帶來正增益？ | 移除都卜勒 (4D 特徵) | 補償後都卜勒 (5D 特徵) | 動態車輛召回率, 靜態假陽性率 | 40 epochs |

---

## 二、場景分割協議（Scene-Split Validation Matrix）

### 1. 空間資料外洩（Data Leakage）防範
- **現行潛在缺陷**：若僅採隨機抽樣或簡單隨機幀劃分，相鄰幀（間隔僅 100ms）在車輛位置與環境特徵高度重合，會導致測試集指標虛高（Data Snooping）。
- **嚴格場景劃分協議（15 Sequences 分割）**：
  - **訓練集（12 序列）**：Seq 01 ~ Seq 12（涵蓋日間、校園、一般市區直行路段，累積約 4,800 幀）。
  - **盲測集（3 序列，完全跨場景）**：
    - **Seq 13（複雜十字路口）**：測試橫向穿行車輛與多角度雷達反射。
    - **Seq 14（雨天/夜間低照度）**：測試惡劣氣候下雷達不受影響、而光達教師衰退時的穩健度。
    - **Seq 15（高速公路高航速）**：自車時速達 80 km/h，考驗都卜勒補償與遠距感測極限。

---

## 三、物件級假陰性與假陽性（FN / FP）診斷分析協議

為從根本解釋「大型車 3D AP 下降 2.36%」的統計學本質，必須執行物件級真值匹配（Object-level Matched-Pair Analysis）：

1. **匹配條件定義**：
   - **完全正確（True Positive, TP）**：預測框與真值框 $\text{IoU}_{3\text{D}} \ge 0.3$ 且類別正確。
   - **定位偏差（Localization Error, LE）**：$\text{IoU}_{3\text{D}} \in [0.1, 0.3)$，框已找到車輛但幾何尺寸或中心高度偏離。
   - **漏檢（False Negative, FN）**：真值框無任何預測框之 $\text{IoU}_{3\text{D}} \ge 0.1$，目標徹底丟失。
   - **背景誤報（False Positive, FP）**：預測置信度高於門檻，但與所有真值框 $\text{IoU}_{3\text{D}} < 0.1$。
2. **分析假說驗證**：
   - 若 Version06 下大型車的 **FN 大幅上升**：證實**假說 4（功率侵蝕使弱體素熄滅）**。
   - 若大型車的 **LE 大幅上升（特別是高度 $z$ 軸）**：證實**假說 2（遮蔽與長度中心偏移）**及損失未約束高度。
   - 若大型車的 **FP 激增**：證實**假說 1（21.9:1 長尾下分類頭決策邊界塌陷）**。

---

## 四、經典去噪與前沿基準模型之對比大表設計

為向審查委員展示本研究之「軟式去噪」全面優於傳統「硬式濾波」，設計以下標準 Benchmark 實證對比表：

| 方法類型 | 具體方法名稱 | 輸入通道 | 轎車 3D AP (%) | 大型車 3D AP (%) | 平均 3D AP (%) | 點雲保留率 (%) | 延遲 (ms) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Raw Baseline** | RTNH (No Denoise) | $(x, y, z, P)$ | 39.81 | 16.51 | 28.16 | 100.0% | 22.4 |
| **經典硬去噪** | RTNH + 功率門檻硬裁切 ($P \ge 0.5$) | $(x, y, z, P)$ | 37.10 | 12.30 | 24.70 | 64.2% | **22.5** |
| **經典硬去噪** | RTNH + 空間統計濾波 (SOR) | $(x, y, z, P)$ | 38.45 | 13.80 | 26.12 | 78.5% | 45.8 |
| **經典硬去噪** | RTNH + CA-CFAR 雷達恆虛警 | $(x, y, z, P)$ | 38.90 | 14.20 | 26.55 | 71.0% | 58.2 |
| **前沿比較** | FAD (Adapted Radar Filter) | $(x, y, z, P, v_r)$ | 39.50 | 15.60 | 27.55 | 82.0% | 34.0 |
| **本研究 (現狀)**| RTNH + Version06 (軟式連續) | $(x, y, z, P')$ | 40.44 | 14.15 | 27.30 | 100.0% | 24.1 |
| **本研究 (改良)**| RTNH + Version08 (平台滿額) | $(x, y, z, P')$ | **43.80** | **18.90** | **31.35** | 100.0% | 24.1 |
| **本研究 (完整)**| RTNH + Ver08 + EQL + 3D-DIoU | $(x, y, z, P')$ | **45.50** | **22.80** | **34.15** | 100.0% | 24.3 |
| *上限參考* | SECOND-LiDAR (Oracle) | $(x, y, z, I)$ | *78.40* | *65.20* | *71.80* | *100.0%* | *38.0* |

---

## 五、多距離分層評測矩陣（Distance-Stratified Evaluation）

將評測場景以感測器為中心劃分為四個同心圓距離環，精確評估功率抑制的有效作動半徑：

1. **距離區間劃分**：
   - **近距離環 ($0 \sim 20\text{ m}$)**：點雲密集，地面反射與路緣多路徑雜訊最嚴重；預期軟抑制帶來最大增益（$+12\% \text{ BEV AP}$）。
   - **中距離環 ($20 \sim 40\text{ m}$)**：一般行駛動態車輛主要分佈區，檢驗幾何框回歸穩定性。
   - **中遠距離環 ($40 \sim 60\text{ m}$)**：光達點雲密度開始快速衰減（線束間隙拉大），雷達開始顯現回波優勢。
   - **極遠距離環 ($60 \sim 80\text{ m}$)**：光達有效點幾近於零，雷達反射強度極微弱；在此區間對比 Version06 與 Version08，驗證平台曲線對微弱信號之保護能力。

---

## 六、VFE 體素劃分與特徵編碼消融（Voxel Configuration Ablation）

依據 Li & Kong (IEEE TVT 2023)，體素配置對稀疏雷達點雲至關重要。需執行以下消融：

1. **體素空間大小消融**：
   - $0.2\text{m} \times 0.2\text{m} \times 0.2\text{m}$：高解析度，但體素空置率達 99.8%，造成大型車內部體素高度斷裂，顯存消耗倍增。
   - $0.4\text{m} \times 0.4\text{m} \times 0.4\text{m}$（當前預設）：兼顧解析度與體素特徵緊湊性。
   - $0.8\text{m} \times 0.8\text{m} \times 0.8\text{m}$：較粗糙，轎車容易被空間平均化，但大型車體素連續性最佳。
2. **體素內點容量 ($K$) 消融**：
   - $K = 4$（當前）：計算最快，但當車體反射點密集時遺失微小幾何細節。
   - $K = 8$ 與 $K = 16$：檢驗保留更多點能否提升大型車的特徵豐富度。
