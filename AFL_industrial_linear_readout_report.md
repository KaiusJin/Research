# AFL 工业质检线性读出技术报告

更新日期：2026-07-01

## 1. 结论

本次审计确认三个数据集可以按官方监督分类划分运行 AFL 线性读出：InsPLAD、SteelDefectX 和 SteelBlastQC。共完成 14 次实验，包括五个 InsPLAD 设备子任务各两个 backbone，以及 SteelDefectX、SteelBlastQC 各两个 backbone。

AeBAD 和 VISION 没有生成 AFL 分类结果：AeBAD 官方训练集只有正常样本，VISION 官方任务是实例分割和少标注检测。为得到分类 Accuracy 而重新混合官方训练/测试集会破坏数据协议，因此这两个数据集被明确排除。

## 2. AFL 实现核验

实验使用 [KaiusJin/AFL_Reproduction](https://github.com/KaiusJin/AFL_Reproduction) 中的 AFL 实现：

1. 冻结 ImageNet 预训练 backbone，整个实验不更新 backbone 参数。
2. 每个客户端接收本地特征 `X_k` 和 one-hot 标签 `Y_k`。
3. 客户端计算 ridge 解析解：

```text
C_k^r = X_k^T X_k + gamma I
W_k^r = solve(C_k^r, X_k^T Y_k)
```

4. 服务器使用 AFL Absolute Aggregation Law 单轮聚合客户端解析解。
5. 服务器执行 RI recovery，恢复未正则化的全局最小二乘解。
6. 对样本数小于特征维度的秩亏情况，RI recovery 使用 `lstsq` 求最小范数解，与 centralized analytic classifier 保持相同数值口径。

独立矩阵测试中，AA Law 与 centralized 解的相对 Frobenius 误差为 `2.94e-14`。14 次工业数据实验中，最大 AFL/centralized 相对误差为 `9.26e-10`，全部低于审计阈值 `1e-8`。

## 3. Backbone 与线性头

| 名称 | 实际模型 | 预训练 | 输出维度 | 降维 |
|---|---|---|---:|---|
| ResNet-50 | torchvision ResNet-50 | `IMAGENET1K_V2` | 2,048 | 无 |
| DeiT-S | `deit_small_patch16_224.fb_in1k` | ImageNet-1K | 384 | 无 |

两个模型都移除分类头、切换到 evaluation mode，并冻结全部参数。提取的特征先做逐样本 L2 归一化，再输入 AFL 解析式线性分类器。不存在神经网络分类头训练、反向传播或随机投影。

## 4. 联邦配置

所有实验使用相同配置：

| 参数 | 值 |
|---|---:|
| 客户端数 | 5 |
| 划分 | Dirichlet label skew |
| Dirichlet alpha | 0.1 |
| Ridge gamma | 1 |
| 随机种子 | 7 |
| 聚合轮数 | 1 |
| RI recovery | 开启 |

每份结果包含 manifest 绝对路径及 SHA-256、类别列表、样本数、客户端最小/最大样本数、混淆矩阵和 AFL/centralized 相对误差。

## 5. 结果

### 5.1 SteelDefectX

官方划分：5,454 张训练图、2,324 张验证图、25 类。

| Backbone | Accuracy | Balanced Accuracy | Macro-F1 | AFL/central error |
|---|---:|---:|---:|---:|
| ResNet-50 | 94.58% | 90.87% | 92.25% | 2.88e-11 |
| DeiT-S | 95.01% | 90.57% | 92.34% | 2.25e-10 |

### 5.2 SteelBlastQC

官方划分：1,404 张训练图、250 张测试图；类别为 `good` 和 `not-good`。

| Backbone | Accuracy | Balanced Accuracy | Macro-F1 | AFL/central error |
|---|---:|---:|---:|---:|
| ResNet-50 | 83.60% | 82.29% | 82.83% | 6.47e-10 |
| DeiT-S | 94.80% | 94.20% | 94.67% | 3.19e-10 |

### 5.3 InsPLAD

InsPLAD 按官方监督基准拆成五个设备分类任务，不再把设备名称和状态组合成自定义 11 类。

| 设备 | Backbone | Accuracy | Balanced Accuracy | Macro-F1 |
|---|---|---:|---:|---:|
| Glass Insulator | ResNet-50 | 72.88% | 73.10% | 72.49% |
| Glass Insulator | DeiT-S | 77.97% | 78.28% | 77.31% |
| Lightning Rod Suspension | ResNet-50 | 98.80% | 97.07% | 96.02% |
| Lightning Rod Suspension | DeiT-S | 98.80% | 94.78% | 95.83% |
| Polymer Insulator Upper Shackle | ResNet-50 | 93.75% | 93.84% | 93.75% |
| Polymer Insulator Upper Shackle | DeiT-S | 92.19% | 92.33% | 92.19% |
| Vari-Grip | ResNet-50 | 93.59% | 84.61% | 83.60% |
| Vari-Grip | DeiT-S | 93.59% | 88.75% | 85.21% |
| Yoke Suspension | ResNet-50 | 99.06% | 89.56% | 68.37% |
| Yoke Suspension | DeiT-S | 97.17% | 93.60% | 58.33% |

五个设备任务的算术平均：

| Backbone | Mean Accuracy | Mean Balanced Accuracy | Mean Macro-F1 |
|---|---:|---:|---:|
| ResNet-50 | 91.62% | 87.64% | 82.85% |
| DeiT-S | 91.94% | 89.55% | 81.77% |

Yoke Suspension 测试集中正常样本远多于锈蚀样本，因此 Accuracy 明显高于 Macro-F1。该结果必须同时报告 Balanced Accuracy、Macro-F1 和混淆矩阵，不能只报告 Accuracy。

## 6. 未运行的数据集

### AeBAD

AeBAD 的官方异常检测训练集只包含正常叶片。监督式 AFL one-hot 线性分类器需要训练集中同时存在正常和异常类别，因此不能在不改变官方协议的前提下给出二分类 Accuracy。AeBAD 应使用 one-class anomaly detection 指标，如图像级 AUROC、像素 AUROC 和 PRO。

### VISION Datasets

VISION 的官方任务是实例分割和有限标注缺陷检测，标签是 COCO polygon/instance 标注，不是单标签图像分类。当前 AFL 线性分类器不输出边界框或掩码，因此不适用于该基准。若研究目标扩展到检测/分割，需要设计新的解析式检测头并使用 mAP/mAR，而不是强行报告分类 Accuracy。

## 7. 完整性与可复现性

审计检查：

- 正式结果文件数量：14。
- 每份 manifest SHA-256 与当前文件匹配。
- ResNet-50 特征维度全部为 2,048。
- DeiT-S 特征维度全部为 384。
- `random_projection_dim` 全部为 `null`。
- 每份 AFL Accuracy 与 centralized Accuracy 完全一致。
- 所有 AFL/centralized 权重相对误差低于 `1e-8`。

批量复现命令：

```bash
git clone https://github.com/KaiusJin/AFL_Reproduction.git
cd AFL_Reproduction
./experiments/run_valid_afl.sh
```

正式结果位于 AFL 复现仓库的：

```text
results/industrial/
```

## 8. 结果解释边界

这些数字是 CraneLab 的 AFL frozen-feature linear-readout 结果，不是原数据集论文中的 SOTA 结果。它们衡量预训练特征经过解析式线性分类器后的可分性；不能替代目标检测 AP、异常检测 AUROC、像素定位 PRO 或实例分割 mAP。
