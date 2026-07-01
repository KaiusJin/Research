# CraneLab Industrial Inspection Benchmarks

CraneLab collects reproducible experiment code and documentation for five small-to-medium industrial inspection datasets. Dataset images are **not redistributed in this repository**. Download every dataset from its official source and comply with its own license and terms.

## Datasets

| Dataset | Domain | Size | Supported tasks | Reported reference result |
|---|---|---:|---|---:|
| [InsPLAD](https://github.com/andreluizbvs/InsPLAD) | Power-line asset inspection | 10,607 images | Object detection, fault classification, anomaly detection | 94.34% image AUROC |
| [SteelDefectX](https://github.com/Zhaosxian/SteelDefectX) | Steel surface inspection | 7,778 images | Vision-language classification, segmentation, cross-dataset transfer | 94.45% Accuracy; 37.49% IoU |
| [SteelBlastQC](https://github.com/ERNIS-LAB/SteelBlastQC) | Shot-blasted steel quality control | 1,654 images | Supervised classification, unsupervised detection, heatmap interpretation | 95.0% Accuracy; CAE 67.6% Accuracy |
| [AeBAD / MMR](https://github.com/zhangzilongc/MMR) | Aero-engine blade inspection | 5,570 images/video frames | Image anomaly detection, pixel localization, video detection, domain shift | 84.7% AUROC; 89.1% PRO |
| [VISION Datasets](https://huggingface.co/datasets/VISION-Workshop/VISION-Datasets) | Mixed industrial inspection | 18,422 images | Instance segmentation, limited-label learning | 48.49% mAP |

The metrics above belong to different tasks and are not directly comparable. AUROC, Accuracy, IoU, PRO, and mAP must be reported separately.

## Official Data

- InsPLAD: https://drive.google.com/drive/folders/1psHiRyl7501YolnCcB8k55rTuAUcR9Ak
- SteelDefectX: https://huggingface.co/datasets/Zhaosxian/SteelDefectX
- SteelBlastQC: https://dataverse.nl/dataset.xhtml?persistentId=doi:10.34894/EKZNN0
- AeBAD: https://drive.google.com/file/d/14wkZAFFeudlg0NMFLsiGwS0E593b-lNo/view
- VISION Datasets: https://huggingface.co/datasets/VISION-Workshop/VISION-Datasets

VISION reference implementation: [Aoi-overfitting-team](https://github.com/love6tao/Aoi-overfitting-team). This is a challenge solution repository, not the official dataset repository.

## AFL Linear Readout

The current AFL experiment uses a frozen ImageNet backbone and a single-round analytic federated linear classifier.

| Backbone | AFL test Accuracy | Centralized Accuracy | AFL/central relative error |
|---|---:|---:|---:|
| ResNet-50 | 93.50% | 93.50% | 7.50e-14 |
| DeiT-S | 94.51% | 94.51% | 1.30e-10 |

Protocol: official supervised InsPLAD train/validation split, an 11-class asset/state label space, five clients, Dirichlet alpha 0.1, gamma 1, and L2-normalized frozen features. ResNet-50 features are projected from 2,048 to 384 dimensions using a fixed data-independent Gaussian projection; DeiT-S uses its native 384-dimensional representation. These are custom AFL linear-readout results, not results reported by the original dataset authors.

## Repository Layout

```text
CraneLab/
├── README.md
├── CraneLab_datasets_technical_report.md
├── InsPLAD/
├── SteelDefectX/
├── SteelBlastQC/
├── MMR/
├── Aoi-overfitting-team/
├── data/                       # local datasets; do not commit
├── manifests/                  # generated local manifests
└── results/                    # generated experiment outputs
```

## Data and Copyright

- Do not commit images, archives, annotations, or generated derivatives without checking the applicable dataset license.
- Keep attribution and citations required by each upstream project.
- Do not assume a code license also licenses dataset images.
- VISION polygon annotations are CC BY-NC 4.0; original image assets retain their original dataset licenses.
- SteelDefectX combines several upstream datasets and does not state one global data license in its current official card; verify NEU, GC10, X-SDD, and S3D terms separately.
- `Aoi-overfitting-team` does not currently include an explicit LICENSE file; linking to it does not grant redistribution rights.

See [CraneLab_datasets_technical_report.md](CraneLab_datasets_technical_report.md) for citations, metric definitions, and dataset-specific license notes.

## Citation

Please cite the original dataset papers used in each experiment. The VISION citation is:

```bibtex
@article{vision-datasets,
  title   = {VISION Datasets: A Benchmark for Vision-based InduStrial InspectiON},
  author  = {Haoping Bai and Shancong Mou and Tatiana Likhomanenko and Ramazan Gokberk Cinbis and Oncel Tuzel and Ping Huang and Jiulong Shan and Jianjun Shi and Meng Cao},
  journal = {arXiv preprint arXiv:2306.07890},
  year    = {2023}
}
```
