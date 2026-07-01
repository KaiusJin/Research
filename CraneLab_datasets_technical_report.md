# CraneLab 工业质检数据集技术报告

更新日期：2026-06-30

## 1. 范围与指标

本文档记录 CraneLab 当前采用的五个工业质检数据集、官方数据入口、可执行任务、已确认公开结果和使用约束。数据集原始图像不随本仓库分发。

不同任务的指标不可直接横向比较：分类使用 Accuracy 或 Balanced Accuracy，异常检测使用 AUROC，定位/分割使用 IoU、PRO 或 mAP。

## 2. 数据集总览

| 数据集 | 工业场景 | 规模 | 主要任务 | 已确认公开水平 |
|---|---|---:|---|---:|
| InsPLAD | 输电线路巡检 | 10,607 张 | 设备检测、故障分类、异常检测 | 94.34% AUROC |
| SteelDefectX | 钢材表面缺陷 | 7,778 张 | 视觉语言分类、缺陷分割、跨数据集迁移 | 94.45% Accuracy；37.49% IoU |
| SteelBlastQC | 钢材喷丸质量检测 | 1,654 张 | 监督分类、无监督检测、热力图解释 | 95.0% Accuracy；CAE 67.6% Accuracy |
| AeBAD | 航空发动机叶片检测 | 5,570 个图像或视频帧 | 图像异常检测、像素定位、视频检测、域偏移 | 84.7% AUROC；89.1% PRO |
| VISION Datasets | 多行业工业质检 | 18,422 张 | 实例分割、少标注学习 | 48.49% mAP |

## 3. InsPLAD

- GitHub：https://github.com/andreluizbvs/InsPLAD
- 数据：https://drive.google.com/drive/folders/1psHiRyl7501YolnCcB8k55rTuAUcR9Ak
- 论文：https://arxiv.org/abs/2311.01619
- 后续异常检测论文：https://arxiv.org/abs/2401.08686

数据包含 10,607 张 UAV 图像、17 类输电线路设备和 28,933 个设备实例。官方提供 `InsPLAD-det` 设备检测、`supervised_fault_classification` 故障分类和 `unsupervised_anomaly_detection` 异常检测三个子集。

AttentDifferNet-SENet 在 InsPLAD-fault 上报告 94.34% 图像级 AUROC；该指标不是分类 Accuracy。

当前 AFL 线性读出把设备类型和状态组合成 11 类，使用官方监督训练/验证划分：

| Backbone | AFL Accuracy | Centralized Accuracy |
|---|---:|---:|
| ResNet-50 | 93.50% | 93.50% |
| DeiT-S | 94.51% | 94.51% |

许可证：官方仓库根目录声明 CC BY-NC 3.0。使用和再分发时必须署名、保留许可证说明并限制为非商业用途。

## 4. SteelDefectX

- GitHub：https://github.com/Zhaosxian/SteelDefectX
- 数据：https://huggingface.co/datasets/Zhaosxian/SteelDefectX
- 论文：https://arxiv.org/abs/2603.21824

数据包含 7,778 张 256 x 256 图像、25 个缺陷类别、像素级二值掩码以及 T1-T4 四种文本描述。官方划分为 5,454 张训练图和 2,324 张验证图。

主要任务包括视觉语言分类、视觉语言分割、图文检索、文本引导定位和跨数据集迁移。论文确认的最高分类结果为 94.45% Accuracy / 93.66% mAcc；AnomalyCLIP 分割结果为 88.21% AUROC、53.36% F1-max 和 37.49% IoU。

许可证：仓库代码为 MIT。数据由 NEU、GC10、X-SDD 和 S3D 整合而来，当前官方数据卡没有声明覆盖全部图像的统一数据许可证。使用或再分发前必须分别核对四个来源数据集条款。

## 5. SteelBlastQC

- GitHub：https://github.com/ERNIS-LAB/SteelBlastQC
- 数据：https://dataverse.nl/dataset.xhtml?persistentId=doi:10.34894/EKZNN0
- 论文：https://arxiv.org/abs/2504.20510

数据包含 1,654 张 512 x 512 RGB 图像，其中 888 张 `good`、766 张 `not-good`。官方划分为 1,404 张训练图和 250 张测试图。

| 方法 | Accuracy | F1 |
|---|---:|---:|
| CCT | 95.0% | 95.0% |
| ResNet-50 特征 + SVM | 94.5% | 95.0% |
| CAE | 67.6% | 68.0% |

该数据集没有人工像素级缺陷掩码，因此热力图属于解释或弱监督定位，不能作为有监督分割结果。

许可证：代码为 MIT；Dataverse 数据集为 CC BY 4.0。使用或再分发数据时必须保留作者、数据集标题、DOI 和许可证信息，并标明修改。

## 6. AeBAD / MMR

- GitHub：https://github.com/zhangzilongc/MMR
- 数据：https://drive.google.com/file/d/14wkZAFFeudlg0NMFLsiGwS0E593b-lNo/view
- 论文：https://arxiv.org/abs/2304.02216

AeBAD 包含 5,570 个图像或视频帧样本：`AeBAD-S` 用于单叶片图像异常检测和像素定位，`AeBAD-V` 用于叶片视频异常检测。测试集包含背景、光照和视角域偏移。

MMR 在 AeBAD-S 上报告 84.7% 平均图像级 AUROC 和 89.1% 平均 PRO；最困难的 view 子集图像级 AUROC 为 79.9%。

许可证：MMR 代码为 Apache-2.0，AeBAD 数据为 CC BY 4.0。代码和数据许可证必须分别保留。

## 7. VISION Datasets

- 数据：https://huggingface.co/datasets/VISION-Workshop/VISION-Datasets
- 数据论文：https://arxiv.org/abs/2306.07890
- 第二名方案：https://github.com/love6tao/Aoi-overfitting-team
- 方案报告：https://arxiv.org/abs/2306.14116

VISION Datasets 由 14 个工业质检子集组成，共 18,422 张图像、44 种缺陷、4,165 张有标注图像和 9,837 个实例掩码。主要任务为实例分割以及监督、半监督、弱监督和无监督的少标注缺陷检测。

`Aoi-overfitting-team` 是 CVPR VISION 2023 Track 1 第二名方案，不是官方数据仓库。该方案使用 HTC、CBNetV2、Swin 和 Mask2Former 等模型，报告超过 48.49% mAP@0.50:0.95 和 66.71% mAR@0.50:0.95。

许可证边界：VISION 新增 polygon 标注为 CC BY-NC 4.0；14 个原始图像子集继续受各自原始许可证约束。官方数据卡明确不保证所有原始资产采用同一许可证，因此不能把整套图像视作统一 CC BY-NC 4.0 数据再分发。`Aoi-overfitting-team` 当前未提供明确 LICENSE 文件，代码仅作外部参考链接。

```bibtex
@article{vision-datasets,
  title   = {VISION Datasets: A Benchmark for Vision-based InduStrial InspectiON},
  author  = {Haoping Bai and Shancong Mou and Tatiana Likhomanenko and Ramazan Gokberk Cinbis and Oncel Tuzel and Ping Huang and Jiulong Shan and Jianjun Shi and Meng Cao},
  journal = {arXiv preprint arXiv:2306.07890},
  year    = {2023}
}
```

## 8. 数据管理与复现要求

1. Git 仓库仅保存代码、配置、清单模板、结果和文档，不保存原始图像或数据压缩包。
2. 数据从上述官方入口下载到本地 `data/<dataset>/`，该目录应被 `.gitignore` 排除。
3. 论文和报告必须使用每项任务对应的标准指标，不能把 AUROC、IoU、PRO 或 mAP 统称为 Accuracy。
4. 每项实验记录数据版本、划分、随机种子、预训练权重、客户端数、non-IID 参数和是否使用外部数据。
5. 发布模型权重或派生标注前再次核查对应数据许可证，尤其是 SteelDefectX 和 VISION 的来源数据条款。

## 9. 结论

五个数据集均满足工业应用和低于 50,000 张的规模要求。InsPLAD 提供最完整的检测/分类/异常检测链路；SteelDefectX 适合多模态与分割；SteelBlastQC 适合小样本监督/无监督对照；AeBAD 适合机械部件域偏移；VISION 的实例分割性能仍明显未饱和。
