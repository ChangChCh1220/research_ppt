---
title: "CORENet: Cross-Modal 4D Radar Denoising Network with LiDAR Supervision for Autonomous Driving"
authors:
  - Fuyang Liu
  - Jilin Mei
  - Fangyuan Mao
  - Chen Min
  - Yan Xing
  - Yu Hu
arxiv: "2508.13485v1"
date: "2025-08-19"
code: "https://github.com/charlesuv/corenet.git"
source_type: "pdf-chat-paste"
ingested: "2026-11-12"
---

# CORENet: Cross-Modal 4D Radar Denoising Network with LiDAR Supervision for Autonomous Driving

**Authors:** Fuyang Liu$^{1,2}$, Jilin Mei$^1$, Fangyuan Mao$^{1,2}$, Chen Min$^1$, Yan Xing$^3$, and Yu Hu$^{1,2}$
- $^1$ Institute of Computing Technology, Chinese Academy of Sciences, China (`{liufuyang2023, meijilin, maofangyuan23s, mincheng, huyu}@ict.ac.cn`)
- $^2$ University of Chinese Academy of Sciences, China
- $^3$ Beijing Institute of Control Engineering, China (`xingyan@bice.org.cn`)
- **arXiv:** `arXiv:2508.13485v1 [cs.CV] 19 Aug 2025`
- **Code:** https://github.com/charlesuv/corenet.git

---

## Abstract

4D radar-based object detection has garnered great attention for its robustness in adverse weather conditions and capacity to deliver rich spatial information across diverse driving scenarios. Nevertheless, the sparse and noisy nature of 4D radar point clouds poses substantial challenges for effective perception. To address the limitation, we present CORENet, a novel cross-modal denoising framework that leverages LiDAR supervision to identify noise patterns and extract discriminative features from raw 4D radar data. Designed as a plug-and-play architecture, our solution enables seamless integration into voxel-based detection frameworks without modifying existing pipelines. Notably, the proposed method only utilizes LiDAR data for cross-modal supervision during training while maintaining full radar-only operation during inference. Extensive evaluation on the challenging Dual-Radar dataset, which is characterized by elevated noise level, demonstrates the effectiveness of our framework in enhancing detection robustness. Comprehensive experiments validate that CORENet achieves superior performance compared to existing mainstream approaches.

---

## I. Introduction

Environmental perception serves as the fundamental module of autonomous driving systems, providing essential support for downstream modules, including path planning, decision-making, and vehicle control [1]. While cameras and LiDAR currently serve as the primary sensor modalities in most autonomous driving platforms, their performance is not ideal in adverse weather conditions [2]. Meanwhile, the radar has gained significant attention due to better environmental adaptability and reliability attributed to their use of relatively longer wavelengths [3]. In the field of autonomous driving, mainstream radar sensors are mainly divided into two categories: 3D radar and 4D radar [4]. Traditional 3D radar suffers from limited resolution and the absence of elevation measurement, which impede precise object characterization and restrict the deployment in autonomous driving scenarios. In contrast, 4D imaging radar provides enhanced spatial perception through four-dimensional data acquisition (range, azimuth, elevation, and Doppler velocity), delivering comprehensive environmental awareness essential for robust operation in complex driving conditions.

However, point clouds collected by radar exhibit strong noise, which is especially prominent in 4D radar, as shown in Fig. 1. The strong noise increases the false detection rate. Accurately distinguishing valid and noise points in 4D radar point clouds poses a significant challenge for 4D radar perception strategies. In the early stages, plenty of radar denoising methods are proposed for 3D radar. Some traditional optimization-based approaches like Moving Least Squares and low-rank learning [5]–[7] leverage geometric projections and self-similarity but often struggle with complex, noisy, and highly irregular radar data. Worth mentioning, neural network-based methods [8], [9] achieve impressive results in low-noise conditions but face challenges with denser and more variable noise levels, as well as larger-scale point clouds typical in radar data. In radar signal processing, denoising is commonly performed using radius-based [10] or statistics-based [11] methods, which can partially eliminate outlier noise but fail to identify extensive noise distributions.

Upon analyzing the noise characteristics of 4D radar and reviewing the fundamental principles of radar signal imaging [12], [13], we identify that the primary noise source arises from energy leakage in sidelobes surrounding strong targets. This phenomenon is predominantly caused by the inherent limitations in the resolution of radar systems [12]. It is known from [4] that the radar has a low resolution in the vertical direction, which results in the distribution of noise along the vertical tangent direction in Fig. 1. Therefore, improving the perceptual capability of 4D radar requires identifying the points associated with actual objects in the point cloud. Drawing inspiration from recent advancements in multi-modal fusion [14]–[16], we try to improve the detection capability of 4D radar-based models by constraining the distribution of 4D radar data with LiDAR. Some works [2], [17] focus on modal fusion and attempt to mitigate radar's limitations by integrating LiDAR information, but they remain largely unexplored for 4D radar, where noise and sparsity are even more pronounced, limiting their robustness and generalizability in real-world scenarios. We propose utilizing cross-modal learning, which leverages LiDAR point clouds during training to make the detection model implicitly capable of recognizing noise.

In this paper, we introduce a cross-modal 4D radar supervision framework CORENet, which effectively distinguishes noise and valid point clouds by using LiDAR point clouds to align with 4D radar point clouds. Experimental results demonstrate that point clouds supervised across modalities can better fit target objects, thereby enhancing object detection capabilities.

In summary, the contributions can be outlined as follows:
- A hierarchical multi-scale denoising module named HMSD-Net for 4D radar is proposed, which is simple and flexible, allowing seamless integration into a voxel-based object detection network to improve detection accuracy.
- We employ a cross-modal supervision mechanism that utilizes LiDAR data to supervise the training of 4D radar data. Our approach requires LiDAR point clouds in the training stage only, ensuring simplicity and efficiency during the inference stage.
- Experimental results show that this supervised denoising framework enables the radar point cloud to better fit target objects and improves the mean accuracy of the original model by 7%-18% in 3D and BEV views.

---

## II. Related Works

### A. Radar-based Object Detection
Radar-based 3D object detection models adopt backbone networks initially designed for LiDAR detectors, tailored specifically to meet the unique requirements of radar data. RadarPointGNN [18] employs Graph Neural Networks (GNNs) [19] to effectively extract features from sparse radar point clouds through graph representations. KPConv Pillars [20] introduces a hybrid architecture that combines grid-based and point-based approaches for radar-only 3D object detection. Recent studies have increasingly focused on detection techniques that fuse radar with LiDAR or cameras [2], [21]–[25], aiming to leverage radar data to compensate for the limitations of each sensor. RadarNet [2] adopts a voxel-based early fusion and an attention-based late fusion approach to integrate radar and LiDAR data. RCM-Fusion [26] fuses radar and camera data at both the feature and instance levels, maximizing the potential of radar information. CRN [22] leverages radar data to transform camera image features into a bird's-eye view (BEV) and then combines these transformed camera features with radar BEV features using a multimodal deformable attention mechanism.

### B. Radar Denoising
Due to the high noise and excessive sparsity of millimeter-wave radar point clouds, perception solutions based on such data are typically underutilized. As a fundamental challenge, point cloud denoising has garnered significant attention. Initially, more traditional approaches such as Moving Least Squares (MLS) [27], [28] were used to project point sets onto effective planes. However, these classical methods were originally designed for reconstructing noise-free surfaces and are not well-suited for describing complex polygonal objects. Moreover, these traditional approaches often involve complex computations and iterative processes, limiting their broader applicability. In radar signal processing, mainstream denoising methods predominantly employ Radius Outlier Removal (ROR) [10] and Statistical Outlier Removal (SOR) [11]. These approaches are widely used due to their excellent efficacy in mitigating isolated outlier noise while exhibiting limited capability in addressing complex spatial noise distributions.

In recent years, with the rapid advancement of neural networks [29]–[33], deep learning techniques have been extensively applied to point cloud denoising, yielding remarkable results. Point-BERT [8] employs a masked learning strategy to restore incomplete point clouds, enhancing their noise resilience. However, this method primarily handles lower noise levels and is suited for smaller-scale point cloud data. DNCNet [9] uses a U-shaped denoising sub-network to eliminate signal noise, followed by a pre-trained classification sub-network to identify the denoised signal types, but its application is limited to one-dimensional signals with simpler forms and features. Furthermore, there are efforts to integrate LiDAR and radar data, such as RadarNet [2], which fuses LiDAR data with radar data from a bird's-eye view (BEV) and utilizes velocity information to improve object detection accuracy. RadarDistill [16] applies a distillation technique to constrain radar feature representations using LiDAR data. Nevertheless, existing methods have primarily been evaluated on 3D radar systems, while their effectiveness in 4D radar with elevated noise levels remains unexplored.

---

## III. Method

The proposed method introduces a novel denoised framework for radar point clouds in 3D object detection task, using LiDAR point clouds to supervise the denoising process of 4D radar data during training only while eliminating the need for LiDAR data during inference, as shown in Fig. 2.

### A. Hierarchical Multi-Scale Denoising Network
As illustrated in Fig. 3, Hierarchical Multi-Scale Denoising Network (HMSD-Net) integrates multi-scale features from two key components: (1) Hierarchical PointNet (HPNet) for topological feature learning and (2) Sparse ConvNet (SConvNet) for spatial encoding. The multi-scale features are later output through a network named Noise Predictor to obtain noise predictions.

Based on the visualization in Fig. 1 and principles of radar imaging [12], the sidelobe signals distributed around targets contain valuable structural information, which can be effectively enhanced through multi-scale local feature aggregation. This signal reinforcement mechanism improves the detection capability in complex environments by preserving fine-grained spatial relationships while suppressing interference patterns. Voxel convolution can capture local spatial information but fails to preserve topological relationships between points [34].

We draw inspiration from works of graph convolution [30], [35] to design the HPNet. The HPNet applies $k$ encoding layers that progressively extract aggregated features from voxels. Specifically, in downsampling layers, the farthest point sampling (FPS) is applied, followed by $k$-nearest neighbor clustering to aggregate voxels. In propagation layers, the clustered voxels undergo non-linear transformations through cascaded multi-layer perceptrons (MLPs) with shared weights, where the MLP layers propagate features from sparse layers to dense layers. These adjacent connections with skip concatenation capture complex interactions within the latent feature representations $\{h_1, h_2, \dots, h_k\}$.

Concurrently, the SConvNet processes voxel features through a multi-stage architecture comprising submanifold sparse convolution layers [36], batch normalization, and ReLU activations at each hierarchy. Specifically, each processing block applies submanifold sparse convolution to maintain geometric sparsity, followed by normalization and ReLU to stabilize feature distributions. The hierarchical features from SConvNet undergo dimensional alignment via upsampling operations, followed by concatenation to synthesize the composite feature representation $f_{\text{conv}}$.

Given the radar point voxels $f_p = \{p_1, p_2, \dots, p_N\}$, where each voxel $p_i \in \mathbb{R}^d$ encapsulates raw radar information: 3D coordinates $[x, y, z]$, signal intensity $i_s$, and radial Doppler velocity $v_D$, the hierarchical processing is formalized as:

$$h_k = \text{HPNet}(f_p), \quad h_k \in \mathbb{R}^{N \times 128},$$
$$f_{\text{conv}} = \text{SConvNet}(f_p), \quad f_{\text{conv}} \in \mathbb{R}^{N \times 64}. \quad (1)$$

The hierarchical features are concatenated along the channel dimension and passed to the Noise Predictor $P$, an MLP network where the output dimension is set to "1". For each voxel, the output is a prediction value ranging from 0 to 1, which indicates the level of confidence. The higher the value, the more likely it is to be a valid voxel, and vice versa.

$$M_{\text{radar}} = P(h_k \oplus f_{\text{conv}}). \quad (2)$$

$M_{\text{radar}}$ denotes the predicted mask representing positions of valid radar voxel clouds. The mask size is the same as the voxel cloud size.

The feature extracted from the final layer of the SConvNet module is projected into the BEV representation with feature dimension reconstruction. The BEV representation $f_{\text{BEV}}$ is subsequently processed by the 2D backbone network and detection head. Finally, the detection head outputs category prediction $c_{\text{cls}}$ and box prediction $y_{\text{reg}}$. We use the binary cross-entropy loss $\text{BCE}(\cdot)$ for classification and the mean squared error $\text{MSE}(\cdot)$ for regression to supervise the network training:

$$L_{\text{cls}} = \text{BCE}(c_{\text{cls}}, c_{\text{gt}}), \quad L_{\text{reg}} = \text{MSE}(y_{\text{reg}}, y_{\text{gt}}), \quad (3)$$

where $y_{\text{gt}}$ represents the ground truth bounding box, $c_{\text{gt}}$ is the ground truth category of the target area.

### B. Cross-modal Supervision Mechanism
In the training stage, the data of LiDAR and radar are jointly provided as inputs. The LiDAR data is processed through the Mask Generator, which accomplishes cross-modal matching through a KDTree structure. Specifically, given the radar data $P = \{p_i \in \mathbb{R}^3\}_{i=1}^N$ and the LiDAR data $Q = \{q_j \in \mathbb{R}^3\}_{j=1}^M$, the KDTree is constructed on $Q$ with axis-aligned spatial partitioning. At each tree level, the splitting axis alternates cyclically (e.g., $x \to y \to z$) to balance spatial discrimination. For each query point $p_i$, the nearest-neighbor search with a fixed distance is performed on $Q$ to identify valid correspondences:

$$N(p_i) = \{q_j \in Q \mid \|p_i - q_j\|_2 < \tau\}, \quad (4)$$

where $\tau$ is a distance threshold. The binary mask $M_{\text{gt}}$ is then generated as:

$$M_{\text{gt}}(i) = \begin{cases} 1 & \text{if } |N(p_i)| \ge 1 \\ 0 & \text{otherwise} \end{cases}. \quad (5)$$

This ensures that only points in $P$ with at least one correspondence in $Q$ within $\tau$ are retained, effectively aligning cross-modal features through efficient spatial indexing. Meanwhile, the radar voxels are passed through the HMSD-Net, where it generates a predicted mask $M_{\text{radar}}$. The two masks are then compared through the loss function:

$$L_{\text{msk}} = \text{SLL}(M_{\text{radar}}, M_{\text{gt}}), \quad (6)$$

where `SLL` (written `SSL` in text) denotes the Smooth $L_1$ Loss [37]. During the inference stage, LiDAR data is no longer required.

### C. Total Loss Function

$$L_{\text{total}} = \alpha \cdot L_{\text{cls}} + \beta \cdot L_{\text{reg}} + \gamma \cdot L_{\text{msk}}. \quad (7)$$

$L_{\text{total}}$ consists of three parts: the classification loss ($L_{\text{cls}}$), the regression loss ($L_{\text{reg}}$), and the effective voxel loss computed based on the predicted mask and supervised mask ($L_{\text{msk}}$). (Sec. III-C states $L_{\text{cls}}$ utilizes focal loss and $L_{\text{reg}}, L_{\text{msk}}$ employ the Smooth L1 function.) The coefficients $\alpha, \beta, \gamma$ are hyperparameters.

---

## IV. Experiment

### A. Datasets and Metrics
Experiments are conducted on the Arbe Phoenix radar data and LiDAR data from the Dual-Radar dataset [4], comprising 9,966 frames (5,100 training, 2,400 testing, 2,466 validation). Evaluation uses KITTI official AP (averaged across Easy, Moderate, Hard) in BEV and 3D views, with IoU thresholds of 50% for Car and 25% for Pedestrian and Cyclist.

### B. Experimental Setup
- CPU: Intel(R) Xeon(R) E5-2650; GPU: single RTX 6000 (46GB)
- Framework: OpenPCDet based on PyTorch 1.8.1
- Training: 80 epochs, lr = 0.003
- KDTree search threshold: $\tau = 0.5\text{ m}$
- Loss hyperparameters: $\alpha = 1, \beta = 2, \gamma = 50$

### C. & D. Quantitative Comparison on Dual-Radar (Table I)

| Baselines | Car 3D AP (%) | Car BEV AP (%) | Cyclist 3D AP (%) | Cyclist BEV AP (%) | Pedestrian 3D AP (%) | Pedestrian BEV AP (%) | Mean 3D AP (%) | Mean BEV AP (%) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| PointPillars [38] | 31.78 | 38.30 | 0.14 | 0.29 | 0.00 | 0.00 | 10.64 | 12.87 |
| VoxelR-CNN [34] | 35.15 | 39.32 | 0.09 | 0.17 | 0.02 | 0.02 | 11.76 | 13.17 |
| RDIoU [39] | 32.02 | 36.18 | 0.41 | 0.72 | 0.00 | 0.01 | 10.81 | 12.30 |
| CasA-T [40] | 8.27 | 15.05 | 0.07 | 0.11 | 0.00 | 0.00 | 2.78 | 5.05 |
| CasA-V [40] | 14.81 | 16.87 | 0.04 | 0.08 | 0.01 | 0.02 | 4.96 | 5.66 |
| CasA-V + ROR [10] | 41.18 | 45.27 | 3.03 | 3.52 | 0.76 | 0.76 | 14.99 | 16.51 |
| CasA-V + SOR [11] | 46.81 | 51.18 | 2.15 | 2.47 | 0.91 | 1.14 | 16.62 | 18.26 |
| Voxel R-CNN + CORENet | 46.48 | 51.09 | 10.02 | 10.12 | 1.82 | 9.09 | 19.44 | **23.43** |
| CasA-V + CORENet | **49.46** | **54.38** | **10.11** | **10.49** | **4.55** | 4.55 | **21.37** | 23.14 |

### E. Ablation Experiments

**Table II: Ablation Experiments of HMSD-Net**

| HPNet | SConvNet | Car 3D AP (%) | Mean 3D AP (%) | Car BEV AP (%) | Mean BEV AP (%) |
|:---:|:---:|---:|---:|---:|---:|
| ✓ | | 67.07 | 26.64 | 70.51 | 28.08 |
| | ✓ | 65.94 | 24.92 | 69.43 | 27.64 |
| ✓ | ✓ | **67.34** | **27.47** | **71.23** | **29.01** |

**Table III: Ablation Experiments of KDTree Distance**

| KDTree Distance (m) | Car 3D AP (%) | Mean 3D AP (%) | Car BEV AP (%) | Mean BEV AP (%) |
|---:|---:|---:|---:|---:|
| 0.3 | 66.94 | 26.86 | 70.26 | 28.00 |
| 0.5 | **67.34** | **27.47** | **71.23** | **29.01** |
| 0.7 | 66.71 | 25.81 | 69.70 | 27.11 |

**Table IV: Ablation Experiments of MLP Layer in Noise Predictor**

| MLP layers | Car 3D AP (%) | Mean 3D AP (%) | Car BEV AP (%) | Mean BEV AP (%) |
|---:|---:|---:|---:|---:|
| 1 | **67.34** | **27.47** | **71.23** | **29.01** |
| 2 | 66.23 | 26.10 | 69.61 | 27.26 |
| 3 | 67.07 | 26.37 | 68.74 | 26.96 |

### F. Visualization and Analysis
Fig. 4 compares LiDAR, raw radar, and CORENet-denoised radar. Fig. 5 validates denoising across four scenarios: (a) daytime, (b) dark tunnel, (c) rainy day, (d) nighttime.

---

## V. Conclusion
The paper introduced CORENet as a novel denoising method for 4D radar, allowing seamless integration into mainstream voxel-based detection networks, with LiDAR supervision used exclusively during training. Limitations: high-precision LiDAR point clouds remain indispensable during the training phase, and detection performance for smaller objects (pedestrians and cyclists) requires further improvement in future work.
