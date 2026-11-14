---
title: 4D 雷達張量 vs. 雷達點雲表徵對比（4D Radar Tensor vs. Point Cloud）
type: comparison
tags:
  - autonomous-driving
  - 4d-radar
  - 4d-radar-tensor
  - point-cloud-denoising
created: 2026-11-12
updated: 2026-11-12
sources_count: 4
status: active
---

# 4D 雷達張量 vs. 雷達點雲表徵對比

## 綜述與核心差異
在自駕 4D 毫米波雷達感知中，輸入特徵表徵可分為兩大流派：
1. **4D 雷達點雲（Radar Point Cloud, RPC）**：以 [[dual-radar-dataset|Dual-Radar]]、[[vod-dataset|View-of-Delft]] 為代表；
2. **4D 雷達張量（4D Radar Tensor, 4DRT / 4DSRT）**：以 [[k-radar-dataset|K-Radar]]、Enhanced K-Radar 為代表。

---

## 橫向維度詳細對比

| 比較維度 | 4D 雷達點雲（RPC: CORENet / L4DR） | 4D 雷達張量（4DRT / 4DSRT: K-Radar） |
|---|---|---|
| **產生階段** | 雷達 DSP 端執行 CFAR（恆虛警率）門檻過濾後輸出 | 保留 FFT 後的未過濾連續能量張量（功率譜） |
| **資料維度** | 離散點集合 $\{[x, y, z, v_D, i_s]_i\}_{i=1}^N$ | 4D 密集格網 $[N_{\text{Doppler}}, N_{\text{Range}}, N_{\text{Azimuth}}, N_{\text{Elevation}}]$ |
| **儲存與傳輸負擔** | **極小**（每幀數百至數千點，數十 KB） | **龐大**（原始 4DRT 達數十 MB/幀，整套資料集 12 TB） |
| **環境微弱特徵保留** | 差（低反射微弱障礙物易被 CFAR 提早抹除） | **極佳**（保留完整微弱反射與連續能量分佈） |
| **前處理運算需求** | 低（直接體素化或 Pillar 化） | 高（需都卜勒池化、極座標轉直角座標雙線性插值） |
| **常用神經網路架構** | Voxel-based / Point-based（Voxel R-CNN、PointNet++） | 3D-SCB（3D 稀疏卷積）或 2D-DCB |
| **最佳稀疏化實踐** | 依賴神經去噪（如 CORENet HMSD-Net 或 L4DR FAD） | **離線預計算 4DSRT 並保留 Top-5% 能量** |

---

## 關鍵技術權衡洞察
1. **CFAR 的不可逆性**：RPC 一旦在硬體端被 CFAR 截斷，遺失的微弱邊界資訊在後續軟體端便無法恢復；4DRT 保留了原始光譜能量，在極端惡劣天氣下能提供更豐富的空間線索（見 [[2022-k-radar]]）。
2. **4DSRT 解決張量計算痛點**：Enhanced K-Radar 證實無需在線上處理 12 TB 稠密張量，離線抽取 **Top-5% 功率的 4DSRT** 既能完整保留 4D 張量的空間精度（Sedan 3D AP 47.9%），又能將訓練速度提升 **17.1 倍**，並將 GPU 記憶體降至 380 MB（見 [[2023-enhanced-k-radar]]）。

---

## 參考來源
- [[2025-corenet]]
- [[2025-l4dr]]
- [[2022-k-radar]]
- [[2023-enhanced-k-radar]]
