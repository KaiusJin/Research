# CraneLab 工业质检数据集与公开基线技术报告

更新日期：2026-06-25

## 1. 文档目的

本文档核验 CraneLab 选定的五个工业质检数据集/代码仓库：

1. InsPLAD
2. SteelDefectX
3. SteelBlastQC
4. AeBAD / MMR
5. VISION Datasets / Aoi-overfitting-team

筛选条件为：

- 面向工业质检、机械工程或工业设备巡检；
- 数据规模低于 50,000 张，优先低于 10,000 张；
- 可支持至少两项任务或两种实验设置；
- 公开结果没有在所有主要任务上普遍达到 98%--99%；
- 有公开数据入口、论文和可复现代码仓库。

## 2. 指标口径说明

这些项目使用的评价指标不同，不能都称为“准确率”，也不能直接横向比较。

| 指标 | 含义 | 常见任务 |
|---|---|---|
| Accuracy (Acc) | 全部样本中预测正确的比例 | 分类 |
| mean class Accuracy (mAcc) | 各类别准确率的宏平均 | 类别不均衡分类 |
| Balanced Accuracy | 各类别召回率的平均 | 不平衡分类 |
| AUROC | 不依赖固定阈值的正常/异常区分能力 | 异常检测 |
| Box AP | 多个 IoU 阈值下的检测平均精度 | 目标检测 |
| mAP | 多类别或多数据集 AP 的平均 | 检测、实例分割 |
| PRO | 对每个异常区域计算重叠程度 | 异常定位 |
| IoU | 预测区域和真实区域的交并比 | 分割 |
| F1-max | 在不同阈值中取得的最佳 F1 | 检测、分割 |

因此，本文使用“公开水平”表示论文或官方仓库报告的代表性结果，不把所有数字统称为 Accuracy。

## 3. 总览

| 项目 | 数据规模 | 原生任务 | 代表性公开水平 | 结论 |
|---|---:|---|---|---|
| InsPLAD | 10,607 张 UAV 图像；28,933 个设备实例 | 目标检测、监督故障分类、无监督异常检测 | 72.1% Box AP；95.4% Balanced Accuracy；90.5%/94.34% AUROC | 分类较高，但检测和异常检测仍有空间 |
| SteelDefectX | 7,778 张；25 类；像素掩码和四种文本形式 | 视觉语言分类、分割、跨数据集迁移、检索和文本引导定位 | 94.45% Acc / 93.66% mAcc；分割 88.21% AUROC / 37.49% IoU | 多任务明显未饱和 |
| SteelBlastQC | 1,654 张，二分类 | 监督分类、无监督质量检测、热力图解释 | CCT 95.0% Accuracy；CAE 67.6% Accuracy | 适合小数据监督/无监督对照 |
| AeBAD | 5,570 个图像或视频帧样本 | 图像级异常检测、像素级定位、视频异常检测、域偏移评估 | AeBAD-S：84.7% AUROC / 89.1% PRO | 难度较高，明显未饱和 |
| VISION V1 | 18,422 张；4,165 张有标注；9,837 个实例掩码 | 实例分割；监督、半监督、弱监督、无监督检测 | 第二名方案 48.49% mAP / 66.71% mAR | 难度高，研究空间大 |

## 4. InsPLAD

### 4.1 仓库和数据

- 本地目录：`/Users/kaius/Project/CraneLab/InsPLAD`
- 官方仓库：https://github.com/andreluizbvs/InsPLAD
- 数据集论文：https://arxiv.org/abs/2311.01619
- 后续异常检测论文：https://arxiv.org/abs/2401.08686

InsPLAD 来自巴西实际运行中的 500 kV 输电线路，由 UAV 拍摄。原始检测数据包含：

- 10,607 张 1920 x 1080 RGB 图像；
- 17 类输电线路设备；
- 28,933 个边界框实例；
- 226 座输电塔；
- 5 类设备具有真实故障样本；
- 共 402 个故障设备裁剪样本。

故障包括腐蚀、绝缘子缺帽和鸟巢等。

### 4.2 可执行任务

1. `InsPLAD-det`：17 类设备目标检测。
2. `supervised_fault_classification`：设备状态和故障类型分类。
3. `unsupervised_anomaly_detection`：只用正常样本训练，测试正常/异常。
4. 仓库另有正在扩展的实例分割标注项目，但它不是原论文完整基准的一部分。

### 4.3 使用方法

- 目标检测：SSD、RetinaNet、YOLO、Faster R-CNN、Cascade R-CNN、TOOD、DetectoRS。
- 监督分类：ResNet、ResNeXt、EfficientNet、MLP-Mixer、Swin Transformer。
- 无监督异常检测：Autoencoder、OGNet、DifferNet、CS-Flow、AttentDifferNet。

推荐把实验拆成两级流水线：

1. 在完整 UAV 图像中检测设备；
2. 裁剪设备后进行故障分类或异常检测。

### 4.4 公开水平及来源

原始 InsPLAD 论文报告：

| 任务 | 方法 | 指标 | 结果 | 来源 |
|---|---|---|---:|---|
| 目标检测 | DetectoRS | Box AP | 72.1% | 原论文 Table 7 |
| 目标检测 | RetinaNet | AP50 | 89.1% | 原论文 Table 7 |
| 监督故障分类 | EfficientNet-B2 | 平均 Balanced Accuracy | 95.4% | 原论文 Table 8 |
| 无监督异常检测 | DifferNet | 平均 AUROC | 90.5% | 原论文 Table 10 |
| 无监督异常检测 | CS-Flow | 平均 AUROC | 90.3% | 原论文 Table 10 |

后续 AttentDifferNet 论文报告：

| 方法 | InsPLAD-fault 平均 AUROC |
|---|---:|
| DifferNet | 92.46% |
| AttentDifferNet-SENet | 94.34% |

重要纠正：`96.67%` 是同一篇 AttentDifferNet 论文在 **MVTec AD** 上的结果，不是 InsPLAD 的结果。InsPLAD 对应的是 `94.34% AUROC`。

### 4.5 饱和度判断

- 监督分类达到 95.4% Balanced Accuracy，单独做分类的提升空间相对较小。
- 目标检测 Box AP 只有 72.1%。
- 原始异常检测基线约 90.5% AUROC，后续方法约 94.34%。

因此 InsPLAD 整体不属于所有任务都达到 98%--99% 的饱和数据集。

## 5. SteelDefectX

### 5.1 仓库和数据

- 本地目录：`/Users/kaius/Project/CraneLab/SteelDefectX`
- 官方仓库：https://github.com/Zhaosxian/SteelDefectX
- 数据入口：https://huggingface.co/datasets/Zhaosxian/SteelDefectX
- 论文：https://arxiv.org/abs/2603.21824

SteelDefectX 整合并重新组织多个钢材表面缺陷数据源，包含：

- 7,778 张图像；
- 25 个缺陷类别；
- 统一为 256 x 256 分辨率；
- 每张缺陷图提供像素级二值掩码；
- 7:3 训练/验证划分；
- 四种文本形式：
  - T1：类别级描述；
  - T2：自由自然语言描述；
  - T3：九字段结构化属性；
  - T4：由结构化属性生成的模板句。

### 5.2 可执行任务

1. 视觉语言分类。
2. 视觉语言缺陷分割。
3. 跨数据集零样本迁移。
4. 图文检索。
5. 文本引导缺陷定位。
6. 基于掩码的普通监督分割。

### 5.3 使用方法

- 分类：CLIP、OpenCLIP、EVA-CLIP、Long-CLIP、FG-CLIP，使用 CLIP-Adapter 微调。
- 分割和定位：Long-CLIP、WinCLIP、AnomalyCLIP。
- 跨域实验：在 SteelDefectX 上训练，在铝表面缺陷和无缝钢管缺陷数据集上零样本测试。
- 文本形式对照：分别使用 T1、T2、T3、T4 训练和测试，比较结构化与自由文本。

### 5.4 公开水平及来源

论文 Table 2 的视觉语言分类结果中：

| 模型和设置 | Accuracy | mAcc |
|---|---:|---:|
| Long-CLIP ViT-L/14，T2-T0 | 93.63% | 92.56% |
| Long-CLIP ViT-L/14，T3-T0 | **94.45%** | **93.66%** |

这里 `Tx-Ty` 表示使用 `Tx` 文本训练、使用 `Ty` 文本评估。

重要纠正：先前提到的“多模态最高约 93.63%”并不准确。`93.63%` 只是 Long-CLIP ViT-L/14 在 `T2-T0` 设置下的 Accuracy。Table 2 的最高分类结果是 `94.45% Acc / 93.66% mAcc`，对应 `T3-T0`。

论文视觉语言分割结果：

| 方法 | AUROC | F1-max | IoU |
|---|---:|---:|---:|
| AnomalyCLIP | **88.21%** | **53.36%** | **37.49%** |
| Long-CLIP T0 | 78.96% | 40.46% | 21.92% |
| WinCLIP 最佳文本设置 | 约 71.87% | 约 31.16% | 约 16.68% |

### 5.5 饱和度判断

分类最高约 94.45%，而像素级 IoU 仅 37.49%。它尤其适合研究：

- 分类和分割之间的性能差距；
- 文本形式对工业缺陷理解的影响；
- 跨数据集泛化；
- 长尾类别和细粒度缺陷。

## 6. SteelBlastQC

### 6.1 仓库和数据

- 本地目录：`/Users/kaius/Project/CraneLab/SteelBlastQC`
- 官方仓库：https://github.com/ERNIS-LAB/SteelBlastQC
- 数据入口：https://doi.org/10.34894/EKZNN0
- 论文：https://arxiv.org/abs/2504.20510

SteelBlastQC 来自真实制造工厂的钢材喷丸质量控制：

- 1,654 张 512 x 512 RGB 图像；
- `good`：已经达到喷漆标准；
- `not-good`：仍需喷丸处理；
- 训练集 1,404 张；
- 测试集 250 张。

图像中包括变色、焊缝、划痕和腐蚀等真实表面问题。

### 6.2 可执行任务

1. 监督式二分类：ready for paint / needs shot-blasting。
2. 只学习正常或良好表面的无监督异常检测。
3. 使用模型热力图进行弱监督缺陷定位和可解释性分析。
4. RGB 与灰度输入对照实验。

注意：数据集没有人工像素级缺陷掩码，因此“定位”应表述为热力图解释或弱监督定位，不能当作有真实掩码监督的分割任务。

### 6.3 使用方法

- CCT：Compact Convolutional Transformer 监督分类。
- SVM：先用 ResNet-50 提取特征，再使用 SVM 分类。
- CAE：卷积自编码器无监督质量检测基线。
- Grad-CAM 或重建误差热力图：解释模型关注区域。

### 6.4 公开水平及来源

官方仓库 README 和论文报告：

| 方法 | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| CCT | **95.0%** | 95.0% | 95.0% | 95.0% |
| ResNet-50 特征 + SVM | 94.5% | 94.5% | 95.5% | **95.0%** |
| CAE | 67.6% | 61.0% | 76.8% | 68.0% |

### 6.5 饱和度判断

监督分类已达到约 95%，但无监督 CAE 基线只有 67.6%。因此它适合：

- 小数据监督学习；
- 监督与无监督方法对照；
- 可解释热力图质量比较。

它不适合作为严格的像素级分割基准。

## 7. AeBAD / MMR

### 7.1 仓库和数据

- 本地目录：`/Users/kaius/Project/CraneLab/MMR`
- 官方代码和数据仓库：https://github.com/zhangzilongc/MMR
- 论文：https://arxiv.org/abs/2304.02216

AeBAD 面向航空发动机叶片异常检测，由西安交通大学机械工程团队提出，包含：

- `AeBAD-S`：单叶片图像；
- `AeBAD-V`：发动机叶盘旋转视频帧；
- 训练样本 1,228 个；
- 测试样本 4,342 个；
- 合计 5,570 个图像或视频帧样本；
- AeBAD-S 的异常样本提供像素级真实掩码；
- AeBAD-V 的所有帧提供图像级正常/异常标签。

缺陷包括 breakdown、ablation、groove 和 fracture。

### 7.2 可执行任务

1. 图像级正常/异常检测。
2. AeBAD-S 像素级异常定位。
3. AeBAD-V 视频帧异常检测。
4. 背景、光照和视角变化下的域偏移鲁棒性测试。

### 7.3 使用方法

- PatchCore：正常特征记忆库和最近邻距离。
- Reverse Distillation：教师-学生特征蒸馏。
- DRAEM、NSA：合成异常训练。
- RIAD、InTra：重建或遮挡恢复。
- MMR：Masked Multi-scale Reconstruction。

MMR 使用冻结的预训练层级编码器产生多尺度目标特征，通过 ViT 和简单 FPN 从被遮挡的输入恢复多尺度特征。测试时以编码特征和恢复特征的差异形成异常图。

### 7.4 公开水平及来源

AeBAD-S 的论文结果：

| 方法 | 图像级平均 AUROC | 像素级平均 PRO |
|---|---:|---:|
| PatchCore | 71.0% | 87.8% |
| Reverse Distillation | 81.0% | 85.6% |
| DRAEM | 62.5% | 63.6% |
| MMR | **84.7%** | **89.1%** |

MMR 的 AUROC 按 `same`、`background`、`illumination` 和 `view` 四个测试子集平均。MMR 在最困难的 `view` 子集上为 79.9% AUROC，说明视角域偏移仍是明显难点。

### 7.5 饱和度判断

AeBAD-S 最佳图像级 AUROC 84.7%，PatchCore 只有 71.0%。它明显不是 98%--99% 饱和数据集，非常适合研究真实机械部件在域偏移下的异常检测。

## 8. VISION Datasets / Aoi-overfitting-team

### 8.1 仓库和数据

- 本地代码目录：`/Users/kaius/Project/CraneLab/Aoi-overfitting-team`
- 第二名方案仓库：https://github.com/love6tao/Aoi-overfitting-team
- 官方数据入口：https://huggingface.co/datasets/VISION-Workshop/VISION-Datasets
- 数据集论文：https://arxiv.org/abs/2306.07890
- 第二名技术报告：https://arxiv.org/abs/2306.14116

重要说明：`Aoi-overfitting-team` 是 CVPR VISION 2023 Track 1 第二名方案代码，不是 VISION 数据集的官方仓库。数据应从官方 Hugging Face 入口获取。

VISION V1 包含：

- 18,422 张工业质检图像；
- 14 类产品；
- 44 种缺陷；
- 4,165 张有标注图像；
- 9,837 个实例分割掩码；
- 其余图像无标签，用于模拟工业中的“数据多、标注少”场景。

产品包括电缆、电容、铸件、气缸、电子元件、PCB、螺丝和木材等。

### 8.2 可执行任务

官方重点任务：

1. 缺陷实例分割。
2. 监督缺陷检测。
3. 半监督、弱监督和无监督缺陷检测。
4. 少样本和数据高效训练。
5. 数据生成、数据增强和数据选择。

可从实例掩码进一步派生：

- 边界框目标检测；
- 语义分割；
- 图像级正常/缺陷分类。

派生任务可执行，但应在论文中说明标签由实例掩码转换得到，并非挑战赛原始主指标。

### 8.3 使用方法

第二名 Aoi-overfitting-team 方案使用：

- Hybrid Task Cascade (HTC) 实例分割；
- CBNetV2 复合连接；
- 两个 Swin-B 主干；
- Mask2Former + Swin-L 辅助提高召回率；
- 多模型融合；
- 多尺度训练；
- Test-Time Augmentation。

### 8.4 公开水平及来源

第二名技术报告在 VISION Track 1 测试集报告：

| 指标 | 结果 |
|---|---:|
| mAP@0.50:0.95 | 48.49% 以上 |
| mAR@0.50:0.95 | 66.71% |

重要说明：`48.49%` 不是分类 Accuracy，而是跨 14 个工业数据集平均的实例分割 mAP。

### 8.5 饱和度判断

即使使用 HTC、Swin、CBNetV2、Mask2Former、模型融合和 TTA，第二名 mAP 仍只有约 48.49%。因此 VISION 是五个项目中最明显未饱和的高难度基准之一。

## 9. 建议的统一实验设计

### 9.1 主实验任务

| 数据集 | 建议主任务 | 建议对照方法 |
|---|---|---|
| InsPLAD | 设备检测 + 故障分类/异常检测 | YOLO、Faster R-CNN、EfficientNet、PatchCore、DifferNet |
| SteelDefectX | 视觉语言分类 + 分割 | CLIP、Long-CLIP、AnomalyCLIP、普通分割网络 |
| SteelBlastQC | 监督分类 + 无监督检测 | ResNet/CCT、SVM、CAE、PatchCore |
| AeBAD | 图像级检测 + 像素定位 + 域偏移 | PatchCore、RD、DRAEM、MMR |
| VISION | 实例分割 + 少标注学习 | Mask R-CNN、HTC、Mask2Former、半监督方法 |

### 9.2 统一报告原则

1. 每个任务单独报告其标准指标，不把 AUROC、AP 和 Accuracy 混成一列。
2. 分类同时报告 Accuracy、Balanced Accuracy 或 macro-F1。
3. 检测报告 AP50 和 AP50:95。
4. 分割报告 IoU、Dice/F1 和 AP。
5. 异常检测分别报告图像级 AUROC与像素级 AUROC/PRO。
6. 至少重复三次训练并报告均值和标准差。
7. 明确是否使用 ImageNet/CLIP 预训练、外部数据、合成异常和测试时增强。

## 10. 最终判断

五个项目均满足工业应用和数据规模低于 50,000 的要求，但“多任务”的性质不同：

- InsPLAD：原生三任务最完整。
- SteelDefectX：视觉、文本和像素掩码结合，适合多模态实验。
- SteelBlastQC：任务较少，但非常适合小数据监督/无监督对照。
- AeBAD：最适合机械工程、域偏移和视频异常检测。
- VISION：规模相对较大但仍低于限制，实例分割难度高，最不饱和。

需要谨慎使用的高值：

- InsPLAD `95.4%` 是 Balanced Accuracy，不是整个数据集综合准确率。
- InsPLAD `94.34%` 是后续方法的图像级 AUROC。
- SteelDefectX `93.63%` 不是最高值；分类最高为 `94.45% Acc / 93.66% mAcc`。
- SteelBlastQC `95.0%` 是二分类测试准确率。
- AeBAD `84.7%` 是 AeBAD-S 四种域设置的平均图像级 AUROC。
- VISION `48.49%` 是实例分割 mAP，不是分类准确率。

## 11. 主要来源

1. InsPLAD repository: https://github.com/andreluizbvs/InsPLAD
2. InsPLAD paper: https://arxiv.org/abs/2311.01619
3. AttentDifferNet paper: https://arxiv.org/abs/2401.08686
4. SteelDefectX repository: https://github.com/Zhaosxian/SteelDefectX
5. SteelDefectX paper: https://arxiv.org/abs/2603.21824
6. SteelBlastQC repository: https://github.com/ERNIS-LAB/SteelBlastQC
7. SteelBlastQC paper: https://arxiv.org/abs/2504.20510
8. MMR/AeBAD repository: https://github.com/zhangzilongc/MMR
9. AeBAD/MMR paper: https://arxiv.org/abs/2304.02216
10. VISION official dataset: https://huggingface.co/datasets/VISION-Workshop/VISION-Datasets
11. VISION dataset paper: https://arxiv.org/abs/2306.07890
12. Aoi-overfitting-team repository: https://github.com/love6tao/Aoi-overfitting-team
13. VISION second-place report: https://arxiv.org/abs/2306.14116
